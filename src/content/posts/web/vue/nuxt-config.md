---
title: Nuxt配置
published: 2025-09-11
description: 'nuxt配置'
image: ''
tags: [nuxt,nuxt-config]
category: 'vue'
draft: false 
lang: 'zh-CN'
---


```ts
// https://nuxt.com/docs/api/configuration/nuxt-config
export default defineNuxtConfig({
  compatibilityDate: '2024-04-03',
  devtools: { enabled: true },
  // 启用仅客户端渲染
  // ssr: false,
  vue: {
    // 这里告诉了vue前缀为"***-"的元素不是vue组件
    compilerOptions: {
      // isCustomElement: tag => tag.startsWith('i-') || tag.startsWith('xl-') || tag.startsWith('x-')
      isCustomElement: tag => tag.startsWith('qwq-') || tag.startsWith('mdui-'),
    }
  },
  // 自动导入组件
  // components: {
  //   "dirs": [
  //     {
  //       "path": "~/components/ui",
  //       "global": true
  //     },
  //     {
  //       "path": "~/components/util",
  //       "global": true
  //     },
  //     "~/components"
  //   ]
  // },
  plugins: [
    // 注册custom element
    { src: '~/plugins/init.config.client.ts', mode: 'client' },
    // { src: '~/plugins/initMduiElement.client.ts', mode: 'client' },
  ],
  // 启动服务器
  devServer: {
    port: 4000,
  },
  // i18n config
  modules: [
    '@nuxtjs/i18n',
    '@pinia/nuxt',
    'pinia-plugin-persistedstate',
    // 'mdui',
  ],
  // pinia: {
  //   autoImports: ['defineStore'], // 自动导入 Pinia 核心方法
  // },
  pinia: {
    // 自动导出stores文件夹下所以文件及文件夹
    storesDirs: ['./stores/**'],
  },
  i18n: {
        strategy: 'prefix_except_default',
        defaultLocale: "zhs",
        locales: [
            { code: "zhs", language: "zh-CN", name: "简体中文" },
            { code: "zht", language: "zh-TW", name: "繁體中文" },
            { code: "en", language: "en-US", name: "English" },
            { code: "ja", name: "日本語" },
            { code: "ko", name: "한국어" },
            { code: "vi", name: "Tiếng Việt" },
            { code: "id", name: "Bahasa Indonesia" },
            { code: "fr", name: "Français" },
            { code: "yue", name: "廣東話" },
        ],
        vueI18n: "./i18n/i18n.config.ts",
        detectBrowserLanguage: {
            useCookie: true,
            cookieKey: "language",
            alwaysRedirect: true,
        },
    },
  // i18n: {
  //   // defaultDirection: 'auto',
  //   lazy: true, // 懒加载语言包
  //   langDir: './locales', // 语言文件目录
  //   locales: [
  //     { code: 'en', iso: 'en-US', file: 'en.json', name: 'English' },
  //     { code: 'zh', iso: 'zh-CN', file: 'zh.json', name: '中文' },
  //   ],
  //   defaultLocale: 'zh', // 默认语言
  //   strategy: 'prefix_except_default', // 路由前缀策略
  //   detectBrowserLanguage: {
  //     useCookie: true, // 使用 Cookie 存储用户语言偏好
  //     cookieKey: 'i18n_redirected', // Cookie 键名
  //     redirectOn: 'root', // 在根路径重定向到语言前缀
  //   },
  //   // vueI18n: './i18n.config.ts'
  // },
  // runtime config
  runtimeConfig: {
    // 仅服务端可用
    apiSecret: process.env.API_SECRET || 'default-secret',
    // 客户端和服务端均可用
    public: {
      baseUrl: process.env.YUSHULINFENG_SERVER_BASE_URL || 'http://localhost:5000',
      appTitle: process.env.YUSHULINFENG_SERVER_APP_TITLE || 'qwq',
      appDir: process.env.YUSHULINFENG_SERVER_APP_DIR || '/_nuxt/',
      version: process.env.YUSHULINFENG_SERVER_API_VERSION || '1.0.0'
    },
  },
  // 打包项目目录命名
  // rootDir: "name",
  // 路由auto
  // pages: false,
  // app id
  appId: "app",
  // 加载css样式
  // css: ['~/assets/styles/main.css'],
  // css: [
  //   'mdui/mdui.css'
  // ],
  // plugins: ['~/plugins/axios.ts'],
  app: {
    buildAssetsDir: process.env.YUSHULINFENG_SERVER_APP_DIR || '/_nuxt/',
    // custom assets
    // buildAssetsDir: "/assets/",
    // teleportAttrs: {
    //   class: "tools"
    // },
    // buildAssetsDir: "/_qwq/",
    rootAttrs: {
      // root dom element id = root
      'id': "root",
      'class': 'root',
      'lang': 'zh-CN',
    },
    // root dom element type
    rootTag: 'div',
    head: {
      // pageTransition: { name: 'page', mode: 'out-in' }, // 页面过渡动画
      // layoutTransition: { name: 'layout', mode: 'out-in' }, // 布局过渡动画
      title: 'i站',
      meta: [
        // <meta name="viewport" content="width=device-width, initial-scale=1">
        { name: 'viewport', content: 'width=device-width, initial-scale=1' },
        {
          name: "description",
          content: "Hi ~, This is my first community website."
        },
        {
          name: "keywords",
          content: "qwq,xiaolin"
        },
        {
          "charset": "utf-8"
        }
      ],
      script: [
        // <script src="https://myawesome-lib.js"></script>
        // { src: 'https://awesome-lib.js' }
        // { src: '/_nuxt/assets/scripts/customElement/*' }
      ],
      link: [
        // <link rel="stylesheet" href="https://myawesome-lib.css">
        // { rel: 'stylesheet', href: 'https://awesome-lib.css' }
        // { rel: 'stylesheet', href: '/' }
      ],
      // please note that this is an area that is likely to change
      style: [
        // <style type="text/css">:root { color: red }</style>
        // { children: ':root { color: red }' }
      ],
      noscript: [
        // <noscript>JavaScript is required</noscript>
        { children: 'JavaScript is required' },
        { children: 'Hello world' }
      ]
    }
  },
})
```