---
title: Pinia持久化配置
published: 2025-09-11
description: 'Pinia持久化配置'
image: ''
tags: [pinia,pinia-persistedstate,persistedstate]
category: 'vue'
draft: false 
lang: 'zh-CN'
---

Pinia 本身不带持久化功能，但我们可以通过插件来实现。

---

## 📦 安装持久化插件

推荐使用官方生态的 `pinia-plugin-persistedstate`：

```bash
npm install pinia-plugin-persistedstate
```

---

## ⚙️ 在 Nuxt 中配置（支持自动导入的写法）

**`plugins/piniaPersist.client.ts`**

> `.client.ts` 确保只在浏览器端执行（避免 SSR 报错）

```ts
import { defineNuxtPlugin } from '#app'
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

export default defineNuxtPlugin((nuxtApp) => {
  nuxtApp.$pinia.use(piniaPluginPersistedstate)
})
```

Nuxt 会自动加载 `plugins/` 下的插件文件。

---

## 🧩 在 store 里开启持久化

```ts
// stores/user.ts
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', {
  state: () => ({
    token: '',
    name: ''
  }),
  persist: true // ⚡ 启用持久化
})
```

默认会使用 `localStorage` 存储。

---

## ⚙️ 进阶：自定义存储位置（localStorage / sessionStorage）

```ts
persist: {
  storage: sessionStorage,
  key: 'my_user'
}
```

---

## ✅ 使用

```vue
<script setup lang="ts">
const userStore = useUserStore()

userStore.token = 'qwq' // 会自动保存到 localStorage
</script>
```