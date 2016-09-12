我们先看看jQuery的原型中初始化了哪些属性和方法：

    jQuery.fn = jQuery.prototype = {
    	jquery: core_version,  //jquery版本号
    
    	constructor: jQuery,   //构造器指向
    
    	init：  //jquery的入口函数，主要用来实现选择器和DOM节点的创建
    
    	selector： //将选择器进行存储
    
    	length:  //当前选择器存储的DOM节点的个数
    
    	toArray:  //通过方法借调的方式，把一个类数组对象转换为一个数组
    			  //类数组对象就是指有数字作为属性，且有length属性，jQuery是一个类数组对象，arguments也是。
    	get: //获取jQuery对象中的某一个DOM节点，返回的是一个DOM节点，
    
    	pushStack: //将一个DOM元素集合加入到jQuery对象的prevObject中。
				   //this.prevObject=this,让当前DOM集合存储到prevObject属性中，方便end()调用是回溯。
    
    	each: //对数组进行遍历
    
    	ready: //当DOM树加载完毕后，回调该函数
    
    	slice:  //类似于toArray方法，只是该方法会进行一次pushStack操作
    
    	first:  //返回第一个元素的jQuery对象
    
    	last:  //返回最后一个元素的jQuery对象
    
    	eq:  //传入一个数字num，获取第num个元素的jQuery对象
    
    	map:   //map将一个数组中的元素转换到另一个数组中，可以传入一个回调函数，作用与each类似，只是map会返回一个新的数组，而each不会
    
    	end:   //返回调用parent()、find()、filter()等方法之前的jQuery对象，就是回溯到上一个DOM合集
    
    	push: core_push,  //存储了数组的push方法
    	sort: [].sort,   //存储了数组的sort方法
    	splice: [].splice  //存储了数组的splice方法
    ｝

上面是对jQuery初始化的一些方法和属性的介绍，这里主要看下init是如何实现，其余的方法在具体用到的时候再看。

首先可以观察到init方法传入了三个参数：
1. selector(选择器)
2. context(上下文环境)
3. rootJQuery( $(document) ) 参见886行

init对传入的选择器进行了一下的区分：

1. 空 ： 包括 '' false undefined null

	return false;

2. string：这个内容有点多，会在后面详细的介绍。<br/><br/>
	
3. DOM节点：修改jQuery对象的属性 0：selector，length：1  ；这就相当于把jQuery对象转成了一个类数组，最后返回this，可用于链式调用。

    this.context = this[0] = selector;
    this.length = 1;
    return this;
    
4. Function：  $(fn) 就相当于 $(document).ready(fn)
    
	return rootjQuery.ready( selector );

5. jQuery对象：

	this.selector = selector.selector;
	this.context = selector.context;
	return jQuery.makeArray( selector, this );

6. 其他任意类型的值：将传入的数据转换成一个数组

	return jQuery.makeArray( selector, this );



----------

下面我们来看当 selector 为一个字符串时是如何进行处理的：

    if ( selector.charAt(0) === "<" && selector.charAt( selector.length - 1 ) === ">" && selector.length >= 3 ) {//匹配单标签
    	match = [ null, selector, null ];
    } else {
    	match = rquickExpr.exec( selector );
    }
    if ( match && (match[1] || !context) ) {   //如果是字符串是html标签
    	if ( match[1] ) { 
    		jQuery.merge( this,  jQuery.parseHTML(match[1])   );
			//先通过parseHTML将字符串转换成DOM对象的数组，然后通过merge将DOM对象的数组合并到this上。
    		if ( rsingleTag.test( match[1] ) && jQuery.isPlainObject( context ) ) {
    			如果传入的是单标签，且第二个参数是一个纯对象,例如：
    			$('<div>',{'id':'box', 'class':'red'})//则把后面对象的属性一一添加到创建的这个DOM节点的属性上。
				//现在知道了通过$不仅可以创建标签，还能在创建单标签的时候直接在后面写属性。
    		}
    	} else { //如果是'#id'，直接通过js的原生方法getElementById获取DOM节点
    		elem = document.getElementById( match[2] );
    	}
	
	//后面的情况都是通过find方法来找复杂的选择器。例如selector是一个'.ClassName'
	//具体过程为jQuery.fn.find->jQuery.find->Sizzle
	//最后会调用Sizzle方法，这是jQuery选择器的核心方法，是一个独立的引擎，等到了后面我看懂了再告诉大家吧 （逃
    } else if ( !context || context.jquery ) {
    	return ( context || rootjQuery ).find( selector );
    } else {
    	return this.constructor( context ).find( selector );
    }

看这段代码前我们要先弄懂match到底是什么东西

	rquickExpr = /^(?:\s*(<[\w\W]+>)[^>]*|#([\w-]*))$/; （75行）
	match = rquickExpr.exec( selector ); (116行)

exec方法用于正则匹配，返回一个数组，第一个元素是匹配到的字符串，后面的元素为匹配子项，[具体用法](http://www.w3school.com.cn/jsref/jsref_exec_regexp.asp)。


举个栗子：

	var rquickExpr = /^(?:\s*(<[\w\W]+>)[^>]*|#([\w-]*))$/;

	console.log( match = rquickExpr.exec( '<ul><li></li></ul>' )  );
	// ["<ul><li></li></ul>", "<ul><li></li></ul>", undefined]

	console.log( match = rquickExpr.exec( '#id' )  );
	// ["#id", undefined, "id"]


现在我们知道了，match其实就是存储了字符串含义的数组，不得不感叹这是人想出来的吗。

- 如果match[1]存在就代表是html标签
- 如果match[2]存在就代表是id名


可以看到我们平时调用 $() 的时候很爽，又能当选择器又能创节点，但是不知道这后面的实现方法原来这么复杂，还真是用起来越方便的东西实现越复杂。