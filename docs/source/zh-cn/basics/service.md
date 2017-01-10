title: service
---

# service

`service` 简单来说，就是当遇到一个复杂的业务场景，我们会把这些逻辑封装到一起，作为一个 `service`。这样可以可以有以下几个好处:

- 保持 Controller 中的逻辑更加简洁。
- 保持业务逻辑的独立性，抽象出来的 Service 可以被多个 Controller 重复调用。
- 将逻辑和展现分离，更容易编写测试用例，测试用例的编写具体可以查看 [这里](../core/unittest.md)。

## service 的一些场景

- 复杂数据的处理，比如要展现的信息需要从数据库获取，还要经过一定的规则计算，才能返回用户显示。或者计算完成后，更新到数据库。
- 第三方服务的调用，比如 Github 信息获取等。

## 定义 service

- `app/service/user.js`

  ```js
  module.exports = app => {
    class User extends app.Service {
      constructor(ctx) {
        super(ctx);
      }
      * find(uid) {
      }
    }
    return User;
  };
  ```

### 注意事项

- service 文件必须放在 `app/service` 目录，可以支持多级目录，访问的时候可以通过目录名级联访问。

  ```
  app/service/biz/user.js => this.service.biz.user.find
  ```

- 一个 service 文件只能包含一个类， 这个类需要通过 `module.exports` 的方式返回。
- service 需要通过 Class 的方式定义，父类必须是 app.Service, 其中 app.Service 会在初始化 Service 的时候，传递进来。
- service 中一定要有有构造函数， 并且执行 `super(ctx)`, 这样我们就可以在下面的函数中，直接可以通过 `this.ctx` 的方式获取上下文的信息 [Context](./extend.md#context), 通过它我们可以拿到框架给我们封装的各种便捷属性和方法。

### Service ctx 详解

为了可以获取用户请求的链路，我们在 Service 中，注入了 ctx, 用户直接可以通过 `this.ctx` 来获取上下文相关信息。比如我们可以用：

- `this.ctx.curl` 发起网络调用
- `this.ctx.service.otherService` 调用其他 Service
- `this.ctx.db` 发起数据库调用等， db 可能是其他插件提前挂载到 app 上的模块。

## 使用 service

下面就通过一个完整的例子，看看怎么使用 service。

```js
// app/router.js
module.exports = app => {
  app.get('/user/:id', 'user.info');
};

// app/controller/user.js
exports.info = function* () {
  const userId = this.params.id;
  const userInfo = yield this.service.user.find(userId);
  this.body = userInfo;
};

// app/service/user.js
module.exports = app => {
  class User extends app.Service {
    constructor(ctx) {
      super(ctx);
    }

    * find(uid) {
      // 假如 我们拿到用户 id 从数据库获取用户详细信息
      const user = this.ctx.db.query(`select * from user where uid = ${uid}`);
      // 假定这里还有一些复杂的计算，然后返回需要的信息。
      const result = yield this.ctx.curl(`http://photoserver/uid=${uid}`, {
        dataType: 'json',
      });
      return {
        name: user.user_name,
        age: user.age,
        picture: result.picture,
      };
    }
  }
  return User;
};

// curl http://127.0.0.1:7001/user/1234
```
