## 使用 Web3 和 Vue.js 来创建你的第一个以太坊 dAPP（第二部分）

欢迎回到这个很棒的系列教程的第二部分，在个教程中我们要亲身实践，创建我们的第一个去中心化应用（decentralized application）。在第二部分中，我们将介绍 VueJS 和 VueX 的核心概念以及 web3js 与 metamask 的交互。


进入正题：VueJS

VueJS 是一个用于构建用户界面的 JavaScript 框架。初看起来，它类似传统的 mustache（译者注：原文为 moustache）模板，但其实 Vue 在后面做了很多工作。

    <div id=”app”>
    {{ message }}
    </div>
    
    var app = new Vue({
    el: '#app',
    data: {
    message: 'Hello Vue!'
    }
    })

这是一个很基本的 Vue 应用的结构。数据对象中的 message 属性会被渲染到屏幕上 id 为「app」的元素中，当我们改变 message 时，屏幕上的值也会实时更新。你可以去这个 jsfiddle 上查看（开启自动运行）：jsfiddle.net/tn1mfxwr/2/

VueJS 的另一个重要特征是组件。组件是小的、可复用的并且可嵌套的小段代码。本质上，一个 Web 应用是由较小组件组成的组件树构成的。当我们着手编写我们前端应用时，我们会愈加清楚。

