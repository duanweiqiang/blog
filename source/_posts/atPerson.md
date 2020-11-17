---
title: 论坛评论@功能
date: 2020-04-26 11:59:22
img: http://ww1.sinaimg.cn/large/987eaf20ly1geanhftcmlj20dw09040x.jpg
top: false
cover: true
coverImg: http://ww1.sinaimg.cn/large/987eaf20ly1geanhftcmlj20dw09040x.jpg
toc: true
mathjax: false
summary: 通用评论中输入@符号后可以调出人员名单进行@某人，本文使用原生dom操作，不借助任何库或第三方框架
categories: 前端
tags:
- React
- keyEvent
---

# 页面@功能实现

本文demo： [gitHub地址](https://github.com/duanweiqiang/atPerson "gitHub")

需要用到的知识：

### 1.将div设置成可输入状态 
- contentEditable 属性用于设置或返回元素的内容是否可编辑。
- 这样就可以在div中使用标签来表示@中的人（只是为了可以在输入框中使用标签）

```html
<div contentEditable="true"
    id="textArea"
    onInput={this.onChangeAtPerson} 
    className={style.testArea}
    placeholder="请输入评论，可以@其他人"
    />
</div>
```

### 2.创建range 对象
- 用来控制光标是否可以进入@的dom

```javascript
    const range = document.createRange();
```

### 3.操作range对象
- 用来控制光标是否可以进入@的dom

**选择节点**

`selectNode()` :选择整个节点，包括子节点

`selectNodeContents()`  选择节点的子节点

- example:
    ```html
    <!-- html -->
    <p id="p1">
        <b>Hello</b> 
        world!
    </p>
    ```
    ```javascript
    var range1 = document.createRange(),
        range2 = document.createRange(),
        p1 = document.getElementById("p1");
    range1.selectNode(p1); //<p id="p1"><b>Hello</b> world!</p>
    range2.selectNodeContents(p1); //<b>Hello</b> world!
    ```

`setStart()`、`setEnd()`：这里选择节点和鼠标选中一样，这个是自动选中

>方法都接受两个参数：一个参照节点，一个节点偏移量

- example
    ```html
    <!-- html -->
    <p id="p1">Hello world!</p> 
    ```
    ```javascript
    //js
    range = document.createRange();
    p1 = document.getElementById("p1").childNodes[0];
    range.setStart(p1,2);
    range.setEnd(p1,8);
    //选中的将会是 llo wo（注意！以0为基数，空格也算一个文本字符，占1个偏移量）
    ```

**操作节点**

`deleteContents()`: 这个方法能够从文档中删除范围缩包含的内容

`extractContents()`: 会删除并返回文档片段

`CloneContents()`: 创建范围对象的一个副本，不会影响原来的节点
>复制 DOM 范围  ： 可以使用 cloneRange()方法复制范围。这个方法会创建调用它的范围的一个副本。
>`var newRange = range.cloneRange();`

`insertNode()`: 向范围选区的开始处插入一个节点

`surroundContents()`: 环绕范围插入内容 

在使用完范围之后，最好是调用 `detach()` 方法，以便从创建范围的文档中分离出该范围。调用
`detach()`之后，就可以放心地解除对范围的引用，从而让垃圾回收机制回收其内存了。来看下面的
例子

`range.detach();` //从文档中分离
`range = null;` //解除引用
推荐在使用范围的最后再执行这两个步骤。一旦分离范围，就不能再恢复使用了。

### 4.Selection 对象

- 这是一个window对象
- 返回一个 `Selection` 对象，表示用户选择的文本范围或光标的当前位置。
- 这里使用它来获取当前光标所处的dom

用法:

```javascript
    const selection = window.getSelection() ;//非react使用
    const selection = document.getSelection();
    this.anchorNode = selection.anchorNode;//获取当前贯标所在位置的dom（使用anchorNode子方法）
```

### 5.dom中设置@人员的演示

- 本文通过`<span>`标签来实现

```html
    <span 
        style="background-color:#F4F4F6;padding: 3px 5px;border-radius: 3px;" 
        id=${item.id} name=${item.name}> //react方法
        @${item.name}
    </span>
```

### 6.通过selection来替换原来的@字符

>1.先获取当前光标的位置
>2.再获取当前输入框的dom
>3.分析dom里的结构
>4.将dom中的光标位置的@替换成`atPresonDom`中的结构
>5.将生成的`reasonDom`插入到dom中
>6.再将光标置于当前替换的`<span>`标签之后

```javascript
selectPerson = item => {
    const anchorNode = this.anchorNode;
    const selectStartIndex = this.selectStartIndex;
    const atPresonDom = `&nbsp;<span style="background-color:#F4F4F6;padding: 3px 5px;border-radius: 3px;" id=${item.id} name=${item.name}>@${item.name}</span>&nbsp;`;
    let reasonDom = '';
    const range = document.createRange();
    let targetDomIndex = 0;
    [...textDom.childNodes].forEach((each, index) => {
      if (each.nodeName === 'SPAN') {
        reasonDom += each.outerHTML;
      } else if (anchorNode === each) {
        targetDomIndex = index;
        const tempEachDom = each;
        const dom1 = tempEachDom.data.substring(0, selectStartIndex - 1);
        const dom2 = atPresonDom;
        const dom3 = tempEachDom.data.substring(
          selectStartIndex + this.selectText.length,
          each.data.length,
        );
        reasonDom += dom1 + dom2 + dom3;
      } else {
        reasonDom += each.data;
      }
    });
    textDom.innerHTML = reasonDom;//插入dom
    const textDomChildList = [...textDom.childNodes];
    const textDomLastIndex =
      textDomChildList.length - 1 > -1 ? textDomChildList.length - 1 : 0;
    const targetIndex =
      targetDomIndex + 2 > textDomLastIndex
        ? textDomLastIndex
        : targetDomIndex + 2;
    const targetAnchorNode = textDomChildList[targetIndex];
    const selection = document.getSelection();
    range.setStart(targetAnchorNode, 1);
    range.setEnd(targetAnchorNode, 1);
    selection.removeAllRanges();
    selection.addRange(range);
    this.selectText = '';
    this.setState({ personDataList: [] });
    this.selectStartIndex = -1;
};
```

### 7.键盘控制选中(后边附录键盘事件)

- 通过键盘来控制循环选中下拉人员列表
- 键盘控制删除整体模块
- 键盘控制光标进入空能

```javascript
keydownEvent = e => {//键盘事件
    if (e.keyCode === 8) {
      const selection = document.getSelection();
      this.anchorNode = selection.anchorNode;
      const range = selection.getRangeAt(0);

      // 删除@模块
      if (selection.focusNode.parentNode.nodeName === 'SPAN') {
        textDom?.removeChild(range.startContainer.parentElement);
      }
    }
    if ([38, 40].indexOf(e.keyCode) > -1) {
      //up&down
      const { personDataList, curSelectItem } = this.state;
      personDataList.length > 0 && e.preventDefault(); //有下拉是阻止冒泡
      let targetIndex = 0;
      personDataList.forEach((item, index) => {
        targetIndex = item.id === curSelectItem.id ? index : targetIndex;
      });
      if (e.keyCode === 40) { //down
        targetIndex = targetIndex === personDataList.length - 1 ? 0 : targetIndex + 1;
      }
      if (e.keyCode === 38) { //up
        targetIndex = targetIndex === 0 ? personDataList.length - 1 : targetIndex - 1;
      }
      this.setState({ curSelectItem: personDataList[targetIndex] });
    }
    if (e.keyCode === 13) {
      e.preventDefault();
      const { curSelectItem } = this.state;
      curSelectItem && this.selectPerson(curSelectItem);
      this.setState({ curSelectItem: null });
    }
  };
```

### 8.当光标删除到整体模块处理

当光标识别到进入@整体模块时，需要整体删除

这里使用的时`<span>`标签来表示的

```javascript
if (selection.focusNode.parentNode.nodeName === 'SPAN') {
    //不允许编辑@人员
    textDom?.removeChild(selection.focusNode.parentNode);
    this.setState({ personDataList: [] });
    this.selectStartIndex = -1;
}
```

### 9.效果图

![demo2.png](http://ww1.sinaimg.cn/large/987eaf20gy1ge7m81xiq0j21dq0m8n17.jpg)

# 附：js键盘事件全面控制详解

主要分四个部分
第一部分：浏览器的按键事件
第二部分：兼容浏览器
第三部分：代码实现和优化
第四部分：总结

## 第一部分：浏览器的按键事件

1.用js实现键盘记录，要关注浏览器的三种按键事件类型，即`keydown`，`keypress`和`keyup`，它们分别对应`onkeydown`、`onkeypress`和`onkeyup`这三个事件句柄。一个典型的按键会产生所有这三种事件，依次是`keydown`，`keypress`，然后是按键释放时候的`keyup`。

2.在这3种事件类型中，`keydown`和`keyup`比较底层，而`keypress`比较高级。这里所谓的高级是指，当用户按下`shift + 1`时，`keypress`是对这个按键事件进行解析后返回一个可打印的“!”字符，而`keydown`和`keyup`只是记录了`shift + 1`这个事件。

3.但是`keypress`只能针对一些可以打印出来的字符有效，而对于功能按键，如`F1-F12`、`Backspace`、`Enter`、`Escape`、`PageUP`、`PageDown`和箭头方向等，就不会产生`keypress`事件，但是可以产生`keydown`和`keyup`事件。然而在FireFox中，功能按键是可以产生`keypress`事件的。

4.传递给`keydown`、`keypress`和`keyup`事件句柄的事件对象有一些通用的属性。如果`Alt`、`Ctrl`或`Shift`和一个按键一起按下，这通过事件的`altKey`、`ctrlKey`和`shiftKey`属性表示，这些属性在FireFox和IE中是通用的。

## 第二部分：兼容浏览器

- 凡是涉及浏览器的js，就都要考虑浏览器兼容的问题。
- 目前常用的浏览器主要有基于IE和基于Mozilla两大类。Maxthon是基于IE内核的，而FireFox和Opera是基于Mozilla内核的。

### 2.1 事件的初始化

首先需要了解的是如何初始化该事件，基本语句如下：

```javascript
    function keyDown(){}
    document.onkeydown = keyDown;
```

***react 绑定***

```javascript
    //直接绑定在原生事件上，这个一般是用来捕捉编辑的时候的事件，用的不多
    <textarea onKeyDown={e=> console.log( e.keyCode ) } />
    //通过声明周期直接绑定到document的事件上，这个方式一般用来做快捷键比较多
    export class KeyBind extends React.Component {
        componentDidMount(){
            document.addEventListener("keydown", this.onKeyDown)
        }

        componentWillUnmount(){
            document.removeEventListener("keydown", this.onKeyDown)
        }

        onKeyDown = (e) => {
            switch( e.keyCode) {
            case 13://回车事件
                break
            }
        }
    }
```

当浏览器读到这个语句时，无论按下键盘上的哪个键，都将呼叫KeyDown()函数。

### 2.2 FireFox和Opera的实现方法

FireFox和Opera等程序实现要比IE麻烦，所以这里先描述一下。

`keyDown()`函数有一个隐藏的变量--一般的，我们使用字母“e”来表示这个变量。  

```javascript
    function keyDown(e)  
```

变量e表示发生击键事件，寻找是哪个键被按下，要使用which这个属性：  

```javascript
    e.which  
```

`e.which`将给出该键的索引值，把索引值转化成该键的字母或数字值的方法需要用到静态函数`String.fromCharCode()`，如下： 

```javascript
    String.fromCharCode(e.which)
```

把上面的语句放在一起，我们可以在FireFox中得到被按下的是哪一个键：  

```javascript
    function keyDown(e) {  
        var keycode = e.which;  
        var realkey = String.fromCharCode(e.which);  
    　　alert("按键码: " + keycode + " 字符: " + realkey);  
    }
    document.onkeydown = keyDown;
```

### 2.3 IE的实现方法

IE的程序不需要e变量，用`window.event.keyCode`来代替`e.which`，把键的索引值转化为真实键值方法类似：`String.fromCharCode(event.keyCode)`，程序如下：

```javascript
    function keyDown() {  
        var keycode = event.keyCode;  
        var realkey = String.fromCharCode(event.keyCode);  
        alert("按键码: " + keycode + " 字符: " + realkey);  
    }  
    document.onkeydown = keyDown;
```

### 2.4 判断浏览器类型

上面了解了在各种浏览器里是如何实现获取按键事件对象的方法，那么下面需要判断浏览器类型，这个方法很多，有比较方便理解的，也有很巧妙的办法，先说一般的方法：就是利用`navigator`对象的`appName属性`，当然也可以用`userAgent`属性，这里用`appName`来实现判断浏览器类型，IE和Maxthon的appName是“Microsoft Internet Explorer” ,而FireFox和Opera的appName是“Netscape”，所以一个功能比较简单的代码如下：

```javascript
    function keyUp(e) {   
        if(navigator.appName == "Microsoft Internet Explorer"){
            var keycode = event.keyCode;  
            var realkey = String.fromCharCode(event.keyCode);  
        }else{
            var keycode = e.which;  
            var realkey = String.fromCharCode(e.which);  
        }
        alert("按键码: " + keycode + " 字符: " + realkey);
    }
    document.onkeyup = keyUp;
```

比较简洁的方法是：

```javascript
    function keyUp(e) {
        var currKey=0, e=e||event;
        currKey=e.keyCode||e.which||e.charCode;
        var keyName = String.fromCharCode(currKey);
        alert("按键码: " + currKey + " 字符: " + keyName);
    }
    document.onkeyup = keyUp;
```

上面这种方法比较巧妙，简单地解释一下：

> 首先，e=e||event;这句代码是为了进行浏览器事件对象获取的兼容。js中这句代码的意思是，如果在FireFox或Opera中，隐藏的变量e是存在的，那么e||event返回e，如果在IE中，隐藏变量e是不存在，则返回event。
> 其次，`currKey=e.keyCode||e.which||e.charCode`;这句是为了兼容浏览器按键事件对象的按键码属性（详见第三部分），如IE中，只有keyCode属性，而FireFox中有which和charCode属性，Opera中有keyCode和which属性等。
> 上述代码只是兼容了浏览器，获取了keyup事件对象，简单的弹出了按键码和按键的字符，但是问题出现了，当你按键时，字符键都是大写的，而按shift键时，显示的字符很奇怪，所以就需要优化一下代码了。

## 第三部分：代码实现和优化

### 3.1 按键事件的按键码和字符码

在IE中，只有一个`keyCode`属性，并且它的解释取决于事件类型。对于`keydown`来说，`keyCode`存储的是按键码，对于 `keypress`事件来说，`keyCode`存储的是一个字符码。而IE中没有`which`和`charCode`属性，所以`which和charCode`属性始终为`undefined`。

FireFox中keyCode始终为0，时间`keydown`/`keyup`时，`charCode`=0，which为按键码。事件`keypress`时，which和charCode二者的值相同，存储了字符码。

在Opera中，`keyCode`和`which`二者的值始终相同，在`keydown/keyup`事件中，它们存储按键码，在`keypress`时间中，它们存储字符码，而charCode没有定义，始终是undefined。

### 3.2 用keydown/keyup还是keypress

第一部分已经介绍了`keydown/keyup`和`keypress`的区别，有一条比较通用的规则，`keydown`事件对于功能按键来说是最有用的，而`keypress`事件对于可打印按键来说是最有用的。

键盘记录主要是针对于可打印字符和部分功能按键，所以`keypress`是首选，然而正如第一部分提到的，IE中`keypress`不支持功能按键，所以应该用`keydown/keyup`事件来进行补充。

### 3.3 代码的实现
总体思路，用keypress事件对象获取按键字符，用keydown事件获取功能字符，如`Enter`，`Backspace`等。

代码实现如下所示

``` javascript
!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.0Transitional//EN">
<html>
<head>
    <title>js 按键记录</title>
    <meta name="Generator" CONTENT="EditPlus">
    <meta name="Author" CONTENT="Duke">
    <meta name="Keywords" CONTENT="js 按键记录">
    <meta name="Description" CONTENT="js 按键 记录">
</head>
<script type="text/javascript">
    var keystring = "";//记录按键的字符串
    
    function $(s){
        return document.getElementByIdx_x(s)?document.getElementByIdx_x(s):s;
    }
    
    function keypress(e){
        var currKey=0,CapsLock=0,e=e||event;
　      currKey=e.keyCode||e.which||e.charCode;
　      CapsLock=currKey>=65&&currKey<=90;
　      
　      switch(currKey){//屏蔽了退格、制表、回车、空格、方向键、删除键
            case 8: 
            case 9: 
            case 13: 
            case 32: 
            case 37: 
            case 38: 
            case 39: 
            case 40: 
            case 46:
                keyName = "";
                break;
            default:
                keyName = String.fromCharCode(currKey); break;
        }
        keystring += keyName;
    }
    
    function keydown(e){
        var e=e||event;
        var currKey=e.keyCode||e.which||e.charCode;
        if((currKey>7&&currKey<14)||(currKey>31&&currKey<47)){
            switch(currKey) {
                case 8: keyName = "[退格]"; break;
                case 9: keyName = "[制表]"; break;
                case 13:keyName = "[回车]"; break;
                case 32:keyName = "[空格]"; break;
                case 33:keyName = "[PageUp]"; break;
                case 34:keyName = "[PageDown]"; break;
                case 35:keyName = "[End]"; break;
                case 36:keyName = "[Home]"; break;
                case 37:keyName = "[方向键左]"; break;
                case 38:keyName = "[方向键上]"; break;
                case 39:keyName = "[方向键右]";break;
                case 40:keyName = "[方向键下]";break;
                case 46:keyName = "[删除]";break;
                default:keyName = "";break;
            }
            keystring += keyName;
        }
        $("content").innerHTML=keystring;
    }
    
    function keyup(e){
        $("content").innerHTML=keystring;
    }
    document.onkeypress=keypress;
    document.onkeydown =keydown;
    document.onkeyup =keyup;
</script>
<body>
    <input type="text" />
    <input type="button" value="清空记录" onclick="$('content').innerHTML = '';keystring = '';"/>
    <br/>请按下任意键查看键盘响应键值：<span id="content"></span>
</body>
</html>
```

代码分析：
`$()`：根据ID获取dom
`keypress(e)`：实现对字符码的截获，由于功能按键要用keydown获取，所以在keypress中屏蔽了这些功能按键。
`keydown(e)`：主要是实现了对功能按键的获取。
`keyup(e)`：展示截获的字符串。

代码基本上就算实现完成了！呵呵

## 第四部分：总结
1.H5端键盘keyCode有部分缺失，使用是要特别注意一下
2.编写代码的最初目的是能够通过js记录按键，并返回一个字符串。

上述代码只是用js实现了基本的英文按键记录，对于汉字是无能为力，记录汉字，我能想到的办法，当然是用js，是用keydown和keyup记录底层按键事件，汉字解析当然无能为力。当然你可以用DOM的方式直接获取input中的汉字，但这已经离开了本文讨论的用按键事件实现按键记录的本意。


本文demo： [gitHub地址](https://github.com/duanweiqiang/atPerson "gitHub")

>上述代码还可以实现添加剪切板的功能，监控删除的功能等等。。。
