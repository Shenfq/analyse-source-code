
>首先我们要知道为什么jQuery为什么会存在Data这个对象，主要是用来干嘛的呢？

很明显Data对象被用来进行数据的缓存，它存在的主要原因是为了解决js的内存泄漏问题。在早期的jQuery版本中，所有的缓存数据都放在DOM节点下，但是这样做就会产生内存泄露的问题。因为js是使用垃圾回收机制来管理内存的，如果DOM对象与普通js对象一旦产生相互引用垃圾回收机制就认为这块内存一直在被使用，迟迟不回收，从而产生内存泄漏的问题。当然那只是老版本浏览器，现在的浏览器已经优化了垃圾回收算法。具体关于js内存泄漏更多细节可以查看这篇[博客](http://blog.csdn.net/qq_16339527/article/details/53287140)。

jQuery为了解决这种内存泄漏引入了Data机制，其主要原理就是将一个生成的唯一标识符作为一个属性名缓存到一个DOM对象或者一个普通对象下，然后该属性名保存的是jQuery对象中的Cache对象的属性名，把要缓存的数据全部挂载到Cache对象的下。描述得不是很清楚，可以通过下图进行理解。

![](http://i.imgur.com/DUOGC3h.png)




下面简单看看这个对象的构造

```javascript
function Data(){  //构造函数
	//该对象下有两个属性：cache和expando
	//一个属性是一个对象，用来存储要缓存的数据
	//一个属性是一个随机字符串，是打通cache对象与指定缓存对象的一座桥梁
	Object.defineProperty( this.cache = {}, 0, { 
		get: function() {//当访问Data.cache[0]时，默认返回一个空对象，且不能被修改。
			return {};
		}
	});
	this.expando = jQuery.expando + Math.random();  //生成一个唯一字符串
}
Data.uid = 1;  //uid 从1开始，表示当前cache对象中的属性名
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
```


----------

这里重点介绍如下几个方法：

首先是Data原型下的key、set、get、access

```javascript
key: function( owner ) {
	if ( !Data.accepts( owner ) ) {  //如果该对象不能满足Data.accepts条件则return 0，直接跳出
		return 0;
	}
	var descriptor = {},
		unlock = owner[ this.expando ];  //检测这个对象是否已经存在缓存键

	// If not, create one
	if ( !unlock ) {  //如果没有被缓存过数据，创建key
		unlock = Data.uid++;
		//此处为了兼容性，如果有defineProperties方法，就是用该方法将expando属性扩展到owner对象，没有这是要jQuery定义的extend
		try {
			descriptor[ this.expando ] = { value: unlock };
			Object.defineProperties( owner, descriptor );
		} catch ( e ) {
			descriptor[ this.expando ] = unlock;
			jQuery.extend( owner, descriptor );
		}
		//有人觉得这样做会多此一举，直接用extend扩展就可以。这里jQuery这样做的目的是为了让owner对象上的expando属性只可读，不可写。
		//也就是所如果使用Object.defineProperties方法进行扩展，就算用户不小心修改了expando的值也没关系，因为该属性不能被修改。
		//因为Android4.0以下版本的浏览器不支持defineProperties方法进行扩展，所有使用extend方法扩展的属性但是很容易被修改，这是不安全的。
	}

	// 给cache对象下创建一个用来缓存属性的新对象。并且属性名为owner对象下expando属性的值，让expando成为打通cache和owner之前的桥梁。
	if ( !this.cache[ unlock ] ) {
		this.cache[ unlock ] = {};
	}

	return unlock;  //最后返回缓存键的键值
},
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
},
remove: function( owner, key ) {
	var i, name, camel,
		unlock = this.key( owner ),//获取当前对象在cache上的属性名
		cache = this.cache[ unlock ];//获取该对象缓存的全部数据

	if ( key === undefined ) {//如果不指定要删除的key名，则会将当前缓存对象置空
		this.cache[ unlock ] = {};

	} else {
		//如果有key时：
		if ( jQuery.isArray( key ) ) {  //先判断是不是数组
			//如果是数组，将当前数组所有key和key的驼峰表示法合并到一个数组
			name = key.concat( key.map( jQuery.camelCase ) );
		} else {//不为数组的情况
			camel = jQuery.camelCase( key ); //获取key的驼峰表示法
			if ( key in cache ) { //key属性存在于cache，则将key和camel合并到一个数组
				name = [ key, camel ];
			} else {//key属性不存在于cache，再判断camel属性是否存在于cache。
				name = camel;
				name = name in cache ?
					[ name ] : ( name.match( core_rnotwhite ) || [] );
			}
		}

		i = name.length;//遍历之前合并的key数组，将cache下的key属性全部删除
		while ( i-- ) {
			delete cache[ name[ i ] ];
		}
	}
}

```


理解上面三个方法后，再看看jQuery扩展的几个方法就一目了然，其实就是调用了Data原型下的几个方法。

```javascript
jQuery.extend({
	acceptData: Data.accepts,

	hasData: function( elem ) {
		return data_user.hasData( elem ) || data_priv.hasData( elem );
	},

	data: function( elem, name, data ) {
		return data_user.access( elem, name, data );
	},

	removeData: function( elem, name ) {
		data_user.remove( elem, name );
	}
});
```


了解了静态方法下的data后，我们在看看实例方法下的data方法，也就是通过jQuery.fn.extend扩展的data。在jQuery中很多方法都会扩展两次，一次是扩展在jQuery对象上，还一次就是扩展在jQuery的原型对象上。我们都知道jQuery是一个以DOM操作为主的库，所以我们很多时候是使用的实例方法，而且还能在一个jQuery对象上进行链式调用。jQuery中的静态方法其实是在内部提供给实例方法使用的，实例方法则是对静态方法的进一步抽象，不信你接下来看看jQuery.fn.data是不是比jQuery.data复杂的多。


```javascript
jQuery.fn.extend({
		data: function( key, value ) {
		var attrs, name,
			elem = this[ 0 ],//elem表示当前$对象下的第一个dom对象
			i = 0,
			data = null;
		//如果key不存在获取elem所有缓存数据
		if ( key === undefined ) {
			if ( this.length ) {
				data = data_user.get( elem );  //获取当前elem下的所有缓存数据
				if ( elem.nodeType === 1 && !data_priv.get( elem, "hasDataAttrs" ) ) {
					attrs = elem.attributes;//获取elem下面所有的属性
					for ( ; i < attrs.length; i++ ) {
						name = attrs[ i ].name;//获取属性名
						if ( name.indexOf( "data-" ) === 0 ) {//判断是不是html5的自定义属性data-*
							name = jQuery.camelCase( name.slice(5) );
							dataAttr( elem, name, data[ name ] );//如果data[name]是undefined，就将该属性及其值缓存到cache对象上，该方法在（3625行）
						}
					}
					data_priv.set( elem, "hasDataAttrs", true );//然后把hasDataAttrs属性设为true
				}
			}
			return data;
		}
		//如果key为对象，遍历jQuery对象，并调用set方法
		if ( typeof key === "object" ) {
			return this.each(function() {//实例方法下，只要是给DOM扩展属性或者设置状态，都要调用each方法，为当前jQuery对象下的所有DOM执行同样的操作
				data_user.set( this, key );
			});
		}
		//key存在且不是一个对象时
		return jQuery.access( this, function( value ) {
			var data,
				camelKey = jQuery.camelCase( key );//获取当前key的驼峰表示

			if ( elem && value === undefined ) {//当value为undefined时，为获取当前节点key的缓存值
				// 同时获取了当前节点缓存对象的key以及key的驼峰表示
				data = data_user.get( elem, key );
				if ( data !== undefined ) {
					return data;
				}
				data = data_user.get( elem, camelKey );
				if ( data !== undefined ) {
					return data;
				}
				data = dataAttr( elem, camelKey, undefined );//如果在缓存对象下找不到存储的数据，则寻找当前节点的data-*属性下缓存的数据，并返回
				if ( data !== undefined ) {
					return data;
				}

				return;//通过三种方式都找不到数据，则直接跳出
			}
			this.each(function() {  //给当前jQuery对象下的每个节点都缓存数据
				var data = data_user.get( this, camelKey );//先获取了其驼峰表示法的值
				data_user.set( this, camelKey, value );//以驼峰表示的属性名来缓存数据对象，因为html5的data属性必须以这种方式缓存数据
				if ( key.indexOf("-") !== -1 && data !== undefined ) {
					data_user.set( this, key, value );//如果key存在'-'，且data有值，则以key的方式缓存一次value
				}
			});
		}, null, value, arguments.length > 1, null, true );
	},
	
	//可以看到实例方法下的data方法多了很多判断，为了用户方便，会在缓存对象时，会把属性名转换成驼峰表示法进行缓存。

	removeData: function( key ) {//移除缓存
		return this.each(function() {
			data_user.remove( this, key );//直接调用了Data.prototype下的remove方法
		});
	}
)};
```


再看看在实例方法中用到dataAttr到底做了什么：
该方法主要是获取了elem节点上使用html5新方式扩展的data-*属性，然后把该属性缓存到cache对象上，并缓存该属性值。

```javascript
	function dataAttr( elem, key, data ) {
		var name;
		if ( data === undefined && elem.nodeType === 1 ) {//如果当前节点的data为undefined
			name = "data-" + key.replace( rmultiDash, "-$1" ).toLowerCase();//将驼峰表示转化成之前的样子
			data = elem.getAttribute( name );//获取缓存值
			if ( typeof data === "string" ) {
				try {
					data = data === "true" ? true :
						data === "false" ? false :
						data === "null" ? null :
						+data + "" === data ? +data :  
						rbrace.test( data ) ? JSON.parse( data ) :
						//rbrace = /(?:\{[\s\S]*\}|\[[\s\S]*\])$/,判断是不是一个json
						data;
				} catch( e ) {}

				// 把data缓存到jQuery的cache对象上
				data_user.set( elem, key, data );
			} else {
				data = undefined;
			}
		}
		return data;
	}
```

----------

到这里我们就已经基本了解了缓存对象主要用来干嘛以及其工作原理，还有一部分源码没有深入研究。说说题外话，最近学起来也有点迷茫，发现前端框架越来越多，也越来越好用，自己是否还有必要坚持jQuery的深入。虽然说jQuery的使用越来越少而且与主流框架的理念有些格格不入了，但jQuery还是小项目的有力支撑。既然开始了就先把开始的事坚持到底吧。