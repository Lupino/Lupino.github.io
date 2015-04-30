---
layout: post
title: "node 内存泄漏分析"
date: 2015-01-23
---

内存泄漏从直觉上来讲，肯定是某些对象被被创建后没有及时销毁。
所以现在的目标是要找出那些对象，然后进行处理。
这里我们使用的是 `node-inspector`



先来个例子：(`leak.js`)
{% highlight javascript %}
function LeakingClass() {

}

var leaks = [];
setInterval(function() {
    for (var i = 0; i < 100; i++) {
        leaks.push(new LeakingClass);
    }
    console.error('Leaks: %d', leaks.length);
}, 1000);
{% endhighlight %}


{% highlight bash %}
npm install -g node-inspector
node --debug leak.js
node-inspector
{% endhighlight %}

使用 chrome 浏览器访问 `http://127.0.0.1:8080/debug?port=5858`

这时你可以看到 chrome 开发者工具的窗口：
![](/images/node-inspector-profile.png)

- 选择 "Profile"
- 选择 "Take Heap Snapshot"
- 然后 "Take Snapshot"
- 如此快照几次
- 选择 "Objects Count" 最后看到如下 `LeakingClass` 泄漏了

![](/images/node-inspector-heap-snapshot.png)

参考文献：

- [Node内存泄漏专题](https://cnodejs.org/topic/4fa94df3b92b05485007fd87)
- [A tutorial for debugging memory leaks in node](https://github.com/felixge/node-memory-leak-tutorial)
