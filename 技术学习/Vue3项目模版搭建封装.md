# Vue3项目模版搭建封装
## 前言

看到咱们目前业务里，项目搭建的比较乱，每个项目使用的都不太一样，不利于统一规范化管理，我结合已有的业务积累和业内优秀的实践经验，搭建了一套模板项目，在这里和大家做一个分享。  
如果能有所收获，或者帮助到大家，那真是太好了！

# 功能点

本项目是基于 vue-cli4.x，webpack5，对 vue-cli 提供的框架做的二次封装，主要封装的功能点主要有以下

- [x] 多入口打包
- [x] 自动化生成项目基本模版
- [x] pinia 状态管理库
- [x] 持久化存储插件封装
- [x] 路由动画的封装
- [x] axios 二次封装
- [x] less sass 变量，函数的处理
- [x] viewport 适配方案
- [x] 配置多环境变量
- [x] vconsole.js
- [x] 链式操作符
- [x] 入口加载动画

## 多入口打包

### 适用场景

移动端开发，一般情况，我们项目开发只需要一个配置一个入口，一个模版文件。但是有时候又明显不够用，移动端，同一个业务下，同类型的项目，就很适合放在一个项目下做多入口打包。  
比如以下场景：

- 活动项目，同类型活动多期迭代，不同类型的活动，新增一个活动就重新建立一套项目，维护成本很大，此时就很适合一个大的活动项目里，去新增多个活动页面入口。
- 当我们去写 App 的内嵌 h5 页面时，每个流程其实都是相互独立的，都是在一个 app 下，也很适合，多入口，维护在一个项目里。

### 优势

- 多入口，项目内各个入口独立，低耦合，不会相互影响。
- 发布成本低，可以维护在一个项目里，不用新建多个项目。
- 结构清晰，可维护性比较好。
  > 这一点主要是说明，如果我们全部使用单入口，也就是 vue 官方提供的框架，可以设想一下，如果是活动页面，我们把所有活动的页面都定义在一个入口里，router，store 全部在一块，项目耦合会越来越严重，路由名也会重合率越来越高！

### 项目结构

如下图所示，我们在 src 目录下面，新增了 module 目录，用来存放我们的多个项目入口，page1 和 page2 就是我用来演示的两个项目文件。  
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1572912/1659075267379-4cc0cf8f-fe71-45b1-adac-7d9ec1c5c4cc.png#averageHue=%233c4146&clientId=uc7c22fba-3af4-4&from=paste&height=925&id=uf51ae4be&name=image.png&originHeight=752&originWidth=345&originalType=binary&ratio=1&rotation=0&showTitle=false&size=56921&status=done&style=none&taskId=ud8452ec2-a0f0-4e46-a58f-7daba92bb21&title=&width=424.5)

### 代码实现

首先介绍几个前置依赖库

#### node 的 glob 库

glob 其实是 linux shell 默认的通配符，主要是做文件匹配，**glob 模式通常用来匹配目录以及文件**  
**这里介绍的 node 的一个方法库 glob**  
glob 在一般用于文件系统中定位文件  
主要用到了同步搜索文件  
**glob.sync(pattern, [options])**

- pattern {String} 待匹配的模式
- options {Object}
- return: {Array<String>} 匹配模式的文件名

#### vue-cli 的 pages  配置

