---
title: Nodejs实现静态资源服务器
date: 2017-12-18 11:19:24
tags:
---

>Nodejs构建静态服务器需要考虑的几个点：

- 路径分析
- 不同类型的文件展示
- 增加文件夹逻辑 形如 'http://xxx.com/a/b/' , 则查找b目录下是否有index.html,如果有就显示，如果没有就列出该目录下的所有文件及文件夹，并可以进一步访问
- 增加缓存机制

## 路由解析

```js
const Koa = require('koa')
const app = new Koa()

app.use( async ( ctx ) => {
    console.log(ctx.request.path);
})

app.listen(3000);
console.log('app started at port 3000 ...');
```

## 纯文本文件查找和读取

```js
app.use( ( ctx ) => {
    // 拼接出真实路径
    let pathName = path.join(staticPath, ctx.request.path);
    console.log(pathName);

    // 读取文件
    let exist = fs.existsSync(pathName);
    if (exist === true) {
        let data = fs.readFileSync(pathName);
        ctx.response.type = "html";
        ctx.response.body = data;
    } else {
        ctx.response.type = 'text/plain';
        ctx.response.body = "The request URL '" + pathName + "' was not found on this server";
    }
    
    console.log(exist);
})
```

## 不同类型文件的读取

MIME值制作映射表
MIME.js
```js
let type = {
    "txt": "text/plain",
    "xml": "text/xml",
    "html": "text/html",
    "css": "text/css",
    "js": "text/javascript",
    "json": "application/json",
    "gif": "image/gif",
    "png": "image/png",
    "jpeg": "image/jpeg",
    "jpg": "image/jpeg",
    "svg": "image/svg+xml",
    "ico": "image/x-icon",
    "pdf": "application/pdf",
    "swf": "application/x-shockwave-flash",
    "tiff": "image/tiff",
    "wav": "audio/x-wav",
    "wma": "audio/x-ms-wma",
    "wmv": "video/x-ms-wmv"
}
exports.type = type;
```

通过映射表，根据后缀名来查找 response.type
```js
const findFileMIME = function (pathName) { // 根据文件后缀名来判断 
    let filetype = path.extname(pathName).substring(1);
    if (MIME.type[filetype]) {
        return MIME.type[filetype];
    }
}
```

```js
let data = fs.readFileSync(pathName);
ctx.response.type = findFileMIME(pathName) || 'text';
ctx.response.body = data;
```

## 完整的文件和文件夹处理

```js
const main = async function (ctx, next) {
    let pathName = path.join(staticPath, ctx.request.path);
    // 判断文件路径是否存在
    let exist = fs.existsSync(pathName);
    
    if (exist === true) {
        // 判断是文件还是文件夹
        var stats = fs.statSync(pathName);
        if (stats.isFile()) { // 如果是文件， 判断文件类型， 并显示文件
            let data = fs.readFileSync(pathName);
            ctx.response.type = findFileMIME(pathName) || 'text';
            ctx.response.body = data;

        } else if (stats.isDirectory()) { // 如果是文件夹, index.html or 文件列表
            if (!fs.existsSync(path.join(pathName, 'index.html'))) { // 判断是否存在 index.html
                ctx.response.type = 'html';
                let htmlbody = "<head><meta charset = 'utf-8'/></head><body><ul>";
                let files = await fs.readdir(pathName);
                for (let i  = 0;i<files.length; i++) {
                    // 排除 DS_STORE 文件
                    if (files[i]!== '.DS_Store') {
                        let hreflink = path.join(ctx.request.path, files[i]);
                        htmlbody = htmlbody + '<li><a href="' + hreflink +'">' + files[i] +'</a></li>';
                        console.log(path.extname(files[i]));
                    }
                }
                htmlbody = htmlbody + '</ul></body>';
                ctx.response.body = htmlbody;
            } else {
                ctx.response.redirect(path.join(ctx.request.path, 'index.html'));
            }
        }

    } else {
        ctx.response.type = 'text/plain';
        ctx.response.body = "The request URL '" + pathName + "' was not found on this server";
    }
};

app.use(main);
```

## 增加缓存机制

目的：为了缓解请求量增大对服务器的压力，采用缓存机制能够减少对服务器文件的读写。

属性：
- request 
    - If-Modified-Since：第一次请求时response 中的 Last-Modified
    - If-Nont-Match： 浏览器第一次请求时 response 中的 ETag

