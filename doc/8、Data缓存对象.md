
>首先我们要知道为什么jQuery为什么会存在Data这个对象，主要是用来干嘛的呢？

很明显Data对象被用来进行数据的缓存，它存在的主要原因是为了解决js的内存泄漏问题。在早期的jQuery版本中，所有的缓存数据都放在DOM节点下，但是这样做就会产生内存泄露的问题。因为js是使用垃圾回收机制来管理内存的，如果DOM对象与普通js对象一旦产生相互引用垃圾回收机制就认为这块内存一直在被使用，迟迟不回收，从而产生内存泄漏的问题。当然那只是老版本浏览器，现在的浏览器已经优化了垃圾回收算法。具体关于js内存泄漏更多细节可以查看这篇[博客](http://blog.csdn.net/qq_16339527/article/details/53287140)。

jQuery为了解决这种内存泄漏引入了Data机制，其主要原理就是将一个生成的唯一标识符作为一个属性名缓存到一个DOM对象或者一个普通对象下，然后该属性名保存的是jQuery对象中的Cache对象的属性名，把要缓存的数据全部挂载到Cache对象的下。描述得不是很清楚，可以通过下图进行理解。

![](http://i.imgur.com/DUOGC3h.png)




下面简单看看这个对象的构造

	function Data(){  //构造函数
		//该对象下只有一个属性，这个属性是一个随机字符串，是沟通cache对象与指定缓存对象的一座桥梁
		this.expando = jQuery.expando + Math.random();  //生成一个唯一字符串
	}

	Data.accepts = function(owner){
		//该方法是用来判断传入的对象是否具有缓存数据的能力
		//只有简简单单的一句代码，判断出了两种情况：
		//1.owner是一个DOM对象，但只能是节点和document对象
		//2.以及不具有nodeType的任何对象
		return owner.nodeType ?
		owner.nodeType === 1 || owner.nodeType === 9 : true;
	}

	//在Data的原型下扩展了几个方法，这几个方法就是实现数据缓存的核心。之后在jQuery对象上拓展的方法都是对Data原型下方法的进一步抽象
	Data.prototype = {
		key: function(owner){},  给owner对象设置一个expando属性
		set: function(){},
		get: function(){},
		access: function(){},
		remove: function(){},
		hasData: function(){},
		discard: function(){}
	}

	data_user = new Data();
	data_priv = new Data();
	
	jQuery.extend({
		acceptData: Data.accepts,
		data: function(){},
		hasData: function(){},
		removeData: function(){},
		//这两个方法现在就是调用data_priv下的方法，现在不建议使用
		_data: function(){},
		_removeData: function(){}
	});
	
	jQuery.fn.extend({
		data: function(){},
		hasData: function(){}
	});