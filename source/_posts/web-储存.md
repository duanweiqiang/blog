---
title: 浏览器储存的前世今生
date: 2020-08-22 21:12:57
img: /medias/featureimages/3.jpg
top: false
cover: true
coverImg: /medias/featureimages3.jpg
toc: true
mathjax: false
summary: 本文章总结浏览器发展以来的各种储存方式，客户端的储存方式也在逐渐进步，总结各种常用或者可用的储存方式，详细介绍储存的使用和注意点兼容性等。
categories: 前端
tags:
- web
- chrome
- Storage
---

### 浏览器储存

1.在代码中我们为了提高性能会请求一次数据储存起来然后在相应的页面进行读取，而不是多次去请求这样是为了提高性能。
2.那么常用的浏览器端存储技术有哪些？在我们工作中会经常遇到需要前端来储存一些数据，除了react model等工具，浏览器也为我们提供来一些可用的储存，那么这些储存都有哪些特点，需要我们怎么注意呢，我们接下来就详细的看看具体的用法。
3.在未来客户端可能会和服务起一样有自己的数据库，提供各种数据的查询方式，提供各种API来提高查询性能。


### 常用的储存方式

- cookie （也叫 Web Cookie 或浏览器 Cookie） 
- userData  比较早历史
- globalStorage  历史
- sessionStorage/localStorage  主流
- IndexedDB  主流或未来主流


#### cookie

- 以下是cookie的官方解释、

```javascript
HTTP Cookie（也叫 Web Cookie 或浏览器 Cookie）是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。通常，它用于告知服务端两个请求是否来自同一浏览器，如保持用户的登录状态。Cookie 使基于无状态的HTTP协议记录稳定的状态信息成为了可能。

Cookie 主要用于以下三个方面：

会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）
个性化设置（如用户自定义设置、主题等）
浏览器行为跟踪（如跟踪分析用户行为等）
```

- 用法：

用来设置http请求的一些规定，但已目前前端来说，更多的是储存用户的登录信息，和后端接口保持通信。

- 简单的demo

```javascript
  Set-Cookie: <cookie名>=<cookie值>
```
服务器通过该头部告知客户端保存 Cookie 信息。
```javascript
  HTTP/1.0 200 OK
  Content-type: text/html
  Set-Cookie: yummy_cookie=choco
  Set-Cookie: tasty_cookie=strawberry
```
但是cookie所能储存的数据是有限的，而且在关闭或者跨域的情况下都是无法访问到的，我们聪明的程序发明者就研究了这一种可以userData保存数据的方式。



#### userData

userData 是IE浏览器专有的数据储存。我们常用这种方式来兼容ie的老版浏览器，最低版本支持到IE5.0版本，最高到IE9以上。现在有H5基本兼容了更优的一些储存APi，这种用法基本快废弃了。
  当时他的出现是为了解决cookie的数据局限性。当浏览器关闭时，他的数据也是可以保留的。
  用户数据允许每个文档最多128KB数据，每个域名最多1MB数据。
  储存格式以xml的形式储存在客户端上。

- 使用方式

给div添加behavior属性（或者动态生成一个behavior的dom标签）
```css
  <div style="behavior:url(#default#userData)" id="dataInfoStore"></div>
```
```javascript
  var dataStore = document.getElementById("dataInfoStore");
  //使用setAttribute()方法保存数据 这是使用的是div的attr元素的道理
  dataStore.setAttribute("oldBool", "true");
  //使用save添加到自定义的变量名空间下
  dataStore.save("listInfo");
```

读取数据
```javascript
  dataStore.load("listInfo"); //load储存数据
  dataStore.getAttribute("oldBool"); //获取key值
```

删除数据 

```javascript
  dataStore.removeAttribute("listInfo") //删除attr信息
```


#### globalStorage

globalStorage也是H5比较老的一种储存方式，他是继userData之后出现的一种储存方式，一个域名下罪错可以储存5120k的数据，也可以跨页面读取。

```javascript
  globalStorage['core.mokahr.com'] 所有core.mokahr.com下面的页面都可以使用这块空间
  globalStorage['mokahr.com'] mokahr.com下面的页面都可以使用这块空间
  globalStorage['com']：所有com域名都可以 共享的使用这一块空间
  globalStorage[''] ：所有页面都可以使用的空间
```
使用方法：

- 设置
```javascript
  globalStorage["mokahr.com"].employName = '张三';
```
- 取值
```javascript
  globalStorage["mokahr.com"].getItem("employName");
  // 张三
```
- 删除
```javascript
  globalStorage["mokahr.com"].removeItem("employName");
```

😊是不是和我们的sessionStorage/localStorage很像啊，其实接下来就发展成sessionStorage/localStorage的形式了。




#### sessionStorage/localStorage

