---
title: ios safari 返回时不执行代码问题解析
date: 2020-12-22 21:12:57
img: /medias/featureimages/7.jpg
top: true
cover: true
coverImg: /medias/featureimages7.jpg
toc: true
mathjax: false
summary: 最近在调试钉钉/企业微信时发现一个问题，在ios上使用返回键时，上一页的内容不执行js代码，导致react的生命周期，包componentDidMount都不执行，由于这两个平台都没有开发者工具，无法进行调试着实废了一番功夫。
categories: 前端
tags:
- debug
- Safari
- web
---

#### 引言

  最近在调试钉钉/企业微信时发现一个问题，在ios上使用返回键时，上一页的内容不执行js代码，导致react的生命周期，包componentDidMount都不执行，由于这两个平台都没有开发者工具，无法进行调试着实废了一番功夫。

- 问题描述

  最近在开发钉钉/企业微信内嵌第三方页面时，使用返回时，会导致页面白屏问题，问题比较棘手，花了3-4day用来定位问题，期间走了一些弯路。
  ![](https://pic.downk.cc/item/5fea06dd3ffa7d37b382ee7e.png)

  如上图，页面上有两个含有滚动条的DIV，一个是顶部的下拉modal，一个是body中的列表，但点击到详情的时候（第三张图），然后再点击返回，这个时候，顶部的下拉modal中就会有部分内容不限时，如图二，其实是有内容的，这个时候，只要将body中的内容滚动一下，modal的内容就都出来了。

- 以下是分析过程：

##### 数据问题

  看到问题后第一个怀疑的就是数据和代码结构问题,导致数据被覆盖或者没恶意截取的前11个，以为切换不同的账号，发现每次都显示前11个，后面的不显示，代码如下
```javascript
  // 渲染全部标签
  renderALlTags = () => {
    const { currentTag, activeIndex, selectTabIndex } = this.state;
    const { backlogTagList } = this.props;
    const showTagList = backlogTagList?.[activeIndex === 0 ? 'pendingList' : 'doneList'] || [];
    return (
      <div className={style.tagList}>
        {showTagList.map((item) => {
          return (
            <div
              className={`${style.tagItem} ${
                Number(currentTag) === item.tag && selectTabIndex === activeIndex
                  ? style.active
                  : ''
              }`}
              key={item.tag}
              onClick={() => this.handleTagChange(item.tag)}
            >
              {item.tagName} {filterNum(item.total)}
            </div>
          );
        })}
      </div>
    );
  };
```
check代码发现和上面的没有关系。
然后使用强制滚动，发现代码压根就没有执行，连console都没有打印。。。。


##### 运行环境问题

  - 因为再本地开发环境没有出现过这个问题，只有再部署到服务器上才出现这个问题，所以出现这个问题应该和环境有关。
  - 接下来发现再有再ios系统上才有这个问题，安卓并没有，果然是运行环境的问题😄，高兴的太早了。
  - 知道环境问题后，就开始一通百度。。。。
  解决方案如下：

```javascript
  $(function () {
      var isPageHide = false;
      window.addEventListener('pageshow', function () {
          if (isPageHide) {
              window.location.reload();
          }
      });
      window.addEventListener('pagehide', function () {
          isPageHide = true;
      });
  })
  //或者
  function pushHistory(){
  window.addEventListener("popstate", function(e){
    alert("回退！");
 
    // window.history.back();
    //在历史记录中后退,这就像用户点击浏览器的后退按钮一样。
 
    // window.history.go(-1);
    //你可以使用go()方法从当前会话的历史记录中加载页面（当前页面位置索引值为0，上一页就是-1，下一页为1）。
 
    // self.location=document.referrer;
    //可以获取前一页面的URL地址的方法,并返回上一页。
  }, false); 
  var state = {
    title:"",
    url: "#"
  }; 
  window.history.pushState(state, "", "#"); 
};

```
巴拉巴拉找了一堆，popstate，pageshow，forceUpdate等。。。一个一个的试，发现都不行，此处省略十万个字。
绝望了。。。。

##### 注释代码和不使用第三方库
  1.没有办法的情况下开始注释代码，不断的注释，不断的尝试，居然没有一次可以，发现问题隐藏比较深。
  2.注释代码不行就开始吧使用的antd库全部用原生div进行重写，写完之后发现问题依然存在，可以肯定这个不是和分装有关，开始怀疑是某些样式或者js在这个环境下不支持。
  3.这个时候开始想怎么能debugger一下，开始找各种工具尝试。
  emm，一天过去了，问题依然没有头绪


##### Safari调试模式

  1.突然想起来，safari在mac上提供了开发者模式，可以联机调试，说干就干，扒token，授权，然后启动。
  2.系统倒是正常运行起来了，但是问题没有复现，反复确定后发现只有在钉钉/企业微信里才会出现这个问题，既然有Safari的调试思路，找钉钉的开发这模式。
  3.钉钉文档：https://ding-doc.dingtalk.com/doc#/kn6zg7/qg4y64 调试工具不支持ios版本。。。无解

  ![](https://pic.downk.cc/item/5fea0bb23ffa7d37b38512c9.png)

  这时候已经陷入绝望了。无法调试，问题就无法定位，只能靠盲猜。

##### react-infinite-scroller研读

  没有其他便捷的方法后，开始分析代码，考虑有可能出现问题的位置进行定位。react-infinite-scroller这个库的文档仔细看了一下
  https://www.npmjs.com/package/react-infinite-scroller
  有几个API比较可疑但是试了也没有问题；
  ![](https://pic.downk.cc/item/5fea0cdd3ffa7d37b3859cf0.png)

  在这里发现了滚动区域的问题，但是试了一下，改变滚动区绑定的位置问题依然存在：
  ```javascript
  <InfiniteScroll
    initialLoad={false}
    pageStart={0}
    loadMore={this.handleInfiniteOnLoad}
    hasMore={true}
    getScrollParent={() => this.scrollParentRef}
    threshold={350}
  >
  //滚动内容
  </InfiniteScroll>
  ```
  看了这个文档后开始往滚动区域上想，然后各种尝试，各种绑定，居然都不能滚动加载更多了😭。
  然后就开始研究滚动问题。最开始怀疑过，但是自认为不可能，没有往这个方面深入。

##### 问题解决

  ios页面加载不全不能滚动

  - 问题描述 ：ios从首页进入，跳转其他页面再后退到首页，首页只显示一屏内容且无法滚动。
  - 问题原因：在于ios浏览器内核的别致渲染逻辑：它会预先找到相应的overflow: scroll元素，如果子元素高度高于父元素，则建立原生的scrollView实现滚动。问题就出现在这个“预先”上，它预先获取的高度并不是子元素渲染后的真实高度。
  - 解决办法：给设置了滚动的#root元素下的子元素wrapper设置min-height: 100vh; 先让wrapper内容设置为滚动区域，进行撑开，然后将滚动加载更多事件绑定在wrapper上进行监听。

  代码如下：
  ```css
    #root{
      position: absolute;
      top: 0;
      left: 0;
      width: 100%!important;
      // height: 100%!important;
      overflow: scroll!important;
      -webkit-overflow-scrolling: touch!important;
    }
    .wrapper {
      overflow: auto;
      height: auto;
      background-color: #fff;
      margin-top: 50px;
      min-height: 100vh;
    }
  ```
```javascript
  <div
    className={style.wrapper}
    ref={(ref) => (this.scrollParentRef = ref)}
    id="scrollableDiv"
  >
    <InfiniteScroll
      initialLoad={false}
      pageStart={0}
      loadMore={this.handleInfiniteOnLoad}
      hasMore={true}
      getScrollParent={() => this.scrollParentRef}
      threshold={350}
      scrollableTarget="scrollableDiv"
    >
    // 内容代码
    </InfiniteScroll>
  </div>
```

然后本地调试发现没有新的问题，部署，check，问题解决。

##### 问题总结

  1.由于环境问题，无法进行直观的调试，这是一个很大的问题，后续研究一下有没有更好的工具。
  2.盲目的以经验来判断是自己的方向错误导致耽误了一些时间。
  3.问题的原因同事也有怀疑过，并且提出过，由于自己试的不够彻底盲目的认为不是这个原因导致走了弯路。