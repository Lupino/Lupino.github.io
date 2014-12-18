---
layout: post
title: "画 caffe 网络连接图"
date: 2014-12-18 11:56:58
---

分析 caffe 网络定义的时候看那配置文件着实难受，想想用 [graphviz](http://graphviz.org/) 把图给画了就容易理解多了。

{% highlight python %}
#!/usr/bin/env python

import sys

if len(sys.argv) < 2:
    print("%s intput [output]"%sys.argv[0])
    sys.exit()

fn = sys.argv[1]

output = ''
if len(sys.argv) == 3:
    output = sys.argv[2]

import os

pro = open(sys.argv[1], 'r').read()

lines = pro.split('\n')
layers = []
layer = []
for line in lines:
    if line.startswith('layers'):
        layers.append(layer)
        layer = [line]
    else:
        layer.append(line)

layers.append(layer)

graph = []
for layer in layers:
    if isinstance(layer, list):
        name = ''
        top = []
        bottom = []
        label = ''
        for line in layer[1:-1]:
            line = line.split(':', 1)
            if len(line) == 1:
                continue
            k = line[0].strip(':," ')
            v = line[1].strip(':," ')

            if k == 'name':
                name = v

            elif k == 'top':
                top.append(v)

            elif k == 'bottom':
                bottom.append(v)

            elif k == 'type' and not label:
                label = v

        for t in top:
            graph.append("    %s -> %s"%(name, t))

        for b in bottom:
            graph.append("    %s -> %s"%(b, name))

        if label:
            graph.append('    %s[label="%s:\n%s"]'%(name, name, label))


graph = list(set(graph))


graph.insert(0, 'digraph G {\n    graph [layout=dot rankdir=LR]')
graph.append('}')

with open('/tmp/layout.dot', 'w') as f:
    f.write('\n'.join(graph))
if output:
    os.system("dot -Tpdf /tmp/layout.dot > %s"%output)
else:
    os.system("dot -Tpdf /tmp/layout.dot")
{% endhighlight %}

## 来两张看看
* caffenet 训练图
![](/images/caffenet_train_val.svg)
* caffenet 分类图
![](/images/caffenet_deploy.svg)
