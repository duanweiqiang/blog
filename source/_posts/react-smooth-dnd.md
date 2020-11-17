---
title: react顺滑拖动实现
date: 2020-05-15 11:59:22
img: /medias/featureimages/7.jpg
top: false
cover: true
coverImg: /medias/featureimages/7.jpg
toc: true
mathjax: false
summary: 一个快速、轻量级的拖放、可排序的库，用于对覆盖许多设计与开发（d&d）场景的许多配置选项进行响应。它使用css转换来制作动画，因此只要有可能，它的硬件就会加速。
categories: 前端
tags:
- React
- 拖动
- drag
---
#### 一、介绍
1.react-smooth-dnd是一个快速、轻量级的拖放、可排序的库，用于对覆盖许多设计与开发（d&d）场景的许多配置选项进行响应。它使用css转换来制作动画，因此只要有可能，它的硬件就会加速，提高了拖动的动画效果。

2.此库是在基于smooth-dnd库开发的React拖动效果组件。

3.npm 安装 `npm install react-smooth-dnd`；

4.鉴于可以查到的文档都是英文的，而且demo都残缺不全，本文进行了中文归纳和详细案例介绍；

5.本文主要介绍基于react引入的方式来写的demo；

![列表拖动排序.jpg](https://i.loli.net/2020/05/18/VpDJzQ4938yr5TO.jpg)

6.目前该库还在维护中，有1.4k星，使用还不错，目前没发现bug，兼容Chrome浏览器
![gitHubInfo.jpg](https://i.loli.net/2020/05/18/xhY1ElczrJBuidj.jpg)

#### 二、一个简单的demo

本案例是一个简单的引入和样式

```javascript
import React, { Component } from 'react';
import { Container, Draggable } from 'react-smooth-dnd';

class SimpleDemo extends Component {
  render() {
    return (
      <div>
        <Container onDrop={this.onDrop}>
            <Draggable key={id}>
                <div>{`${我是一个可拖动的元素，拖拖试试看}`}</div>
            </Draggable>
        </Container>
      </div>
    );
  }
}

```
下面文章就介绍详细的使用方法。。。

### 三、Dom引入

引入标签解释：

- Container 指的是拖动的容器，即可拖动的有效使用范围,它的内部可有有多个`<Draggable>`标签。

- Draggable 指的是可拖动元素，在页面中把可拖动的内容用`<Draggable>`标签包裹后，该标签就可以脱离文档进行拖动。

* 用map实现的拖动内部元素
```javascript
<Container onDrop={this.onDrop}>
    {itemList.map(item => {
    return (
        <Draggable key={item.id}>
        ...//你的拖动代码块
        </Draggable>
    );
    })}
</Container>
```

### 四、Container 常用API

#### 1.groupName

解释：定义当前拖动容器的名称（唯一性），如果代码中有多个`groupName="col"` 则表示这几个拖动区域中的内容是相互之间可以拖动。比如我们要实现一个跨组拖动。

```javascript
<Container groupName="col">
    <Draggable>....<Draggable>
</Container>
```

#### 2.behaviour的使用

解释：当前容器中的元素拖动后，本容器中的元素状态。标示当前容器状态；

可选值：`move`（默认，移动）、`copy`（复制）、`drop-zone`（跌落）、`contain`（包含）

```javascript
//实现复制本容器中的内容而不是move拖动离开
<Container behaviour="copy">
    <Draggable>....<Draggable>
</Container>
```

#### 3.lockAxis的使用

解释：设置限制当前拖动的方向。

可选值：`x`、`y`，表示只能x轴或者y轴方向拖动；

```javascript
//限制该拖动只能y轴方向进行拖动
<Container lockAxis="y">
    <Draggable>....<Draggable>
</Container>
```

#### 4.dragClass的使用

解释：拖动元素被拖动时可添加的样式，（拿起来样式）；可以使用react的Style进行引入css

可以设置字体，缩放，旋转等`css`的样式实现拖动过程中的样式；

```css
.card_ghost{
    background: #FFF;
    transform:rotate(1.5deg);
    -ms-transform:rotate(1.5deg); 	/* IE 9 */
    -moz-transform:rotate(1.5deg); 	/* Firefox */
    -webkit-transform:rotate(1.5deg); /* Safari 和 Chrome */
    -o-transform:rotate(1.5deg); 	/* Opera */
    box-shadow: 0 0 15px grey;
    opacity: 0.8;
}
```

```javascript
<Container dragClass={style.card_ghost}>
    <Draggable>....<Draggable>
</Container>
```

#### 5.dropClass的使用

解释：拖动释放时的样式，写法如上；

#### 6.dropPlaceholder的使用

解释：拖动时的占位效果

当拖动元素拖走时或进入其他位置时，用于占位当前阴影配置

参数：`className`, `animationDuration`, `showOnTop`;

calssNmae： 占位元素的延时
animationDuration： 延时
showOnTop： （暂时还未知作用）

```css
.cards_drop_preview{
    background: #DDDFE3;
}
```

```javascript
<Container 
    dropPlaceholder={{
        animationDuration: 150,//动画延时
        showOnTop: false,//暂时还不知道作用
        className: style.cards_drop_preview, //占位元素样式
    }}
>
    <Draggable>....<Draggable>
</Container>
```

#### 7.dragBeginDelay的使用

解释：延时拖动，时间单位为毫秒。按下项目后延迟开始拖动。在延迟超过5px之前移动光标将取消拖动。

防止有误操作的情况发生；也可用于该item上既有拖动又又其他事件，延时可以区分需要触发的是那种事件。

#### 8.onDragStart、onDragEnd、onDropReady、onDrop的使用

解释： 
onDragStart：拖动开始后出发该函数；
onDragEnd：拖动结束；
onDropReady：拖动ready；
onDrop：拖动释放；

参数：
`isSource` : boolean (true/false) 如果是从其他容器中拖动来的则是false
`payload` : object （ { removedIndex, addedIndex, payload } ）
    - removedIndex移走元素的索引；
    - addedIndex添加元素的索引；
    - payload移动的元素数据，配合`getChildPayload`函数使用；
`willAcceptDrop` : boolean 如果拖动的项可以放入容器中，则为true，否则为false。

可以认为是辅助函数，用于获取动作结束后的一些状态事件获取。

```javascript
//拖动开始后出发
onDragStart = ({isSource, payload, willAcceptDrop})=>{
    const { removedIndex, addedIndex, payload } = dragResult;
    if (removedIndex === null && addedIndex === null) return arr;
    const result = [...arr];
    let itemToAdd = payload;
    if (removedIndex !== null) {
      itemToAdd = result.splice(removedIndex, 1)[0];
    }
    if (addedIndex !== null) {
      result.splice(addedIndex, 0, itemToAdd);
    }
    this.setState({ dataList: result });
}
<Container onDragStart={this.onDragStart}>
    <Draggable>....<Draggable>
</Container>
```

#### 9.getChildPayload的使用

解释：设置上述的payload的值

getChildPayload 函数return一个自定义的值；

用来记录当前拖动元素的信息，参数是index即Container中dataList的索引，当释放（onDrop）函数触发是，payLoad会自动带入该参数，用于做数据处理。

```javascript
//拖动开始后出发
getChildPayload = (index)=>{
    console.log(index);//当前拖动的索引
    return {
        ...
    }
}
<Container getChildPayload={this.getChildPayload}>
    <Draggable>....<Draggable>
</Container>
```

#### 10.onDragEnter、onDragLeave的使用

解释： 监控拖动进入或者离开响应区时的状态

函数参数数据和和7一样，同上onDragStart；

该功能可以在拖动的中间触发事件，进行一些业务逻辑处理

### 五、Draggable API

Draggable可以使用render进行渲染，默认情况下，Draggable对组件根使用div元素。
如果设置了render函数，则Draggable的子属性将被忽略，而render的返回值将用于render Draggable。

return 是一个React的dom元素集（React Element）；

demo
```javascript
<Draggable render={() => {
  return (
    <li>
      ...
    </li>
  )
}}/>
```

### 六、典型Demo

我们最长使用的就是`dragClass`、`getChildPayload`、`onDrop`这些函数；

下面来个拖动排序的demo实现拖动排序

拖动排序最重要的时顺序的索引值，如果先删除了就会导致索引对应不上了，这点要特别注意

```javascript
import React, { Component } from 'react';
import { Container, Draggable } from 'react-smooth-dnd';


class SimpleDemo extends Component {
    constructor(props) {
        super(props);
        this.state = {
            dataList: [ //排序数组
                {
                    id: 1,
                    name: '第1个'
                },
                {
                    id: 1,
                    name: '第2个'
                },
                {
                    id: 1,
                    name: '第3个'
                },
                {
                    id: 1,
                    name: '第4个'
                }
            ]
        };
    }
    //设置拖动内容
    getChildPayload = (index)=>{
        const { dataList } = this.state;
        return {
            item:dataList[index]
            index,
        }
    }
    //拖动释放时出发重新排序事件
    onDrop = (dragResult)=>{
        const { removedIndex, addedIndex, payload } = dragResult;
        const { dataList } = this.state;
        let resutl = dataList;
        let itemToAdd = null;
        if (removedIndex !== null) {
            itemToAdd = result.splice(removedIndex, 1)[0];
        }
        if (addedIndex !== null) {
            result.splice(addedIndex, 0, itemToAdd);
        }
        this.setState({dataList: result})
    }
    render() {
        const { dataList } = this.state;
        return (
        <div>
            <Container 
                onDrop={this.onDrop}
                getChildPayload={this.getChildPayload}
            >{
                dataList.map(k=>{
                    return <Draggable key={k.id}>
                        <div>{k.name}</div>
                    </Draggable>
                })
            }
                
            </Container>
        </div>
        );
    }
}
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

更多详细使用请参考如下gitHub地址
github demo地址：https://kutlugsahin.github.io/smooth-dnd-demo/

