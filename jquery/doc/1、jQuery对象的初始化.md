### 1、命名空间
为什么要使用命名空间？

在一些语言中会看到有命名空间的概念，可在js中并没有，但是可以通过闭包来实现。在js闭包中定义的变量会被保存到一个作用域且不会污染全局变量，在程序运行完之后也不会被销毁。

我们可以看到，jQuery的做法就是使用了一个匿名函数形成一个闭包,然后该匿名函数自调用，所有的代码都在这个闭包中完成，这样就不会对全局变量污染，但是无法在其他地方访问该闭包内的变量，所以我们可以将window对象传入该匿名函数，然后将jQuery对象进行暴露，并且在暴露jQuery对象时，给jQuery对象赋了一个别名 $ ，这样就可以通过暴露的 jQuery 或者 $ 访问闭包内的其他变量了。

```javascript
(function(window,undefined){
    //code
    
    if ( typeof window === "object" && typeof window.document === "object" ) {
        window.jQuery = window.$ = jQuery;
    }
})(window);
```

我们可以看到在这个匿名函数自调用时，传入了window对象，这样做不仅是为了暴露接口，还有两个原因：

1. 使得window由全局变量变为局部变量，当在jQuery代码块中访问window对象时，不需要将作用域链回退到顶层作用域，这样可以更快的访问window。
2. 将window作为参数传入，可以在压缩代码时进行优化。

那么为什么要在参数列表中增加undefined呢？这样做也有两个目的：

1、因为有些低版本的浏览器中Undefined是可以被重新赋值的，在自调用匿名函数的作用域内，确保undefined是真的未定义。因为undefined能够被重写，赋予新的值。

```javascript
undefined = "now it's defined";
alert( undefined );
//我们会发现在有些低版本的浏览器会弹出 "now it's defined" 的字符串
```

2、在jquery经常要判断一个值是否为undefined，在压缩时将undefinde压缩一字母u，这样可以节省大量的空间。



### 2、一些变量的申明
```javascript
    _jQuery = window.jQuery
    _$ = window.$
```

在申明变量时，jQuery先将window对象下的jQuery和$存储到临时变量_jQuery和_$中，这是为了防止命名冲突，防止在导入其他库的时候命名出现重复导致一个库无法被调用，比如prototype也使用 $ 作为接口的。

```javascript
    class2type = {},
    core_deletedIds = [],
    core_version = "2.0.3",
    core_concat = core_deletedIds.concat,
    core_push = core_deletedIds.push,
    core_slice = core_deletedIds.slice,
    core_indexOf = core_deletedIds.indexOf,
    core_toString = class2type.toString,
    core_hasOwn = class2type.hasOwnProperty,
    core_trim = core_version.trim,
```

class2type、core_deletedIds、core_version就代表三种变量类型，对象、数组、字符串。然后将这三种变量类型常用的方法进行存储，再利用call和apply进行方法的借用。
另一方面，调用实例arr的方法concat时，首先需要辨别当前实例arr的类型是Array，在内存空间中寻找Array的concat内存入口，把当前对象arr的指针和其他参数压入栈，跳转到concat地址开始执行。 当保存了concat方法的入口core_concat时，完全就可以省去前面两个步骤，从而提升一些性能。 


后面还申明了一些正则表达式，在后面用到的时候再一一讲解，正则表达式真是一个让人懵逼的东西。 


### 3、严格模式

>"use strict";

开头有一句"use strict",表示使用js的严格模式。

因为这是ES5中新增的内容，所以低版本的浏览器不兼容，低版本的jq默认是不执行严格模式的。高版本的jq中默认是执行严格模式的。

在查看jq源码时，注释中的有些括号里有#符号开头的字符，例如(#13335)。
这是对jq版本升级后一些bug的说明，可以在[jq官网](http://bugs.jquery.com/)查看

### 4、初始化一个jQuery对象

```javascript
    jQuery = function( selector, context ) {
    	return new jQuery.fn.init( selector, context, rootjQuery );
    }
```

可以看到jQuery对象其实相当于一个对jQuery.fn.init对象进行实例化的工厂，每次调用jQuery时，就返回了一个init对象的实例，这样就实现了免new调用jQuery对象。

那么这个jQuery.fn.init到底是一个什么东西呢？

```javascript
    jQuery.fn = jQuery.prototype = {  //96行
    	init: function(selector, context, rootjQuery) {}
    ｝
    jQuery.fn.init.prototype = jQuery.fn;  //283行
```

可以看到jQuery.fn 就是jQuery对象的原型，然后让**jQuery.fn.init.prototype = jQuery.fn**，这样就相当于让init继承了jQuery对象上的方法。所以调用jQuery时，返回的init的实例就相当于继承了jQuery原型上的所有方法，我靠，怎么能这么绕。

但是为什么在调用jQuery时不直接返回一个jQuery对象，还要先返回init的实例，然后让init来继承jQuery的方法呢（我为什么要问这么白痴的问题）？

首先，如果让jQuery返回jQuery的话这样就会形成一个死循环，其次如果返回init实例，init又是jQuery原型下的一个方法，init中的this就是jQuery对象，这样就能轻松的实现链式调用，只要在每个方法中返回this，也就是返回了jQuery对象。

刚开始总会有瑕疵，慢慢加油（我不是鸡汤，认真脸）。