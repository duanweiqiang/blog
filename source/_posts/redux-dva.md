---
title: react redux-dva简析
date: 2020-06-20 11:59:22
img: /medias/featureimages/2.jpg
top: false
cover: true
coverImg: /medias/featureimages/8.jpg
toc: true
mathjax: false
summary: 本文介绍关于react的state储存机制，以及state的隔离问题，使我们的state机制更加有效方便，降低组件之间状态的耦合度。
categories: 前端
tags:
- React
- 储存
- 数据隔离
---

##### 简介 
  React 是一个数据单项的JavaScript库，可以实现重复调用，全局修改全局生效，在这样的情况下我们就会涉及到数据的模块封装和数据隔离，由于react使用的是state的机制故而我们就想让react的state机制像闭包的数据隔离一样对不同组件的state进行单独封装，单独隔离，但是呢又可以全局引用，因此就出现了redux到dva等一系列的react的状态管理框架。

#### 最初史的state介绍
  react用的比较多的朋友应该知道，redux的思想是视图与状态是一一对应的；但是呢redux的所有的状态，都保存在一个对象state里面；

  store：是一个数据池，redux会把所有的数据都放在里面，没有做数据隔离，而一个 State 对应一个 View。只要 State 相同，View 就相同。

  redux-saga和dva非常相似，可以把触发条件理解为action，具体业务处理是是reducer，那么我们触发事件只能通过action去操作整个store的状态，对业务来说，我们可以不需要知道reducer的具体实现，更新reducer对业务来说也是无感的。这样就把数据的处理和更新与业务分离开来，相互没有直接的依赖关系。


##### redux-saga:
 - 1.Redux 内里只需一个 Store,全局的数据都在这个大 Store 内里。Store 的 State 不能直接修正，每次只能返回一个新的 State。Redux 整了一个createStore函数来添加 Store。
 - 2.同一个state里不能有相同的key，即同一个state里不能有两个List，虽然两个list是在不同的页面里使用。

demo
```javascript
  import { createStore } from 'redux';
  const store = createStore(fn);
```

action的代码如下：
```javascript
  import { takeEvery } from 'redux-saga';
  
  function* demo() {
    yield takeEvery('actionName', function* (result) {
      console.log(result);
    });
  }
```
如果有多个saga函数来监听不同的action事件的情况：

``` javascript
  import { takeEvery } from 'redux-saga/effects'

  // action 1
  function* demo1(action) { ... }

  // action 2
  function* demo2(action) { ... }

  // 同时抛出它们
  export default function* rootSaga() {
    yield takeEvery('actionName1', demo1)
    yield takeEvery('actionName2', demo2)
  }
  //yield 右边的任何表达式都会被求值，结果会被 yield 给调用者。
```
  - 我们也可以在action函数中写fetch-request对接口进行调用，同时可以吐出到组件，yield为异步api时使用saga提供call()方法，我们可以在触发业务代码的地方使用then函数来接受api的返回数据也可以在action函数这里进行state处理。
  这里要说明的是，一般我们都会在action函数这里做一层数据校验，对接口返回数据进行一个初步的检测。

##### 1.接口校验
```javascript
import { call } from 'redux-saga/effects';
import Api from './path/to/api'

function* fetchApi() {
  const result = yield call(Api.fetch, '/products')
  if(result.code===200||...){
    alert('success');
    return result.data
  }else{
    alert('error');
    return null
  }
}
```

##### 2.使用try...catch来处理错误，并统一提示处理

```javascript
//错误统一处理

import Api from './path/to/api';
import { call, put } from 'redux-saga/effects';

// ...

function* fetchApi() {
  try {
    const result = yield call(Api.fetch, '/products')
    yield put({ type: 'resultSuccess', products })
  }
  catch(error) {
    yield put({ type: 'resultError', error })
  }
}
```

##### 3.使用throw来抛出一个错误
```javascript
import { call, put } from 'redux-saga/effects'
import Api from '...'

const iterator = fetchProducts()

// 期望一个 call 指令
assert.deepEqual(
  iterator.next().value,
  call(Api.fetch, '/products'),
  "fetchProducts should yield an Effect call(Api.fetch, './products')"
)

// 创建一个模拟的 error 对象
const error = {}

// 期望一个 dispatch 指令
assert.deepEqual(
  iterator.throw(error).value,
  put({ type: 'resultError', error }),
  "fetchProducts should yield an Effect put({ type: 'PRODUCTS_REQUEST_FAILED', error })"
)
```
##### redux-dva：
学过React的都知道他的技术栈，各种库插件多的如天，所以每当你使用React的时候都需要引入很多的模块，配置好多数据，那么dva就是把这些用到的模块集成在一起，形成一定的架构规范。

1.dva 是 framework，不是 library库；
2.dva封装了redux，减少很多重复代码比如action reducers 等的重复编写。
3.dva的核心是module模块，通过module我们可以实现结构上的多个store来储存数据，saga是所有的都写在一个里面，这样就可以对不同的业务组件或者模块来分别管理和引用。
4.call, put其实是saga的写法，dva集成了集成了redux、redux-saga、react-router-redux、react-router。将initState、saga、reducer集成到一个model里面统一管理，对某个模块进行单独的维护，减少成本量。

