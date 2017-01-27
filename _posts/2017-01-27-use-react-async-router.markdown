---
layout: post
title: 使用 react 异步路由按需加载组件
date: "2017-01-27"
---

今天在刷 GITHUB 的时候无意间发现 [react-async-router](https://github.com/QianmiOpen/react-async-router) 这一个项目，我意识到原来组建也可以按需加载。

我之前写的项目都是直接合并到一个文件，
若碰到一个较复杂的应用合并到一个文件会很大，
加载很长时间影响用户体验。

按需加载可以很好的解决这一个问题，
我一口气就把它给 clone 下来，阅读其代码，
然后找了个之前写的项目，添加 bundle-loader (`yarn add bundle-loader`)，
随手写了一个高阶组件 `AsyncLoader`。

{% highlight javascript %}
import React, { Component } from 'react';

export default function (component) {
  return class AsyncLoader extends Component {
    state = {
      Component: null
    }
    componentDidMount() {
      component((com) => {

        this.setState({ Component: com.default || com });
      });
    }
    render() {
      const { Component } = this.state;
      if (Component) {
        return <Component {...this.props} />;
      } else {
        return <div> 正在加载中... </div>;
      }
    }
  }
}
{% endhighlight %}

然后更改路由组件 `import` 的方法

{% highlight javascript %}
import XXXX from './XXXX';
// 改为
import _XXXX from 'bundle?lazy!./XXXX';
const XXXX = AsyncLoader(_XXXX);
{% endhighlight %}

重新 build 一下 发现多了好多 `chunk` 文件，浏览器看一下，果然是按需加载。
