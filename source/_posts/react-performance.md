---
title: react性能优化
date: 2020-07-25 11:59:22
img: /medias/featureimages/9.jpg
top: false
cover: true
coverImg: /medias/featureimages/9.jpg
toc: true
mathjax: false
summary: 在一个庞大的项目中，后期我们一定会去思考怎样提高react的性能，下面是我平时开发总结出来的一些经验。
categories: 前端
tags:
- React
- 性能优化
---

#### React组件性能优化

React是一个专注于UI层的框架，它使用虚拟DOM技术，以保证它UI的高速渲染；使用单向数据流，因此它数据绑定更加简单；那么它内部是如何保持简单高效的UI渲染呢？这种渲染机制有可能存在什么性能问题呢？



##### 1.render函数
  (1)在调用组件时，如果某个属性值是函数，避免使用箭头函数，不然每次比较props中该属性时都是不同的。
  正确的做法
```javascript
  import React, { Component } from 'react';
  class Demo extends Component {
    handleClick = () =>{
      //TODO
    }
    render() {
        return (
            <ChildDom  onClick={this.handleClick} /> //构造函数每一次渲染的时候只会执行一遍；
        );
    }
  }
```
不推介例子：
```javascript
  <ChildDom  onClick={()=>{this.handleClick()}} /> //都会生成一个新的箭头函数，即使两个箭头函数的内容是一样的。
  <ChildDom  onClick={this.handleClick.bind(this)} /> //在每次render()的时候都会重新执行一遍函数；
```
  (2)在shouldComponentUpdate生命周期中进行比较，减少render函数刷新次数。
  使用浅比较时，如果是对象类型需要特别注意（这里也可以使用ladash库来实现）
  demo:
```javascript
  shouldComponentUpdate(nextProps, nextState) {
    if (
      JSON.stringify(nextState.rosterList) === JSON.stringify(this.staterosterList)
    ){
      return false;
    }
    return true;
  }
```
使用React.memo进行组件记忆；
纯函数的话也可以使用React.memo来实现类似的功能。或者可以减少不必要的state参数使用。

##### 2.key值的使用

  (1)react组件在装载过程中，react通过在render方法在内存中产生一个树形结构，树上的节点代表一个react组件或者原生的Dom元素，这个树形结构就是我们所谓的Vitural Dom，react根据这个来渲染产生浏览器的Dom树。
  (2)react在更新阶段对比原有的Vitural Dom和新生成的Vitural Dom，找出不同之处，在根据不同来渲染Dom树。
  (3)react为了追求高性能，采用了时间复杂度为O(N)来比较两个属性结构的区别，因为要确切比较两个树形结构，需要通过O(N^3),这会降低性能.

##### 3.减少使用state来共享数据,跨组件使用数据是推介使用redux。
  如果有如下情况意味着你要使用redux
  1.组件首其他不相关组件的状态控制
  2.组件之间不是子父组件关系
  3.许多不相关的组件以相同的方式更新状态
  4.状态以许多不同的方式更新

##### 4.组件懒加载
1、webpack+es6的import(this.props.children为回调函数);

```javascript

import React , { Component } from 'react';

export default class  extends Component {
    constructor ( props ) {
        super ( props );
        this.load(props); //调用下面load
        this.state={
            Com:null
        };
    };
    load(props){ //this.props.load()就是调用indexrou.jsx传过来的函数
        props.load().then((Com)=>{
           console.log(Com.default);//得到的就是传过来的函数
            this.setState({
                Com:Com.default?Com.default:null
            });
        });
    };
    render () {
        if(!this.state.Com){
            return null;
        }else{
            return this.props.children(this.state.Com);
        }
    };
};

```
```javascript
import Load from '../components/lazy';

let Demo2=function(){
    return <Load load={()=>import('../components/demo2')}>
        {(Com)=><Com/>} 
    </Load>;
};
```

2、webpack+es6的import纯粹的高阶组价
```javascript
import React , { Component } from 'react';

export default function(loading){//传过来一个函数
    return class extends Component {
        constructor ( props ) {
            super ( props );
            this.state={
                Com:null
            };
            this.load();
        };
        load(props){
            loading().then((Com)=>{  //调用函数获取它传过来的路径
                this.setState({
                    Com:Com.default?Com.default:null
                });
            });
        };
        render () {
            let Com=this.state.Com;
            return Com?<Com/>:null;
        };
    };
}
```
在router路由里 indexrou.js
```javascript
import Load from '../components/lazy';
let Demo2=Load(()=>import('../components/demo2'));

```
3、webpack+es6的import +async（高阶函数）
```javascript
import React , { Component } from 'react';

export default function(loading){
    return class extends Component {
        constructor ( props ) {
            super ( props );
            this.state={
                Com:null
            };
        };
        //即使是同步的话执行的也是promise.resolve这个方法，将同步代码包装一层，进行同步
        //await后面接收的是值或promise
        async componentWillMount(){
            let Com=await loading();  //依次执行，只有一个await往下走，Com是有值的
            this.setState({
                Com:Com.default?Com.default:null
            });
        };
        render () {
            let Com=this.state.Com;
            return Com?<Com/>:null;
        };
    };
}
```
在router路由里 indexrou.jsx

