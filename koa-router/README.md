# koa-router

koa-router应该是最常使用的koa的路由库，其源码比较简单，而且有十分详细的注释与使用案例。使用方式也比tj大神的koa-route要简洁。

按照惯例，先看看koa-router的使用方法。

```javascript
var Koa = require('koa');
var Router = require('koa-router');

var app = new Koa();
var router = new Router();

router
  .get('/', (ctx, next) => {
    ctx.body = 'Hello World!';
  })
  .post('/users', (ctx, next) => {
    // ...
  })
  .put('/users/:id', (ctx, next) => {
    // ...
  })
  .del('/users/:id', (ctx, next) => {
    // ...
  })
  .all('/users/:id', (ctx, next) => {
    // ...
  });

app
  .use(router.routes())
  .use(router.allowedMethods());
```