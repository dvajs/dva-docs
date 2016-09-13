## 定义Services

之前我们已经：

1. 设计好了 model state -> 抽象数据
2. 完善了组件 -> 完善展示
3. 添加了 Reducers -> 数据同步处理
4. 添加了 Effects -> 数据异步处理

接下来就是将请求相关（与后台系统的交互）抽离出来，单独放到 `/services/` 中，进行统一维护管理，所以我们只需要将之前定义在 Effects 的以下代码，移动到 `/services/users.js` 中即可：

```jsx
// request 是我们封装的一个网络请求库
import request from '../utils/request';
import qs from 'qs';

export async function query(params) {
  return request(`/api/users?${qs.stringify(params)}`);
}
```

```jsx
// ./src/utils/request.js
import fetch from 'dva/fetch';

function parseJSON(response) {
  return response.json();
}

function checkStatus(response) {
  if (response.status >= 200 && response.status < 300) {
    return response;
  }

  const error = new Error(response.statusText);
  error.response = response;
  throw error;
}

/**
 * Requests a URL, returning a promise.
 *
 * @param  {string} url       The URL we want to request
 * @param  {object} [options] The options we want to pass to "fetch"
 * @return {object}           An object containing either "data" or "err"
 */
export default function request(url, options) {
  return fetch(url, options)
    .then(checkStatus)
    .then(parseJSON)
    .then((data) => ({ data }))
    .catch((err) => ({ err }));
}

```

然后在 users model 中引入：

```jsx
import { query } from '../services/users';
```

之后无论是更新，删除、添加等操作，跟用户相关的都可以统一放置在 `/services/users.js` 中。

代码写到这里，你可以看到浏览器中的显示是这样的：

![image](https://zos.alipayobjects.com/rmsportal/blvkEJOTjrXCIVf.png)

没错，我们虽然有了接口，但是我们还没有数据，在 dva 中，我们配套的工具能够很方便的模拟数据，这样就可以脱离服务器复杂的环境进行模拟的本地调试开发。下面一节就会一起来看下，如何 mock 数据。

下一步，进入[mock数据](./09-mock数据.md)。
