
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
		key: function(owner){},  判断owner有没有expando属性，没有就给owner对象设置一个expando属性，有则返回expando属性缓存的键值
		set: function(){},设置缓存数据
		get: function(){},获得缓存数据
		access: function(){},set方法与get方法的抽象
		remove: function(){},删除一个缓存数据
		hasData: function(){},判断当前对象是否有缓存对象或缓存对象是否为空
		discard: function(){}清空当前对象的缓存数据
	}

	//  创建了两个Data()对象的实例
	data_user = new Data(); //该对象实例提供给jquery对象外部使用
	data_priv = new Data();	//该对象实例提供给jquery对象内部使用，用得比较少
	

	//下面几个方法扩展到jQuery对象下，主要是对Data原型下方法的调用
	jQuery.extend({   //$.data($("div",key,value)), 调用该方法时，将缓存键直接jQuery对象上
		acceptData: Data.accepts,
		data: function(){},
		hasData: function(){},
		removeData: function(){},
		//这两个方法现在就是调用data_priv下的方法，现在不建议使用
		_data: function(){},
		_removeData: function(){}
	});
	
	//下面几个方法扩展到jQuery实例下,data方法将Data原型下的get和set方法进一步抽象
	jQuery.fn.extend({  //$("div").data(key,value), 调用该方法时，将缓存键放在jQuery对象缓存的第一个DOM节点上
		data: function(){},
		hasData: function(){}
	});



----------

这里重点介绍如下几个方法：

首先是Data原型下的set、get、access
	
	set: function( owner, data, value ) {
		var prop,
			unlock = this.key( owner ),  //获取缓存键值，没有则设置一个缓存键并将值返回
			cache = this.cache[ unlock ];
		//通过判断data类型来处理缓存数据的方式  1：$.data(obj, 'key', 'value'); 2: $.data(obj,{key:'value',key2:'value'});
		if ( typeof data === "string" ) {  //进行赋值操作  Data.cache[unlock][data] = value
			cache[ data ] = value;
		} else { //如果data是一个对象，表示要添加多个属性到缓存对象上
			if ( jQuery.isEmptyObject( cache ) ) {//判断当前cache是否为空，如果为空使用extend，不为空使用forin遍历添加
				jQuery.extend( this.cache[ unlock ], data );
			} else { 
				for ( prop in data ) {
					cache[ prop ] = data[ prop ];
				}
			}
		}
		return cache;  //最后返回该对象下的全部缓存值
	},
	get: function( owner, key ) {
		var cache = this.cache[ this.key( owner ) ];  //先获取该对象上的全部缓存数据，然后看key是否存在

		return key === undefined ?  //key不存在则返回全部缓存数据，如果存在就返回要找的key存储的数据
			cache : cache[ key ];
	},
	access: function( owner, key, value ) {  //该方法是对get方法与set方法的抽象
		var stored;
		if ( key === undefined ||  //如果没有指定key或者key为string但是value没有被指定
				((key && typeof key === "string") && value === undefined) ) {
			stored = this.get( owner, key );
			return stored !== undefined ?
				stored : this.get( owner, jQuery.camelCase(key) );
		}
		this.set( owner, key, value );
		return value !== undefined ? value : key; //最后返回缓存的数据
	}

理解上面三个方法后，再看看jQuery扩展的data方法就一目了然

	jQuery.extend({
		data: function( elem, name, data ) {
			return data_user.access( elem, name, data );
		}
	});

我们能看出其实data方法就是Data.prototype下的access方法。

到这里我们就已经基本了解了缓存对象主要用来干嘛以及其工作原理，还有一部分源码没有深入研究。说说题外话，最近学起来也有点迷茫，发现前端框架越来越多，也越来越好用，自己是否还有必要坚持jQuery的深入。虽然说jQuery的使用越来越少而且与主流框架的理念有些格格不入了，但jQuery还是小项目的有力支撑。既然开始了就先把开始的事坚持到底吧。