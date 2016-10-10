---
title: react.js components的生命周期
date: 2016-10-10
tags:
  - react
categories:
  - react
author: 田小龙
---


## react.js components的生命周期

React提供了和以往不一样的方式来看待视图，它以组件开发为基础。组件是React的核心概念，React 允许将代码封装成组件（component），然后像插入普通 HTML 标签一样，在网页中插入这个组件。React.createClass 方法就用于生成一个组件类。对React应用而言，你需要分割你的页面，使其成为一个个的组件。也就是说，你的应用是由这些组件组合而成的。你可以通过分割组件的方式去开发复杂的页面或某个功能区块，组件是可以被复用的。

之前我们简单介绍了react的由来、特点、应用场景。以及，jsx语法糖，使用React.createClass生成自定义标签，插入节点，添加css样式，这些都是react的基础，接下来，我们继续react compenents的生命周期。
<!--more-->
组件的生命周期分成三个状态：
  Mounting：已插入真实 DOM，即Initial Render
  Updating：正在被重新渲染，即Props与State改变
  Unmounting：已移出真实 DOM，即Component Unmount


React 为每个状态都提供了两种处理函数，will 函数在进入状态之前调用，did 函数在进入状态之后调用，三种状态共计五种处理函数。
     componentWillMount()
     componentDidMount()
     componentWillUpdate(object nextProps, object nextState)
     componentDidUpdate(object prevProps, object prevState)
     componentWillUnmount()
此外，React 还提供两种特殊状态的处理函数。
 componentWillReceiveProps(object nextProps)：已加载组件收到新的参数时调用
 shouldComponentUpdate(object nextProps, object nextState)：组件判断是否重新渲染时调用

Mounting阶段：
  componentWillMount—render—componentDidMount
Updating阶段：
  componentWillReceiveProps—shouldCOmponentUpdate—componentWillUpdate—render—componentDidUpdate
Unmounting阶段：
  coponentWillUnmount


一个完整的React Component的写法应该如下：

    var myComponent = React.createClass({

        // The object returned by this method sets the initial value of this.state
        getInitialState: function(){
            return {};
        },

        // The object returned by this method sets the initial value of this.props
        // If a complex object is returned, it is shared among all component instances
        getDefaultProps: function(){
            return {};
        },

        // Returns the jsx markup for a component
        // Inspects this.state and this.props create the markup
        // Should never update this.state or this.props
        render: function(){
            return (<div></div>);
        },

        // An array of objects each of which can augment the lifecycle methods
        mixins: [],

        // Functions that can be invoked on the component without creating instances
        statics: {
            aStaticFunction: function(){}
        },

        // -- Lifecycle Methods --

        // Invoked once before first render
        componentWillMount: function(){
            // Calling setState here does not cause a re-render
        },

        // Invoked once after the first render
        componentDidMount: function(){
            // You now have access to this.getDOMNode()
        },

        // Invoked whenever there is a prop change
        // Called BEFORE render
        componentWillReceiveProps: function(nextProps){
            // Not called for the initial render
            // Previous props can be accessed by this.props
            // Calling setState here does not trigger an an additional re-render
        },

        // Determines if the render method should run in the subsequent step
        // Called BEFORE a render
        // Not called for the initial render
        shouldComponentUpdate: function(nextProps, nextState){
            // If you want the render method to execute in the next step
            // return true, else return false
            return true;
        },

        // Called IMMEDIATELY BEFORE a render
        componentWillUpdate: function(nextProps, nextState){
            // You cannot use this.setState() in this method
        },

        // Called IMMEDIATELY AFTER a render
        componentDidUpdate: function(prevProps, prevState){
        },

        // Called IMMEDIATELY before a component is unmounted
        componentWillUnmount: function(){
        }
    });
  * * *
