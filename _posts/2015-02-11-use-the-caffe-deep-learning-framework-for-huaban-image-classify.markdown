---
layout: post
title: "caffe 在花瓣的使用"
date: 2015-02-11
---

有这么一种场景，我们需要对图片进行标注，比如打 tag 呀，分类呀......

可以用机器来完成吗？答案是可以的。

[caffe](http://caffe.berkeleyvision.org/) 是一个深度学习的框架，
它提供了一些 [models](https://github.com/BVLC/caffe/tree/master/models)，
很方便让工程师或研究员重新训练和使用。

经过多次尝试，我们决定使用 CaffeNet 做图片分类的网络。

以[花瓣发现](http://huaban.com/discovery/)为例子吧

首先我们要收集标注数据
------------------

{% highlight text %}
# file: explore.txt
日系简约风格家居	explore/rixijianyue
针织	explore/zhenzhi
不织布	explore/buzhibu
蕾丝	explore/leisi
软陶	explore/ruantao
临摹练习	explore/linmolianxi
水彩手绘	explore/shuicaishouhui
萝莉	explore/luoli
混搭	explore/hunda
刺绣	explore/cixiu
萌娃	explore/mengwa
设计配色	explore/shejipeise
...
...
{% endhighlight %}

{% highlight python %}
# file: get_huaban_images.py
import requests
import os

ROOT_PATH = 'datasets'
LIMIT = 1000

def get_pins(uri):
    api = 'http://api.huaban.com/{}'.format(uri)
    url = api

    count = 0
    while True:
        rsp = requests.get(url, headers={
            'Accept': 'application/json',
            'User-Agent': 'Huabot/1.0'})
        print(rsp)
        pins = rsp.json()['pins']
        if len(pins) == 0:
            break

        min_pin_id = pins[0]['pin_id']
        for pin in pins:
            if min_pin_id > pin['pin_id']:
                min_pin_id = pin['pin_id']

            download_pin(uri, pin)
            count += 1

        url = api + '?limit=100&max=%s'%min_pin_id

        if count > LIMIT:
            break


def download_pin(path, pin):
    dirctory = os.path.join(ROOT_PATH, path)

    if not os.path.isdir(dirctory):
        os.makedirs(dirctory)

    fn = os.path.join(dirctory, str(pin['file_id']) + '.jpg')
    print('Download:', fn)
    if os.path.isfile(fn):
        return

    file_url = 'http://img.hb.aicdn.com/' + pin['file']['key'] + '_fw320'
    rsp = requests.get(file_url)
    data = rsp.content
    with open(fn, 'wb') as f:
        f.write(data)


explores = [explore.split('\t')
            for explore in open('explore.txt', 'r').read().split('\n') if explore]

for explore in explores:
    uri = explore[1].strip()
    get_pins(uri)

{% endhighlight %}

然后执行 `python get_huaban_images.py` 接下来就是漫长的等待中......

准备训练数据和验证数据
-----------------

我们把前面下载下来的数据分为两类 `train.txt` 和 `val.txt`
{% highlight python %}
# file: create_train.py
import glob
import random

filelist = []
VAL_LENGTH = 10000

filelist = glob.glob('datasets/explore/*/*')

random.shuffle(filelist)
# filelist = filelist[:3000]

filelist = [fn.split('/', 2)[2] for fn in filelist]
cats = [fn.split('/', 1)[0] for fn in filelist]
cats = list(set(cats))

test = filelist[-VAL_LENGTH:]
filelist = filelist[:-VAL_LENGTH]

with open('cats.txt', 'w') as f:
    f.write('\n'.join(cats))

with open('train.txt', 'w') as f:
    for fn in filelist:
        cat = fn.split('/', 1)[0]
        cat_id = cats.index(cat)
        f.write("explore/{} {}\n".format(fn, cat_id))

with open('val.txt', 'w') as f:
    for fn in test:
        cat = fn.split('/', 1)[0]
        cat_id = cats.index(cat)
        f.write("explore/{} {}\n".format(fn, cat_id))
{% endhighlight %}

一样的执行 `python create_train.py`, 这样就把数据集分成两部分, cats.txt 我们是用来保存分类的信息

将数据写到 lmdb 里面
-----------------
{% highlight bash %}
# file: create_huaban.sh
TOOLS=/path/to/caffe/build/tools
TRAIN_DATA_ROOT=datasets/
VAL_DATA_ROOT=datasets/
RESIZE_HEIGHT=256
RESIZE_WIDTH=256

$TOOLS/convert_imageset \
    --resize_height=$RESIZE_HEIGHT \
    --resize_width=$RESIZE_WIDTH \
    --shuffle \
    $TRAIN_DATA_ROOT \
    train.txt \
    huaban_train_lmdb

$TOOLS/convert_imageset \
    --resize_height=$RESIZE_HEIGHT \
    --resize_width=$RESIZE_WIDTH \
    --shuffle \
    $TRAIN_DATA_ROOT \
    val.txt \
    huaban_val_lmdb
{% endhighlight %}

计算训练数据集的均值
----------------
{% highlight bash %}
/path/to/caffe/build/tools/compute_image_mean huaban_train_lmdb huaban_mean.binaryproto
{% endhighlight %}

修改 [train_val.prototxt](https://github.com/BVLC/caffe/blob/master/models/bvlc_reference_caffenet/train_val.prototxt)
----------------------

- 将 mean_file 改为 huaban_mean.binaryproto
- 将 TRAIN source 改为 huaban_train_lmdb
- 将 TEST source 改为 huaban_val_lmdb

修改 [solver.prototxt](https://github.com/BVLC/caffe/blob/master/models/bvlc_reference_caffenet/solver.prototxt)
---------
{% highlight text %}
net: "train_val.prototxt"
test_iter: 50
test_interval: 100
base_lr: 0.01
lr_policy: "ploy"
gamma: 0.1
stepsize: 1000
display: 20
max_iter: 450000
momentum: 0.9
weight_decay: 0.0005
snapshot: 1000
snapshot_prefix: "models/huaban_train"
solver_mode: CPU
{% endhighlight %}

训练网络
------

{% highlight bash %}
/path/to/caffe/build/tools/caffe train --solver=solver.prototxt
{% endhighlight %}

可恶又要进行漫长的等待......

训练结果
------
经过长达半个月的训练:

{% highlight bash %}
Iteration 40000, Testing net (#0)
    Test net output #0: accuracy = 0.354531
    Test net output #1: loss = 2.85665 (* 1 = 2.85665 loss)
{% endhighlight %}

![](/images/caffe-train-result.png)
35.4% 的准确率，可以接受的。

预测图片的分类
-----------

- [deploy.prototxt](https://github.com/BVLC/caffe/blob/master/models/bvlc_reference_caffenet/deploy.prototxt)

- huaban.caffemodel --  huaban_train_iter_40000.caffemodel
{% highlight python %}
# file: classify_image.py
import numpy as np
import caffe

model_def_file = 'deploy.prototxt'
pretrained_model_file = 'huaban.caffemodel'
mean_file = 'huaban_mean.binaryproto'
label_file = 'cats.txt'

MAX_PREDICT_LENGTH = 5

def load_binaryproto(fn):
    blob = caffe.proto.caffe_pb2.BlobProto()
    data = open(fn, 'rb').read()
    blob.ParseFromString(data)
    return np.array(caffe.io.blobproto_to_array(blob))[0]

def load_labels(fn):
    labels = [line.strip() for line in open(fn).read().split() if line.strip()]
    return np.array(labels)

def classify(image):
    net = caffe.Classifier(model_def_file,
                           pretrained_model_file,
                           image_dims=(256, 256),
                           raw_scale=255.,
                           mean=load_binaryproto(mean_file),
                           channel_swap=(2, 1, 0))

    labels = load_labels(label_file)
    scores = net.predict([image], oversample=True).flatten()
    indices = (-scores).argsort()[:MAX_PREDICT_LENGTH]
    predictions = labels[indices]
    meta = [(p, '%.5f'%scores[i]) for i, p in zip(indices, predictions)]
    return meta

if __name__ == '__main__':
    import sys
    fn = sys.argv[1]
    image = caffe.io.load_image(open(fn))
    meta = classify(image)
    print(meta)

{% endhighlight %}

最后
---

到此为止，你已经了解 Huaban Brain 的基本情况了。
当然在生产环境中还需要更多的考虑，这只是冰山一角而已。
