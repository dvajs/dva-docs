## mock数据

我们采用了 [dora-plugin-proxy](https://github.com/dora-js/dora-plugin-proxy) 工具来完成了我们的数据 mock 功能。

在 `package.json` 中：

```json
  "scripts": {
    "start": "dora --plugins \"proxy,webpack,webpack-hmr\"",
    "lint": "eslint --fix --ext .js,.jsx .",
    "build": "atool-build"
  }
```

的start命令中，可以看到使用 [dora](https://github.com/dora-js/dora) 工具的相关内容，其中`proxy`就是dora的一个插件，在你的项目不需要代理的时候，去除proxy插件即可。

mock文件如下：

```js
// ./mock/users.js
'use strict';

const qs = require('qs');

// 引入 mock js
const mockjs = require('mockjs');

module.exports = {
  'GET /api/users' (req, res) {
    const page = qs.parse(req.query);

    const data = mockjs.mock({
      'data|100': [{
        'id|+1': 1,
        name: '@cname',
        'age|11-99': 1,
        address: '@region'
      }],
      page: {
        total: 100,
        current: 1
      }
    });

    res.json({
      success: true,
      data: data.data,
      page: data.page
    });
  },
};
```

在完整样例中，mock 数据更为全面。

具体更多的使用方法，可以参看 [dora-plugin-proxy 文档](https://github.com/dora-js/dora-plugin-proxy)。

到此为止，我们围绕 UserList 组件的实现，从各方面展示了 dva 项目的设计思路以及方法，剩下的样例项目内容大同小异，关于应用中其它组件的实现和相关内容，请参看[完整内容](https://github.com/dvajs/dva/tree/master/examples/user-dashboard)。

下一步，进入[添加样式](./10-添加样式.md)。
