jQuery在定义了extend之后，马上利用这个函数，对自身扩展了一些常用的静态方法。

	jQuery.extend({    (349行--817行)
		expando: 生成一个唯一的标识符
	
		noConflict:  用于防止冲突
	
		isFunction:   判断是不是一个函数
	
		isArray: 判断是不是一个函数
	
		isWindow: 判断是不是一个window对象
	
		isNumeric: 判断是不是一个数
	
		type: 用来判断类型
	
		isPlainObject: 判断是不是一个继承自Object的对象
	
		isEmptyObject: 判断是不是一个空对象
	
		error:   用于抛出错误
			
		parseHTML:  将HTML字符串转换为DOM结点对象
	
		parseJSON: 将JSON字符串转换为JSON对象
	
		parseXML: 将XML字符串转换为XML结点对象
	
		noop:   一个空函数
	
		globalEval:   eval一段js代码，并将作用域放在全局
		
		camelCase: 转换为驼峰命名法
	
		nodeName: 检查DOM元素的节点名称与指定的值是否相等，不区分大小写
	
		each: 遍历一个数组或对象
	
		trim: 去掉一个字符串前后的空格
	
		makeArray: 将一个变量转换成数组
	
		inArray: 判断一个元素是否在一个数组中
	
		merge: 合并对象和数组
	
		grep: 
	
		map: 遍历一个数组或对象
	
		guid: 
	
		proxy: 

		access: 为集合中的元素设置一个或多个属性值，或者获取第一个元素的属性值

		now: 获取当前时间戳
	
		swap: 
	});



----------




下面分析几个经常使用的方法，多多熟悉熟悉jquery的写法和它的代码思想。

先看看type方法，该方法就是用来返回一个变量类型字符串的，相当于typeof的加强版。

	type: function( obj ) {
		//判断是否为null类型
		if ( obj == null ) {
			return String( obj );//如果是，则直接返回 'null' 字符串
		}
		//判断是不是一个对象，如果是对象
		//再通过object的tostring方法判断具体的对象类型。
		return typeof obj === "object" || typeof obj === "function" ?
			class2type[ core_toString.call(obj) ] || "object" : typeof obj;
	}


class2type[ core_toString.call(obj) ] 具体判断类型就是通过这个方法，可以看到844行。其原理主要是通过对象原型下的toString方法返回一个字符串，该字符串可以判定该对象的具体类型。

	jQuery.each("Boolean Number String Function Array Date RegExp Object Error".split(" "), function(i, name) {
		class2type[ "[object " + name + "]" ] = name.toLowerCase();
	});


其后的isFunction、isPlainObject都用到了type方法。

isArray方法在2.0版本的jquery中，因为不考虑ie8一下的兼容，直接使用了ECMA5中数组的原生方法来判断该变量是否为一个数组。

`isArray: Array.isArray`

isWindow只需要判断传入的对象是否有window属性等于本身就可以知道该对象是否为window对象。

	isWindow: function( obj ) {
		return obj != null && obj === obj.window;
	}

----------

下面看看camelCase方法，将传入的字符串转换成驼峰命名法

	var rmsPrefix = /^-ms-/,
		rdashAlpha = /-([\da-z])/gi,
		// Used by jQuery.camelCase as callback to replace()
		fcamelCase = function( all, letter ) {//该函数是camelCase最后replace的回调函数。
			//all表示rdashAlpha匹配到的字符串，letter表示第一个匹配子项
			//该函数的返回值表示将匹配字符串替换为返回的字符串，返回值将匹配子项转换成大写
			return letter.toUpperCase();
		}

	var camelCase = function( string ) {//转成驼峰命名法
		//在转换之前先把-ms-转换成了ms-
		return string.replace( rmsPrefix, "ms-" ).replace( rdashAlpha, fcamelCase );
	}

----------

noConflict该方法用来防止重名的冲突。因为jQuery变量和$变量是暴露在window下的，可能会有其他库使用该变量名。
在看该方法源码之前可以先看38--41行，可以看到jquery在加载之前先把全局变量中的jQuery变量和$变量存储到了_jQuery和_$中。

	_jQuery = window.jQuery,
	_$ = window.$,  
	

