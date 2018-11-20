> koa2相关源码都放在 `./src/` 文件夹下，并且有添加一些注释。

在看koa2的源码之前，按照惯例先看看koa2的`hello world`的写法。

```javascript
const Koa = require('koa');
const app = new Koa();

// response
app.use(ctx => {
  ctx.body = 'Hello Koa';
});

app.listen(3000);
```

一开始就通过`new`关键词对koa进行了实例化，并且实例化后的对象具有`use`和`listen`方法。废话不多说，先看`require('koa')`引入的`application.js`。

```javascript
module.exports = class Application extends Emitter {
  constructor() {
    super();

    this.proxy = false;
    this.middleware = [];
    this.env = process.env.NODE_ENV || 'development';
    this.context = Object.create(context);
    this.request = Object.create(request);
    this.response = Object.create(response);
  }
  //...
}
```