---
title: 学习jquery源码-5 （jquery-2.0.3）（不兼容ie6,7,8）
comments: true
toc: true
date: 2018-05-27 15:29:14
categories: jquery
tags: jquery
---
## jquery代码349-817行 扩展一些工具方法（静态方法和静态属性）
使用之前$.extend方法来扩展方法，供jquery内部调用方便

首先列举以下属性和方法
```
jQuery.extend({
    expando: 生成唯一JQ字符串（内部）,
    noConfict(): 防止冲突,
    isReady： DOM是否价值完（内部）
    readyWait: 等待多时文件的计数器（内部）
    holdReady(): 推迟DOM触发
    isFunction(): 是否为函数
    isArray(): 是否为数组
    isWindow(): 是否为window
    isNumeric: 是否欸数字
    type() : 判断数据类型
    isPlainObject():  是否为对象自变量
    isEmptyObject(): 是否为空的对象
    error(): 抛出异常
    parseHTML(): 解析节点
    parseJSON(): 解析JSON
    noop(): 空函数
    gl
})
```
接下来是源码 正在研读中----
```
jQuery.extend({
	// 唯一jQuery字符串
	expando: "jQuery" + ( core_version + Math.random() ).replace( /\D/g, "" ),
	// 如果$或jquery被使用了，那么需要使用该函数来更名，防止和其他冲突
	noConflict: function( deep ) {
		if ( window.$ === jQuery ) {
			window.$ = _$; // 针对$一开始被复制 ，现在还原
		}

		if ( deep && window.jQuery === jQuery ) {
			window.jQuery = _jQuery; // 针对$一开始被复制 ，现在还原,并且用参数true来控制
		}

		return jQuery;
	},

	// DOM是否加载完成标志位
	isReady: false,

	// 等待次数
	readyWait: 1,

	// 推迟reday触发 $.holdReady(true) 推迟， $.holdReady(false)再触发 
	// 使用场景， ajax异步加载数据有先后关系时，可以使用
	holdReady: function( hold ) {
		if ( hold ) {
			jQuery.readyWait++;
		} else {
			jQuery.ready( true );
		}
	},

	// Handle when the DOM is ready
	ready: function( wait ) {

		// 第一次不进入这边，第二次开始进入，判断是否处于reday等待
		if ( wait === true ? --jQuery.readyWait : jQuery.isReady ) {
			return;
		}

		// dom加载完毕
		jQuery.isReady = true;

		// 第一次进入后返回，如果有ready等待
		if ( wait !== true && --jQuery.readyWait > 0 ) {
			return;
		}

		// 解析ready函数 $(function() {})
		readyList.resolveWith( document, [ jQuery ] );

		// 这种判断是否有reday事件，有就触发：$(doc).on('reday' ,func(){})
		if ( jQuery.fn.trigger ) {
			jQuery( document ).trigger("ready").off("ready");
		}
	},

	// 判断是否为函数
	isFunction: function( obj ) {
		return jQuery.type(obj) === "function";
	},
	// 判断是否为数组
	isArray: Array.isArray,
	// 判断是否为window
	isWindow: function( obj ) {
		return obj != null && obj === obj.window;
	},
	// 判断是否为数字
	isNumeric: function( obj ) {
		// 判断是否为NAN  和是否为有限数字
		return !isNaN( parseFloat(obj) ) && isFinite( obj );
	},
	// 判断数据类型
	type: function( obj ) {
		if ( obj == null ) {
			return String( obj );
		}
		// typeof 无法区分对象的详细类型
		return typeof obj === "object" || typeof obj === "function" ?
			class2type[ core_toString.call(obj) ] || "object" :
			typeof obj;
	},
    //  是否为对象自变量:json 或new对象
	isPlainObject: function( obj ) {
		// 如果不是对象或者节点，window直接排除（节点和window也返回object）
		if ( jQuery.type( obj ) !== "object" || obj.nodeType || jQuery.isWindow( obj ) ) {
			return false;
		}

		try {
			// 判断是否有构造函数，以及判断原型链上的方法是否有
			if ( obj.constructor &&
					!core_hasOwn.call( obj.constructor.prototype, "isPrototypeOf" ) ) {
				return false;
			}
		} catch ( e ) {
			return false;
		}

		return true;
	},
	// 是否为空
	isEmptyObject: function( obj ) {
		var name;
		// 通过遍历自身参数
		for ( name in obj ) {
			return false;
		}
		return true;
	},
	// 抛出错误
	error: function( msg ) {
		throw new Error( msg );
	},

	// data: string of html
	// context (optional): If specified, the fragment will be created in this context, defaults to document
	// keepScripts (optional): If true, will include scripts passed in the html string
	parseHTML: function( data, context, keepScripts ) {
		if ( !data || typeof data !== "string" ) {
			return null;
		}
		if ( typeof context === "boolean" ) {
			keepScripts = context;
			context = false;
		}
		context = context || document;

		var parsed = rsingleTag.exec( data ),
			scripts = !keepScripts && [];

		// Single tag
		if ( parsed ) {
			return [ context.createElement( parsed[1] ) ];
		}

		parsed = jQuery.buildFragment( [ data ], context, scripts );

		if ( scripts ) {
			jQuery( scripts ).remove();
		}

		return jQuery.merge( [], parsed.childNodes );
	},

	parseJSON: JSON.parse,

	// Cross-browser xml parsing
	parseXML: function( data ) {
		var xml, tmp;
		if ( !data || typeof data !== "string" ) {
			return null;
		}

		// Support: IE9
		try {
			tmp = new DOMParser();
			xml = tmp.parseFromString( data , "text/xml" );
		} catch ( e ) {
			xml = undefined;
		}

		if ( !xml || xml.getElementsByTagName( "parsererror" ).length ) {
			jQuery.error( "Invalid XML: " + data );
		}
		return xml;
	},

	noop: function() {},

	// Evaluates a script in a global context
	globalEval: function( code ) {
		var script,
				indirect = eval;

		code = jQuery.trim( code );

		if ( code ) {
			// If the code includes a valid, prologue position
			// strict mode pragma, execute code by injecting a
			// script tag into the document.
			if ( code.indexOf("use strict") === 1 ) {
				script = document.createElement("script");
				script.text = code;
				document.head.appendChild( script ).parentNode.removeChild( script );
			} else {
			// Otherwise, avoid the DOM node creation, insertion
			// and removal by using an indirect global eval
				indirect( code );
			}
		}
	},

	// Convert dashed to camelCase; used by the css and data modules
	// Microsoft forgot to hump their vendor prefix (#9572)
	camelCase: function( string ) {
		return string.replace( rmsPrefix, "ms-" ).replace( rdashAlpha, fcamelCase );
	},

	nodeName: function( elem, name ) {
		return elem.nodeName && elem.nodeName.toLowerCase() === name.toLowerCase();
	},

	// args is for internal usage only
	each: function( obj, callback, args ) {
		var value,
			i = 0,
			length = obj.length,
			isArray = isArraylike( obj );

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
	},

	trim: function( text ) {
		return text == null ? "" : core_trim.call( text );
	},

	// results is for internal usage only
	makeArray: function( arr, results ) {
		var ret = results || [];

		if ( arr != null ) {
			if ( isArraylike( Object(arr) ) ) {
				jQuery.merge( ret,
					typeof arr === "string" ?
					[ arr ] : arr
				);
			} else {
				core_push.call( ret, arr );
			}
		}

		return ret;
	},

	inArray: function( elem, arr, i ) {
		return arr == null ? -1 : core_indexOf.call( arr, elem, i );
	},

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
	},

	grep: function( elems, callback, inv ) {
		var retVal,
			ret = [],
			i = 0,
			length = elems.length;
		inv = !!inv;

		// Go through the array, only saving the items
		// that pass the validator function
		for ( ; i < length; i++ ) {
			retVal = !!callback( elems[ i ], i );
			if ( inv !== retVal ) {
				ret.push( elems[ i ] );
			}
		}

		return ret;
	},

	// arg is for internal usage only
	map: function( elems, callback, arg ) {
		var value,
			i = 0,
			length = elems.length,
			isArray = isArraylike( elems ),
			ret = [];

		// Go through the array, translating each of the items to their
		if ( isArray ) {
			for ( ; i < length; i++ ) {
				value = callback( elems[ i ], i, arg );

				if ( value != null ) {
					ret[ ret.length ] = value;
				}
			}

		// Go through every key on the object,
		} else {
			for ( i in elems ) {
				value = callback( elems[ i ], i, arg );

				if ( value != null ) {
					ret[ ret.length ] = value;
				}
			}
		}

		// Flatten any nested arrays
		return core_concat.apply( [], ret );
	},

	// A global GUID counter for objects
	guid: 1,

	// Bind a function to a context, optionally partially applying any
	// arguments.
	proxy: function( fn, context ) {
		var tmp, args, proxy;

		if ( typeof context === "string" ) {
			tmp = fn[ context ];
			context = fn;
			fn = tmp;
		}

		// Quick check to determine if target is callable, in the spec
		// this throws a TypeError, but we will just return undefined.
		if ( !jQuery.isFunction( fn ) ) {
			return undefined;
		}

		// Simulated bind
		args = core_slice.call( arguments, 2 );
		proxy = function() {
			return fn.apply( context || this, args.concat( core_slice.call( arguments ) ) );
		};

		// Set the guid of unique handler to the same of original handler, so it can be removed
		proxy.guid = fn.guid = fn.guid || jQuery.guid++;

		return proxy;
	},

	// Multifunctional method to get and set values of a collection
	// The value/s can optionally be executed if it's a function
	access: function( elems, fn, key, value, chainable, emptyGet, raw ) {
		var i = 0,
			length = elems.length,
			bulk = key == null;

		// Sets many values
		if ( jQuery.type( key ) === "object" ) {
			chainable = true;
			for ( i in key ) {
				jQuery.access( elems, fn, i, key[i], true, emptyGet, raw );
			}

		// Sets one value
		} else if ( value !== undefined ) {
			chainable = true;

			if ( !jQuery.isFunction( value ) ) {
				raw = true;
			}

			if ( bulk ) {
				// Bulk operations run against the entire set
				if ( raw ) {
					fn.call( elems, value );
					fn = null;

				// ...except when executing function values
				} else {
					bulk = fn;
					fn = function( elem, key, value ) {
						return bulk.call( jQuery( elem ), value );
					};
				}
			}

			if ( fn ) {
				for ( ; i < length; i++ ) {
					fn( elems[i], key, raw ? value : value.call( elems[i], i, fn( elems[i], key ) ) );
				}
			}
		}

		return chainable ?
			elems :

			// Gets
			bulk ?
				fn.call( elems ) :
				length ? fn( elems[0], key ) : emptyGet;
	},

	now: Date.now,

	// A method for quickly swapping in/out CSS properties to get correct calculations.
	// Note: this method belongs to the css module but it's needed here for the support module.
	// If support gets modularized, this method should be moved back to the css module.
	swap: function( elem, options, callback, args ) {
		var ret, name,
			old = {};

		// Remember the old values, and insert the new ones
		for ( name in options ) {
			old[ name ] = elem.style[ name ];
			elem.style[ name ] = options[ name ];
		}

		ret = callback.apply( elem, args || [] );

		// Revert the old values
		for ( name in options ) {
			elem.style[ name ] = old[ name ];
		}

		return ret;
	}
});
```
