## Redux源码解析 

> redux版本3.7.2

redux中index.js 暴露的5个api，大家应该都用过
```
export {
  createStore,
  combineReducers,
  bindActionCreators,
  applyMiddleware,
  compose,
}
```

这里先提三个问题：
1. 5个api的关系是什么?
2. applyMiddleware这里面做了什么?
3. 多个中间件是如何一起发挥作用的?

主要围绕这三个问题进行探究，先看createStore方法，一般来说create开头的方法总是用来初始化的。


```
import isPlainObject from 'lodash/isPlainObject'
import $$observable from 'symbol-observable'

export const ActionTypes = {
  INIT: '@@redux/INIT'
}
/**
 * createStore接受三个参数
 * 1. reducer处理action的函数
 * 2. reducer的初始状态
 * 3. enhancer增强器，是个函数
 */
export default function createStore(reducer, preloadedState, enhancer) {
  // 一系列的参数合法性判断，这里允许用户第二参数传入enhancer
  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState
    preloadedState = undefined
  }
  // enhancer合法性判断
  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.')
    }
    // 将createStore函数作为参数传递执行enhancer，再执行返回enhancer(createStore)的函数，将reducer, preloadedState作为参数
    // enhancer通过applyMiddleware生成，看不懂没关系，后面会重点说。
    return enhancer(createStore)(reducer, preloadedState)
  }

  if (typeof reducer !== 'function') {
    throw new Error('Expected the reducer to be a function.')
  }
  // 记录传入的参数
  let currentReducer = reducer
  let currentState = preloadedState
  // 初始化内部参数
  // 双缓冲读写分离
  // currentListeners负责读取数据
  // nextListeners负责写入数据
  let currentListeners = []
  let nextListeners = currentListeners
  // 标志位，dispatch状态
  let isDispatching = false

  function ensureCanMutateNextListeners() {
    if (nextListeners === currentListeners) {
      nextListeners = currentListeners.slice()
    }
  }

  function getState() {
    return currentState
  }

  function subscribe(listener) {
    if (typeof listener !== 'function') {
      throw new Error('Expected listener to be a function.')
    }

    let isSubscribed = true

    ensureCanMutateNextListeners()
    nextListeners.push(listener)

    return function unsubscribe() {
      if (!isSubscribed) {
        return
      }

      isSubscribed = false

      ensureCanMutateNextListeners()
      const index = nextListeners.indexOf(listener)
      nextListeners.splice(index, 1)
    }
  }

  function dispatch(action) {
    if (!isPlainObject(action)) {
      throw new Error(
        'Actions must be plain objects. ' +
        'Use custom middleware for async actions.'
      )
    }

    if (typeof action.type === 'undefined') {
      throw new Error(
        'Actions may not have an undefined "type" property. ' +
        'Have you misspelled a constant?'
      )
    }

    if (isDispatching) {
      throw new Error('Reducers may not dispatch actions.')
    }

    try {
      isDispatching = true
      currentState = currentReducer(currentState, action)
    } finally {
      isDispatching = false
    }

    const listeners = currentListeners = nextListeners
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }

    return action
  }

  function replaceReducer(nextReducer) {
    if (typeof nextReducer !== 'function') {
      throw new Error('Expected the nextReducer to be a function.')
    }

    currentReducer = nextReducer
    dispatch({ type: ActionTypes.INIT })
  }

  function observable() {
    const outerSubscribe = subscribe
    return {
      subscribe(observer) {
        if (typeof observer !== 'object') {
          throw new TypeError('Expected the observer to be an object.')
        }

        function observeState() {
          if (observer.next) {
            observer.next(getState())
          }
        }

        observeState()
        const unsubscribe = outerSubscribe(observeState)
        return { unsubscribe }
      },

      [$$observable]() {
        return this
      }
    }
  }

  dispatch({ type: ActionTypes.INIT })

  return {
    dispatch,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
  }
}

```
isPlainObject这个方法是用来判断参数是否是普通对象。$$observable不在本次的讨论范围。
