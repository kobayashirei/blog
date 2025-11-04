---
title: Golang JWT
published: 2025-08-09
description: ''
image: ''
tags: [go,golang,jwt]
category: 'golang'
draft: false 
lang: ''
---

# 自定义Claims
```go

// CustomClaims JWT Claims结构
type CustomClaims struct {
	UserID    string `json:"user_id"`
	DeviceID  string `json:"device_id"`
	SessionID string `json:"session_id"`
	jwt.RegisteredClaims
}

// GenerateTokens 生成双Token
func (ax *AuthX) GenerateTokens(userID string) (accessToken, refreshToken string, sessionID string, err error) {
	// 生成唯一sessionID
	sessionID = fmt.Sprintf("u%s_%s", userID, uuid.NewString()) // u10001_f47ac10b-58cc-4372-a567-0e02b2c3d479

	// 生成Access Token
	accessClaims := CustomClaims{
		UserID:    userID,
		SessionID: sessionID,
		RegisteredClaims: jwt.RegisteredClaims{
			ExpiresAt: jwt.NewNumericDate(time.Now().Add(ax.config.AccessTokenExpire)),
		},
	}
	accessToken, err = jwt.NewWithClaims(jwt.SigningMethodHS256, accessClaims).SignedString(ax.config.SecretKey)
	if err != nil {
		return "", "", "", err
	}

	// 生成Refresh Token
	refreshClaims := jwt.RegisteredClaims{
		ExpiresAt: jwt.NewNumericDate(time.Now().Add(ax.config.RefreshTokenExpire)),
	}
	refreshToken, err = jwt.NewWithClaims(jwt.SigningMethodHS256, refreshClaims).SignedString(ax.config.SecretKey)
	if err != nil {
		return "", "", "", err
	}

	// 存储到Redis (sessionID作为关键key)
	//ctx := context.Background()
	//err = redisClient.Set(ctx,
	//	fmt.Sprintf("user:%s:session", userID),
	//	sessionID,
	//	refreshTokenExpire,
	//).Err()

	return accessToken, refreshToken, sessionID, err
}

```

- redis custom key

> user_id:${uid}:did:${did}


```txt
user_id:10001:device:web/pc
user_id:10001:device:ios/app/mobile
```

---

# 设计概览（要点）

1. **单端登录**：每个用户在 Redis 中只保留一份会话标识（jti）。登录时写入 `user_session:{userID} = jti`，校验任何 token 时都要比对其 `jti` 是否和 Redis 中一致。这样旧设备/旧 token 即被作废。
2. **Access Token / Refresh Token**：

   * Access：短期（例如 15 分钟），用于接口认证。
   * Refresh：长期（例如 7 天），用于在 Access 过期时“无感”获得新的 Access（并做 **refresh token 轮换**）。
3. **无感刷新（中间件自动刷新）**：

   * 中间件先尝试验证 Access。
   * 若 Access 过期但 Refresh cookie 存在且合法，解析 Refresh；若 Refresh 的 jti 与 Redis 中一致，则生成新的 Access（和新的 Refresh，可选择轮换），更新 Redis 的 jti，从而实现“无感刷新”并继续本次请求。
   * 新 token 会以 HttpOnly、Secure cookie 返回给客户端，或用 header 返回（建议 cookie）。
4. **安全**：

   * 强烈建议使用 HTTPS、HttpOnly + Secure cookie、SameSite=strict/lax。
   * Refresh token 轮换（每次刷新都发新 Refresh 并使旧 Refresh 失效）能降低刷新令牌被滥用风险。
   * 将 `user_session:{userID}` 存 Redis，设置 TTL 为 Refresh 的时长（或每次刷新重置 TTL）。
5. **依赖**：

   * Gin: `github.com/gin-gonic/gin`
   * Redis: `github.com/redis/go-redis/v9`（go-redis v9）
   * JWT: `github.com/golang-jwt/jwt/v5`
   * Context/标准库


