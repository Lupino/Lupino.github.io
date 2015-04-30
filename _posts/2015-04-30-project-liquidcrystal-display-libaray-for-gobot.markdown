---
layout: post
title: "PROJECT: LiquidCrystal display libaray for gobot"
date: 2015-04-30
---

最近在使用 [Gobot](http://gobot.io) 发现居然不支持 LCD，
手上还有一个 LCM1602 的 LCD 板子，而且在 [Arduino](http://arduino.cc) 板子上有
[I2C LCM1602](https://github.com/fdebrabander/Arduino-LiquidCrystal-I2C-library)
的库，遂就写了一个 <https://github.com/Lupino/LiquidCrystal>

硬件准备
------

- Arduino 板子
- LCM1602 LCD
- 一台 PC or Macbook (PS: 我用的是 Raspberry pi)

硬件连接
------

如下图连接硬件，通过 usb 线连接电脑。
![](/images/LCM1602_bb.png)

### 1、 安装 gort

{% highlight bash %}
$ go get -v github.com/hybridgroup/gort
{% endhighlight %}
### 2、 上传 firmata

{% highlight bash %}
$ gort scan serial
1 serial port(s) found.

1 [/dev/ttyACM0] - [usb-Arduino__www.arduino.cc__0043_7543730383035110B2C1-if00]

$ gort arduino upload firmata /dev/ttyACM0
{% endhighlight %}

### 3、Get LiquidCrystal.

{% highlight bash %}
$ go get -v github.com/Lupino/LiquidCrystal
{% endhighlight %}

### 4、Hello World
{% highlight go %}
# file: helloworld.go
package main

import (
	"github.com/Lupino/LiquidCrystal"
	"github.com/hybridgroup/gobot"
	"github.com/hybridgroup/gobot/platforms/firmata"
)

func main() {
	gbot := gobot.NewGobot()

	firmataAdaptor := firmata.NewFirmataAdaptor("firmata", "/dev/ttyACM0")
	lcd := LiquidCrystal.NewLiquidCrystalDriver(firmataAdaptor,
		"LiquidCrystal",
		0x27,
		16,
		2)

	work := func() {
		lcd.Print("Hello World!")
	}

	robot := gobot.NewRobot("LiquidCrystal",
		[]gobot.Connection{firmataAdaptor},
		[]gobot.Device{lcd},
		work,
	)

	gbot.AddRobot(robot)

	gbot.Start()
}
{% endhighlight %}

### 6、测试
{% highlight bash %}
$ go run main.go
2015/04/30 11:22:09 Initializing Robot LiquidCrystal ...
2015/04/30 11:22:09 Initializing connections...
2015/04/30 11:22:09 Initializing connection firmata ...
2015/04/30 11:22:09 Initializing devices...
2015/04/30 11:22:09 Initializing device LiquidCrystal ...
2015/04/30 11:22:09 Starting Robot LiquidCrystal ...
2015/04/30 11:22:09 Starting connections...
2015/04/30 11:22:09 Starting connection firmata on port /dev/ttyACM0...
2015/04/30 11:22:23 Starting devices...
2015/04/30 11:22:23 Starting device LiquidCrystal...
2015/04/30 11:22:24 Starting work...
{% endhighlight %}

### 更多例子
<https://github.com/Lupino/LiquidCrystal/tree/master/examples>
