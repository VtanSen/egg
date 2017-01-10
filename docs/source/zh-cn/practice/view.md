title: View 插件开发规范
---

# View 插件开发规范

绝大多数情况，我们都需要读取数据后渲染模板，然后呈现给用户。故我们需要引入对应的模板引擎。

而 egg 并不强制你使用某种模板引擎，开发者可以依据下文的规范, 来封装对应的模板引擎插件。

目前官方支持的 view 插件有：
- [egg-view-nunjucks](https://github.com/eggjs/egg-view-nunjucks)

## 术语

本文中：
- 插件开发者：指开发 egg view 插件的开发者，输出物为 npm 包
- 应用开发者：指开发 egg 应用的开发者，在应用中需要使用 view 插件

## 插件命名规范

- 遵循 [插件开发规范](../advanced/plugin.md)
- 插件命名约定以 `egg-view-` 开头
- package.json 中增加:

```json
{
  "name": "egg-view-nunjucks",
  "eggPlugin": {
    "name": "view"
  },
  "keywords": [
    "egg",
    "eggPlugin",
    "egg-plugin",
    "egg-plugin-view",
    "nunjucks"
  ],
}
```

## 应用开发者使用方式

1. 引入 view 插件

```bash
$ npm i egg-view-nunjucks --save
```

2. 启用插件

```js
// config/plugin.js
exports.view = {
  enable: true,
  package: 'egg-view-nunjucks',
}
```

3. 使用 view 插件

框架暴露了 3 个接口:
- ctx.render(name, locals) 渲染模板文件, 并赋值给 ctx.body
- ctx.renderView(name, locals) 渲染模板文件, 并赋值给 ctx.body
- ctx.renderString(tpl, locals) 渲染模板字符串, 仅返回不赋值

```js
// app/controller/home.js
module.exports = app => {
  const data = { name: 'egg' };

  // render a template, path releate to `app/view`
  yield this.render('path/to/file.tpl', data);

  // or manually set render result to this.body
  this.body = yield this.renderView('path/to/file.tpl', data);

  // or render string directly
  this.body = yield this.renderString('hi, {{ name }}', data);
};
```


## View 插件实现的约定

### View 基类

需提供一个 View 基类：

```js
// lib/view.js
module.exports = class MyCustomView {
  constructor(ctx) {
    this.ctx = ctx;
    this.app = ctx.app;
  }

  render(name, locals) {
    // name is a tpl file releate to `app/view`
    return Promise.resolve('some html');
  }

  renderString(tpl, locals) {
    // tpl is a string
    return Promise.resolve('some html');
  }
};
```

再把该基类挂载到 application 上：

```js
// app/extend/application.js
const View = require('../../lib/view');

module.exports = {
  // mount `View` class to app
  // egg will create an instance to `ctx.view` at every request
  // you can use `this.render` at controller
  get [Symbol.for('egg#view')]() {
    return View;
  },
}
```

### engine

### filter

### helper

### security


app.viewEngine 用于暴露给应用开发者


## 深度解析

