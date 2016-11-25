
首先我们要知道为什么jQuery为什么会存在Data这个对象，主要是用来干嘛的呢？
这个对象主要是用来做数据的缓存的



下面看看这个对象的具体构造

	function Data(){}  //构造函数

	Data.accepts = function(){	}

	Data.prototype = {
		key: function(){},
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
		//这两个方法现在就是调用data_priv下的方法，与data和removeData没有区别，现在可以弃用
		_data: function(){},
		_removeData: function(){}
	});
	
	jQuery.fn.extend({
		data: function(){},
		hasData: function(){}
	});