![](https://i.imgur.com/bGJEgn6.png)


这个页面示例是由组件构成的。页面由三个组件组成的，其中的两个有子组件。

### 状态的整合: Vuex
我们使用 Vuex 管理应用的状态。类似于 Redux，Vuex 实现了一个对于我们应用数据「单一数据源」的容器。Vuex 允许我们使用一种可预见的方法操作和提供应用程序使用的数据。

它工作的方式是非常直观的。当组件需要数据进行渲染时，它会触发（dispatch）一个 action 获取所需的数据。Action 中获取数据的 API 调用是异步的。一旦取得数据，action 会将数据提交（commit）给一个变化（mutation）。然后，Mutation 会使得我们容器（store）的状态发生改变（alert the state）。当组件使用的容器中的数据改变时，它会重新进行渲染。

![](https://i.imgur.com/doJqM9z.png)


Vuex 的状态管理模式。

在我们继续之前...

在第一部分中，我们已经通过 vue-cli 生成了一个 Vue 应用，我们也安装了所需的依赖。如果你没有这样做的话，请查看上面第一部分的链接。

如果你正确完成了各项的话，你的目录看起来应该是这样的：


![](https://i.imgur.com/wicjT55.png)

新生成的 vue 应用。

小提示：如果你要从这里复制粘贴代码段的话，请在你的 .eslintignore 文件中添加 /src/，以免出现缩进错误。

你可以在终端中输入 npm start 运行这个应用。首先我们需要清理它包含的这个默认的 Vue 应用。

注解：尽管只有一个路由，但是我们还是会使用 Vue Router，虽然我们并不需要，但是因为这个教程相当简单，我想将其保留会更好。

贴士：在你的 Atom 编辑器右下角中将 .vue 文件设置为 HTML 语法（高亮）

现在处理这个刚生成的应用：

- 在 app.vue 中删除 img 标签和 style 标签中的内容。

- 删除 components/HelloWorld.vue，创建两个名为 casino-dapp.vue（我们的主组件）和 hello-metamask.vue（将包含我们的 Metamask 数据）的两个新文件。

- 在我们的新 hello-metamask.vue 文件中粘贴下面的代码，它现在只显示了在一个 p 标签内的「hello」文本。


    <template>
    <p>Hello</p>
    </template>

    <script>
    export default {
    name: 'hello-metamask'
    }
    </script>

    <style scoped>

    </style>

现在我们首先导入 hello-metamask 组件文件，通过导入文件将其加载到主组件 casino-app 中，然后在我们的 vue 实例中，引用它作为模板中一个标签。在 casino-dapp.vue 中粘贴这些代码：

    <template>
    <hello-metamask/>
    </template>

    <script>
    import HelloMetamask from '@/components/hello-metamask'
    export default {
    name: 'casino-dapp',
    components: {
    'hello-metamask': HelloMetamask
    }
    }
    </script>

    <style scoped>

    </style>

现在如果你打开 router/index.js 你会看到 root 下只有一个路由，它现在仍指向我们已删除的 HelloWorld.vue 组件。我们需要将其指向我们主组件 casino-app.vue。

    import Vue from 'vue'
    import Router from 'vue-router'
    import CasinoDapp from '@/components/casino-dapp'

    Vue.use(Router)

    export default new Router({
    routes: [
    {
    path: '/',
    name: 'casino-dapp',
    component: CasinoDapp
    }
    ]
    })


关于 Vue Router：你可以增加额外的路径并为其绑定组件，当你访问定义的路径时，在 App.vue 文件中的 router-view 标签中，对应的组件会被渲染，并进行显示。

- 在 src 中创建一个名为 util 的新文件夹，在这个文件夹中创建另一个名为 constants 的新文件夹，并创建一个名为 networks.js 的文件，粘贴下面的代码。我们用 ID 来代替以太坊（Ethereum）网络名称显示，这样做会保持我们代码的整洁。

    export const NETWORKS = {
    '1': 'Main Net',
    '2': 'Deprecated Morden test network',
    '3': 'Ropsten test network',
    '4': 'Rinkeby test network',
    '42': 'Kovan test network',
    '4447': 'Truffle Develop Network',
    '5777': 'Ganache Blockchain'
     }


- 最后的但同样重要的（实际上现在用不到）是，在 src 中创建一个名为 store 的新文件夹。我们将在下一节继续讨论。

如果你在终端中执行 npm start，并在浏览器中访问 localhost:8000，你应该可以看到「Hello」出现在屏幕上。如果是这样的话，就表示你准备好进入下一步了。

### 设置我们的 Vuex 容器

在这一节中，我们要设置我们的容器（store）。首先从在 store 目录（上一节的最后一部分）下创建两个文件开始：index.js 和 state.js；我们先从 state.js 开始，它是我们所检索的数据一个空白表示（Blank representation）。

    let state = {
    web3: {
    isInjected: false,
    web3Instance: null,
    networkId: null,
    coinbase: null,
    balance: null,
    error: null
    },
    contractInstance: null
    }
    export default state


好了，现在我们要对 index.js 进行设置。我们会导入 Vuex 库并且告诉 VueJS 使用它。我们也会把 state 导入到我们的 store 文件中。

    import Vue from 'vue'
    import Vuex from 'vuex'
    import state from './state'

    Vue.use(Vuex)

    export const store = new Vuex.Store({
    strict: true,
    state,
    mutations: {},
    actions: {}
    })


最后一步是编辑 main.js ，以包含我们的 store 文件：

    import Vue from 'vue'
    import App from './App'
    import router from './router'
    import { store } from './store/'

    Vue.config.productionTip = false

    /* eslint-disable no-new */
    new Vue({
    el: '#app',
    router,
    store,
    components: { App },
    template: '<App/>'
    })

现在已经准备好通过 web3 API 获取我们 Metamask 的数据，并使其在我们的应用发挥作用了

### 入门 Web3 和 Metamask
就像前面提到的，为了让 Vue 应用能获取到数据，我们需要触发（dispatch）一个 action 执行异步的 API 调用。我们会使用 promise 将几个方法链式调用，并将这些代码提取（封装）到文件 util/getWeb3.js 中。粘贴以下的代码，其中包含了一些有助你遵循的注释。我们会在代码块下面对它进行解析：

    import Web3 from 'web3'

    /*
    * 1. Check for injected web3 (mist/metamask)
    * 2. If metamask/mist create a new web3 instance and pass on result
    * 3. Get networkId - Now we can check the user is connected to the right network to use our dApp
    * 4. Get user account from metamask
    * 5. Get user balance
    */

    let getWeb3 = new Promise(function (resolve, reject) {
    // Check for injected web3 (mist/metamask)
    var web3js = window.web3
    if (typeof web3js !== 'undefined') {
    var web3 = new Web3(web3js.currentProvider)
    resolve({
      injectedWeb3: web3.isConnected(),
      web3 () {
        return web3
      }
    })
    } else {
    // web3 = new Web3(new Web3.providers.HttpProvider('http://localhost:7545')) GANACHE FALLBACK
    reject(new Error('Unable to connect to Metamask'))
     }
    })
    .then(result => {
    return new Promise(function (resolve, reject) {
      // Retrieve network ID
      result.web3().version.getNetwork((err, networkId) => {
        if (err) {
          // If we can't find a networkId keep result the same and reject the promise
          reject(new Error('Unable to retrieve network ID'))
        } else {
          // Assign the networkId property to our result and resolve promise
          result = Object.assign({}, result, {networkId})
          resolve(result)
        }
      })
    })
     })
    .then(result => {
    return new Promise(function (resolve, reject) {
      // Retrieve coinbase
      result.web3().eth.getCoinbase((err, coinbase) => {
        if (err) {
          reject(new Error('Unable to retrieve coinbase'))
        } else {
          result = Object.assign({}, result, { coinbase })
          resolve(result)
        }
      })
    })
    })
    .then(result => {
    return new Promise(function (resolve, reject) {
      // Retrieve balance for coinbase
      result.web3().eth.getBalance(result.coinbase, (err, balance) => {
        if (err) {
          reject(new Error('Unable to retrieve balance for address: ' + result.coinbase))
        } else {
          result = Object.assign({}, result, { balance })
          resolve(result)
        }
      })
    })
    })

    export default getWeb3


第一步要注意的是我们使用 promise 链接了我们的回调方法，如果你不太熟悉 promise 的话，请参考此链接。下面我们要检查用户是否有 Metamask（或 Mist）运行。Metamask 注入 web3 本身的实例，所以我们要检查 window.web3（注入的实例）是否有定义。如果是否的话，我们会用 Metamask 作为当前提供者（currentProvider）创建一个 web3 的实例，这样一来，实例就不依赖于注入对象的版本。我们把新创建的实例传递给接下来的 promise，在那里我们做几个 API 调用：

- web3.version.getNetwork() 将返回我们连接的网络 ID。
- web3.eth.coinbase() 返回我们节点挖矿的地址，当使用 Metamask 时，它应该会是已选择的账户。
- web3.eth.getBalance(<address>) 返回作为参数传入的该地址的余额。

还记得我们说过 Vuex 容器中的 action 需要异步地进行 API 调用吗？我们在这里将其联系起来，然后再从组件中将其触发。在 store/index.js 中，我们会导入 getWeb3.js 文件，调用它，然后将其（结果）commit 给一个 mutation，并让其（状态）保留在容器中。
在你的 import 声明中增加：


    import getWeb3 from '../util/getWeb3'

然后在（store 内部）的 action 对象中调用 getWeb3 并 commit 其结果。我们会添加一些 console.log 在我们的逻辑中，这样做是希望让 dispatch-action-commit-mutation-statechange 流程更加清楚，有助于我们理解整个执行的步骤。

    registerWeb3 ({commit}) {
      console.log('registerWeb3 Action being executed')
      getWeb3.then(result => {
        console.log('committing result to registerWeb3Instance mutation')
        commit('registerWeb3Instance', result)
      }).catch(e => {
        console.log('error in action registerWeb3', e)
      })
    }

现在我们要创建我们的 mutation，它会将数据存储为容器中的状态。通过访问第二个参数，我们可以访问我们 commit 到 mutation 中的数据。在 mutations 对象中增加下面的方法：

    registerWeb3Instance (state, payload) {
    console.log('registerWeb3instance Mutation being executed', payload)
    let result = payload
    let web3Copy = state.web3
    web3Copy.coinbase = result.coinbase
    web3Copy.networkId = result.networkId
    web3Copy.balance = parseInt(result.balance, 10)
    web3Copy.isInjected = result.injectedWeb3
    web3Copy.web3Instance = result.web3
    state.web3 = web3Copy
    }

现在剩下要做的是在我们的组件中触发（dispatch）一个 action，取得数据并在我们的应用中进行呈现。为了触发（dispatch）action，我们将会用到 Vue 的生命周期钩子。在我们的例子中，我们要在它创建之前触发（dispatch）action。在 components/casino-dapp.vue 中的 name 属性下增加以下方法：

    export default {
    name: 'casino-dapp',
    beforeCreate () {
    console.log('registerWeb3 Action dispatched from casino-dapp.vue')
    this.$store.dispatch('registerWeb3')
    },
    components: {
    'hello-metamask': HelloMetamask
    }
    }

现在我们要渲染 hello-metamask 组件的数据，我们账户的所有数据都将在此组件中进行呈现。从容器（store）中获得数据，我们需要在计算属性中增加一个 getter 方法。然后，我们就可以在模板中使用大括号来引用数据了。

    <template>
    <div class='metamask-info'>
    <p>Metamask: {{ web3.isInjected }}</p>
    <p>Network: {{ web3.networkId }}</p>
    <p>Account: {{ web3.coinbase }}</p>
    <p>Balance: {{ web3.balance }}</p>
    </div>
    </template>

    <script>
    export default {
    name: 'hello-metamask',
    computed: {
    web3 () {
     return this.$store.state.web3
     }
     }
    }
    </script>

    <style scoped></style>

现在一切都应该完成了。在你的终端（terminal）中通过 npm start 启动这个项目，并访问 localhost:8080。现在，我们可以看到 Metamask 的数据。当我们打开控制台，应该可以看到 console.log 输出的 —— 在 Vuex 那段中的描述状态管理模式信息。


以防出现错误，可以[ Github 仓库 ](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fkyriediculous%2Fdapp-tutorial%2Ftree%2Fhello-metamask)的 hello-metamask 分支上查看有此部分完整的代码