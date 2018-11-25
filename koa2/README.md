> koa2相关源码都放在 `./src/` 文件夹下，并且有添加一些注释。

## 如何使用koa

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

## koa的构造函数

```javascript
const Emitter = require('events');

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

可以看到整个Application继承自node原生的events，这是node的事件系统，这里不做过多展开，与浏览器中的事件绑定类似。
对koa实例化之后，先使用use方法进行中间件的绑定，然后调用listen方法监听系统端口，并且启动http服务。

```javascript
use(fn) {
  // 判断中间件是否为函数
  if (typeof fn !== 'function')
    throw new TypeError('middleware must be a function!');
  // 判断是否为迭代器，如果是，需要提示这种用法在下个版本会被抛弃
  if (isGeneratorFunction(fn)) {
    deprecate('Support for generators will be removed in v3. ')
    fn = convert(fn);
  }
  // 将中间件函数放入middleware栈中
  this.middleware.push(fn);
  return this;
}
```

首先需要判断传入的中间件是否为一个函数，koa1的版本传入的为迭代器来做异步操作，且需要使用`convert`进行包装，koa2的中间件使用`async function`，然后将这些中间件函数放入middleware栈中。

## 启动http服务

接下来调用listen方法启动一个http服务，http服务的回调函数由callback方法生成。

```javascript
const http = require('http');

listen(...args) {
  // koa内部其实还是调用node自带的http服务
  const server = http.createServer(this.callback());
  return server.listen(...args);
}


const compose = require('koa-compose');

callback() {
  const fn = compose(this.middleware); // 用于生成koa的洋葱模型

  if (!this.listenerCount('error')) this.on('error', this.onerror);

  const handleRequest = (req, res) => {
    // 创建ctx对象，该对象贯穿整个koa，根据http类提供的req和res对象生成
    const ctx = this.createContext(req, res);
    return this.handleRequest(ctx, fn); // 返回值为http服务的回调函数
  };

  return handleRequest;
}

handleRequest(ctx, fnMiddleware) {
  // 开始执行洋葱模型
  return fnMiddleware(ctx)
    .then(() => respond(ctx))
    .catch(err => ctx.onerror(err));
}
```

## 洋葱模型

![洋葱模型](./images/onion.png)

通过compose函数包装中间件数组，用于生成洋葱模型，该函数属于koa的一个模块`koa-compose`，现在深入`koa-compose`模块看看，它到底对middleware做了什么。

```javascript
module.exports = compose
function compose (middleware) {
  return function (context) {
    return dispatch(0) // 先调用第一个中间件
    function dispatch (i) {
      let fn = middleware[i]
      // fn不存在表示已经没有中间件了，直接进行resolve操作
      if (!fn) return Promise.resolve()
      try {
        // 调用中间件函数，第二个参数表示调用下一个中间件
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}
```

可以很明显的看到这是一个闭包，为了方便阅读我省略了一些错误处理，完整源码请看：[链接](./src/koa-compose/index.js)。闭包返回的函数最后在handleRequest方法中被调用，而这个方法就是每次http服务的回调函数，所以每次发起http请求都会走入这个洋葱模型。

这个闭包里面进行了一个递归操作，先取出middleware的第0个函数，然后给这个函数传入两个参数第一个是ctx对象，这个对象属于koa自己封装的对象，后面再讲，第二个参数就是我们在中间件中使用的的`next`方法。

```
dispatch.bind(null, i + 1)
```

可以看到next方法其实就是递归调用下一个中间件函数。还有一点需要注意dispatch方法返回的是Promise对象，因为我们在使用koa的时候，中间件都是`async function`，而且调用next的时候使用的是`await next()`，这里必须返回的是一个Promise对象。

## ctx对象

看完洋葱模型之后看看koa的另一个重点，ctx对象。该对象是通过http服务的回调函数传入的`req`、`res`两个对象生成的。

```javascript
const ctx = this.createContext(req, res);


createContext(req, res) {
  const context = Object.create(this.context);
  const request = context.request = Object.create(this.request);
  const response = context.response = Object.create(this.response);
  context.app = request.app = response.app = this;
  context.req = request.req = response.req = req;
  context.res = request.res = response.res = res;
  request.ctx = response.ctx = context;
  request.response = response;
  response.request = request;
  context.originalUrl = request.originalUrl = req.url;
  context.state = {};
  return context;
}
```

这里基本上是几个对象之间的相互挂载，进行了一系列的循环引用，为使用者提供方便。

// todo ctx对象的代理

