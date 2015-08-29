---
layout: post
title: "利用 hole 服务实现内网穿透"
date: 2015-08-29
---

最近研究 [nupic-community](https://github.com/nupic-community) 的一个项目：[htmengine-traffic-tutorial](https://github.com/nupic-community/htmengine-traffic-tutorial)。 好不容易运行成功，想向朋友展示一下，发现在服务器再配置一遍实在是太麻烦了。
我想起最近开发 [hole](https://github.com/Lupino/hole) 这个项目刚好可以很好的解决这一个问题，我可以利用它实现内网穿透。

我们经常会有「把本机开发中的 web 项目给朋友看一下」这种临时需求，为此专门在 VPS 上部署一遍就有点太浪费了。之前通常是在 ADSL 路由器上配个端口映射让本机服务在外网可以访问，但现在大部分运营商不会轻易让你这么干了。一般小运营商也没有公网 IP，自己的路由器出口还是在局域网内，端口映射这种做法就不管用了。

这时候我们可以利用公网 IP 的主机中转来实现这种需求。Hole 就是为了这一目标而开发出来的，并且我还为它建立一个公共服务 [HoleHUB](http://holehub.com)。现在我展示手动搭建一套 hole 服务给自己用，一劳永逸地解决了内网穿透这个难题。下面展示这一过程。

编译 hole
---------

我的 VPS 系统是 [coreos](https://coreos.com) 所以得本地编译后上传上去或者利用 [docker](https://docker.com)。
这里我是利用 mac osx 来编译的：

{% highlight bash %}
brew install go --with-cc-common
go get -v github.com/Lupino/hole/...
GOOS=linux GOARCH=amd64 go build github.com/Lupino/hole/cmd/holed
{% endhighlight %}

然后生成证书：

{% highlight bash %}
$ hole-keys
2015/08/29 16:56:58 write to ca.pem
2015/08/29 16:56:58 write to ca.key
2015/08/29 16:56:58 write to cert.pem
2015/08/29 16:56:58 write to cert.key
2015/08/29 16:56:58 check signature true
{% endhighlight %}

最后把编译后的 `holed` `ca.pem` `ca.key` scp 到 VPS 上面。

服务端
------

Hole 支持使用证书认证和非证书认证，而我这里使用证书。

Hole 支持 udp, tcp, unix socket 方式链接，而现在演示的 web 项目是 http，tcp 的一种。
所以:

{% highlight bash %}
$ holed --help
Usage of holed:
  -addr string
        server address. (default "tcp://127.0.0.1:4000")
  -ca string
        The ca file. (default "ca.pem")
  -key string
        The ca key file. (default "ca.key")
  -use-tls
        use TLS

$ holed -addr tcp://holehub.com:4000 -ca ca.pem -key ca.key -use-tls
2015/08/29 17:09:18 Hole proxy server started on tcp://holehub.com:4000
{% endhighlight %}

客户端
------

我现在是要把本地的 :3000 端口映射出去:

{% highlight bash %}
$ hole --help
Usage of hole:
  -addr string
        Hole server address. (default "tcp://127.0.0.1:4000")
  -cert string
        The cert file. (default "cert.pem")
  -key string
        The cert key file. (default "cert.key")
  -src string
        Source server address. (default "tcp://127.0.0.1:8080")
  -use-tls
        use TLS

$ hole -addr tcp://holehub.com:4000 -cert cert.pem -key cert.key -use-tls -src tcp://:3000
2015/08/29 17:15:08 Connect Hole server: tcp://holehub.com:4000
2015/08/29 17:15:08 Process: tcp://:3000
{% endhighlight %}

到这里的话整个端口映射已经完成。只要通过 http://holehub.com:4000 就可以访问应用了。

扩展
----

朋友们没有自己的 VPS 也不用担心可以使用 <http://holehub.com> 同样可以实现内网穿透。
并且 holehub 帮你做了很多事情，穿透就更加简单了。

既然 hole 支持 tcp, udp, unix socket, 所以它还有很多玩法。比如：

* 转发 ssh 端口
* 和 shadowsocks 配合实现 local proxy
* 本地主机服务，什么智能家居...
* 其它 ...
