---
title: react框架教程
date: 2026-09-24 12:36:07
tags:
---

首先我们需要了解一个工具vite

Vite（法语意为“快速的”，发音 /viːt/）是由尤雨溪及其团队于 2019 年开发的前端构建工具，核心目标是解决传统 JavaScript 工具在大型项目中面临的启动缓慢、热更新延迟等性能瓶颈问题。它的思路和 Webpack 这样的传统打包器有本质区别：不在开发阶段打包
传统的打包器需要先把整个项目的所有模块分析、编译、打包成一个或多个 bundle，然后才能启动开发服务器。Vite 的做法是直接利用浏览器原生的 ES 模块支持，开发服务器按需编译模块——浏览器请求哪个模块，Vite 就实时编译哪个模块并返回。这意味着无论项目有 10 个模块还是 1000 个模块，启动速度几乎一样快。（Vite 这个工具，本身是用 JavaScript 写的。）

如何安装vite？

1.环境检查
Vite 需要 Node.js 版本 20.19+ 或 22.12+。在终端里检查一下：

```bash
node -v
npm -v
```
如果版本不够，用 nvm 或 nvm-windows 升级

2.安装
如果你已经安装好了node.js，估计要在目录下做一次初始化
npm init -y
运行完，目录里会多出一个文件 package.jso（暂且不用管）
然后在目录下
npm install -D vite
运行完目录里多出一个 node_modules 文件夹（暂且不用管）
package.json 现在多了一段 devDependencies：
同时还多出一个 package-lock.json

3.启动
npm create vite@latest my-react-app -- --template react-swc-ts（生成完整的项目结构默认选择SWC 作为编译器）

然后
cd my-react-app
npm install
npm run dev

终端输出：
  VITE v8.x.x  ready in 320 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose

4.概览
一个典型的 react-swc-ts 项目长这样：
my-react-app/
├── index.html              # 入口 HTML，包含 <div id="root">
├── package.json
├── vite.config.ts          # Vite 配置
├── tsconfig.json           # TypeScript 配置
├── src/
│   ├── main.tsx            # React 挂载入口
│   ├── App.tsx             # 根组件
│   ├── App.css
│   ├── index.css
│   └── assets/             # 静态资源
└── public/                 # 不需要处理的公共资源


index.html 在项目根目录，而不是在 public/ 里。在 Vite 中，index.html 是应用的真正入口，Vite 以它为中心来解析和构建整个应用。你可以直接在 index.html 里写 <script type="module" src="/src/main.tsx"></script>，Vite 会处理这个入口的依赖关系。

src/main.tsx 是 React 的挂载点：

import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import App from './App.tsx'
import './index.css'

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <App />
  </StrictMode>,
)
StrictMode 在开发模式下会做额外的检查（比如检测副作用是否被正确清理），生产构建中不会产生任何开销。建议保留。

理解 vite.config.ts
脚手架生成的配置文件基本是空的，但它是你后续定制构建行为的核心入口。Vite 会自动解析项目根目录下的 vite.config.js 或 vite.config.ts。
{% asset_img vite.png "示例图" %}



一个基础的 React 项目配置长这样：

import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react-swc'

export default defineConfig({
  plugins: [react()],
})

defineConfig 是一个辅助函数，主要作用是提供 TypeScript 类型提示。没有它也能跑，但有了它，你在编辑器里写配置项时会有完整的补全和类型检查。

如果你选了 react-ts 模板（Babel 版），插件是 @vitejs/plugin-react；react-swc-ts 则是 @vitejs/plugin-react-swc。两者 API 一致，SWC 版在大型项目中编译更快。

5.写一个自己的组件
默认的 App.tsx 包含了一些演示内容，可以清掉，写一个简单的组件来确认一切正常：

// src/components/Greeting.tsx
interface GreetingProps {
  name: string
}

export function Greeting({ name }: GreetingProps) {
  return <h2>你好，{name}！React + Vite 已就绪。</h2>
}
然后在 App.tsx 中使用：

{% raw %}
import { Greeting } from './components/Greeting'

function App() {
  return (
    <div style={{ padding: '2rem' }}>
      <Greeting name="开发者" />
    </div>
  )
}

export default App
{% endraw %}

保存完成





