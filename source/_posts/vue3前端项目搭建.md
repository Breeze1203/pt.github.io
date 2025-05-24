---
title: "vue3前端项目搭建"
date: 2024-11-18 15:41:52
---

当你想要开始一个 Vue 3 项目时，下面是一份完整的搭建流程：

#### 1. 创建项目

首先，你需要安装 Vue CLI 工具。如果你还没有安装，你可以通过以下命令进行安装：

    npm install -g @vue/cli

然后，你可以使用 Vue CLI 创建一个新的 Vue 3 项目。在命令行中运行：

    vue create my-vue3-project

然后按照提示进行选择，选择 Vue 3 作为你的项目模板。

#### 2. 安装依赖

进入到你的项目目录，并安装其他依赖项。通常情况下，你需要安装 Vue Router 和 Vuex（如果需要）：

    cd my-vue3-project

    npm install vue-router@next vuex@next

#### 3. 创建路由器和状态管理器（可选）

如果你的项目需要使用路由和状态管理，你可以创建路由器和状态管理器。在 `src` 目录下创建 `router` 和 `store` 文件夹，并分别创建 `index.js` 文件来配置路由器和状态管理器。

``` javascript
// router/index.js

import { createRouter, createWebHistory } from 'vue-router';

import Home from '../views/Home.vue';

const routes = [

  { path: '/', component: Home }

  // 其他路由配置

];

const router = createRouter({

  history: createWebHistory(),

  routes

});

export default router;


// store/index.js

import { createStore } from 'vuex';

const store = createStore({

  state() {

    return {

      // 状态管理的状态

    };

  },

  mutations: {

    // 状态管理的变化

  },

  actions: {

    // 异步操作

  },

  getters: {

    // 获取状态

  }

});

export default store;
```

#### 4.集成Element UI

    # 选择一个你喜欢的包管理器

    # NPM
    $ npm install element-plus --save

    # Yarn
    $ yarn add element-plus

    # pnpm
    $ pnpm install element-plus

###### **完整引入**

如果你对打包后的文件大小不是很在乎，那么使用完整导入会更方便。

    // main.ts
    import { createApp } from 'vue'
    import ElementPlus from 'element-plus'
    import 'element-plus/dist/index.css'
    import App from './App.vue'

    const app = createApp(App)

    app.use(ElementPlus)
    app.mount('#app')

###### **Volar 支持**

如果您使用 Volar，请在 `tsconfig.json` 中通过 `compilerOptions.type` 指定全局组件类型。

    // tsconfig.json
    {
      "compilerOptions": {
        // ...
        "types": ["element-plus/global"]
      }
    }

###### **按需导入**

您需要使用额外的插件来导入要使用的组件。

###### **自动导入<span color="" fontsize="">推荐</span>**

首先你需要安装`unplugin-vue-components` 和 `unplugin-auto-import`这两款插件

    npm install -D unplugin-vue-components unplugin-auto-import

然后把下列代码插入到你的 `Vite` 或 `Webpack` 的配置文件中

###### **Vite**

    // vite.config.ts
    import { defineConfig } from 'vite'
    import AutoImport from 'unplugin-auto-import/vite'
    import Components from 'unplugin-vue-components/vite'
    import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

    export default defineConfig({
      // ...
      plugins: [
        // ...
        AutoImport({
          resolvers: [ElementPlusResolver()],
        }),
        Components({
          resolvers: [ElementPlusResolver()],
        }),
      ],
    })

###### **Webpack**

    // webpack.config.js
    const AutoImport = require('unplugin-auto-import/webpack')
    const Components = require('unplugin-vue-components/webpack')
    const { ElementPlusResolver } = require('unplugin-vue-components/resolvers')

    module.exports = {
      // ...
      plugins: [
        AutoImport({
          resolvers: [ElementPlusResolver()],
        }),
        Components({
          resolvers: [ElementPlusResolver()],
        }),
      ],
    }

想了解更多打包 (<a href="https://rollupjs.org/" class="vp-link" rel="noopener noreferrer" target="_blank"><strong>Rollup</strong></a>, <a href="https://cli.vuejs.org/" class="vp-link" rel="noopener noreferrer" target="_blank"><strong>Vue CLI</strong></a>) 和配置工具，请参考 <a href="https://github.com/antfu/unplugin-vue-components#installation" class="vp-link" rel="noopener noreferrer" target="_blank"><strong>unplugin-vue-components</strong></a> 和 <a href="https://github.com/antfu/unplugin-auto-import#install" class="vp-link" rel="noopener noreferrer" target="_blank"><strong>unplugin-auto-import</strong></a>。

###### **Nuxt**

对于 Nuxt 用户，只需要安装 `@element-plus/nuxt` 即可。

    npm install -D @element-plus/nuxt

然后将下面的代码写入你的配置文件.

    // nuxt.config.ts
    export default defineNuxtConfig({
      modules: ['@element-plus/nuxt'],
    })

配置文档参考 <a href="https://github.com/element-plus/element-plus-nuxt#readme" class="vp-link" rel="noopener noreferrer" target="_blank"><strong>docs</strong></a>.

###### **手动导入**

Element Plus 提供了基于 ES Module 的开箱即用的 <a href="https://webpack.js.org/guides/tree-shaking/" class="vp-link" rel="noopener noreferrer" target="_blank"><strong>Tree Shaking</strong></a> 功能。

但你需要安装 <a href="https://github.com/element-plus/unplugin-element-plus" class="vp-link" rel="noopener noreferrer" target="_blank"><strong>unplugin-element-plus</strong></a> 来导入样式。 配置文档参考 <a href="https://github.com/element-plus/unplugin-element-plus#readme" class="vp-link" rel="noopener noreferrer" target="_blank"><strong>docs</strong></a>.

> App.vue

    <template>
      <el-button>我是 ElButton</el-button>
    </template>
    <script>
      import { ElButton } from 'element-plus'
      export default {
        components: { ElButton },
      }
    </script>

    // vite.config.ts
    import { defineConfig } from 'vite'
    import ElementPlus from 'unplugin-element-plus/vite'

    export default defineConfig({
      // ...
      plugins: [ElementPlus()],
    })

#### 5.集成Ant Design

###### 传送门 https://ant.design/docs/react/use-with-vite-cn

#### 6. 创建组件

开始编写你的 Vue 组件。在 `src` 目录下创建 `components` 文件夹，并在其中创建你的 Vue 组件。

#### 7. 使用组件

在你的应用程序中使用你创建的组件。编辑 `App.vue` 文件，并在其中使用你的组件。

#### 8. 编写样式

编写你的样式文件，可以使用 CSS、Sass、Less 等预处理器。确保将样式文件导入到你的组件中。

#### 9. 启动项目

最后，运行你的 Vue 3 项目：

    npm run serve

然后在浏览器中打开 http://localhost:8080/ 查看你的项目。

这就是一个 Vue 3 项目搭建的基本流程。根据你的项目需求，可能还需要进行其他配置和操作，但这个流程可以帮助你开始一个简单的 Vue 3 项目。
