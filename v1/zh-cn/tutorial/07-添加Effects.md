## 添加Effects

在之前的教程中，我们已经完成了静态数据的操作，但是在真实场景，数据都是从服务器来的，我们需要发起异步请求，在请求回来以后设置数据，更新 state，那么在 dva 中，这一切是怎么操作的呢，首先我们先来简单了解一下 Effects。

### 理解 Effects

Effects 来源于 dva 封装的底层库 [redux-sagas](http://yelouafi.github.io/redux-saga) 的概念，主要指的是处理 `Side Effects` ，指的是副作用（源于函数式编程），在这里可以简单理解成异步操作，所以我们是不是可以理解成 Reducers 处理同步，Effects 处理异步？这么理解也没有问题，但是要认清 Reducers 的本质是修改 model 的 state，而 Effects 主要是 __控制数据流程__ ，所以最终往往我们在 Effects 中会调用 Reducers。

> 在函数式编程中，我们强调纯函数的使用：纯函数是这样一种函数，即相同的输入，永远会得到相同的输出，而且没有任何可观察的副作用；简单理解就是每个函数职责纯粹，所有行为就是返回新的值，没有其他行为，真实情况最常见的副作用就是异步操作，所以 dva 提供了 effects 来专门放置副作用，不过可以发现的是，由于 effects 使用了 Generator Creator，所以将异步操作同步化，也是纯函数。更多相关可以参看 [JS函数式编程指南](https://www.gitbook.com/book/llh911001/mostly-adequate-guide-chinese/details)。

### 给 Users Model 添加 Effects

```jsx
// ./src/models/users.js
import { hashHistory } from 'dva/router';
//import { create, remove, update, query } from '../services/users';

// 处理异步请求
import request from '../utils/request';
import qs from 'qs';
async function query(params) {
  return request(`/api/users?${qs.stringify(params)}`);
}

export default {
  namespace: 'users',

  state: {
    list: [],
    total: null,
    loading: false, // 控制加载状态
    current: null, // 当前分页信息
    currentItem: {}, // 当前操作的用户对象
    modalVisible: false, // 弹出窗的显示状态
    modalType: 'create', // 弹出窗的类型（添加用户，编辑用户）
  },

  subscriptions: {
    setup({ dispatch, history }) {
      history.listen(location => {
        if (location.pathname === '/users') {
          dispatch({
            type: 'query',
            payload: {}
          });
        }
      });
    },
  },

  effects: {
    *query({ payload }, { select, call, put }) {
      yield put({ type: 'showLoading' });
      const { data } = yield call(query);
      if (data) {
        yield put({
          type: 'querySuccess',
          payload: {
            list: data.data,
            total: data.page.total,
            current: data.page.current
          }
        });
      }
    },
    *create(){},
    *'delete'(){},
    *update(){},
  },
  reducers: {
    showLoading(state, action){
      return { ...state, loading: true };
    }, // 控制加载状态的 reducer
    showModal(){}, // 控制 Modal 显示状态的 reducer
    hideModal(){},
    // 使用服务器数据返回
    querySuccess(state, action){
      return {...state, ...action.payload, loading: false};
    },
    createSuccess(){},
    deleteSuccess(){},
    updateSuccess(){},
  }
}
```

首先我们需要增加 `*query` 第二个参数 `*query({ payload }, { select, call, put })` ，其中 call 和 put 是 dva 提供的方便操作 effects 的函数，简单理解 call 是调用执行一个函数而 put 则是相当于 dispatch 执行一个 action，而 select 则可以用来访问其它 model，更多可以参看 [redux-saga-in-chinese](https://github.com/superRaytin/redux-saga-in-chinese)。

而在 `query` 函数里面，可以看到我们处理异步的方式跟同步一样，所以能够很好的控制异步流程，这也是我们使用 Effects 的原因，关于相关的更多内容可以参看 [Generator 函数的含义与用法](http://www.ruanyifeng.com/blog/2015/04/generator.html)。

这里我们把请求的处理直接写在了代码里面，接下来我们需要把它拆分到 `/services/` 里面统一处理：

```jsx
import request from '../utils/request';
import qs from 'qs';
async function query(params) {
  return request(`/api/users?${qs.stringify(params)}`);
}
```

关于 async 的用法，可以参看 [async 函数的含义和用法](http://www.ruanyifeng.com/blog/2015/05/async.html)，需要注意的是，无论是 Generator 函数，yield 亦或是 async 目的只有一个： __让异步编写跟同步一样__ ，从而能够很好的控制执行流程。

下一步，进入[定义Services](./08-定义Services.md)。

