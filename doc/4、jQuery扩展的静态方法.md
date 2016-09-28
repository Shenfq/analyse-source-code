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
	
		nodeName: 
	
		each: 遍历一个数组或对象
	
		trim: 去掉一个字符串前后的空格
	
		makeArray: 将一个变量转换成数组
	
		inArray: 
	
		merge: 合并对象和数组
	
		grep: 
	
		map: 遍历一个数组或对象
	
		guid: 
	
		proxy: 

		access: 

		now: 
	
		swap: 
	});


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


class2type[ core_toString.call(obj) ] 具体判断类型就是通过这个方法，可以看到844行。

	jQuery.each("Boolean Number String Function Array Date RegExp Object Error".split(" "), function(i, name) {
		class2type[ "[object " + name + "]" ] = name.toLowerCase();
	});