`sessionStorage`对象是存储特定于某个会话的数据，也就是数据只保存到浏览器关闭，这个对象就像会话cookie，也会在浏览器关后消失，存储在`sessionStorage`中的数据可以跨越页面刷新而存在。而`localStorage`的数据是永久性的，他不随着浏览器的关闭而清除。用这种方法我们可以储存一些需要保留的数据，希望下次打开后直接显示出来（比如用户的习惯行为）；
这两种是我们目前常用的储存数据的方式；

支持`sessionStorage/localStorage`的浏览器最小版本：IE8、Chrome 5。所以现在主流的浏览器都支持这个写法，我们基本现在都用这种写法来实现会话数据的储存。最多储存5M数据。

注意：`sessionStorage/localStorage` 的数据是可见的，一些敏感的数据不建议储存在这里。


- 设值：
```javascript
  //  使用方法存储数据 json对象要使用JSON.Stringify()方法转换成文本格式
  sessionStorage.setItem("userData", "这是一个测试数据可以是json string");
  //  使用属性存储数据
  sessionStorage.userData = "这是一个测试数据可以是json string";
  sessionStorage['userData'] = "这是一个测试数据可以是json string";
```
- 取值
```javascript
  //通过getItem（）方法取值
  const userData = sessionStorage.getItem('userData'); //这里是key名
  //通过属性取值
  const userData = sessionStorage['userData']; 

  //如果储存的是一个json字符串需要通过JSON.parse()来序列化成对象
  const JsonUserdata = JSON.parse(userData);
```
- 删除属性
```javascript
  //通过removeItem()来删除
  sessionStorage.removeItem('userData');//移除userData属性的数据
```

#### IndexedDB

官方解释：IndexedDB 是一种可以让你在用户的浏览器内持久化存储数据的方法。IndexedDB 为生成 Web Application 提供了丰富的查询能力，使我们的应用在在线和离线时都可以正常工作。

IndexedDB兼容支持浏览器情况：
  fireFox: 50+;
  chrome: 57+;
  safari: 10+;
  opera: 45+

查询支持情况：
```javascript
  if (!window.indexedDB) {
    window.alert("不支持")
  }
```
具体使用：

- 引入

```javascript
  // 打开我们的数据库
  var MokaDataStore = window.indexedDB.open("mokaDataStore",'1200');//储存空间和版本号
  // 打开数据库成功后，自动调用onsuccess事件回调。
  MokaDataStore.onsuccess = function(e) {};

  // 打开数据库失败
  MokaDataStore.onerror = function(e) {
    console.log(e.currentTarget.error.message);
  };

  // 第一次打开成功后或者版本有变化自动执行以下事件：一般用于初始化数据库。
  MokaDataStore.onupgradeneeded = function(e) {
    let dv = e.target.result; // 获取到 demoDB对应的 MokaDataStore,也就是我们的数据库。

    if (!db.objectStoreNames.contains(employeeStore)) {
      //如果表格不存在，创建一个新的表格（keyPath，主键 ； autoIncrement,是否自增），会返回一个对象（objectStore）
      // objectStore就相当于数据库中的一张表。IDBObjectStore类型。
      var objectStore = db.createObjectStore(employeeStore, {
        keyPath: 'id',
        autoIncrement: true
      });

      //指定可以被索引的字段，unique字段是否唯一。类型： IDBIndex
      objectStore.createIndex('employeeId', 'employeeId', {
        unique: true
      });
      objectStore.createIndex('employeeName', 'employeeName', {
        unique: false
      });
    }
    console.log('数据库版本更改为： ' + dbVersion);
  };
```

- 向数据库中增加数据

```javascript
  var transaction = db.transaction(employeeStore, "readwrite");
  // 在所有数据添加完毕后的处理
  transaction.oncomplete = function(event) {
    alert("添加完成");
  };

  transaction.onerror = function(event) {
    // 错误处理！
  };

  var objectStore = transaction.objectStore(employeeStore);
  var addEmployeeRequest = objectStore.add({
    employeeId: 6758,
    employeeName: '张三',
  });

```
transaction() 方法接受两个参数（一个是可选的）并返回一个事务对象。第一个参数是事务希望跨越的对象存储空间的列表。如果你希望事务能够跨越所有的对象存储空间你可以传入一个空数组，但请不要这样做，因为标准规定传入一个空数组会导致一个InvalidAccessError（可以使用属性db.objectStoreNames）。如果你没有为第二个参数指定任何内容，你得到的是只读事务。如果你想写入数据，你需要传入 "readwrite" 标识。


- 从数据库中获取数据

```javascript
  var transaction = db.transaction(employeeStore);
  var objectStore = transaction.objectStore(employeeStore);
  var request = objectStore.get(6758); // IDBIndex
  request.onerror = function(event) {
    // 错误处理!
  };
  request.onsuccess = function(event) {
    // 对 request.result 做些操作！
    alert("employeeId === 6758的数据是" + request.result.name);
  };
```

- 从数据库中删除数据

```javascript
  var request = db.transaction(["customers"], "readwrite")
                  .objectStore("customers")
                  .delete(6758);
  request.onsuccess = function(event) {
    // 删除employeeId是6758的数据成功！
  };
```