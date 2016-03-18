---
layout: post
title: 在 arduino  上面玩 flappy
date: "2016-03-18"
---

前几天无意间在某宝上买了一个 2.6 寸 TFT液晶触摸彩屏，到货后研究了老半天不知道咋用
Google 了许久未搞定，最后找了技术支持，给了个资料库，然后搞定了。
对比代码发现原来我用的库的例子少了一个 `0x9341` 的判断。

相应的库： [Adafruit-GFX-Library](https://github.com/adafruit/Adafruit-GFX-Library),[TFTLCD-Library](https://github.com/adafruit/TFTLCD-Library),[Touch-Screen-Library](https://github.com/adafruit/Touch-Screen-Library)

为了这一个显示器我可是把 `arduino` 的亚克力外壳给破坏了，能用可是不够的。
于是又 Google 一下，发现有个牛人（mrt-prodz）写了
 [ATmega328-Flappy-Bird-Clone](https://github.com/mrt-prodz/ATmega328-Flappy-Bird-Clone)
遂 clone 了下来试试。

阅读代码时，发现他用 `Adafruit_ST7735` 相对比较旧，应该是 `TFTLCD-Library` 早期的版本吧，所以就直接把它给替换掉，

```cpp
#include <Adafruit_GFX.h>
#include <Adafruit_TFTLCD.h> // Hardware-specific library
#include <SPI.h>

// The control pins for the LCD can be assigned to any digital or
// analog pins...but we'll use the analog pins as this allows us to
// double up the pins with the touch screen (see the TFT paint example).
#define LCD_CS A3 // Chip Select goes to Analog 3
#define LCD_CD A2 // Command/Data goes to Analog 2
#define LCD_WR A1 // LCD Write goes to Analog 1
#define LCD_RD A0 // LCD Read goes to Analog 0

#define LCD_RESET A4 // Can alternately just connect to Arduino's reset pin

// When using the BREAKOUT BOARD only, use these 8 data lines to the LCD:
// For the Arduino Uno, Duemilanove, Diecimila, etc.:
//   D0 connects to digital pin 8  (Notice these are
//   D1 connects to digital pin 9   NOT in order!)
//   D2 connects to digital pin 2
//   D3 connects to digital pin 3
//   D4 connects to digital pin 4
//   D5 connects to digital pin 5
//   D6 connects to digital pin 6
//   D7 connects to digital pin 7
// For the Arduino Mega, use digital pins 22 through 29
// (on the 2-row header at the end of the board).

// Assign human-readable names to some common 16-bit color values:
#define    BLACK   0x0000
#define    BLUE    0x001F
#define    RED     0xF800
#define    GREEN   0x07E0
#define    CYAN    0x07FF
#define    MAGENTA 0xF81F
#define    YELLOW  0xFFE0
#define    WHITE   0xFFFF

Adafruit_TFTLCD TFT(LCD_CS, LCD_CD, LCD_WR, LCD_RD, LCD_RESET);
```

 并删除 `ST7735_` 开头的颜色，换成自己定义的颜色。

 编译时报错 `#define drawPixel(a, b, c) TFT.setAddrWindow(a, b, a, b); TFT.pushColor(c)`, 所以也改成`#define drawPixel(a, b, c) TFT.drawPixel(a, b, c)`

很好编译可以通过，并下载到 `arduino` 板子上，可以显示 `FIAPPY BIRD` 了，不过是小屏幕的。

现在的目标是把它变成大屏幕的 `240x320`

```cpp
#define TFTW            240     // screen width
#define TFTH            320     // screen height
#define TFTW2           120     // half screen width
#define TFTH2           160     // half screen height
```

很好已经全屏显示 `FLAPPY BIRD`。

我的显示器可是触屏的，得把它利用起来，原来的代码是利用按钮的，所以添加代码:

```cpp
#include <TouchScreen.h>
#define YP A3  // must be an analog pin, use "An" notation!
#define XM A2  // must be an analog pin, use "An" notation!
#define YM 9   // can be a digital pin
#define XP 8   // can be a digital pin

#define MINPRESSURE 10
#define MAXPRESSURE 1000

// For better pressure precision, we need to know the resistance
// between X+ and X- Use any multimeter to read it
// For the one we're using, its 300 ohms across the X plate
TouchScreen ts = TouchScreen(XP, YP, XM, YM, 300);
```

并通过 `TSPoint p = ts.getPoint();`  来判断屏幕是否被按下。

修改完后编译下载运行，一点击屏幕就白屏这下子蛋疼了。
接下来就是漫长的调试中。

最后发现是数据类型错误导致死机，
原来 mrt-prodz 用的是 `char` 而这对于 `128x160` 来讲足够了，
`320` 已将超过 `256`, 所以就不行了，所以我通通改成 `int`。

可以玩了，第一台游戏机终于搞定，于是我在 arduino 上面玩 flappy。
