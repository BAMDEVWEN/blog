---
title: 'Vuex'
description: 'Vuex的基本使用'
pubDate: 'DEC 29 2024'
tags: ['Vuex']

---

# Vuex

## Store Example

```js
import { createApp } from 'vue'
import { createStore } from 'vuex'

// 创建一个新的 store 实例
const store = createStore({
  state () {
    return {
      count: 0
    }
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})

const app = createApp({ /* 根组件 */ })

// 将 store 实例作为插件安装
app.use(store)
```

## Store

### State

- Vuex使用的是单一状态树

```js
//在Vue组件获取Vuex状态
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return store.state.count
    }
  }
}
```



### Getter

- Getter 接受state作为第一个参数

- 通过属性访问

```js
onst store = createStore({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    doneTodos (state) {
      return state.todos.filter(todo => todo.done)
    }
  }
})
//访问
store.getters.doneTodos
//接受其他getter作为第二个参数
getters: {
  // ...
  doneTodosCount (state, getters) {
    return getters.doneTodos.length
  }
}
```





### Mutation

**每个Mutation都有一个字符串的时间类型和一个回调函数,会接受state作为第一个参数**

```js
const store=createStore({
    state:{
        count:1
    },
    mutations:{
        increment(state){
            state.count++
        }
    }
})
```

**不能直接调用一个mutation函数。需要使用store.commit方法**

```js
store.commit('increment')
```

#### 提交荷载（Payload）

```js
mutations:{
    increment(state,n){
        state.count+=n;
    }
}
```

大多数情况下荷载是一个对象

```js
mutations:{
    increment(state,payload){
        state.count+=payload.amount
    }
}
//使用
store.commit('increment',{
    amount:10
})
```

#### 对象风格的提交

```js
store.commit({
    type:'increment',
    amount:10
})
```

#### 使用常量代替Mutation代替事件类型

```js
//// mutation-types.js
export const SOME_MUTATION = 'SOME_MUTATION'
```



```js
import { createStore } from 'vuex'
import { SOME_MUTATION } from './mutation-types'

const store = createStore({
  state: { ... },
  mutations: {
    // 我们可以使用 ES2015 风格的计算属性命名功能
    // 来使用一个常量作为函数名
    [SOME_MUTATION] (state) {
      // 修改 state
    }
  }
})
```

#### Mountion必须是同步函数

- 一条重要的原则就是要记住 **mutation 必须是同步函数**。

### Action

Action类似于mutation不同于：

- Action 提交的是mutation，不是直接变更状态
- Action可以包含任意异步操作

```js
const store=createStore({
    state:{
        count:0
    },
    mutations:{
        increment(state){
            state.count++
        },
        actions:{
            increment(comtext){
                content.commit('increment')
            }
        }
    }
})
```

#### 分发Action

Action通过`store.dispatch('increment')`

Action在内部可以执行异步函数

```js
actions: {
  incrementAsync ({ commit }) {
    setTimeout(() => {
      commit('increment')
    }, 1000)
  }
}
```

#### 组合Action

Action是异步通信的，`tore.dispatch` 可以处理被触发的 action 的处理函数返回的 Promise，并且 `store.dispatch` 仍旧返回 Promise：

```js
actions: {
  actionA ({ commit }) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        commit('someMutation')
        resolve()
      }, 1000)
    })
  }
}
```

```js
store.dispatch('actionA').then(() => {
  // ...
})
```





### Module

Vuex允许store分割成模块，每个模块有字节的state，mutation、action、getter

```js
const moduleA = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... }
}

const store = createStore({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a // -> moduleA 的状态
store.state.b // -> moduleB 的状态
```