---

# 示例

> 假设文件 `main.go` 单文件实现（为了演示把主要逻辑放一起）。真实项目建议分包（auth, middleware, storage 等）。

```go
// main.go
package main

import (
	"context"
	"errors"
	"fmt"
	"net/http"
	"os"
	"time"

	"github.com/gin-gonic/gin"
	"github.com/golang-jwt/jwt/v5"
	"github.com/redis/go-redis/v9"
)

// ----------------- 配置 -----------------
var (
	AccessTokenTTL  = 15 * time.Minute
	RefreshTokenTTL = 7 * 24 * time.Hour

	JWTSecret = []byte("replace-with-strong-secret") // 生产请从 env/config 注入
)

var redisClient *redis.Client
var ctx = context.Background()

// ----------------- Claims -----------------
type MyClaims struct {
	UserID string `json:"uid"`
	// 可以放更多字段
	jwt.RegisteredClaims
}

// ----------------- Redis Key helpers -----------------
func userSessionKey(userID string) string {
	return fmt.Sprintf("user_session:%s", userID)
}

// ----------------- Token generation -----------------
func genToken(userID string, ttl time.Duration) (tokenString string, jti string, err error) {
	jti = fmt.Sprintf("%d", time.Now().UnixNano()) // 简单 jti；也可以用 uuid
	now := time.Now()
	claims := MyClaims{
		UserID: userID,
		RegisteredClaims: jwt.RegisteredClaims{
			ID:        jti,
			IssuedAt:  jwt.NewNumericDate(now),
			ExpiresAt: jwt.NewNumericDate(now.Add(ttl)),
			// Issuer, Subject 等根据需要设置
		},
	}
	t := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	tokenString, err = t.SignedString(JWTSecret)
	return
}

// ----------------- Token parse & validate -----------------
func parseToken(tokenStr string) (*MyClaims, error) {
	claims := &MyClaims{}
	parser := jwt.NewParser()
	_, err := parser.ParseWithClaims(tokenStr, claims, func(t *jwt.Token) (interface{}, error) {
		// ensure method
		if _, ok := t.Method.(*jwt.SigningMethodHMAC); !ok {
			return nil, fmt.Errorf("unexpected signing method: %v", t.Header["alg"])
		}
		return JWTSecret, nil
	})
	if err != nil {
		return nil, err
	}
	// jwt lib already checks expiry in Valid() when parsing (unless you skip)
	return claims, nil
}

// ----------------- Redis helpers -----------------
func setUserSession(userID, jti string, ttl time.Duration) error {
	return redisClient.Set(ctx, userSessionKey(userID), jti, ttl).Err()
}

func getUserSession(userID string) (string, error) {
	return redisClient.Get(ctx, userSessionKey(userID)).Result()
}

func deleteUserSession(userID string) error {
	return redisClient.Del(ctx, userSessionKey(userID)).Err()
}

// ----------------- Handlers -----------------

// Mock: validate username/password -> return userID
func authenticateUser(username, password string) (string, error) {
	// 真实系统要查 DB
	if username == "alice" && password == "pass123" {
		return "user-1001", nil
	}
	return "", errors.New("invalid credentials")
}

// POST /login
func loginHandler(c *gin.Context) {
	var body struct {
		Username string `json:"username"`
		Password string `json:"password"`
	}
	if err := c.BindJSON(&body); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": "bad request"})
		return
	}
	uid, err := authenticateUser(body.Username, body.Password)
	if err != nil {
		c.JSON(http.StatusUnauthorized, gin.H{"error": "invalid username/password"})
		return
	}

	// generate tokens
	accessToken, accessJti, err := genToken(uid, AccessTokenTTL)
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": "token err"})
		return
	}
	refreshToken, refreshJti, err := genToken(uid, RefreshTokenTTL)
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": "token err"})
		return
	}

	// 单端登录：把 refreshJti（或任意代表会话的 jti）写到 Redis（覆盖旧会话）
	// 我们将 Redis 的值视作“当前有效 jti”
	if err := setUserSession(uid, refreshJti, RefreshTokenTTL+time.Minute); err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": "redis err"})
		return
	}

	// 建议：通过 HttpOnly Secure cookie 发送 Refresh token，Access token 也可以放 cookie 或返回给前端
	// 这里：Access 返回 JSON，Refresh 写 cookie（HttpOnly）
	c.SetCookie("refresh_token", refreshToken, int(RefreshTokenTTL.Seconds()), "/", "", true, true)
	// 返回 access token（也可以放 cookie）
	c.JSON(http.StatusOK, gin.H{
		"access_token": accessToken,
		"expires_in":   int(AccessTokenTTL.Seconds()),
		"user_id":      uid,
	})
}

// POST /logout
func logoutHandler(c *gin.Context) {
	// 从 context 中拿 user（中间件会设置）
	userID, _ := c.Get("userID")
	if userIDStr, ok := userID.(string); ok {
		deleteUserSession(userIDStr)
	}
	// 清 cookie
	c.SetCookie("refresh_token", "", -1, "/", "", true, true)
	c.JSON(http.StatusOK, gin.H{"msg": "logged out"})
}

// GET /profile（受保护示例）
func profileHandler(c *gin.Context) {
	uid, _ := c.Get("userID")
	c.JSON(http.StatusOK, gin.H{"user_id": uid})
}

// ----------------- Auth Middleware（含无感刷新） -----------------
func authMiddleware() gin.HandlerFunc {
	return func(c *gin.Context) {
		// 先尝试从 Authorization header 或 access cookie 获取 Access token
		var accessToken string
		authHeader := c.GetHeader("Authorization")
		if len(authHeader) > 7 && authHeader[:7] == "Bearer " {
			accessToken = authHeader[7:]
		} else {
			// 或者从 cookie 中获取 access（如果你选择把 access 存 cookie）
			if cookie, err := c.Cookie("access_token"); err == nil {
				accessToken = cookie
			}
		}

		if accessToken != "" {
			claims, err := parseToken(accessToken)
			if err == nil {
				// 校验 jti 与 redis 中的一致性（单端）
				if ok := checkJtiMatch(claims.UserID, claims.ID); ok {
					// token 合法且为当前会话
					c.Set("userID", claims.UserID)
					c.Next()
					return
				}
				// jti 不匹配 -> token 已失效（被踢）
				c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{"error": "session invalidated"})
				return
			} else {
				// 如果是过期错误，尝试无感刷新
				var verr *jwt.ValidationError
				if errors.As(err, &verr) && verr.Errors&jwt.ErrTokenExpired != 0 {
					// 尝试用 refresh 做无感刷新
					if doSilentRefresh(c) {
						// doSilentRefresh 会在成功时设置 userID
						c.Next()
						return
					}
					// 刷新失败 -> 需要重新登录
					c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{"error": "token expired, please login"})
					return
				}
				// 其他解析错误
				c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{"error": "invalid token"})
				return
			}
		}

		// 没有 access token -> 尝试无感刷新（某些场景，客户端只保存 refresh cookie）
		if doSilentRefresh(c) {
			c.Next()
			return
		}

		c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{"error": "missing token"})
	}
}

// doSilentRefresh: 返回 true 表示刷新成功并已设置 userID 到 context
func doSilentRefresh(c *gin.Context) bool {
	refreshCookie, err := c.Cookie("refresh_token")
	if err != nil || refreshCookie == "" {
		return false
	}
	claims, err := parseToken(refreshCookie)
	if err != nil {
		return false
	}
	uid := claims.UserID
	jti := claims.ID

	// 校验 Redis 中的 jti（单端登录）：确保 refresh 的 jti 是当前会话
	curJti, err := getUserSession(uid)
	if err != nil {
		return false
	}
	if curJti != jti {
		// 说明该 refresh 已被替换（可能在另一端登录了）
		return false
	}

	// 通过了：生成新的 access（和新的 refresh（轮换））
	newAccess, _, err := genToken(uid, AccessTokenTTL)
	if err != nil {
		return false
	}
	newRefresh, newRefreshJti, err := genToken(uid, RefreshTokenTTL)
	if err != nil {
		return false
	}

	// 更新 Redis 会话 jti，覆盖旧 refresh（实现单端 + refresh 轮换）
	if err := setUserSession(uid, newRefreshJti, RefreshTokenTTL+time.Minute); err != nil {
		return false
	}

	// 设置新 refresh cookie（HttpOnly）
	c.SetCookie("refresh_token", newRefresh, int(RefreshTokenTTL.Seconds()), "/", "", true, true)
	// 可把 access 返回 header 或 cookie：这里放 header（也可改为 cookie）
	c.Header("X-Access-Token", newAccess)
	c.Set("userID", uid)
	return true
}

func checkJtiMatch(userID, jti string) bool {
	cur, err := getUserSession(userID)
	if err != nil {
		return false
	}
	return cur == jti
}

// ----------------- Init Redis & main -----------------
func initRedis() {
	redisClient = redis.NewClient(&redis.Options{
		Addr:     "127.0.0.1:6379",
		Password: "", // set if needed
		DB:       0,
	})
	if err := redisClient.Ping(ctx).Err(); err != nil {
		fmt.Fprintln(os.Stderr, "redis connect error:", err)
		os.Exit(1)
	}
}

func main() {
	initRedis()

	r := gin.Default()

	r.POST("/login", loginHandler)
	r.POST("/logout", authMiddleware(), logoutHandler)

	protected := r.Group("/api")
	protected.Use(authMiddleware())
	{
		protected.GET("/profile", profileHandler)
		// ... 其他受保护路由
	}

	r.Run(":8080")
}
```