vue-cli 接收一个参数，用来控制输入文件，输出文件  
详情可参考文档[pages](https://cli.vuejs.org/zh/config/#pages)  
官方例子如下  
比较重要的三个参数就是 entry template filename  
 page 的入口，模板来源， 在 dist/index.html 的输出

```javascript
module.exports = {
  pages: {
    index: {
      // page 的入口
      entry: "src/index/main.js",
      // 模板来源
      template: "public/index.html",
      // 在 dist/index.html 的输出
      filename: "index.html",
      // 当使用 title 选项时，
      // template 中的 title 标签需要是 <title><%= htmlWebpackPlugin.options.title %></title>
      title: "Index Page",
      // 在这个页面中包含的块，默认情况下会包含
      // 提取出来的通用 chunk 和 vendor chunk。
      chunks: ["chunk-vendors", "chunk-common", "index"],
    },
    // 当使用只有入口的字符串格式时，
    // 模板会被推导为 `public/subpage.html`
    // 并且如果找不到的话，就回退到 `public/index.html`。
    // 输出文件名会被推导为 `subpage.html`。
    subpage: "src/subpage/main.js",
  },
};
```

#### devServer 的 setupMiddlewares

提供执行自定义函数和应用自定义中间件的能力。  
我们可以在 devServer 启动项目之前，做一些自定义的处理  
我在启动时渲染了一段 html

```javascript
 devServer: {
    host: '0.0.0.0',
    setupMiddlewares: (middleware, devServer) => {
      devServer.app.get('', (_, response) => {
        response.write(html)
        response.end()
      })
      return middleware
    },
    proxy: {}
 }
```

#### 核心代码实现

```javascript
const entryName = "";
const entry1 = glob.sync("src/main.js"); // 单入口
const entry2 = glob.sync("src/module/*/main.js"); // 多入口
const entryList = [...entry1, ...entry2];
const pages = {}; //pages 参数
let html = ""; // 要渲染的html

entryList.forEach((filePath) => {
  const name1 = filePath.match(/src\/main.js$/); // 单入口
  const name2 = filePath.match(/src\/module\/(\w+)\/main.js$/); // 多入口
  const name = name1 ? name1[1] : name2[1];

  const filename = name1 ? name + "/index" : name;
  if (filename === entryName || !entryName) {
    const pagePath = glob.sync(`src/module/${name}/index.html`)[0];
    // 读文件夹下的index.html文件
    pages[name] = {
      entry: filePath,
      template: pagePath || "public/index.html",
      // 如果没有自动去取public下的通用index.html
      filename: filename + "/index.html",
    };
    // 构造html 结构
    html += `<a href="${filename}/">${filename}/</a><br>`;
  }
});
console.log(pages);
```

#### pages 结构

hehehhe 项目的 index.html 文件是取自 public，其他的是取自本项目  
考虑到部分页面会有在入口文件引用第三方的库的情况，比如 swiper,jq。  
所以这里设置的比较灵活，可以直接用通用的项目模版，也可以在项目里自定义模版。

```javascript
{
  hehehe: {
    entry: 'src/module/hehehe/main.js',
    template: 'public/index.html',
    filename: 'hehehe/index.html'
  },
  page1: {
    entry: 'src/module/page1/main.js',
    template: 'src/module/page1/index.html',
    filename: 'page1/index.html'
  },
  page2: {
    entry: 'src/module/page2/main.js',
    template: 'src/module/page2/index.html',
    filename: 'page2/index.html'
  }
}
```

### 选择编译

当随着目录越来越大，我们启动项目编译过程也会越来越长，大多时候，我们只需要修改多入口的一个页面，因此启动一个页面很有必要

```javascript
npm run dev --page=xxxx
```

### 打包目录

![image.png](https://cdn.nlark.com/yuque/0/2022/png/1572912/1659270705199-07a1dd80-0b32-4a17-bec9-03871be3c62c.png#averageHue=%230a0a0a&clientId=ue2c240f3-82ff-4&from=paste&height=1024&id=ufded46f3&name=image.png&originHeight=1024&originWidth=722&originalType=binary&ratio=1&rotation=0&showTitle=false&size=135510&status=done&style=none&taskId=u27f15ad0-10ae-4438-b8df-3cc234e6fc0&title=&width=722)

## 自动化生成项目基本模版

当我们在新建项目时，一般是手动新建文件夹，然后定义项目名字，新建入口文件，index.html，.vue 文件，新建 router store 文件等等，这个是每次新建时必不可少的步骤。  
其实，初始化项目的时候，新建的内容都差不多，如果我们能用一行指令，帮助我们生成这些模版文件，我们只需要定义个文件名字，可以显著提升开发体验。

### 新建项目指令

我在 package.json 里新增了一个指令 init

```json
"init": "node ./src/common/initTemplate/index.js"
```

当执行这个命令时，会自动去执行我在本地 common 目录下新建的 js 脚本，在 module 目录下自动生成一个新的文件。

### 演示

- 输入 npm run init

会提示用户“正在创建项目”

- 要求输入项目名称

![image.png](https://cdn.nlark.com/yuque/0/2022/png/1572912/1659079382181-b360ae79-4455-49c1-a862-dba35cfc8beb.png#averageHue=%23303030&clientId=uc7c22fba-3af4-4&from=paste&height=194&id=u11f91631&name=image.png&originHeight=235&originWidth=822&originalType=binary&ratio=1&rotation=0&showTitle=false&size=28697&status=done&style=none&taskId=u96f3f45d-1941-4842-ad55-3cc86c3f237&title=&width=678)

- 确认之后

返回用户“该项目已经创建”，我们在 src 项目下能看到刚刚新建的目录 hehehe，有内置的 index.vue，App.vue，main.js，router.js  
因为考虑到并不是所有用户都会用到 store，所以基础模版没有引入。  
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1572912/1659079567963-e2b95363-eb7b-4cff-b336-6bdc5e1b1378.png#averageHue=%233c3f41&clientId=uc7c22fba-3af4-4&from=paste&height=248&id=uf46c0c16&name=image.png&originHeight=275&originWidth=360&originalType=binary&ratio=1&rotation=0&showTitle=false&size=20573&status=done&style=none&taskId=u2515e3b1-be0c-4210-8050-732f0146ebb&title=&width=324)

- 执行 npm run dev 之后

就可以在浏览器里看到我们刚刚生成的 hehehe 项目了！  
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1572912/1659080287575-2864d1f8-5a17-4123-a467-05a29ec9a891.png#averageHue=%23fcfcfb&clientId=uc7c22fba-3af4-4&from=paste&height=171&id=ubdc60d51&name=image.png&originHeight=342&originWidth=1004&originalType=binary&ratio=1&rotation=0&showTitle=false&size=106371&status=done&style=none&taskId=u99f21124-53eb-461e-ae84-041097a6047&title=&width=502)

- 重复名检测

当新建的项目已有时，会提示用户，“目录已经存在”，要求用户继续输入  
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1572912/1659080375617-0fd8a3d0-efa5-4ba6-8564-6f549ba9f13f.png#averageHue=%23313131&clientId=uc7c22fba-3af4-4&from=paste&height=123&id=uf4f88c56&name=image.png&originHeight=245&originWidth=807&originalType=binary&ratio=1&rotation=0&showTitle=false&size=34272&status=done&style=none&taskId=ue4bf4f99-6784-4480-86d8-d6d19e32869&title=&width=403.5)

### 代码实现

下图是 initTempate 文件结构  
实现比较简单  
核心代码在 src/common/initTemplate/index.js  
template 文件夹是我们要生成的项目模版

```json
initTemplate
├── index.js
└── template
    ├── App.vue
    ├── main.js
    ├── router.js
    └── view
        └── index.vue
// 2 directories, 5 files
```

#### inquirer

这个库用来询问用户输入项目名称，这是一个比较在处理命令行交互比较常见的库，详情可以查阅文档  
[前端 Node 命令行交互工具](http://www.baidu.com/link?url=LwKD5CZQCby8PJ57ipiAhi7XBu_A47JlhxLA7k-AhE_IeymkktWklIxSXqfOIZs_Pz9DlKLxlxrkZXGJTMs5Vq) —— inquirer  
用 vue-cli 脚手架新建项目的应该都进行过命令交互，vue 创建的时候会让你选择 vue2 还是 vue3，也有多选要什么配置，也有输入 y 或者 n 选择是否用 history 路由等，这其实用 inquire 这个包都能实现。

#### fs.access

fs.access()方法可以用于测试文件是否存在

#### 流程图

![](https://cdn.nlark.com/yuque/0/2022/jpeg/1572912/1659082239050-a45c0097-b254-4864-bfa7-3de280a4742a.jpeg)

#### 代码

```json
#!/usr/bin/env node
console.log('您正在创建项目')
const path = require('path')
const fs = require('fs')
const inquirer = require('inquirer')
const stat = fs.stat
const targetDir = path.resolve(__dirname, './template')
//copy文件目录常用函数，都是常见api,
const copyFile = (targetDir, resultDir) => {
  // 读取文件、目录
  fs.readdir(targetDir, function(err, paths) {
    if (err) {
      throw err
    }
    paths.forEach(function(p) {
      const target = path.join(targetDir, '/', p)
      const res = path.join(resultDir, '/', p)
      let read
      let write
      stat(target, function (err, statsDta) {
        if (err) {
          throw err
        }
        if (statsDta.isFile()) {
          read = fs.createReadStream(target)
          write = fs.createWriteStream(res)
          read.pipe(write)
        } else if (statsDta.isDirectory()) {
          fs.mkdir(res, function() {
            copyFile(target, res)
          })
        }
      })
    })
  })
}

const question = [
  {
    type: 'input',
    name: 'name',
    message: '请输入项目名称：'
  }
]

const createProject = () => {
  // 询问用户问题
  inquirer.prompt(question).then(({ name }) => {
    // name 为输入的项目名称
    name = name.trim()
    if (!name) {
      console.log('项目目录不能为空')
      // 如果输入空，继续询问
      createProject()
      return false
    }
    // 目标路径，要放在module目录下
    const resultDir = path.resolve(__dirname, '../../module/', name)
    // fs.access()方法用于测试文件是否存在
    fs.access(resultDir, function(err, data) {
      if (err) {
        // 创建文件
        fs.mkdir(resultDir, function(err, data) {
          if (err) {
            throw err
          }
          // 复制模版文件
          copyFile(targetDir, resultDir)
        })
        console.log(`${name} 项目已创建成功`)
      } else {
        console.log(`${name} 项目目录已存在，请输入其他名称`)
        // 不存在，继续询问
        createProject()
      }
    })
  }).catch(err => {
    console.log(err)
  })
}
createProject()
```

## Pinia 状态管理

![image.png](https://cdn.nlark.com/yuque/0/2022/png/1572912/1659193040042-1ca2071a-4e0e-4217-a512-8d20a129778c.png#averageHue=%23fefcf6&clientId=ua0cf60ba-bf7a-4&from=paste&height=948&id=u2bc233b4&name=image.png&originHeight=948&originWidth=1562&originalType=binary&ratio=1&rotation=0&showTitle=false&size=136497&status=done&style=none&taskId=uc5997acd-1bda-4083-a41e-c95cb965060&title=&width=1562)  
文档地址：[https://pinia.web3doc.top/](https://pinia.web3doc.top/)  
Pinia /piːnjʌ/ 中文名：皮你啊

### Pinia 优势

**1.Pinia 是一个全新的 Vue 状态管理库，是 Vuex 的代替者，尤雨溪强势推荐**  
翻译翻译 => 官方背书  
**2.Vue2 和 Vue3 都能支持**  
翻译翻译 => 既支持 options api 又支持 compostions api，维护成本低  
**3.抛弃传统的 Mutation ，只有 state, getter 和 action ，简化状态管理库**  
翻译翻译 => 少写代码,降低心智负担，再也不用写 mutation  
**4.不需要嵌套模块，符合 Vue3 的 Composition api，让代码扁平化**  
翻译翻译 => 多个 state,不需要再使用 module，现在可以定义多个 store，相互之间调用  
**5.TypeScript 支持**  
翻译翻译 => 没有使用 ts，不好解释  
**6.代码简单，很好的代码自动分割**  
**翻译翻译 => **可以构建多个 store，打包管理会自动拆分模块化的设计，便于拆分状态，能很好支持代码分割；  
**7.极轻， 仅有 1 KB**  
翻译翻译 => 体积小，不会成为项目的负担

### 核心概念

1. **State**: 用于存放数据，有点儿类似 data 的概念；
2. **Getters**: 用于获取数据，有点儿类似 computed 的概念；
3. **Actions**: 用于修改数据，有点儿类似 methods 的概念；
4. **Plugins**: Pinia 插件。

### Pinia 与 Vuex 代码分割机制

举个例子：某项目有**3 个 store「user、job、pay」**，另外有**2 个路由页面「首页、个人中心页」**，**首页用到 job store，个人中心页用到了 user store**，分别用 Pinia 和 Vuex 对其状态管理。  
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1572912/1659255649356-4ed7d350-c971-41c5-a06b-979cb382cdcd.png#averageHue=%23fafaf4&clientId=ua0cf60ba-bf7a-4&from=paste&height=417&id=u04feac2e&name=image.png&originHeight=417&originWidth=620&originalType=binary&ratio=1&rotation=0&showTitle=false&size=31095&status=done&style=none&taskId=ua34627da-470c-4e8d-9dd3-046268a45f1&title=&width=620)

#### Vuex 的代码分割

打包时，vuex 会把 3 个 store 合并打包，当首页用到 Vuex 时，这个包会引入到首页一起打包，最后输出 1 个 js chunk。这样的问题是，其实首页只需要其中 1 个 store，但其他 2 个无关的 store 也被打包进来，造成资源浪费。  
**解决方案：**  
经常在首页优化时，会考虑到这个场景，一般处理方案是去做 vuex 的按需加载，beforeCreate 的时候，可以去判断当前页面需要加载哪些 store,利用这个 api 可以实现$store.registerModule  
详情可参考文章  
[vuex 按需加载，避免首页初始化所有数据](https://segmentfault.com/a/1190000038440206)

#### pinia 的代码分割

打包时，Pinia 会检查引用依赖，当首页用到 job store，打包只会把用到的 store 和页面合并输出 1 个 js chunk，其他 2 个 store 不耦合在其中。Pinia 能做到这点，是因为它的设计就是 store 分离的，解决了项目的耦合问题。  
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1572912/1659257283076-e37aed9d-d91a-45ca-8755-541b6186de04.png#averageHue=%23fafaf4&clientId=ua0cf60ba-bf7a-4&from=paste&height=421&id=u2c8214e0&name=image.png&originHeight=421&originWidth=593&originalType=binary&ratio=1&rotation=0&showTitle=false&size=27920&status=done&style=none&taskId=uce6d3862-be55-40fd-bc2e-92e3172e2bc&title=&width=593)

### 基本使用

定义 state，getters 和 vuex 基本一样，具体使用可以去官网学  
这里仅仅对比修改数据时，两者的区别  
明显 pinia 的代码量更少，逻辑更清晰，心智负担更小

```javascript
//store.js
import { defineStore } from "pinia";

export const main = defineStore("main", {
  state: () => {
    return {
      configInfo: {},
    };
  },
  getters: {},
  actions: {},
});

import * as piniaStore from "../piniaStore";
piniaStore.main().$patch({
  configInfo: data,
});
```

```javascript
//store.js
import { createStore } from "vuex";

export const store = createStore({
  state: {
    count: 0,
  },
  getters: {},
  mutations: {
    increment(state) {
      state.count++;
    },
  },
  actions: {},
  modules: {},
});
export default store;

//index.vue
import { useStore } from "vuex";
const storeVuex = useStore();

storeVuex.commit("increment");
```

### 总结

总得来说，Pinia 就是 Vuex 的替代版，可以更好的兼容 Vue2，Vue3 以及 TypeScript。在 Vuex 的基础上去掉了 Mutation，只保留了 state, getter 和 action。Pinia 拥有更简洁的语法， 扁平化的代码编排，符合 Vue3 的 Composition api

## Pinia 数据持久化插件

### 使用场景

把数据缓存下来，可以避免页面刷新时，重复调用接口，提升用户体验

### 封装 sessionStorage localStorage

代码在 src/common/utils/storage.ts

```javascript
let hasSessionStorage = true;
let hasLocalStorage = true;
//判断当前浏览器是否支持sessionStorage
if (sessionStorage) {
  try {
    sessionStorage.setItem("_sessionStorageTest", "Hello World!");
    sessionStorage.removeItem("_sessionStorageTest");
  } catch (e) {
    hasSessionStorage = false;
  }
} else {
  hasSessionStorage = false;
}
//判断当前浏览器是否支持localStorage
if (localStorage) {
  try {
    localStorage.setItem("_localStorageTest", "Hello World!");
    localStorage.removeItem("_localStorageTest");
  } catch (e) {
    hasLocalStorage = false;
  }
} else {
  hasLocalStorage = false;
}
/**
 * 设置本地缓存
 * @param key
 * @param val
 */
export function setLocalStorage(key: string, val?: any): void {
  if (!hasLocalStorage) {
    return;
  }
  localStorage.setItem(key, JSON.stringify(val));
}
/**
 * 设置会话级别缓存
 * @param key
 * @param val
 */
export function setSessionStorage(key: string, val: any): void {
  if (!hasSessionStorage) {
    return;
  }
  sessionStorage.setItem(key, JSON.stringify(val));
}
```

### 代码实现

核心就是，Pinia 的监听 API subscribe  
state 中的数据变化时，就会触发 subscribe  
这样我们就可以判断当前变化的 store 的 id，是否在我们的需要持久化的数组中  
如果在，我们就将数据存到本地缓存

```javascript
const getStorageTypeMap: AnyObj = {
  sessionStorage: getSessionStorage,
  localStorage: getLocalStorage,
};

const setStorageTypeMap: AnyObj = {
  sessionStorage: setSessionStorage,
  localStorage: setLocalStorage,
};

const plugin = (options: Options): any => {
  // key 为标识前缀，避免命名空间冲突
  const { key, storeList } = options;
  return (context: PiniaPluginContext) => {
    const { store } = context;
    let storageType: any;
    let obj: any = {};
    for (const item of storeList) {
      if (item.storeName.includes(store.$id)) {
        // storeName 为哪个store，path 为store下某个字段
        const { storeName, path } = item;

        storageType = item.storageType;
        // 如果key 不存在默认走 pinia
        const data = getStorageTypeMap[storageType](
          `${key ?? "pinia"}-${store.$id}`
        );
        if (data) {
          // 更新store
          store.$patch(data);
        }
        if (path && path.length > 0) {
          // 如果存在path 则需要判断
          if (storeName.length === 1) {
            path.forEach((item) => {
              obj[item] = store.$state[item];
            });
          } else {
            return new Error("配置path 时只允许配置一个storeName");
          }
        }
        obj = path && path.length > 0 ? obj : store.$state;
        storeName.includes(store.$id) &&
          store.$subscribe(() => {
            setStorageTypeMap[storageType](
              `${key ?? "pinia"}-${store.$id}`,
              toRaw(obj)
            );
          });
      }
    }
  };
};
```

### 全局引入

这个是我定义的 store 文件，里面定义了三个 store,分别为 main，test，test1

```javascript
// piniaStore.js
import { defineStore } from "pinia";

export const main = defineStore("main", {
  state: () => {
    return {
      test: "hello word",
      test1: "hello word1",
      configInfo: {},
    };
  },
  getters: {},
  actions: {},
});

export const test = defineStore("test", {
  state: () => {
    return {
      age: 18,
      name: "yz",
    };
  },
});

export const test1 = defineStore("test1", {
  state: () => {
    return {
      age: 18,
      name: "yz",
    };
  },
});
```

在入口文件处 引入插件，然后对进行需要存储的 store 和字段进行配置

- key 为这个项目使用的一个命名前缀，保证不会污染到其他缓存数据
- storeName 为一个数组，可以为空，默认存储所有 store，可以配置自己需要存储的 store 名字
- storageType 为字符串，可以配置需要会话级存储还是本地化存储
- path 为一个数组，可以为空，默认存储该 store 下所有字段，可以配置自己需要的字段

```javascript
import { createPinia } from "pinia";
import piniaPlugin from "@/common/utils/piniaPlugin";
// 创建pinia 实例
const pinia = createPinia();
pinia.use(
  piniaPlugin({
    key: "XXX", // 这是给缓存到本地时，加一个特殊的前缀，以免造成污染到其他缓存数据
    storeList: [
      {
        storeName: ["main"], // 对于特定store进行持久化，空或者不传，则对所有的store进行缓存到本地
        storageType: "sessionStorage",
        path: ["configInfo"],
      },
      {
        storeName: ["test"], // 对于特定store进行持久化，空或者不传，则对所有的store进行缓存到本地
        storageType: "sessionStorage",
      },
      {
        storeName: ["test1"], // 对于特定store进行持久化，空或者不传，则对所有的store进行缓存到本地
        storageType: "localStorage",
      },
    ],
  })
);
```

存储之后的结果，可以在浏览器里看到  
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1572912/1659408422586-de00c09b-72fe-466a-b259-e75c4f11c1ea.png#averageHue=%23fdfcfc&clientId=u0809bbdf-46d1-4&from=paste&height=716&id=u875d0ef2&name=image.png&originHeight=1432&originWidth=2230&originalType=binary&ratio=1&rotation=0&showTitle=false&size=233233&status=done&style=none&taskId=u436120cf-936e-41ce-98a1-3772c65e368&title=&width=1115)  
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1572912/1659410572569-da764f08-33fc-46d4-aee8-3197be0b3f89.png#averageHue=%23fdfdfd&clientId=u0809bbdf-46d1-4&from=paste&height=692&id=u5baa2987&name=image.png&originHeight=1384&originWidth=2106&originalType=binary&ratio=1&rotation=0&showTitle=false&size=254046&status=done&style=none&taskId=ubae19b06-f336-4005-a948-b3375f4240f&title=&width=1053)

### 参考库

[https://github.com/Seb-L/pinia-plugin-persist](https://github.com/Seb-L/pinia-plugin-persist)  
当时是参考了网路上一个开源库的实现  
这个库如果使用，需要每个 store 下，都进行配置。  
我们开发时，不可能只定义一个 store，一般是按功能，模块划分代码，保证可读性。  
所以我觉得使用这个库，代码开发成本更大  
因此在这个基础上自行写了一套持久化插件，在入口处全局管理

```javascript
export const useUserStore = defineStore("storeUser", {
  state() {
    return {
      firstName: "S",
      lastName: "L",
      accessToken: "xxxxxxxxxxxxx",
    };
  },
  persist: {
    enabled: true,
    strategies: [
      {
        key: "user", //自定义 Key值
        storage: localStorage, // 选择存储方式
      },
    ],
  },
});
```

### 总结

我现在开发的插件一定程度上，增加了使用者的心智负担，需要去了解配置项的规则。  
持久化存储的场景非常多，对于小型页面小型项目，直接在修改 store，再手动设置一次 sessionStroage，在页面中使用的时候，再去主动取一下 sessionStorage 即可，没有任何问题。  
但是如果是对于多人维护的大型项目，如果这么写，随着迭代，将搞不清楚到底哪些数据存在了 session 中，在什么时机存储的，无法统一管理。  
有这样一个插件，就可以在全局统一配置，增删改查在入口文件统一管理。

## 路由动画的封装

Vue 路由过渡动画是对 Vue 程序一种快速简便的增加个性化效果的的方法。  
可以让你在程序的不同页面之间增加平滑的动画和过渡。  
如果使用得当，可以使你的程序显得更加专业，从而增强用户体验。  
页面切换的动画时间的同时，下一个页面初始化也在进行了，对用户体验来说，可以有效避免下一个页面的加载 dom，初始化页面的时间。

### 封装思路

- 使用 transition 方式给根路由设置全局动画
- 给 router 的路径设置 meta 的 level 层级
- 在入口页面的 setup 中，在路由守卫中，根据 level 的大小，设置相应的动画
- 定义相应的动画类样式

### 代码

css 写出动画效果  
根组件 App.vue，监听路由的变化，如果 to 索引大于 from 索引,使用前进的动画,反之使用后退的动画  
level 可以使用数字，也可以字母，也可以数字加字母，能体现大小关系即可  
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1572912/1659085747542-b2f29a77-265c-43f4-8122-c681f53576a5.png#averageHue=%232d2b2b&clientId=u0ad80e95-051d-4&from=paste&height=545&id=u0cfdd8e6&name=image.png&originHeight=638&originWidth=541&originalType=binary&ratio=1&rotation=0&showTitle=false&size=53213&status=done&style=none&taskId=ucf7af2ef-1737-4b10-9e8c-b5307ca437e&title=&width=462.5)

```json
export default {
  setup() {
    const router = useRouter()
    //默认值
    const state = reactive({
      transitionName: 'slide-left'
    })
    router.beforeEach((to, from) => {
      if (to.meta.level > from.meta.level) {
        state.transitionName = 'slide-left'// 向左滑动
      } else if (to.meta.level < from.meta.level) {
        // 由次级到主级
        state.transitionName = 'slide-right'// 向右滑动
      } else {
        state.transitionName = ''// 同级无过渡效果
      }
    })

    return {
      ...toRefs(state)
    }
  }
}
```

### 效果展示

![QQ20220729-172949-HD.gif](https://cdn.nlark.com/yuque/0/2022/gif/1572912/1659087049176-0068cb95-914d-4eb7-b5ae-38439c2461b5.gif#averageHue=%23cf7da5&clientId=u0ad80e95-051d-4&from=ui&id=u33ec7add&name=QQ20220729-172949-HD.gif&originHeight=600&originWidth=349&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9513513&status=done&style=none&taskId=u848e48c4-237b-41ae-a882-1db3a53a796&title=)  
对比不加动画的，效果还是非常好的  
![QQ20220729-173253-HD.gif](https://cdn.nlark.com/yuque/0/2022/gif/1572912/1659087198172-00e43c55-b5ec-4531-9f4b-52b7b72451c4.gif#averageHue=%23d07ca4&clientId=u0ad80e95-051d-4&from=drop&id=u7b0256d2&name=QQ20220729-173253-HD.gif&originHeight=600&originWidth=352&originalType=binary&ratio=1&rotation=0&showTitle=false&size=5992183&status=done&style=none&taskId=ub468f606-a363-476e-8467-d01f934eeb3&title=)

### 修改动画样式

一个项目里使用的肯定是一个动画，目前我在项目中默认使用的左滑，右滑。  
可以根据业务需要，和自己的喜好，去使用更多的动画效果。  
可参考大佬写的过渡效果，有兴趣可以再研究研究  
[4 个 Vue 路由过渡动效](https://juejin.cn/post/6963205355702583303#heading-4)

## axios 二次封装

### 封装目的

- 降低心智负担
- 减少冗余代码
- 使用更加高效

### 封装效果

下图是封装前后的使用代码对比  
我们在业务调用中，省略了 showLoading 这个过程，不关心业务 code 和 msg，可以直接获取 data 进行处理，省略了显示错误信息的过程，所以代码量大大减小。

- 封装后

![image.png](https://cdn.nlark.com/yuque/0/2022/png/1572912/1659176446819-2c87c5fc-5d87-48f7-b867-f1dabc8ef5e4.png#averageHue=%232f2e2c&clientId=uf171db09-bd4c-4&from=paste&height=254&id=u4007ab21&name=image.png&originHeight=254&originWidth=1086&originalType=binary&ratio=1&rotation=0&showTitle=false&size=47619&status=done&style=none&taskId=udf33b770-ba1d-48b9-b309-0ff4c862fc6&title=&width=1086)

- 封装前

![image.png](https://cdn.nlark.com/yuque/0/2022/png/1572912/1659176636736-2491229b-a0bc-45e6-867d-5fb880dfcd41.png#averageHue=%232d2e2c&clientId=uf171db09-bd4c-4&from=paste&height=788&id=u619e7ae4&name=image.png&originHeight=788&originWidth=1256&originalType=binary&ratio=1&rotation=0&showTitle=false&size=301583&status=done&style=none&taskId=ud529cf9e-e75a-4739-9bd1-e470f3bec4d&title=&width=1256)

### 通用能力

这里说明一下我封装时候的思路和想法  
首先列一下我想要这个通用请求能达到什么样的效果  
1.正常请求基础的配置，比如超时配置，baseUrl，跨域携带 cookie 等等  
2.响应拦截处理

- 请求成功，业务状态码成功，直接解析接口中的 data，不用一层一层再去取 code，判断，拿结果
- 请求成功，业务状态码不成功，可以选择自己处理特殊状态码，也可以选择全局 message 提示服务端的报错，业务开发中，大部分都是直接提示服务端报错，但是也有需要前端处理状态码的逻辑
- 请求失败，全局 messagege 提示报错
- 统一的特殊请求码处理，或者状态码做特殊逻辑，比如丢失登陆态，请求参数有误等等

  3.全局统一的 loading 配置

- 默认开启，可配置关闭
- 统一管理，业务中不用再去关心这个逻辑

### 代码实现

#### 基础类型

config 是传入参数

- baseURL 是请求地址
- timeOut 是请求超时时间
- slientError 是我自定义的值，拿到的结果非成功的请求时，这个参数控制，要不要直接 message 服务端返回的错误，一般都是直接返回，所以默认值时 false
- loading 是我自定义的值，用来控制，调用接口是需不需要展示 loading，业务开发中大部分都需要展示 loading 提示用户，防止用户多次点击，少部分场景不想让用户感知在调用接口，需要关闭，所以默认值我定义了 true

```javascript
type TAxiosOption = {
  baseURL: string
  timeout: number
  slientError: boolean
  loading: boolean
}

const config:TAxiosOption = {
  baseURL: process.env.VUE_APP_ACTIVITY_SERVER_TARGET,
  timeout: process.env.VUE_APP_API_TIMEOUT,
  slientError: false,
  loading: true
}

class Request {
  instance:AxiosInstance
  constructor(config: TAxiosOption) {
    this.instance = axios.create(config)
    this.instance.interceptors.request.use(data=>{
      return data
    })
    this.instance.interceptors.response.use(data=>{
      return data
    }）
  }
  get<T, U>(url: string, data?: U, config = {}): Promise<T> {
    return this.instance.get(url, { params: data, ...config })
  }

  post<T, U>(url: string, data?: U, config = {}): Promise<T> {
    return this.instance.post(url, data, config)
  }
}
export default new Request(config)
```

#### api 管理

我们在调用的时候，可以在 post 的第三个参数中，传入自己需要的配置  
如果不传全部走默认值  
slientError: true 表示业务中自己处理异常，网路库不 message 异常  
需要我们在业务中的 catch 中 就可以拿到 接口返回的所有信息，去做相应的处理  
loading: true 表示展示 loading

```javascript
import request1 from "@/common/axios/request1";
// 查询拉新活动规则
export function queryInviteActivityRule(params) {
  return request1.post(
    "/kyf-activity-api/activity/queryInviteActivityRule",
    params,
    { slientError: true, loading: true }
  );
}
```

#### 全局统一 loading 处理

在请求拦截器中 执行 addLoading  
在响应拦截器中 执行 cacelLoaing  
保证全局无论多少接口调用，requestNum 参数 控制都只有一个 loading  
目前的 shoowLoading 引用的是 vant 组件，也可以根据自己业务需要去自定义 loading 组件

```javascript
let requestNum = 0;

const addLoading = () => {
  // 增加loading 如果pending请求数量等于1，弹出loading, 防止重复弹出
  requestNum++;
  if (requestNum === 1) showLoading();
};
const cancelLoading = () => {
  // 取消loading 如果pending请求数量等于0，关闭loading
  requestNum--;
  if (requestNum === 0) hideLoading();
};
```

#### 响应拦截器

拦截器的代码可以直接去看我的源代码  
在 src/common/axios/request.ts 中  
主要就是在响应拦截器中对返回值做了处理，如果成功，直接返回结果，不成功，返回接口全部内容

```javascript
this.instance.interceptors.response.use(
  (responseData: MyAxiosResponse) => {
    const status = responseData.status;
    const res = responseData.data;
    const { data, code } = res;
    const { loading = true } = responseData.config;
    if (loading) cancelLoading();
    // 如果调用其他中台，此处需要单独对code做转化
    if (status === 200 && code === "000001") {
      return Promise.resolve(data || res || {});
    } else {
      // 此处可以处理其他特殊的业务逻辑
      // 比如丢失登陆态，传参数有误等逻辑
      if (responseData.config.slientError) {
        return Promise.reject(res);
      }
      showErrorInfo(res.msg);
      return Promise.reject(res);
    }
  },
  (error: any) => {
    const { loading = true } = error.config;
    if (loading) cancelLoading();
    let errMsg;
    if (error && error.response) {
      if (error.response.status >= 500 && error.response.status <= 599) {
        errMsg = "服务器繁忙，请稍后重试";
      } else if (error.response.status === 404) {
        errMsg = "服务不存在";
      } else {
        errMsg = "网络繁忙，请稍后重试";
      }
    } else {
      errMsg = "( ⊙ o ⊙ )啊！网络不太顺畅哦~";
    }
    if (error.config.slientError) {
      return Promise.reject(error);
    } else {
      showErrorInfo(errMsg);
    }
    return Promise.reject(errMsg);
  }
);
```

#### 请求拦截器

请求拦截器中一般注入 token，我的代码中暂时没有做处理

## less sass 的优化处理

### 背景

我们使用 less sass 主要是为了使用他们提供的变量，函数等特性。如果不进行配置，只在入口文件引入，在各个子页面里是无法直接使用，如果使用，还需要再次引用，这个对使用者非常不友好。

### 官方方案

对 sass 文件 可以在 loaderOptions 增加 additionalData 选项  
但是对于 less 文件的处理非常不友好，需要一个个去定义变量，因此我了舍弃官方的 less 处理方案

```json
module.exports = {
  css: {
    loaderOptions: {
      // 给 sass-loader 传递选项
      sass: {
        // @/ 是 src/ 的别名
        // 所以这里假设你有 `src/variables.sass` 这个文件
        // 注意：在 sass-loader v8 中，这个选项名是 "prependData"
        additionalData: `@import "~@/variables.sass"`
      },
      // 默认情况下 `sass` 选项会同时对 `sass` 和 `scss` 语法同时生效
      // 因为 `scss` 语法在内部也是由 sass-loader 处理的
      // 但是在配置 `prependData` 选项的时候
      // `scss` 语法会要求语句结尾必须有分号，`sass` 则要求必须没有分号
      // 在这种情况下，我们可以使用 `scss` 选项，对 `scss` 语法进行单独配置
      scss: {
        additionalData: `@import "~@/variables.scss";`
      },
      // 给 less-loader 传递 Less.js 相关选项
      less:{
        // http://lesscss.org/usage/#less-options-strict-units `Global Variables`
        // `primary` is global variables fields name
        globalVars: {
          primary: '#fff'
        }
      }
    }
  }
}
```

### 单独处理 less

安装 style-resources-loader  
在 vue.config.js 新增配置  
对如果使用了 less，预先引入我们基础 less 文件

```javascript
pluginOptions: {
  'style-resources-loader': {
    preProcessor: 'less',
      patterns: [
        path.resolve(__dirname, './src/common/style/base.less')]
  }
}
```

### 总结

三个预处理器的全局引入方案，可以参考这个文章  
[CSS 预编译语言 变量全局引用方式 vue-cli@3.0 stylus/sass/less 使用方法](https://blog.csdn.net/rudy_zhou/article/details/103683140)  
一般来说，一个项目使用一个 css 预处理器，less 或者 sass，因为我极其不适应 stylus 的无括号缩进模式，所以就里没有说 stylus

只要在 webpack 引入了就不用再全局再次引入了，但是我始终觉得这种业务代码不应该出现在 webpack 中，这种方式大大增加了开发者的心智负担，也是现在脚手架设计不合理的地方，希望未来能有更好的解决方案！

至于为什么，在入口文件引入了，不配置的话，还不能使用 less，或者 sass 的。这个深层原因，我猜测和 vue 的框架设计有关，有兴趣可以深入研究

## viewport 适配方案

postcss-px-to-viewport 是一款 postcss 插件，用于将单位转化为 vw， 现在很多浏览器对 vw 的支持都很好，适配首选方案。

### PostCSS 配置

下面提供了一份基本的 postcss 配置，可以在此配置的基础上根据项目需求进行修改

```javascript
// postcss.config.js
const path = require("path");

module.exports = ({ file }) => {
  // const designWidth = file.includes(path.join('node_modules', 'vant')) ? 375 : 750
  return {
    plugins: {
      autoprefixer: {},
      "postcss-px-to-viewport": {
        // 要转化的单位
        unitToConvert: "px",
        // 视口的宽度
        viewportWidth: designWidth,
        // 保留几位小数
        unitPrecision: 3,
        // 哪些属性需要修改
        propList: ["*"],
        // 预期单位
        viewportUnit: "vw",
        // 字体单位
        fontViewportUnit: "vw",
        // 最小转化单位
        minPixelValue: 1,
        // 媒体查询里面的单位要不要转化
        mediaQuery: true,
        // 替换包含vw单位的
        replace: true,
        // 排除转化文件
        exclude: [],
        // 添加横向转化值
        landscape: false,
        // 横向采用的单位
        landscapeUnit: "vw",
        // 横屏的视口宽度
        landscapeWidth: false,
      },
    },
  };
};
```

### autoprefixer

如果要配置目标浏览器，可使用 package.json 的 [browserslist](https://cli.vuejs.org/zh/guide/browser-compatibility.html#browserslist) 字段  
你只使用无前缀的 CSS 规则即可

### 对比 rem

下面这个是 rem 适配方案肯定会有的代码

```javascript
const deviceWidth =
  document.documentElement.clientWidth || document.body.clientWidth;
document.querySelector("html").style.fontSize = deviceWidth / 7.5 + "px";
```

弊端：

- px 和 rem 需要一个计算比例系数，开发时需要计算，后来又提出 px-to-rem 不如直接在代码中写 px 直观高效。
- rem 是相对于 html 元素字体单位的一个相对单位，从本质上来说，它属于一个字体单位，用字体单位来布局，并不是太合适。

### 兼容第三方 UI 库

vant 团队的是根据 375px 的设计稿去做的，理想视口宽度为 375px。  
如果读取的是 vant 相关的文件，viewportWidth 就设为 375，如果是其他的文件，我们就按照我们 UI 的宽度来设置 viewportWidth，即 750  
目前网上没有找到完全正确的例子，博客错误的 demo 相互抄袭，我在做代码验证时走了很多弯路，现在下面这行代码肯定是可行的。

```javascript
// 核心代码就这么一行
const designWidth = file.includes(path.join("node_modules", "vant"))
  ? 375
  : 750;
```

下面是没有配置之前引入 vant 的 cell 组件，整个缩小了  
配置之后，样式恢复正常  
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1572912/1659148722660-1b9f4d3b-5b7e-4b7c-864d-4dc27a60f834.png#averageHue=%23c67fa2&clientId=uf171db09-bd4c-4&from=paste&height=641&id=u23b3bd46&name=image.png&originHeight=1286&originWidth=716&originalType=binary&ratio=1&rotation=0&showTitle=false&size=56521&status=done&style=none&taskId=u3f4c5976-618f-4208-b868-31288c1183e&title=&width=357)![image.png](https://cdn.nlark.com/yuque/0/2022/png/1572912/1659148591685-e69c1e28-0c0f-4189-bc78-41077020fe30.png#averageHue=%23c781a3&clientId=uf171db09-bd4c-4&from=paste&height=639&id=u0260835e&name=image.png&originHeight=1278&originWidth=728&originalType=binary&ratio=1&rotation=0&showTitle=false&size=218106&status=done&style=none&taskId=ud77fd811-2dbe-4f09-8b83-19a9c27f9c4&title=&width=364)

## 配置多环境变量

### 命令

package.json 里的 scripts 配置 build  
"build:test": "vue-cli-service build --mode test",  
"build:online": "vue-cli-service build --mode online  
可以根据自己业务情况去实际扩展

#### 环境变量

目录下 3 个环境变量文件

- .env
- .env.test
- .env.online

分别对应，通用的环境变量，测试环境变量，线上环境变量  
获取时，无论什么命令都会优先获取 .env 里的环境变量，再根据不同命令，执行不同的环境变量文件，遇到相同的环境变量会进行覆盖  
目前我对主要定义的几个环境变量进行说明

```javascript
# 是否是线上版本
VUE_APP_ONLINE_ENV=false
# 环境
NODE_ENV=development
# 请求超时时间
VUE_APP_API_TIMEOUT=30000
# public-path
VUE_APP_PUBLIC_PATH=/
# 请求后端地址
VUE_APP_ACTIVITY_SERVER_TARGET = xxx
```

## 兼容性处理方案

### 可选链操作符和空值合并操作符的兼容处理

我们判断嵌套对象属性是否存在时，常常会用到链式操作符 **?. **，是一个非常好用的语法糖  
高版本 api,需要加 babel 插件，转化成 es5  
我们对 babel.config.js 进行了单独配置  
同理空值合并操作符也是如此  
注释里已经很清晰了

```javascript
module.exports = {
  presets: ["@vue/cli-plugin-babel/preset"],
  plugins: [
    // 空值合并操作符号 ？？
    "@babel/plugin-proposal-nullish-coalescing-operator",
    // 可选链 ？.
    "@babel/plugin-proposal-optional-chaining",
    //vant 按需引入
    [
      "import",
      {
        libraryName: "vant",
        libraryDirectory: "es",
        style: true,
      },
      "vant",
    ],
  ],
};
```

### transpileDependencies

默认情况下 babel-loader 会忽略所有 node*modules 中的文件。你可以启用本选项，以避免构建后的代码中出现未转译的第三方依赖。  
不过，对所有的依赖都进行转译可能会降低构建速度。如果对构建性能有所顾虑，你可以只转译部分特定的依赖：给本选项传一个数组，列出需要转译的第三方包包名或正则表达式即可。  
上面这种方式会影响项目构建速度和部署包大小，不能把 module 中所有包都放进去  
明确哪个包有转义问题，可以用这种方式 babel 重新编译  
目前已知需要这样的做的库，有一个加解密库\*\*\_CryptoJS \_\_当前项目并没有使用*\*\*

```javascript
transpileDependencies: ["crypto-js"];
```

### vConsole.js 的兼容处理

vConsole 是一个轻量、可拓展、针对手机网页的前端开发者调试面板。  
[vConsole](https://github.com/Tencent/vConsole)  
当我们直接 inpmort 引入时，在低版本手机上，在安卓 5.0 手机上直接白屏，这个函数库使用了一些高版本语法，没有进行转义  
因为 vconsole 存在引用第三包的情况，无法确定范围，因此没有采用 transpileDependencies 方案  
推荐引入链接方式引入，不推荐直接 import  
可以直接在入口，根据环境变量，测试环境自动加载一个转义后的 js 文件

```javascript
<script>
    var WEB_ENV = '<%= VUE_APP_ONLINE_ENV %>'
    // 根据环境变量引入VConsole(判断非生产环境)
    if (!JSON.parse(WEB_ENV)) {
        var scriptEl = document.createElement('script')
        scriptEl.src = '<%= BASE_URL %>vConsole.js'
        scriptEl.async = true
        document.head.appendChild(scriptEl)
        scriptEl.onload = function () {
            window.vConsole = new VConsole()
        }
    }
</script>
```

## 其他

### 入口加载动画

没有挂在 dom 时，先加载 loading 动画，也可以加载 svg，JSON 格式动画，或者骨架屏，持续优化用户体验

```javascript
<div id="app">
  <img class="no-app-img-loading" src="data:image/gif;base64,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" alt="">
</div>
```

# 总结

脚手架这个东西，本质是工具，工具应该是拿来即用，应该助力开发，不应该成为学习的负担。

目前前端打包工具 webpack，vite 等等，框架迭代升级快速，vite 我暂时没有研究，但是 webpack 从 3 到 5，三个版本，能明显感受的是，配置在简单化，学习成本在降低，很多业内大量使用的通用配置，已经成为官方的默认配置。可能官方也意识到配置复杂性的这个问题，在慢慢做改进。

但是 webpack 生态没有跟上，vue-cli 也是基于 webpack5 之上封装一层，我在做这个 vue-template 项目时去查阅 webpack 的文档，资料不是很多，最佳实践不多，也能理解，这种优秀的实践一般都是公司内部使用，不会往外分享传播。

**vue3 +vue-cli4 + webpack5 + 多入口打包 + 自动生成项目模版 + pinia + 数据持久化 + 路由动画 + axios 二次封装 + less sass 变量函数处理 + viewport 适配方案等等**

这个二次封装的框架，融合了我工作以来的经验和实践，平时学习到的优秀的类库，以及和大家交流中发现的问题和解决方案，还是遗留了很多我暂时找不到解决方案的问题，后续看精力和兴趣，如果愿意的话，都能一一攻克。

目前来说，是能够支持移动端的大多数业务，也希望能够帮助到大家业务开发！

# 后期规划

- 路由配置 history，目前使用 history 无法访问二级页面，等待后期研究 ngnix
- 统一的格式控制管理，能够适用于 webstrom 和 vscode
- 整个项目 cli 化，可以像 vue-cli 那样，直接一行命令下载下来
- 增量编译，随着项目变大，每次发布把所有项目都打包一遍是不现实的，能否有一个方案只编译有修改部分，这样编译效率大大提高
- common 基础方法库 打包封装 rollup
- 使用 vite 搭建组件库 componet vant

##

##

<br>
  
> 语雀地址 https://www.yuque.com/yuqueyonghuyv23kd/xebk9e/hrokgg