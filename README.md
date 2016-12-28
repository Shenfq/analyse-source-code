## jQuery 源码分析(版本2.0.3)


因为是初学者，所以很多都是自己片面的理解，希望大家多多包涵。

下面为大神总结的jQuery-2.0.3源码的结构，我会按照这个目录慢慢将源码慢慢（及其缓慢）看完，然后按照自己的理解对源码进行解读，解读后的文档都会放在doc文件夹中，方便查阅。同时我也会把我写过注释的源码上传，希望大家多多支持。这个过程可能会比较慢，希望能坚持到底别半途而废。



(function(window,undefined){ 

[(21 , 94) 定义了一些变量和函数，并初始化了jQ对象。jQuery = function(){};](https://github.com/Shenfq/Analyse-jQuery-source-code/blob/master/doc/1%E3%80%81jQuery%E5%AF%B9%E8%B1%A1%E7%9A%84%E5%88%9D%E5%A7%8B%E5%8C%96.md)

[(96 , 283) 给JQ对象，添加一些方法和属性](https://github.com/Shenfq/Analyse-jQuery-source-code/blob/master/doc/2%E3%80%81JQ%E5%AF%B9%E8%B1%A1%E6%B7%BB%E5%8A%A0%E4%B8%80%E4%BA%9B%E6%96%B9%E6%B3%95%E5%92%8C%E5%B1%9E%E6%80%A7.md)

[(285 , 347) extend : JQ的扩展方法的函数](https://github.com/Shenfq/Analyse-jQuery-source-code/blob/master/doc/3%E3%80%81jQuery%E5%AF%B9%E8%B1%A1%E7%9A%84%E6%89%A9%E5%B1%95--extend.md)

[(349 , 870) jQuery.extend() : 扩展一些工具方法](https://github.com/Shenfq/Analyse-jQuery-source-code/blob/master/doc/4%E3%80%81jQuery%E6%89%A9%E5%B1%95%E7%9A%84%E9%9D%99%E6%80%81%E6%96%B9%E6%B3%95.md)

(877 , 2856)  Sizzle : 复杂选择器的实现 

[(2880 , 3042) Callbacks : 回调对象 : 对函数的统一管理](https://github.com/Shenfq/Analyse-jQuery-source-code/blob/master/doc/5%E3%80%81Callback%E6%96%B9%E6%B3%95.md)
	
[(3043 , 3183) Deferred : 延迟对象 : 对异步的统一管理](https://github.com/Shenfq/Analyse-jQuery-source-code/blob/master/doc/6%E3%80%81Deferred%E5%BB%B6%E8%BF%9F%E5%AF%B9%E8%B1%A1.md)

[(3184 , 3295) support : 功能检测 : 对浏览器的功能检查](https://github.com/Shenfq/Analyse-jQuery-source-code/blob/master/doc/7%E3%80%81Support%E5%AF%B9%E8%B1%A1.md)

[(3308 , 3652) data() : 数据缓存](https://github.com/Shenfq/Analyse-jQuery-source-code/blob/master/doc/8%E3%80%81Data%E7%BC%93%E5%AD%98%E5%AF%B9%E8%B1%A1.md)

[(3653 , 3797) queue() : 队列方法 : 执行顺序的管理 ](https://github.com/Shenfq/Analyse-jQuery-source-code/blob/master/doc/9%E3%80%81queue%E9%98%9F%E5%88%97%E5%AF%B9%E8%B1%A1.md)

(3803 , 4299) attr() prop() val() addClass()等 : 对元素属性的操作 :
[1-对元素属性的操作](https://github.com/Shenfq/Analyse-jQuery-source-code/blob/master/doc/10%E3%80%81%E5%AF%B9%E5%85%83%E7%B4%A0%E5%B1%9E%E6%80%A7%E7%9A%84%E6%93%8D%E4%BD%9C.md)
[2-对元素Class名的操作](https://github.com/Shenfq/Analyse-jQuery-source-code/blob/master/doc/10-2%E3%80%81%E5%AF%B9%E5%85%83%E7%B4%A0Class%E7%9A%84%E6%93%8D%E4%BD%9C.md)

(4300 , 5128) on() trigger() : 事件操作的相关方法 

(5140 , 6057) DOM操作 : 添加 删除 获取 包装 DOM筛选 

(6058 , 6620) css() : 样式的操作 

(6621 , 7854) 提交的数据和ajax() : ajax() load() getJSON() 

(7855 , 8584) animate() : 运动的方法 

(8585 , 8792) offset() : 位置和尺寸的方法 

(8804 , 8821) JQ支持模块化的模式 

(8826)  window.jQuery = window.$ = jQuery; 

})(window);




