---
title: 浏览器token验证
date: 2020-10-17 11:12:57
img: /medias/featureimages/5.jpg
top: false
cover: true
coverImg: /medias/featureimages5.jpg
toc: true
mathjax: false
summary: 很多的网站、APP都弱化了甚至没有搭建属于自己的账号体系，而是使用其它社会化的第三方登陆的方式，比如在登陆某个网站的时候选择通过github或者微信、微博等方式登陆，这样不仅免去了用户注册账号的麻烦，还可以获取用户的好友关系来增强自身的社交功能。
categories: 前端
tags:
- web
- token
---

### OAuth认证流程

#### 第三方登陆

举个板栗：如果要通过第三方网站（例如github）登录没有自己的帐户系统的平台，最传统的方法是直接在平台的着陆页上输入github帐户密码， 可以通过用户帐户和密码用户数据从github获得它，但是这样做有很多缺陷：


- 该平台需要以明文格式保存用户的github帐户和密码，这是不安全的；
- 该平台拥有在github中获取用户的所有权限信息；
- 只有修改密码后，用户才能取消授权平台的权限，但是这将导致用户授权的所有其他第三方应用程序无效，不知这一个应用；
- 如果第三方应用被破解，就会有用户密码泄露的风险，和所有使用github登录的网站的数据泄漏；

为了解决上述问题，有OAuth。

#### 原理

OAuth在“客户端”和“服务器”之间建立了一个授权层。 “客户端”不可以直接请求登陆“服务器”，而只能通过登录授权层来进行登陆客户端，通过服务端获取用户信息。 “客户端”登录授权层所使用的密码与用户的登陆密码不同。 登录时，用户可以指定授权层的token的授权范围和有效期。

授权层允许“客户端”登陆之后，“服务端”根据token的权限范围和有效期，向“客户端”返回用户的信息（userInfo）。

例如现在有一个平台为平台A，平台A的登陆方式只有通过gitHub账号第三方登录机制登录。流程图大致如下：


![授权登陆.png](https://img-blog.csdnimg.cn/20201018165245119.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h5YWxvaWR6,size_16,color_FFFFFF,t_70#pic_center)




1.用户点击Sign in with gitHub
 需要跳转到授权页面， 授权页面的URL中包含的主要参数是如下：
 
- client_id: 在gitHub中申请应用ID；
- redirect_uri: 授权成功之后要跳转到的地址；

2.该页面会自动跳转到由redirect_uri在初始参数中定义的URL，并将code参数自动添加到URL的末尾

- gitHub验证：https://github.com/login/oauth/authorize?client_id=myclient_id&scope=user:email

3.平台A通过上一步获取的code参数换取Token，平台A请求如下接口获取Token

`https://github.com/login/oauth/access_token`，需要包含以下参数：

- client_id: 在gitHub申请的应用ID；
- client_secret: 在gitHub申请时提供的APP Secret；
- grant_type: 需要填写authorization_code；
- code: 上一步获得的code；
- redirect_uri: 回调地址，需要与注册应用里的回调地址以及第一步的redirect_uri参数一致；

4.通过第三步的请求，接口返回Token和相关授权数据

```javascript
{
	"access_token": "ACCESS_TOKEN",		// Token的值
	"expires_in": 1000,		// 过期时间
	"uid": "1234567", 		// 当前授权用户的UID
}
```
5.使用在第四步中获取到的access_token，就可以去获取用户的资源了。

调用 `https://api.github.com/user?access_token=access_token` 这个API，就可以获取到用户的基本信息。 

用户的基本信息内容如下：
```
{
    "login": "Diamondtest",
    "id": 28478049,
    "avatar_url": "https://avatars0.githubusercontent.com/u/28478049?v=3",
    "gravatar_id": "",
    "url": "https://api.github.com/users/Diamondtest",
    "html_url": "https://github.com/Diamondtest",
    "followers_url": "https://api.github.com/users/Diamondtest/followers",
    "following_url": "https://api.github.com/users/Diamondtest/following{/other_user}",
    "gists_url": "https://api.github.com/users/Diamondtest/gists{/gist_id}",
    "starred_url": "https://api.github.com/users/Diamondtest/starred{/owner}{/repo}",
    "subscriptions_url": "https://api.github.com/users/Diamondtest/subscriptions",
    "organizations_url": "https://api.github.com/users/Diamondtest/orgs",
    "repos_url": "https://api.github.com/users/Diamondtest/repos",
    "events_url": "https://api.github.com/users/Diamondtest/events{/privacy}",
    "received_events_url": "https://api.github.com/users/Diamondtest/received_events",
    "type": "User",
    "site_admin": false,
    "name": null,
    "company": null,
    "blog": "",
    "location": null,
    "email": null,
    "hireable": null,
    "bio": null,
    "public_repos": 0,
    "public_gists": 0,
    "followers": 0,
    "following": 0,
    "created_at": "2017-05-06T08:08:09Z",
    "updated_at": "2017-05-06T08:16:22Z"
}
```

获取到用户信息，平台A进行登陆成功处理，授权登陆流程到此结束😊；


通过以上的步骤，在平台A和gitHub之间建立了独立的权限层。 该权限由用户授予，并且可以由用户取消。 它不同于第三方应用程序之间的独立性，并且不会相互干扰。 解决了以明文存储帐户密码的问题。

### Access Token

Access Token是用于访问被保护资源的一种凭证，它是一个加密的字符串。

一般Access Token的有效时间都比较短，需要频繁的使用登陆抄错。
如果要解决用户的频繁登陆问题，就需要用到Refresh Token了操作了。
那么我们是不是直接多次获取Access Token也是可以的呢？答案是不方便的，主要是因为获取Access Token的时候需要使用到一个code，而这个code是需要用户进行授权操作的，威力避免频繁点击授权操作，就有了Refresh Token，用户不用再次进行操作了。

### Refresh Token

在OAuth机制中，Refresh Token并不是必须设置的，但是不设置Refresh Token，则会增加用户登录的次数，交互不是很友好哈。

Refresh token的作用是刷新Access token，保证使用的token保持最新的有效token。Refresh Token是保存在客户端的服务器上的，当前的Access Token失效或者过期时，Refresh Token就会去获取一个新的Token，Refresh Token也是一个对客户端加密的字符串。

一个有效的token返回结果如下：

```
    HTTP/1.1 200 OK
    Content-Type: application/json
    Cache-Control: no-store
    Pragma: no-cache
    
    {
    "access_token":"MTQ0NjJkZmQ5OTM2NDE1ZTZjNGZmZjI3",
    "token_type":"bearer",
    "expires_in":3600,
    "refresh_token":"IwOGYzYTlmM2YxOTQ5MGE3YmNmMDFkNTVk",
    "scope":"create"
    }
```
获取流程原理大致如下：

![Refresh.png](https://img-blog.csdnimg.cn/20201018181410884.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h5YWxvaWR6,size_16,color_FFFFFF,t_70#pic_center)

1.客户端通过认证服务器请求认证；

2.认证服务器校验客户端认证是否有效，如果有效，返回一个Access Token和一个Refresh Token；

3.客户端通过Access Token去请求服务器的资源；

4.如果Access Token有效，服务器返回给客户端资源，如果Access Token失效，服务器返回给客户端Token失效的信息，然后客户端会通过Refresh Token再次请求获取新的Access Token；

 - Refresh Token本身也是有过期时间的，一般会比Access Token的过期时间长很多，如果想要将Refresh Token设置为永久有效，则可以通过配置参数实现。比如可以设置prompt=consent。