##### 引入图文说明
<!-- ![dva.png](https://i.loli.net/2020/06/29/OkzifJLM7wGR1TZ.png) -->
![dva.png](https://i.loli.net/2020/06/29/9Zkes2tvRAHMxLf.png)

缺点：
1.如现在react-router已经到了4.x了，但是dva内置的版本却还是2.x，如果react-router升级就要在使用 dva 和使用新版本的组件上做出选择了。
2.可扩展性不强，后期升级也是个问题。

##### 引入连接器：
```javascript
import { connect } from 'dva';
```

```javascript
import queryString from 'query-string';
import * as todoService from '../services/todo'
 
export default {
  namespace: 'storeA', //model的命名空间key，全局唯一，用来识别调用或引入的store
   state: {  //状态的初始值
     list: [],
     message:"ffff"
   },
 
  //类似于redux的 reducer 是一个纯函数用来处理同步函数。
  reducers: {   
    setList(state, { payload: { list } }) {
      return { ...state, list }
    },
 
    setState(state, { payload: { message } }) {     
      return { ...state, message }
    },
  },
  //处理请求等异步函数
  effects: {
    *addForm({ payload: value }, { call, put, select }) {
      const data = yield call(todoService.query, value)
      let tempList = yield select(state => state.todo.list);
      yield put({ type: 'setList', payload: { list }}) 
    },
 
    *test({ payload: message }, { call, put, select }) {
      yield put({ type: 'setState', payload: { message } })
      yield put({ type: 'namespaceB/setState', payload: { message } })//可控制其他model中的state
    }
  },
 
  //用于订阅某些数据 如：监听路由的变化等
  subscriptions: {
    setup({ dispatch, history }) {
      // 监听路由的变化，请求页面数据
      return history.listen(({ pathname, search }) => {
        const query = queryString.parse(search);
        let list = []
        if (pathname === 'addForm') {//路由名称
          dispatch({ type: 'setList', payload: {list} }) //更新action
        }
      })
    }
  }
 ```
  - namespace 代表命名空间，全局唯一
  - state 代表初始化数据是一个对象，通过props调用
  - reducer 就跟我们平时用的reducer一样，action事件触发的函数
  - effects 处理Api等异步问题
  - subscriptions 订阅，常用来监听

##### 跨model的通信：
```javascript
  new Promise((resolve, reject) => {
    dispatch({ type: 'storeA/addLog', payload: { ... } });
  })
  .then((data) => {
    console.log(data);
    // ...
  });
```
dispatch中的type/前面的是model的key（namespace）后面的才是action。
也可以在effects中通过key/action的方式来通知其他model更新state状态，达到跨model通行。

```javascript
  *test({ payload: message }, { call, put, select }) {
    yield put({ type: 'setState', payload: { message } })
    yield put({ type: 'namespaceB/setState', payload: { message } })//可控制其他model中的state
  }
```

##### 多任务的情况：
```javascript
  const [result1, result2]  = yield all([
    call(service1, param1),
    call(service2, param2)
  ])
```

##### model数据共享（共享state数据）

在B的model中的effects中获取A的state，其中a为A的namespace，response 为A的state，如
```javascript
effects: {
  *fetchResult({ callback }, { call, put, select }) {
    const response = yield select(_ => _.a);
    // response为A model中的state值
    // ···其他操作
    // yield put({ type: "saveEvents", payload:{response} });
    // if (callback) callback(response);
  }
}
```
也可以在B页面中connect A model取值。

#### 附：初始化:

```javascript
  // 1. Initialize
  const app = dva({
  history, // 指定给路由用的 history，默认是 hashHistory
  initialState,  // 指定初始数据，优先级高于 model 中的 state
  onError, // effect 执行错误或 subscription 通过 done 主动抛错时触发，可用于管理全局出错状态。
  onAction, // 在 action 被 dispatch 时触发
  onStateChange, // state 改变时触发，可用于同步 state 到 localStorage，服务器端等
  onReducer, // 封装 reducer 执行。比如借助 redux-undo 实现 redo/undo
  onEffect, // 封装 effect
  onHmr, // 热替换相关
  extraReducers, // 指定额外的 reducer，比如 redux-form 需要指定额外的 form reducer
  extraEnhancers, // 指定额外的 StoreEnhancer ，比如结合 redux-persist 的使用
});

  // 2. Plugins
  // app.use({});//也可以可配置hooks的相关

  // 3. Model
  // app.model(require('./models/example').default);

  // 4. Router
  app.router(require('./router').default);

  // 5. Start
  app.start('#root');
```
挂载组件时，dva中的state通过connect将model、状态数据与组件相连。通过dispatch 调用key/action来触发model。这样就形成一个完整的数据流向。

#### 总结
dva 帮你自动化了Redux 架构一些繁琐的设置，比如上面所说的redux store 的创建，中间件的配置，路由的初始化等都自动生成好了。

dva 降低了组件之间数据之间的的耦合度，可以对单个模块进行封装单独的model通过connect来连接，模块数据单独设置清晰易维护。

dva编写方便，基本不需要过多的配置，提高团队多人开发的效率。

dva配合umi达到了开箱即用的状态
