这对元素节点的操作一共有2部分：

1. 对节点属性的操作
2. 对节点class的操作

前面已经介绍了节点属性的操作，现在我们再看看元素节点class名的操作。

jQuery中对class名的操作（主要是class名而不是对style的操作）一共提供了四个方法：

1. addClass()  添加类名
	- addClass(str)：  通过字符串的方式添加类名，多个类名用空格隔开
	- addClass（function(){}）; 通过传入回调函数的返回值来添加类名,回调函数接受2个参数(当前节点索引、节点类名)，this指向节点
2. removeClass()  删除类名
	- removeClass(str)  通过传入字符串的方式删除类名，多个类名用空格隔开
	- removeClass（function(){}）; 通过传入回调函数的返回值来删除类名,回调函数接受2个参数(当前节点索引、节点类名)，this指向节点
3. hasClass()   类名是否存在
	- hasClass(str)  通过传入字符串的方式判断类名是否存在，只能判断一个类名
4. toggleClass()  添加或删除类名
	- toggleClass(boolean) 传入一个布尔值，如果是false则删除节点上的所有类名，并通过data缓存，如果是true则把之前缓存的类名重新添加到节点上
	- toggleClass(str) 传入一个字符串，如果该类名存在就删除，不存在就添加
	- toggleClass(str，boolean) 判断传入的boolean，如果是true调用add方法，如果是false调用remove方法
	- toggleClass（function(){},boolean）; 通过传入回调函数的返回值来添加或删除类名,回调函数接受3个参数(当前节点索引、节点类名、布尔值)，this指向节点

这三个方法都是直接扩展在jQuery的实例方法中，因此也比较简单，没有像之前的实例方法一样先实现了内部的同名静态方法，再通过调用静态方法来扩展实例方法。所以我们直接看源码：

    
	jQuery.fn.extend({
		addClass: function( value ) {
			var classes, elem, cur, clazz, j,
				i = 0,
				len = this.length,
				proceed = typeof value === "string" && value;
	
			if ( jQuery.isFunction( value ) ) {  //判断要添加的class是不是函数
				return this.each(function( j ) {  //支持通过回调函数返回值的方式添加class
					jQuery( this ).addClass( value.call( this, j, this.className ) );  //回调传入两个值：当前节点index和当前节点的class名
				});
			}
	
			if ( proceed ) { //只有传入的value是字符串时才添加到class
				classes = ( value || "" ).match( core_rnotwhite ) || [];  
				//通过匹配非空格字符的方式将字符串分割为数组
	
				for ( ; i < len; i++ ) { //对节点进行遍历
					elem = this[ i ];
					cur = elem.nodeType === 1 && ( elem.className ?  //只有节点是元素节点才能添加class，并给元素节点的class前后添加空格
						( " " + elem.className + " " ).replace( rclass, " " ) :   // rclass = /[\t\r\n\f]/g  将换行换页tab等空白字符替换成空格
						" "//当前节点没有class则返回一个空格字符，注意不是空字符
					);
	
					if ( cur ) { //cur不存在表示当前节点不是元素节点
						j = 0;
						while ( (clazz = classes[j++]) ) { //遍历传入的class
							if ( cur.indexOf( " " + clazz + " " ) < 0 ) { //只有class不存在当前节点才添加，避免重复添加
								cur += clazz + " ";
							}
						}
						elem.className = jQuery.trim( cur );  //去掉前后空格
	
					}
				}
			}
	
			return this;
		},
	
		removeClass: function( value ) {
			var classes, elem, cur, clazz, j,
				i = 0,
				len = this.length,
				proceed = arguments.length === 0 || typeof value === "string" && value;
	
			if ( jQuery.isFunction( value ) ) {
				return this.each(function( j ) { //同样支持回调函数返回值的方式移除class
					jQuery( this ).removeClass( value.call( this, j, this.className ) );
				});
			}
			if ( proceed ) {
				classes = ( value || "" ).match( core_rnotwhite ) || [];
				for ( ; i < len; i++ ) {
					elem = this[ i ];
					cur = elem.nodeType === 1 && ( elem.className ?
						( " " + elem.className + " " ).replace( rclass, " " ) :
						""
					);

					if ( cur ) {
						j = 0;
						while ( (clazz = classes[j++]) ) {
							// Remove *all* instances
							while ( cur.indexOf( " " + clazz + " " ) >= 0 ) {  //使用while的方式移除所有名为clazz的class名，避免一个节点上有多个同名class
								cur = cur.replace( " " + clazz + " ", " " );
							}
						}
						elem.className = value ? jQuery.trim( cur ) : "";  //如果value没有传入，清空class
					}
				}
			}
	
			return this;
		},
	
		toggleClass: function( value, stateVal ) {
			var type = typeof value;
	
			if ( typeof stateVal === "boolean" && type === "string" ) {  //通过传入stateVal的方式，判断是添加class还是删除class
				return stateVal ? this.addClass( value ) : this.removeClass( value );
			}
	
			if ( jQuery.isFunction( value ) ) { //同样支持回调函数返回值的方式添加和删除class
				return this.each(function( i ) {
					jQuery( this ).toggleClass( value.call(this, i, this.className, stateVal), stateVal );
				});
			}
	
			return this.each(function() {
				if ( type === "string" ) {
					// toggle individual class names
					var className,
						i = 0,
						self = jQuery( this ),  
						classNames = value.match( core_rnotwhite ) || [];
	
					while ( (className = classNames[ i++ ]) ) { //toggle的基本思想就是有该class就移除，没有就添加
						// check each className given, space separated list
						if ( self.hasClass( className ) ) { //通过hasClass进行判断
							self.removeClass( className );
						} else {
							self.addClass( className );
						}
					}

				} else if ( type === core_strundefined || type === "boolean" ) {  //如果没有传入class名，或者只传入了一个boolean值
					if ( this.className ) {  //移除所有的class，并通过data进行缓存，当传入true时再把之前缓存的class重新设置到节点
						data_priv.set( this, "__className__", this.className );
					}
					this.className = this.className || value === false ? "" : data_priv.get( this, "__className__" ) || "";
				}
			});
		},
	
		hasClass: function( selector ) {
			var className = " " + selector + " ",
				i = 0,
				l = this.length;
			for ( ; i < l; i++ ) {  //遍历节点，然后判断节点是否存在传入class(通过indexOf判断)，存在返回true，否则返回false
				if ( this[i].nodeType === 1 && (" " + this[i].className + " ").replace(rclass, " ").indexOf( className ) >= 0 ) {
					return true;
				}
			}
			return false;
		},
	});