- response
    - Cache-Control: max-age=【秒】 || no-cache || no-store || public || private
    - ETag: 当前资源在服务器的唯一标志符
    - Expires
    - Last-Modified: 告诉浏览器当前资源的最后修改时间
    
HTTP 缓存策略
- **本地缓存阶段：** 先在本地查找该资源，如果存在，且资源未过期，则使用这一资源，不发送 http 请求到服务器
- **协商缓存阶段：** 如果在本地缓存找到对应资源，但是不知道是否过期，则发一个HTTP请求到服务器，然后服务器判断请求的资源在服务器上是否改动过，没有改动过则返回 304，让浏览器使用本地找到的资源
- **缓存失败：** 当服务器发现请求的资源已经修改过，或这是一个新的请求，服务器则返回该资源的数据，并返回200。如果资源未找到则返回 404

### 本地缓存阶段

- Expires：缓存过期的绝对时间GMT，如果设置了max-age，expires会失效
- Cache-Control：max-age=【秒】

### 前端存储方式

- Cookie <4k 且会在同域网络请求中进行传输，消耗网络带宽，大量Cookie会导致请求变慢，所以Cookie只保存与权限有关的用户信息。
- LocalStorage 存储非敏感的静态数据
- SeesionStorage 关闭浏览器后会清空

### 协商缓存阶段

- Last-Modified === if-modified-since 最后修改时间判断
- ETag === If-None-Match 文件的唯一符

协商流程：
1、客户端请求一个页面（A）。
2、服务器返回页面A，并在给A加上一个Last-Modified/ETag。
3、客户端展现该页面，并将页面连同Last-Modified/ETag一起缓存。
4、客户再次请求页面A，并将上次请求时服务器返回的Last-Modified/ETag一起传递给服务器。
5、服务器检查该Last-Modified或ETag，并判断出该页面自上次客户端请求之后还未被修改，直接返回响应304和一个空的响应体。


**Etag** 主要为了解决 Last-Modified 无法解决的一些问题：
1、一些文件也许会周期性的更改，但是他的内容并不改变(仅仅改变的修改时间)，这个时候我们并不希望客户端认为这个文件被修改了，而重新GET；
2、某些文件修改非常频繁，比如在秒以下的时间内进行修改，(比方说1s内修改了N次)，If-Modified-Since能检查到的粒度是s级的，这种修改无法判断(或者说UNIX记录MTIME只能精确到秒)；
3、某些服务器不能精确的得到文件的最后修改时间。

**Content-Length：**尽管并没有在缓存中明确涉及，Content-Length头部在设置缓存策略时很重要。某些软件如果不提前获知内容的大小以留出足够空间，则会拒绝缓存该内容。

**Vary：**缓存系统通常使用请求的主机和路径作为存储该资源的键。当判断一个请求是否是请求同样内容时，Vary头部可以被用来提醒缓存系统需要注意另一个附加头部。它通常被用来告诉缓存系统同样注意Accept-Encoding头部，以便缓存系统能够区分压缩和未压缩的内容。

```js
 // ************ 判断 缓存 ***********************
let Expires = {
    fileMatch: /^(\.gif|\.png|\.jpg|\.js|\.css|\.html)$/ig,
    maxAge: 60*60
};
if (path.extname(ctx.request.path).match(Expires.fileMatch)) {
    //获取最后修改时间 last-modified 
    var stats = fs.statSync(pathName);
    let lastModified = stats.mtime.toUTCString();
    
    // last-modified
    if (ctx.request.header['if-modified-since']) {
        if (ctx.request.header['if-modified-since'] === lastModified) {
            ctx.status = 304;
            await next();
            return;
        }
    }
    // ETag 
    if (ctx.request.header['if-none-match']) {
        if (ctx.request.header['if-none-match'] == lastModified) {
            ctx.status = 304;
            await next();
            return;
        }
    }
    // cache 失效 fetch new data
    ctx.set('ETag', lastModified);
    ctx.set('Last-Modified', lastModified);
    ctx.status = 200;
    let data = fs.readFileSync(pathName);
    ctx.response.type = findFileMIME(pathName) || 'text';
    ctx.response.body = data;
    await next();
}

```

## 参考资料
- [缓存策略](http://imweb.io/topic/55c6f9bac222e3af6ce235b9)
