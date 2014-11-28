---
layout: post
title: "Mahotas: Computer Vision in Python"
date: 2014-11-28
---

Mahotas 是一个 Python 的图像处理库，包含大量的图像处理算法，使用 C++ 实现的算法，处理性能相当好。

# 安装 Mahotas

{% highlight bash %}
# create a python venv
virtualenv -p python venv
source venv/bin/activate

# first install the reqirements
easy_install numpy imread
easy_install mahotas
easy_install matplotlib
{% endhighlight %}

# 先来个例子
{% highlight python %}
from matplotlib import pylab as plt
import numpy as np
import mahotas as mh

f = np.ones((256,256), bool)
f[200:,240:] = False
f[128:144,32:48] = False
# f is basically True with the exception of two islands: one in the lower-right
# corner, another, middle-left

dmap = mh.distance(f)
plt.imshow(dmap)
plt.show()
{% endhighlight %}

![](/images/mahotas-a-distance-transform-example.png)