下面看看noConflict的源码，可以看到调用该方法后，jquery把$变量名重新交还给了jquery初始化之前的$变量，如果传入了deep为true，jquery也会把jQuery变量名交还出去。最后返回了jQuery变量，用户可以通过另一个自定义的变量名来接收jQuery变量。

	noConflict: function( deep ) {  //用来防止jquery的 $ 的命名冲突
		if ( window.$ === jQuery ) {
			window.$ = _$;  
		}

		if ( deep && window.jQuery === jQuery ) {
			window.jQuery = _jQuery;
		}

		return jQuery;
	}


----------

接下来看看使用比较多的工具方法数组遍历和映射，each和map。

**each：**
each方法可以接受三个值：
1.要遍历的数组或类数组变量；
2.为每个数组元素执行的回调函数，该函数传入两个值，当前索引和当前元素；
3.如果传入了最后一个值（数组），那么回调函数传入的参数为最后一个数组的元素。

	each: function( obj, callback, args ) {
		var value,
			i = 0,
			length = obj.length,//创建一个变量存储数组（或类数组对象）的长度
			isArray = isArraylike( obj );//判断传入的对象是否为一个数组（或类数组对象）
		//后面有两个if判断
		//判断是否传入了args参数，如果有，则通过apply调用回调函数传入args
		//判断是数组还是类数组对象，如果是数组使用for遍历数组，如果是类数组对象使用forin遍历对象
		if ( args ) {
			if ( isArray ) {
				for ( ; i < length; i++ ) {
					value = callback.apply( obj[ i ], args );

					if ( value === false ) {
						break;
					}
				}
			} else {
				for ( i in obj ) {
					value = callback.apply( obj[ i ], args );

					if ( value === false ) {
						break;
					}
				}
			}

		// A special, fast, case for the most common use of each
		} else {
			if ( isArray ) {
				for ( ; i < length; i++ ) {
					value = callback.call( obj[ i ], i, obj[ i ] );

					if ( value === false ) {
						break;
					}
				}
			} else {
				for ( i in obj ) {
					value = callback.call( obj[ i ], i, obj[ i ] );

					if ( value === false ) {
						break;
					}
				}
			}
		}

		return obj;
	}



**map：**
map方法是对一个数组进行映射，并返回一个新的数组。使用时传入三个参数：
1.要遍历的数组或类数组对象；
2.为该数组的每个元素执行的回调函数且返回值会插入的新数组，该函数接收三个值，当前元素、当前索引值、map方法传入的第三个值；
3.给回调函数使用的值。


	map: function( elems, callback, arg ) {
		var value,
			i = 0,
			length = elems.length,
			isArray = isArraylike( elems ),
			ret = [];

		// 判断当前对象是否为数组，如果是数组使用for来遍历数组元素，如果对象使用forin遍历对象的所有属性。
		
		if ( isArray ) {
			for ( ; i < length; i++ ) {
				//接受回调函数的返回值。
				value = callback( elems[ i ], i, arg );
				//如果对调函数的返回值不为空就将其存入到新数组。
				if ( value != null ) {
					ret[ ret.length ] = value;
				}
			}
		} else {
			for ( i in elems ) {
				value = callback( elems[ i ], i, arg );

				if ( value != null ) {
					ret[ ret.length ] = value;
				}
			}
		}

		//最后将映射过的新数组返回
		return core_concat.apply( [], ret );
	}

----------

merge方法是用来将两个数组或者对象进行合并。

	merge: function( first, second ) {
		var l = second.length,
			i = first.length,
			j = 0;

		if ( typeof l === "number" ) {
			for ( ; j < l; j++ ) {
				first[ i++ ] = second[ j ];
			}
		} else {
			while ( second[j] !== undefined ) {
				first[ i++ ] = second[ j++ ];
			}
		}

		first.length = i;

		return first;
	}