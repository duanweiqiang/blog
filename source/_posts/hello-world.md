---
title: hexo 配置教程
date: 2020-02-12 18:50:00
categories: 前端
img: /medias/featureimages/1.jpg
top: false
cover: false
coverImg: /medias/featureimages/1.jpg
toc: true
mathjax: false
summary: Github Page + Hexo 搭建个人博客的方式可以用来托管博客、项目官网等静态网页，具体的内容下面在文章内介绍。
tags:
- hexo
- blog
---

原博客出处：https://segmentfault.com/a/1190000017986794
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## 前言

博客有第三方平台，也可以自建，比较早的有博客园、CSDN，近几年新兴的也比较多诸如：WordPress、segmentFault、简书、掘金、知乎专栏、Github Page 等等。

这次我要说的就是 Github Page + Hexo 搭建个人博客的方式！Github Page 是 Github 提供的一种免费的静态网页托管服务（所以想想免费的空间不用也挺浪费的哈哈哈），可以用来托管博客、项目官网等静态网页。支持 Jekyll、Hugo、Hexo 编译静态资源，这次我们的主角就是 Hexo 了，具体的内容下面在文章内介绍。

## 准备环境

准备 node 和 git 环境，
首先，安装 NodeJS，因为 Hexo 是基于 Node.js 驱动的一款博客框架，相比起前面提到过的 Jekyll 框架更快更简洁，因为天*朝网络被墙的原因尝试过安装 Jekyll 失败而放弃了。
然后，安装 git，一个分布式版本控制系统，用于项目的版本控制管理，作者是 Linux 之父。如果 Git 还不熟悉可以参考廖雪峰大神的 Git 教程。

两个工具不同的平台安装方法有所不一样，可自行了解按步骤安装，这里不详述了。安装成功后打开git bash（Windowns）或者终端（Mac），下方中将统一称为命令行。
在命令行中输入相应命令验证是否成功，如果成功会有相应的版本号。

```
git version
node -v
npm -v
```

### 安装 Hexo

如果以上环境准备好了就可以使用 npm 开始安装 Hexo 了。也可查看 Hexo 的详细文档。
在命令行输入执行以下命令：

```
npm install -g hexo-cli
```

安装 Hexo 完成后，再执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。

```
hexo init myBlog//文件夹名，可自定义
cd myBlog
npm install
```
新建完成后，指定文件夹的目录如下：

```
.
├── _config.yml # 网站的配置信息，您可以在此配置大部分的参数。 
├── package.json
├── scaffolds # 模版文件夹
├── source  # 资源文件夹，除 _posts 文件，其他以下划线_开头的文件或者文件夹不会被编译打包到public文件夹
|   ├── _drafts # 草稿文件
|   └── _posts # 文章Markdowm文件 
└── themes  # 主题文件夹
```

好了，如果上面的命令都没报错的话，就恭喜了，运行 hexo s 命令，其中 s 是 server 的缩写，在浏览器中输入 http://localhost:4000 回车就可以预览效果了。

```
hexo s
或者 hexo server
```
至此，你本地的博客就已经搭建成功，接下来就是部署到 Github Page 了。

## 注册 Github

（自行baidu、google）

## 新建仓库

点击 Start project 或者下面的 new repository 创建一个新的仓库

- 注意点来了，Github 仅能使用一个同名仓库的代码托管一个静态站点，这个网上很多教程没说到的。
- 这里有个硬性要求：repository的名称必须要"用户名.github.io"

## 配置 SSH key

要使用 git 工具首先要配置一下SSH key，为部署本地博客到 Github 做准备。

打开命令行输入 cd ~/.ssh 如果没报错或者提示什么的说明就是以前生成过的，直接使用 cat ~/.ssh/id_rsa.pub 命令就是可以查看本机上的 SSH key 了。

```
cat ~/.ssh/id_rsa.pub
```
### 如果之前没有创建，则执行以下命令全局配置一下本地账户：

```
git config --global user.name "用户名"
git config --global user.email "邮箱地址"
```

然后开始生成密钥 SSH key
```
ssh-keygen -t rsa -C '上面的邮箱'
```

按照提示完成三次回车，即可生成 ssh key。通过查看 ~/.ssh/id_rsa.pub 文件内容，获取到你的 SSH key

-首次使用还需要确认并添加主机到本机SSH可信列表。若返回 Hi xxx! You've successfully authenticated, but GitHub does not provide shell access. 内容，则证明添加成功。

```
ssh -T git@github.com
```
到这还没完，还要登录 Github 上添加刚刚生成的SSH key(自行baidu、google)；

## 部署到 Github

此时，本地和Github的工作做得差不了，是时候把它们两个连接起来了。你也可以查看官网的部署教程。
先不着急，部署之前还需要修改配置和安装部署插件。

第一：打开项目根目录下的 _config.yml 配置文件配置参数。拉到文件末尾，填上如下配置（也可同时部署到多个仓库，后面再说）：

```
deploy:
  type: git
  repo: 
    github: https://github.com/duanweiqiang/duanweiqiang.github.io.git
  branch: master

```

第二：要安装一个部署插件 hexo-deployer-git。
```
npm install hexo-deployer-git --save
```

最后执行以下命令就可以部署上传啦，以下 g 是 generate 缩写，d 是 deploy 缩写：
```
hexo g -d//生成静态文件并部署到GitHub
```
稍等一会，在浏览器访问网址： https://你的用户名.github.io 就会看到你的博客啦！！

## 写博客

``` bash
$ hexo new "My New Post"//博客名文件夹
```

More info: [Writing](https://hexo.io/docs/writing.html)

More info: [Server](https://hexo.io/docs/server.html)

### 使用 Generate 命令生成 static files

``` bash
$ hexo generate/hexo g
```

More info: [Generating](https://hexo.io/docs/generating.html)

### 将本地命令推送到git仓库

``` bash
$ hexo clean
$ hexo deploy/hexo g -d
```
部署前最好能先执行一下 hexo clean 命令，清除缓存文件 (db.json) 和已生成的静态文件 (public)。在某些情况（尤其是更换主题后），如果发现您对站点的更改无论如何也不生效，您可能需要运行该命令。

另外，如果你的文章暂时不发布可以先保存在草稿里面。生成草稿的方法和文章差不多 hexo new draft "文章标题"，生成后会在 /source/_drafts 里看到你的草稿文章。当你想发布时只要执行 publish 命令即可发布到博客。

```
$ hexo publish [layout] <filename>
```
## 静态图床
文章里用的一些图片放哪里比较好呢？比对了几个免费的图床七牛、sm.ms和微博图床，最后我决定选用微博的，七牛的好像最近是测试域名不能用了，虽然有解决方案，但怕以后还会有其他问题，所以放弃啦，毕竟免费的东西才是最贵的，特别像云储存这种东西，感觉都是钱钱钱，哈哈哈，万一有一天不让用就比较麻烦了，另外sm.ms这个口碑也不错，好像是个人开发的，免费好几年了，有同样的担心就放弃了，最好抱了新浪的大腿，感觉新浪应该会靠谱一点吧，唯一的问题就是如果有一天新浪禁止外链的话就不行了，再看吧。

可以去chrome网上应用商店下载一个叫微博图床的chrome插件，下图是插件的界面，操作简单方便，具体使用看说明就可以啦，比较简单，这样图床的问题就解决了。

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)