这一部分jQuery还扩展了一个val方法，专门用来获取一些表单元素的value属性的值。这个方法有几个hooks，想让我们来看看这几个hooks是如何解决不同浏览器兼容的：


	valHooks: {
		option: {
			get: function( elem ) {
				// attributes.value is undefined in Blackberry 4.7 but
				// uses .value. See #6932
				var val = elem.attributes.value;
				return !val || val.specified ? elem.value : elem.text;
			}
		},
		select: {
			get: function( elem ) {
				var value, option,
					options = elem.options,
					index = elem.selectedIndex,
					one = elem.type === "select-one" || index < 0,
					values = one ? null : [],
					max = one ? index + 1 : options.length,
					i = index < 0 ?
						max :
						one ? index : 0;

				// Loop through all the selected options
				for ( ; i < max; i++ ) {
					option = options[ i ];

					// IE6-9 doesn't update selected after form reset (#2551)
					if ( ( option.selected || i === index ) &&
							// Don't return options that are disabled or in a disabled optgroup
							( jQuery.support.optDisabled ? !option.disabled : option.getAttribute("disabled") === null ) &&
							( !option.parentNode.disabled || !jQuery.nodeName( option.parentNode, "optgroup" ) ) ) {

						// Get the specific value for the option
						value = jQuery( option ).val();

						// We don't need an array for one selects
						if ( one ) {
							return value;
						}

						// Multi-Selects return an array
						values.push( value );
					}
				}

				return values;
			},

			set: function( elem, value ) {
				var optionSet, option,
					options = elem.options,
					values = jQuery.makeArray( value ),
					i = options.length;

				while ( i-- ) {
					option = options[ i ];
					if ( (option.selected = jQuery.inArray( jQuery(option).val(), values ) >= 0) ) {
						optionSet = true;
					}
				}

				// force browsers to behave consistently when non-matching value is set
				if ( !optionSet ) {
					elem.selectedIndex = -1;
				}
				return values;
			}
		}
	}

	

	jQuery.each([ "radio", "checkbox" ], function() {
		jQuery.valHooks[ this ] = {
			set: function( elem, value ) {
				if ( jQuery.isArray( value ) ) {
					return ( elem.checked = jQuery.inArray( jQuery(elem).val(), value ) >= 0 );
				}
			}
		};
		if ( !jQuery.support.checkOn ) {  //检查选择框的默认值是否为on
			jQuery.valHooks[ this ].get = function( elem ) {
				// 用来支持低版本的webkit浏览器，选择框的默认值为空
				return elem.getAttribute("value") === null ? "on" : elem.value;
			};
		}
	});














----------










	val: function( value ) {
		var hooks, ret, isFunction,
			elem = this[0];

		if ( !arguments.length ) { //如果没有传入参数（通过判断arguments长度的方式）
			if ( elem ) {
				hooks = jQuery.valHooks[ elem.type ] || jQuery.valHooks[ elem.nodeName.toLowerCase() ];

				if ( hooks && "get" in hooks && (ret = hooks.get( elem, "value" )) !== undefined ) {
					return ret;
				}

				ret = elem.value;

				return typeof ret === "string" ?
					// handle most common string cases
					ret.replace(rreturn, "") :
					// handle cases where value is null/undef or number
					ret == null ? "" : ret;
			}

			return;
		}

		isFunction = jQuery.isFunction( value );

		return this.each(function( i ) {
			var val;

			if ( this.nodeType !== 1 ) {
				return;
			}

			if ( isFunction ) {
				val = value.call( this, i, jQuery( this ).val() );
			} else {
				val = value;
			}

			// Treat null/undefined as ""; convert numbers to string
			if ( val == null ) {
				val = "";
			} else if ( typeof val === "number" ) {
				val += "";
			} else if ( jQuery.isArray( val ) ) {
				val = jQuery.map(val, function ( value ) {
					return value == null ? "" : value + "";
				});
			}

			hooks = jQuery.valHooks[ this.type ] || jQuery.valHooks[ this.nodeName.toLowerCase() ];

			// If set returns undefined, fall back to normal setting
			if ( !hooks || !("set" in hooks) || hooks.set( this, val, "value" ) === undefined ) {
				this.value = val;
			}
		});
	}