## jQuery 源码分析(版本2.0.3)


因为是初学者，所以很多都是自己片面的理解，希望大家多多包涵。

下面为大神总结的jQuery-2.0.3源码的结构，我会按照这个目录慢慢将源码看完，将看完的模块划掉，然后按照自己的理解对源码进行解读，解读后的文档都会放在doc文件夹中，方便查阅。同时我也会把我写过注释的源码上传，希望大家多多支持。这个过程可能会比较慢，希望年底能全部完成。



(function(window,undefined){ 

~~(21 , 94) 定义了一些变量和函数 jQuery = function(){};~~

~~(96 , 283) 给JQ对象，添加一些方法和属性~~

~~(285 , 347) extend : JQ的扩展方法的函数~~

~~(349 , 870) jQuery.extend() : 扩展一些工具方法~~ 

(877 , 2856)  Sizzle : 复杂选择器的实现 

~~(2880 , 3042) Callbacks : 回调对象 : 对函数的统一管理~~ 
	
~~(3043 , 3183) Deferred : 延迟对象 : 对异步的统一管理~~ 

~~(3184 , 3295) support : 功能检测 : 对浏览器的功能检查~~ 

(3308 , 3652) data() : 数据缓存 

(3653 , 3797) queue() : 队列方法 : 执行顺序的管理 

(3803 , 4299) attr() prop() val() addClass()等 : 对元素属性的操作 

(4300 , 5128) on() trigger() : 事件操作的相关方法 

(5140 , 6057) DOM操作 : 添加 删除 获取 包装 DOM筛选 

(6058 , 6620) css() : 样式的操作 

(6621 , 7854) 提交的数据和ajax() : ajax() load() getJSON() 

(7855 , 8584) animate() : 运动的方法 

(8585 , 8792) offset() : 位置和尺寸的方法 

(8804 , 8821) JQ支持模块化的模式 

(8826)  window.jQuery = window.$ = jQuery; 

})(window);




