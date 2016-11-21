作为一个前端开发者，很多时候我们要实现多浏览器的兼容，这一关往往是一个坎。早期的web开发者需要考虑各种ie567与w3c标准之间的兼容，好在现在这些低版本的浏览器已经逐渐淘汰，随着html5与css3的发展，浏览器的标准逐渐统一，而且从jQuery2.0之后的版本就不再考虑ie8以下（包括ie8）低版本ie的兼容，只做现代浏览器标准。

而jQuery是一个做到多浏览器兼容的js库，它通过下面这个support做支撑，来判断浏览器是否支持一些操作，因为2.0以上的版本为了代码量与性能，舍弃了低版本ie的兼容，所以support对象也减少了很多判断。我们可以看看1.7与2.0两个版本之间有多大区别。


![jquery1.7-support](http://i.imgur.com/TGpbTMY.png)

![jquery2.0-support](http://i.imgur.com/kFJzjyT.png)


当然我们要做新时代的前端工程师，所以我们看看2.0的support对象就可以了，未来的前端更多应该是实现不同分辨率手机屏幕的适配与ios和android浏览器直接的兼容，而不是还把时间用在如何兼容低版本ie上，那样的时代已经过去，应该好好感谢w3c组织的努力和谷歌浏览器的飞速发展，哈哈哈哈。

	Object{
			ajax:true,  检测能否创建ajax对象
			boxSizing:true,如果css支持box-sizing属性，则返回true
			boxSizingReliable:true,
			checkClone:true,检测浏览器能否正确克隆文档片段中的复选框或单选按钮的状态
			checkOn:true,检测复选框的默认值是否为 on
			clearCloneStyle:true,如果被克隆节点的样式改变，源节点样式也会改变，返回true
			cors:true,如果浏览器能创建 XMLHttpRequest 对象，并且该 XMLHttpRequest 对象含有 withCredentials 属性的话，则返回 true。主要是跨域检测
			focusinBubbles:false,  如果支持focusin事件被支持，返回true
			noCloneChecked:true, 如果克隆后的 DOM 元素保持了 .checked 状态，则返回 true。
			optDisabled:true,如果含有被禁用的 option 元素的 select 元素没有被自动禁用的话，则返回 true。
			optSelected:true,如果被默认选中的 <option> 元素是通过 selected 属性被选中的，则返回 true。
			pixelPosition:true,
			radioValue:true,
			reliableMarginRight:true
	}


	

----------

其实jq的support很少在开发时用到，因为jq提供给我们的方法都做了兼容，我们在使用jq的时候很少会做浏览器兼容，大部分都只在jq内部使用。其实jq做的这些兼容都是小方面，都不是大方面的，而不是常见的改变样式表、添加事件这样的大方向的兼容，所以这一部分源码不会看得那么详细。

	jQuery.support = (function( support ) {
	var input = document.createElement("input"),
		fragment = document.createDocumentFragment(),
		div = document.createElement("div"),
		select = document.createElement("select"),
		opt = select.appendChild( document.createElement("option") );

	// Finish early in limited environments
	if ( !input.type ) {
		return support;
	}

	input.type = "checkbox";

	// Support: Safari 5.1, iOS 5.1, Android 4.x, Android 2.3
	// Check the default checkbox/radio value ("" on old WebKit; "on" elsewhere)
	support.checkOn = input.value !== "";

	// Must access the parent to make an option select properly
	// Support: IE9, IE10
	support.optSelected = opt.selected;

	// Will be defined later
	support.reliableMarginRight = true;
	support.boxSizingReliable = true;
	support.pixelPosition = false;

	// Make sure checked status is properly cloned
	// Support: IE9, IE10
	input.checked = true;
	support.noCloneChecked = input.cloneNode( true ).checked;

	// Make sure that the options inside disabled selects aren't marked as disabled
	// (WebKit marks them as disabled)
	select.disabled = true;
	support.optDisabled = !opt.disabled;

	// Check if an input maintains its value after becoming a radio
	// Support: IE9, IE10
	input = document.createElement("input");
	input.value = "t";
	input.type = "radio";
	support.radioValue = input.value === "t";

	// #11217 - WebKit loses check when the name is after the checked attribute
	input.setAttribute( "checked", "t" );
	input.setAttribute( "name", "t" );

	fragment.appendChild( input );

	// Support: Safari 5.1, Android 4.x, Android 2.3
	// old WebKit doesn't clone checked state correctly in fragments
	support.checkClone = fragment.cloneNode( true ).cloneNode( true ).lastChild.checked;

	// Support: Firefox, Chrome, Safari
	// Beware of CSP restrictions (https://developer.mozilla.org/en/Security/CSP)
	support.focusinBubbles = "onfocusin" in window;

	div.style.backgroundClip = "content-box";
	div.cloneNode( true ).style.backgroundClip = "";
	support.clearCloneStyle = div.style.backgroundClip === "content-box";

	// Run tests that need a body at doc read
	jQuery(function() {
		var container, marginDiv,
			// Support: Firefox, Android 2.3 (Prefixed box-sizing versions).
			divReset = "padding:0;margin:0;border:0;display:block;-webkit-box-sizing:content-box;-moz-box-sizing:content-box;box-sizing:content-box",
			body = document.getElementsByTagName("body")[ 0 ];

		if ( !body ) {
			// Return for frameset docs that don't have a body
			return;
		}

		container = document.createElement("div");
		container.style.cssText = "border:0;width:0;height:0;position:absolute;top:0;left:-9999px;margin-top:1px";

		// Check box-sizing and margin behavior.
		body.appendChild( container ).appendChild( div );
		div.innerHTML = "";
		// Support: Firefox, Android 2.3 (Prefixed box-sizing versions).
		div.style.cssText = "-webkit-box-sizing:border-box;-moz-box-sizing:border-box;box-sizing:border-box;padding:1px;border:1px;display:block;width:4px;margin-top:1%;position:absolute;top:1%";

		// Workaround failing boxSizing test due to offsetWidth returning wrong value
		// with some non-1 values of body zoom, ticket #13543
		jQuery.swap( body, body.style.zoom != null ? { zoom: 1 } : {}, function() {
			support.boxSizing = div.offsetWidth === 4;
		});

		// Use window.getComputedStyle because jsdom on node.js will break without it.
		if ( window.getComputedStyle ) {
			support.pixelPosition = ( window.getComputedStyle( div, null ) || {} ).top !== "1%";
			support.boxSizingReliable = ( window.getComputedStyle( div, null ) || { width: "4px" } ).width === "4px";

			// Support: Android 2.3
			// Check if div with explicit width and no margin-right incorrectly
			// gets computed margin-right based on width of container. (#3333)
			// WebKit Bug 13343 - getComputedStyle returns wrong value for margin-right
			marginDiv = div.appendChild( document.createElement("div") );
			marginDiv.style.cssText = div.style.cssText = divReset;
			marginDiv.style.marginRight = marginDiv.style.width = "0";
			div.style.width = "1px";

			support.reliableMarginRight =
				!parseFloat( ( window.getComputedStyle( marginDiv, null ) || {} ).marginRight );
		}

		body.removeChild( container );
	});

	return support;
	})( {} );




	jQuery.support.cors = !!xhrSupported && ( "withCredentials" in xhrSupported );
	jQuery.support.ajax = xhrSupported = !!xhrSupported;