```javascript
import Load from '../components/lazy';
let Demo2=Load(()=>import('../components/demo2'));
```

4、webpack+require.ensure (高阶组价)
```javascript
import React , { Component } from 'react';

export default function(loading){
    return class extends Component {
        constructor ( props ) {
            super ( props );
            this.state={
                Com:null
            };
        };
        componentWillMount(){
            new Promise((resolve,reject)=>{
                require.ensure([], function(require) {//[]依赖项
                    var c = loading().default;
                    console.log(c);
                    resolve(c);
                });
            }).then((data)=>{
                this.setState({
                    Com:data
                });
            });
        };
        render(){
            let Com=this.state.Com;
            return Com?<Com/>:null;
        };
    };
};
```
在router路由里 indexrou.jsx

```javascript
  import Load from '../components/lazy';

  let Demo2=Load(()=>
    require('../components/demo2')
  );

````

##### 5.使用 React Fragments 避免额外标记

```javascript
radiosBtns()
    return(
        <React.Fragment>
            <RadioButton value="1" style={{marginLeft:'10px'}}>
                <Icon type="flag" className={flag(1)+' fs16'} />
            </RadioButton>
            <RadioButton value="2" style={{marginLeft:'10px'}}>
                <Icon type="flag" className={flag(2)+' fs16'} />
            </RadioButton>
            <RadioButton value="3" style={{marginLeft:'10px'}}>
                <Icon type="flag" className={flag(3)+' fs16'}/>
            </RadioButton>
            <RadioButton value="4" style={{marginLeft:'10px'}}>
                <Icon type="flag" className={flag(4)+' fs16'}/>
            </RadioButton>
            <RadioButton value="5" key="5" style={{marginLeft:'10px'}}>
                <Icon type="flag" className={flag(5)+' fs16'}/>
            </RadioButton>
        </React.Fragment>
    )
}
```
##### 6.不要使用内联函数定义
eg: 一个内联的“DOM组件”事件处理程序
```javascript

class Demo extends Component {
  render() {
    return (
      <div>
        <button
          onClick={() => {
            this.setState({ clicked: true })
          }}
        >
          XXX
        </button>
      </div>
    )
  }
}
```
##### 7.优化 React 中的条件渲染
  可以使用三元，if-else，and运算符(&&)。推荐使用&&来表示；
  尽量使用return null来判断显隐，而不是用css的display：none，避免回流；
```javascript
  render(){
    let con = this.state.goods[0].text ? <h1>{this.state.goods[0].text}</h1>:null   //条件渲染 
    return(
        <div>
          {con}
        </div>
    )
  }
```
##### 8.事件节流和防抖
（参考节流防抖功能实现，这里不多赘述）

##### 9.不要在render中生成新的引用
定义函数、使用内联样式或者动态生成一些不依赖state或props的jsx这些，凡是不依赖state或props的都应该提到render之外，否则会造成每次render的时候生成新的引用，会导致在diff算法对比属性或节点的过程中发现两个引用不一致，对于react来说，这说明属性值发生了改变，最后会被替换成新的引用，造成性能浪费。例如函数应该放在render函数外定义，样式应该单独建一个文件，然后引入，使用生命周期函数或者自定义函数动态生成一些不依赖state或props的jsx。
```javascript
import React, { Fragment } from 'react';
import 'antd/dist/antd.css'
import {Input,Button,List} from 'antd'
 const TodoListUI=(props)=>{
     return (
        <Fragment>
            <div className="form-box">
            <Input placeholder="请输入" value={props.inputValue} onChange={props.handleChange}style={{width:'300px'}}/>
            <Button type="primary" style={{marginLeft:'20px'}} onClick={props.handleSubmit}>提交</Button>
        </div>
        <div  className="list">
            <List
                size="small"
                bordered
                dataSource={props.list}
                renderItem={(item,index) => (<List.Item onClick={()=>{props.handleDelete(index)}}>{item}</List.Item>)}
            />
        </div>      
        </Fragment>
     )
 }

export default TodoListUI;
```
##### 10.ProtoTypes 的验证会损耗性能
  建议用环境变量控制在生产环境关闭类型验证

```javascript
  IncomingTasks.propTypes = {
    reTryGetData: PropTypes.func,
  }
```

##### 11.尽量少直接操作dom
  用setTimeout或不可控的refs等dom操作；
  setTimeout会导致state多次更新，refs会直接操作dom，消耗大量性能；
```javascript
  if(!textDom){
      textDom = document.getElementById('textArea');
  }
  return (
    <div contentEditable="true"
      id="textArea"
      onInput={this.onChangeAtPerson}
      className={style.testArea}
      placeholder="请输入评论，可以@其他人"
    />
  )
```
```javascript
  return (<div ref={com => (this.refs = com)}>
    //TODO
  </div>)
```