---

# 使用说明与前端协作建议

1. **前端**：

   * 登录成功后把 `access_token` 存内存（如 Vuex / Pinia），不要放 localStorage（XSS 风险）。也可以把 access 也通过 HttpOnly cookie 返回，客户端不直接访问。
   * 将 Refresh 放 `HttpOnly; Secure` cookie；浏览器自动带上。
   * 当后端返回新 `X-Access-Token` header 或新的 access cookie 时，前端要更新内存中的 Access（如果放内存）。
   * 如果你的前端是 SPA，可在每次发请求前从内存取 Access，若 401 且响应提示需要登录，则跳转登录；但若使用本示例中“无感刷新”，后端会在中间件自动刷新并在响应中返回新 Access（header 或 cookie），前端应读取并替换。
2. **刷新策略**：

   * 可以用“滑动窗口”（每次请求刷新 Refresh TTL）或“轮换 Refresh token”（示例采用轮换：每次刷新发新 Refresh 并使旧的失效）。
   * 轮换更安全，但后端必须把最新 jti 存起来（示例用 `user_session:{userID}`）。
3. **并发登录/并发刷新**：为避免并发刷新时的竞态，可在 Redis 做简单乐观锁或使用 `GETSET` / `SET` with NX 等手段控制。生产系统应谨慎处理刷新并发。
4. **登出**：删除 `user_session:{userID}` 即可立即使所有 token 失效。

---

# 进一步改进建议（生产级）

* 用更强的 jti（UUIDv4）。
* 把 JWT secret 放到 KMS / 环境变量中，并按需做密钥轮换（kid）。
* 在 Redis 中保存更多会话信息（设备、IP、UserAgent），支持管理会话（踢人）。
* 在 token 中加入 `aud`/`iss`/`sub` 等字段增强校验。
* 记录 refresh 操作日志，监控异常频繁的 refresh（可能是攻击）。
* 使用 TLS（HTTPS），并对 cookie 设置 `SameSite=Lax` 或更严格。
* 对 logout、换设备、管理员强制下线等场景增加接口。