## Store（仓库）

每一个 Vuex 应用的核心就是 store（仓库）。“store”基本上就是一个容器，它包含着你的应用中大部分的状态 (state)。Vuex 和单纯的全局对象有以下两点不同：
+ Vuex 的状态存储是响应式的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。
+ 你不能直接改变 store 中的状态。改变 store 中的状态的唯一途径就是显式地提交 (commit) mutation。这样使得我们可以方便地跟踪每一个状态的变化，从而让我们能够实现一些工具帮助我们更好地了解我们的应用

### 创建一个 store

通过提交 mutation 的方式，而非直接改变 store.state.count，是因为我们想要更明确地追踪到状态的变化。

```js
const store = new Vuex.Store({
    state: {
        count: 0
    },
    mutations: {
        increment(state) {
            state.count ++
        }
    }
})
```

## State（状态）

Vuex 使用的是单一状态树，用一个对象包含了全部的应用级别的状态。

### 在 Vue 组件中获取 Vuex 状态

1. 由于 Vuex 的状态存储是响应式的，从 store 实例中读取状态最简单的方式就是使用**计算属性**返回某个状态。

```js
const app1 = new Vue({
    el: '#app1',
    computed: {
        count() {
            return store.state.count;
        }
    },
    methods: {
        add() {
            store.commit('increment')
        }
    }
})
```

2. 在跟组件中注册 store 选项，根组件的所有子组件能通过 this.$store 访问到。

```js
const Counter = {
    template: `<div>Counter: {{ count }}</div>`,
    computed: {
        count() {
            return this.$store.state.count;
        }
    }
}

const app3 = new Vue({
    el: '#app3',
    store,
    components: {Counter},
    template: `
        <div>
            <counter></counter>
        </div>
    `
})
```

### mapState 辅助函数

当一个组件需要获取多个状态时候，将这些状态都声明为计算属性会有些重复和冗余。为了解决这个问题，我们可以使用 mapState 辅助函数帮助我们生成计算属性。

```js
const app4 = new Vue({
    el: '#app4',
    store,
    data: {
        localCount: 10
    },
    computed: Vuex.mapState({
        // 箭头函数可使代码更简练
        count: state => state.count,
        // 传字符串参数 'count' 等同于 `state => state.count`
        countAlias: 'count',
        // 为了能够使用 `this` 获取局部状态，必须使用常规函数
        countPlusLocalState(state) {
            return state.count + this.localCount;
        }
    })
})
```

当映射的计算属性的名称与 state 的子节点名称相同时，我们也可以给 mapState 传一个字符串数组。

```js
const app5 = new Vue({
    el: '#app5',
    store,
    computed: Vuex.mapState([
        'count'
    ])
})
```

### 对象展开运算符

mapState 函数返回的是一个对象。使用**对象展开运算符**可以将 mapState 返回的对象展开，从而可以于局部计算属性混用。

```js
const app6 = new Vue({
    el: '#app6',
    store,
    computed: {
        first() {
            return this.$store.state.count + 100;
        },
        ...Vuex.mapState({
            second: 'count'
        })
    }
})
```

## Getter

Vuex 允许在 store 中定义 getters 。getters 可以从 store 中的 state 中派生出一些状态。<br>
像计算属性一样，getter 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。<br>
Getter 接受的第一个参数是 state。<br>
Getter 可以像 store 一样通过属性访问，可以通过让 getter 返回一个函数然后通过调用方法进行访问，也可以使用 mapGetters 辅助函数。

```js
const store = new Vuex.Store({
    state: {
        count: 0,
        todos: [
            {id: 1, text: 'hello', done: true},
            {id: 2, text: 'world', done: false}
        ]
    },
    getters: {
        doneTodos: state => {
            return state.todos.filter(todo => !todo.done)
        },
        getTodoById: (state) => (id) => {
            return state.todos.find(todo => todo.id === id)
        }
    },
    mutations: {
        increment(state) {
            state.count ++
        }
    }
})
    
const app7 = new Vue({
    el: '#app7',
    store,
    computed: {
        todos() {
            return this.$store.getters.doneTodos;
        },
        todoById() {
            return this.$store.getters.getTodoById(1)
        }
    }
})
```

## Mutation

更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。<br>
Vuex 中的 mutation 类似于事件，每个 mutation 都有一个字符串的**事件类型**（type）和一个**回调函数**(handler)。回调函数就是实际进行状态更改的地方，它接收 state 作为第一参数。<br>
可以向 store.commit 传入额外的参数，即 mutation 的载荷（payload）。在大多数情况下，载荷应该是一个对象。<br>
Mutation 必须是同步函数。

```js
const store = new Vuex.Store({
    state: {
        count: 0,
        user: {
            id: '1',
            name: 'w'
            
        }
    },
    mutations: {
        increment(state) {
            state.count ++
        },
        add(state, n) {
            state.count += n
        },
        incrementPayload(state, payload) {
            state.count += payload.amount
        },
        addAge(state) {
            // 当需要在对象上添加新属性时,应该使用 Vue.set(obj, 'newProp', 123), 或者以新对象替换老对象
            state.obj = { ...state.obj, age: 20 }
        }
        
    }
})

store.commit('increment')
store.commit('add', 10)

// 提交 mutation 的另一种方式是直接使用包含 type 属性的对象：
store.commit({
    type: 'incrementPayload',
    amount: 10
})
```

## Action

Action 类似于 mutation，不同在于：
+ Action 提交的是 mutation，而不是直接变更状态。
+ Action 可以包含任意异步操作。

Action 通过 store.dispatch 方法触发。

```js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    },
    add(state, payload) {
        state.count += payload.amount
    }
  },
  actions: {
    increment (context) {
      context.commit('increment')
    },
    incrementAlias ({ commit }) {
        commit('increment')
    },
    incrementAsync ({ commit }) {
        setTimeout(() => {
            commit('increment')
        }, 1000)
    },
    add ({ commit, payload }) {
        commit('add', payload)
    }
  }
})

store.dispatch('increment')
store.dispatch('add', {amount: 10})
store.dispatch({
    type: 'add',
    amount: 10
})
```

## Module

Vuex 允许我们将 store 分割成模块（module）。每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块。<br>
对于模块内部的 mutation 和 getter，接收的第一个参数是模块的局部状态对象。<br>
对于模块内部的 action，局部状态通过 context.state 暴露出来，根节点状态则为 context.rootState。

```js
const moduleA = {
  state: { ... },
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: { ... },
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a // -> moduleA 的状态
store.state.b // -> moduleB 的状态
```
