---
title: 浅谈JavaScript错误
date: 2018-06-07 23:07:19
categories: 
- javascript
tags:
- javascript
- error
- 大圣
author: 大圣
---

本文主要从前端开发者的角度谈一谈大多数前端开发者都会遇到的js错误，对错误产生的原因、发生阶段，以及如何应对错误进行分析、归纳和总结，希望得到一些有益的结论用来指导日常开发工作。

概念辨析
---

### 错误（Error）和异常（Exception）
对于Java来说错误和异常是两个相近但是不同的概念，而在JavaScript中可以认为错误和异常是等同的，js里只有Error关键字，并无Exception关键字。下文指的js错误也指通常理解的js异常。

### js错误和bug

#### js错误：
通常是非程序设计的原因导致的错误，大部分是发生在应用环境中的外部错误，比如硬件故障导致的I/O Error、网络不稳定导致的Network Error，调用不被信任的外部方法，DOM操作，使用new Image、new FileReader加载资源。错误可以被忽略或者捕获，代码可以通过预设错误处理流程，例如使用了重试机制的代码可以通过重试有可能让程序恢复到正常状态。
#### bug：
通常是程序设计的原因导致的计算机程序或系统中的缺陷，能够引发错误或意外结果，或使程序或系统以非预期方式运行。通常无法继续和恢复，需要程序员进入程序并且修改代码来修复。

javaScript错误发生阶段
---

由于JavaScript语言解释型的特性，js错误发生在运行时，这一点和编译型语言相比错误更加难以发现。幸运的是技术发展到今天已经有非常多成熟的工具比如Eslint、IDE的代码检查，可以帮助我们在早期不需要运行程序的阶段发现错误。还有一些JavaScript语言的超集语言，为语言添加了可选的静态类型，也可以帮助在早期发现错误。程序运行开始后，在进行用户交互之前，一些语法错误，常见的比如Uncaught SyntaxError是非常容易发现的。而剩下的那些，基本上需要通过用户交互来触发，较难以发现，本文着重讨论这部分错误。

错误的几种应对方式
---

### 不捕获错误
如果判断程序当前位置可能会发生错误，不捕获错误是一种消极的应对方式。依赖全局window.onerror错误监听能够获取未捕获的错误信息。
### 捕获错误，不处理，抛出
如果捕获错误后马上抛出，抛出的error对象就是原来的对象，相当于啥也没干，等同于不捕获错误。一般来说采用这种应对方式时都会干点什么，比如抛出一个自定义的错误，而不是原始错误对象。
### 捕获错误，不处理，不抛出
要小心不处理不抛出意味着全局window.onerror错误监听也无法获取到该错误信息，一般来说这种应对方式用在一些不影响程序主流程的错误处理上，比如调用DOM节点的focus方法，但还是需要注释合理的理由。
捕获错误后静默处理的示例:
```
try {
    obj[0].focus();
} catch (e) {
    // IE8 can throw "Can't move focus to the control because it is invisible,
    // not enabled, or of a type that does not accept the focus." for all kinds of
    // reasons that are too expensive and fragile to test.
}
```
### 捕获错误，处理
通常会在处理方法体中使用错误日志上报、失效保护、重试恢复等技术。失效保护即降级处理，举一个比较简明的例子，`try { a = JSON.parse(b); } catch (e) { a = {}; }`

几个引申出的问题
---

