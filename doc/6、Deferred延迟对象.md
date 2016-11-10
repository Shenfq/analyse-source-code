jQuery的延迟对象主要作用是回调函数的异步处理，是一种Promises/A规范的一种实现。具体什么是Promises/A规范大家可以看看[这里](http://blog.csdn.net/qq_16339527/article/details/53097891)。

在jQuery中通过$.extend()方法扩展了两个对象。
    
    jQuery.extend(
    	Defferred: function(){},
    	when:function() {}
    );
