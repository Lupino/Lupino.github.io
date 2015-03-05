---
layout: post
title: "Huabot Brain 一个简单易用的图片识别系统"
date: 2015-03-05
---

关于 Huabot Brain
================

Huabot Brain 可以说是 [caffe](http://caffe.berkeleyvision.org/) 的 GUI 工具。
Huabot Brain 提供 restful api 和一个基于 react 的 web 页面，使得开发者或者研究人员可以
方便的使用 caffe，查看训练状态，提供 demo 和应用。
可作为一个嵌入式的子系统，通过简单的 restful api 给原有的系统提供深度学习的能力。

来几张截图：

1、训练曲线
---------
![训练曲线](/images/train-line.png)

2、数据页面
---------
![](/images/huabot-brain-waterfal.png)

3、Demo 页面
---------------
![](/images/huabot-brain-demo.png)

安装 Huabot Brain
================

* 安装 golang <http://golang.org/doc/install>
* 安装 caffe <http://caffe.berkeleyvision.org/installation.html>
* 安装 gnuplot
* 安装 Huabot Brain <https://github.com/Lupino/huabot-brain#install>

{% highlight bash %}
# 设置 GOPATH
export GOPATH=$HOME/gopath
export PATH=$GOPATH/bin:$PATH

# 安装 Huabot Brain
go get -v github.com/Lupino/huabot-brain
# 安装 Train worker
go get -v github.com/Lupino/huabot-brain/tools/train_worker
{% endhighlight %}

启动 Huabot Brain
================

{% highlight bash %}
# 启动 Gearman job server
gearmand -d

# 启动 Huabot Brain (默认端口 3000)
cd $GOPATH/src/github.com/Lupino/huabot-brain
huabot-brain --gearmand=127.0.0.1:4730 --dbpath=dataset.db

# 启动 Train worker
train_worker --gearmand=127.0.0.1:4730

# 启动 Predict Worker (这一步要等训练一端时间或完成后，才可执行， xxx 为数字)
ln -s resourses/models/huabot-brain_iter_xxx.caffemodel resourses/models/huabot-brain.caffemodel
env GEARMAND_PORT=tcp://127.0.0.1:4730 python tools/predict_worker/main.py resources
{% endhighlight %}

提交数据
=======

{% highlight bash %}
# 用 curl 提交数据
curl -XPOST -F file=@/path/to/image.png -F tag=xxx -F data_type=1 http://127.0.0.1:3000/api/datasets

# 用脚本提交数据
cd tools/datasets
python get_datasets.py
{% endhighlight %}

自己的数据集请使用 [API](https://github.com/Lupino/huabot-brain/blob/master/API.md) 提交

训练 Huabot Brain
================

打开 [Dashboard](http://127.0.0.1:3000/#/dashboard) 点击 Train 就可以了。
也可以用命令

{% highlight bash %}
curl http://127.0.0.1:3000/api/train
{% endhighlight %}

详细信息请见 [API 文档](https://github.com/Lupino/huabot-brain/blob/master/API.md)

预测图片的信息
===========

但这里需要启动 predict worker。

通过 [DEMO](http://127.0.0.1:3000/#/demo) 页面，在这里面贴入一个图片的地址，回车就可以。

也可以用 API:

{% highlight bash %}
curl -d img_url=http://image http://127.0.0.1:3000/api/predict
{% endhighlight %}

结束语
=====

就此饱览了 Huabot Brain 的盛宴。
