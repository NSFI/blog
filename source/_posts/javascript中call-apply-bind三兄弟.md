---
title: 'javascript中call,apply,bind三兄弟'
date: 2018-06-25 16:58:31
tags:
    - giflee
    - javascript
author: giflee
---

# javascript中call,apply,bind三兄弟
call，apply，bind这三兄弟在日常开发中并不少见，尤其是bind，简直是回调函数绑定this的神器有没有！
### 兄弟从哪里来
无论是js中原生的函数还是自定义的函数都是可以直接调用这三个兄弟的，为什么可以这么方便，这就不得不得益于javascript的原型大法了。

新建一个Function的实例func，可以在发现console面板里发现call，apply和bind的影子。



![1](https://user-images.githubusercontent.com/9374871/34679195-297dff32-f4d0-11e7-97c9-48b71871668e.jpg)



### call和apply
call和apply除了接收参数有区别外，在显式绑定this方面是一样的。它们的第一个参数是一个对象，然后再调用函数的时候将this指向这个对象，区别是call的第二个以后的参数是调用函数传入的若干参数，apply第二个参数传入的是一个函数参数组成的数组，ES5以后可以传入一个类数组的对象，例如函数的内置的arguments，NodeList等。

语法：

```func.apply(thisObj, [c1,c2,c3,c4]);```

```func.call(thisObj, c1,c2,c3,c4);```

thisObj的取值有以下几种情况:

	function a(){
		console.log(this);
	}
> thisObj不传，传null或者undefined，this指向window

a.call();      	// window

a.apply(); 		// window

a.call(null); 		// window

a.apply(undefined); // window
> thisObj传一个函数的名字，this指向这个函数的引用

	var b = function(){
		console.log('b')
	}
	
a.call(b);  // ƒ (){console.log('b')}
> thisObj传入一个基本类型，string，boolean类型等，返回基本类型的构造类型

a.call('www') // String {"wew"}

a.apply(false) // Boolean {false}
> thisObj传入一个对象，this指向这个对象

	var o = {
		name: 'giflee'
	}
	
a.apply(o); // {name: "giflee"}

### bind
bind函数创建了一个新的函数，并将this绑定为传入的第一个参数，此时并不会调用这个函数。

```func.bind(thisObj,arg2,arg3,arg4,...)```

除了要绑定的thisObj以外，其他的参数都是在新函数被调用之后在实参之前传入到调用方法中。

### 日常用法
由于call和apply的特性，可以灵活运用javaScript中的内置方法来解决一些小问题。

借助push的行为来拼接两个数组：

	var arr = [1,2,3,4];
	Array.prototype.push.apply(arr,[5,6,7])
	arr   //[1,2,3,4,5,6,7]
利用push的行为，用起来比concat体验更好些。

找出数组中的最大最小值：

	Math.max.apply(null,[1,2,3,4]);   // 4
	Math.min.apply(null,[1,2,3,4]);   // 1
	
对一些类数组元素使用一些数组的方法：

可以通过call和apply对dom的NodeList和函数参数的arguments使用原生的数组方法。
	

  