# vant 组件学习
## 目录结构

```javascript
.
├── LICENSE
├── README.md // readme.md,可参考reamde开头的写法
├── README.zh-CN.md // 中文readme
├── SECURITY.md
├── package.json
├── packages
│   ├── create-vant-cli-app // 自动搭建组件库的脚手架
│   ├── vant //组件库 npm run dev 实际运行，供用户预览的项目
│   ├── vant-area-data// 中国省市区数据，适用于 Vant Area 组件。
│   ├── vant-cli// 脚手架 Vant CLI 是一个基于 Vite 实现的 Vue 组件库构建工具，通过 Vant CLI 可以快速搭建一套功能完备的 Vue 组件库。
│   ├── vant-eslint-config // vant eslint配置  npm 包方式引入
│   ├── vant-icons//vant 图标库 vant里实际引用
│   ├── vant-popperjs// 例如：按钮，以及描述它的工具提示元素，Popper 会自动将工具提示放在按钮附近的正确位置。 主要用于Popover 气泡弹出框
│   ├── vant-touch-emulator // 桌面模拟touch事件
│   └── vant-use// vant自行封装的componstion Api
├── pnpm-lock.yaml
├── pnpm-workspace.yaml
└── tsconfig.json
```

## 组件依赖结构图

## package.json

### 构建命令

```javascript
//package.json
"scripts": {
    "prepare": "husky install",
    "dev": "pnpm --dir ./packages/vant dev",
    "lint": "pnpm --dir ./packages/vant lint",
    "test": "pnpm --dir ./packages/vant test",
    "test:watch": "pnpm --dir ./packages/vant test:watch",//什么意思
    "build": "pnpm --dir ./packages/vant build",
    "build:site": "pnpm --dir ./packages/vant build:site"
}
```

### 核心库 vant 的 package.json

```javascript
//vant/packages/vant/package.json
"scripts": {
    "dev": "vant-cli dev",
    "lint": "vant-cli lint",
    "test": "vant-cli test",
    "build": "vant-cli build",
    "build:site": "vant-cli build-site",
    "release": "cp ../../README.md ./ && vant-cli release && rm ./README.md",
    "release:site": "pnpm build:site && npx gh-pages -d site-dist --add",
    "test:watch": "vant-cli test --watch",
    "test:coverage": "open test/coverage/index.html" //什么意思
  },

```

### 依赖库

nano-staged：用于在 GIT 存储库中为修改、暂存和提交的文件运行命令的微型工具。它有助于加快测试、过梁、脚本等的运行。  
rimraf：rimraf 的作用：以包的形式包装 rm -rf 命令，用来删除文件和文件夹的，不管文件夹是否为空，都可删除；  
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1572912/1660201394763-732ce754-f25f-42bb-931f-b42b93ff1f2f.png#averageHue=%23f7f7f7&clientId=udf1c76ad-0e66-4&from=paste&height=752&id=u3d93b49a&name=image.png&originHeight=752&originWidth=2202&originalType=binary&ratio=1&rotation=0&showTitle=false&size=86701&status=done&style=none&taskId=u81086c39-6e8b-4343-8889-edf5b227d7c&title=&width=2202)

## vant-cli 

## 问题积累

1.怎么做到 各个子包下载依赖，相互独立  
2.自动生成更新日志  
3.模拟器嵌入的 iframe,链接是什么  
4.学习格式控制方式  
5.学习自己写一个在线文档  
6.pnpm-worksapce.yaml  
7.前端测试  
8.nano-staged  
 9.husky  
10.输出 dist/style-deps.json 文件 这个文件是干什么的

11. vite 基本配置  
    root  
    plugin
12. hooks

 

### npm run dev 之后

先输出 dist/ package-entry.js style-deps.json  
合并处理 vantConfig，返回 vite 的 config  
创建 createServer，打印 url

```javascript
{
  root: '/Users/yinzhuo/Desktop/yinzhuo/node_modules/@vant/cli/site',
    // 项目根目录（index.html 文件所在的位置）。可以是一个绝对路径，或者一个相对于该配置文件本身的相对路径。
  plugins: [
    {
      name: 'vite-plugin(vant-cli):gen-site-base-code',
      resolveId: [Function: resolveId],
      load: [Function: load]
    },
    {
      name: 'vite:vue',
      handleHotUpdate: [Function: handleHotUpdate],
      config: [Function: config],
      configResolved: [Function: configResolved],
      configureServer: [Function: configureServer],
      buildStart: [Function: buildStart],
      resolveId: [AsyncFunction: resolveId],
      load: [Function: load],
      transform: [Function: transform]
    },
    {
      name: 'vite-plugin-md',
      enforce: 'pre',
      transform: [Function: transform],
      handleHotUpdate: [AsyncFunction: handleHotUpdate]
    },
    {
      name: 'vite:vue-jsx',
      config: [Function: config],
      configResolved: [Function: configResolved],
      resolveId: [Function: resolveId],
      load: [Function: load],
      transform: [Function: transform]
    },
    {
      name: 'vite:inject-html',
      enforce: 'pre',
      configResolved: [Function: configResolved],
      transformIndexHtml: [Object]
    }
  ],
  server: { host: '0.0.0.0' }
}
```

### npm run build 之后

- 清空文件
- 安装依赖 默认使用 yarn yarn install --prod=false
- copySourceCode 把 src 目录代码复制到 es lib
- buildPackageScriptEntry es lib 下加入 index.js，在 es 的 index.js 中写入组件入口固定模版，lib 复制到 es 目录下
- buildStyleEntry 生成样式入口文件
- buildPackageStyleEntry lib 下生成样式文件
- buildTypeDeclarations 生成声明文件
- buildESMOutputs 
- buildCJSOutputs
- buildBundledOutputs

   
   
 

 

<br>
  
> 语雀地址 https://www.yuque.com/yuqueyonghuyv23kd/xebk9e/zzkglu