### 不处理错误造成的影响？
这个问题缺乏前提，到底是捕获了不处理还是未捕获错误。前者已有解答，这里说未捕获错误可能会造成的影响。错误发生后，在出错位置之前的代码已经执行过了，之后的代码不再执行，这种情况基本上就是bug了。由于js的并发模型与事件循环机制，如果在出错位置之前执行过异步代码，比如setTimeout、new Promise，异步代码中的回调函数仍然会在“执行栈”中的所有同步任务执行完毕之后按照回调函数在任务队列里的顺序执行。值得注意的是，利用这种特性，可以将错误包装在一个异步执行的函数中使用异步抛出错误的技术，能够避免阻断错误处之后的代码执行。
异步抛出错误示例:
```javascript
const asyncThrowError = (error) => {
    setTimeout(function() {
        throw error || new Error('异步抛出的异常');
    });
};
try {
    let a = JSON.parse('{a: a}');
} catch(e) {
    // 使用异步抛出异常的技术，依靠window.onerror捕获异常并记录日志
    asyncThrowError(e);
}
```
### 什么时候应该捕获错误？
当然是预感程序某一处可能会出现问题的时候啦。具体什么时候则见仁见智啦，依赖程序员的经验。
### 什么时候应该抛出错误，错误抛出后会怎么样？
人为主动抛出的错误和应用环境中的发生的错误同样会导致错误位置之后的代码无法执行。常见的一种用法是在函数检查传入的参数是否合法，不合法就使程序快速失效（Fail Fast），提醒开发者修复问题。还可以将抛出错误技术用在收集用户填写的不合法的表单数据上，在内层代码中抛出错误，在外层代码中捕获错误并判断错误类型，获取到错误信息。
### 错误的重试与恢复
如果前提得是可恢复类型的错误，在程序中加入重试机制才有意义。可恢复的错误通常是外部原因导致的，典型例子如网络错误。重试机制根据是否需要用户交互触发可分为自动重试和手动重试。自动重试的一种设计是，通过线性增长的间隔时间或者成指数增长的间隔时间循环重试直到没有错误发生，尤其是指数增长的间隔时间循环重试机制可以避免程序太快将计算机资源占满。手动重试的一个设计例子是，在一些关键业务流程，比如电商场景中的添加到购物车，当接口返回失败时，可以通过让用户手动点击重试按钮触发新的一轮接口调用。
自动重试的一种设计方案:
```javascript
const sendMesg = (errorResendTimes = 0) => {
    console.log(errorResendTimes);
    try {
        throw 'hahaha';
    } catch (e) {
        setTimeout(
            () => sendMesg(errorResendTimes), 
            100 * Math.pow(2, errorResendTimes) // 使用指数增长的时间间隔
            // 100 * errorResendTimes // 使用线性增长的时间间隔
        );
        errorResendTimes++;
    }
};
```
### 错误的隔离与降级
这个是js中捕获并处理错误的最核心的出发点，有经验的程序员为了避免未捕获的错误阻断程序执行造成bug，需要在程序中设置陷阱（使用try/catch语句）捕获错误。当错误发生的时候，使用了错误捕获技术的地方可以防止错误影响当前调用栈之后的代码执行，隔离了错误的影响范围，将影响范围限制在当前函数作用域。错误的降级处理指的是当错误发生后，虽然已使用错误捕获技术隔离了影响范围，但是假如当前调用栈之后的代码仍然依赖当前的执行结果，错误导致执行结果为非预期的数据类型，那么之后的代码使用该执行结果必然又会出错。
错误的降级处理示例:
```javascript
let a;
try {
    a = JSON.parse('{a: a}');
} catch (e) {
    a = {};
}
console.log(a.a);
```
#### 如何上报错误信息到日志服务器
这个问题分为两个步骤，首先要获取到错误信息，其次是预处理这些错误信息，通常情况下使用一些技术手段合并、降频，最后再上报日志服务器。这里着重谈如何获取到更全面细致的错误信息。直接看示例：
```
// 监听js运行时异常
window.onerror = (message, source, lineno, colno, error) => {
    var errorMesg = wrapError({message, source, lineno, colno, error});
    console.log(message)
    sendErrorToServer('js运行时异常：' + errorMesg);
}

// 监听document资源加载异常
window.addEventListener('error', function(e) {
    // 过滤非window上捕获的异常
    if (e.target === window) {
        return;
    }
    var errorMesg = wrapError(e);
    sendErrorToServer('document资源加载异常：' + errorMesg);
}, true);

// 监听未捕获的Promise异常
window.addEventListener('unhandledrejection', function(e) {
    var errorMesg = e.reason;
    sendErrorToServer('未捕获的Promise异常：' + errorMesg);
});

// 监听最初未捕获稍后又被捕获的Promise异常
window.addEventListener('rejectionhandled', function(e) {
    var errorMesg = e.reason;
    sendErrorToServer('最初未捕获稍后又被捕获的Promise异常：' + errorMesg);
});

// 已捕获的Promise异常
new Promise(function(resolve, reject) {
    // reject('我被捕获了');
    throw new Error('我被捕获了');
}).catch(function(reason) {
    var errorMesg = reason;
    sendErrorToServer('已捕获的Promise异常：' + errorMesg);
});

// 未捕获的Promise异常
new Promise(function(resolve, reject) {
    reject('我没有被捕获');
});

// 最初未捕获稍后又被捕获的Promise异常：
var p1 = new Promise(function(resolve, reject) {
    reject('我后来被捕获了');
});
setTimeout(function(){
    p1.catch(function(e) {
    });
}, 200);

function sendErrorToServer(error) {
    const img = new Image();
    var from = window.location.href;
    error += '\n' + from;
    img.src = "http://www.baidu.com?error=" + error;
}
function wrapError({message='', source='', lineno='', colno='', error, target}) {
    var returned = '';
    var mesg = error && error.toString() || message;
    var stack = error && error.stack || '';
    returned += '\n' + mesg;
    returned += '\n' + source;
    returned += '\n' + lineno;
    returned += '\n' + colno;
    returned += '\n' + stack;
    if (target && target.outerHTML) {
        returned += '\n' + target.outerHTML;
    }
    return returned;
}
```
名词解释
---

### 捕获错误：
使用try/catch块的try语句将可能出错的代码包含进去。
### 处理错误：
在try/catch块的catch语句中存在非注释的代码逻辑可以认为错误经过处理。
### 不处理错误：
与之相反，catch(e) {}内部没有代码逻辑，可以认为错误没有经过处理。
### 抛出错误：
使用throw语句后跟错误对象，`throw new Error('error message')`或
`throw 'error message'`。