
---
layout: post
title:  "论如何科学搭建梯子"
date:   2015-10-10 14:30:00
---

前段时间余弦在他的公众账号（@Lazy-Thought）中提到了一种优雅且靠谱的翻墙方案，就是利用一台海外的服务器去转发本地的网络请求，很值得尝试。所以本文为对该方案的一次探索和记录。如果你也有幸采用这种方案，希望我的记录可以帮到你。

准备工作是需要购买一台海外服务器。当然我需要小心翼翼的说出我用的是亚马逊的服务器，之所以选择这家，是因为亚马逊的流量通道中有大量的商业流量在里面，个人的流量充斥在里面不算什么，真要是被防火墙嗅探到，以亚马逊的商业博弈能力，相信会解决问题。说不定博弈来博弈去，防火墙都没了也说不定。（哈哈，我就是这么天真）

## 原理

### 防火墙是怎么工作的

`GFW`会将自己不喜欢的东西统统过滤掉，当用户触发`GFW`的过滤规则的时候，连接就会被重置。


![](http://photo-coder.b0.upaiyun.com/img/how-gfw-work01.png)


### SSH Tunnel是怎么工作的

先跟境外服务器基于`ssh`建立一条加密通道，接下来，当用户访问`GFW`的过滤规则内的内容的时候是通过建立起的隧道进行代理，实际上就是到了境外的服务器才会触发真实的网络请求。但由于创建隧道和数据传输的过程中，ssh本身的特征是比较明显，所以`GFW`一度通过分析连接的特征进行干扰，导致`ssh`存在被定向进行干扰的问题。

![](http://photo-coder.b0.upaiyun.com/img/how-gfw-work02.png)


### Shadowsocks是怎么工作的

简单来说，就是把上图常见`ssh`通道的过程拆分成为两个部分，local和server. 这样一来，创建`ssh`的过程挪到了局域网内部，`GFW`无法嗅探其特征，故无法进行干扰。

![](http://photo-coder.b0.upaiyun.com/img/how-gfw-work02.png)

## 服务器端配置

服务器端需要安装`shadowsocks`, 并作为server部分来工作:

``` 
sudo -s // 获取超级管理员权限
apt-get update // 更新apt-get
apt-get install python-pip // 安装python包管理工具pip
pip install shadowsocks // 安装shadowsocks
```

`shadowsocks`的启动依赖一份配置文件，需要创建这个文件：

``` 
sudo vi /etc/shadowsocks.json
```

如果是单个用户使用，则写入以下内容：

``` 
{
    "server":"0.0.0.0",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"myvpnpwd",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```

如果是多个用户，则按下面的格式写入：

``` 
{
    "server":"0.0.0.0",
    "port_password":
    {
        "8388": "pwd",
        "8389": "pwd",
        "8390": "pwd"
    },
    "local_address": "127.0.0.1",
    "local_port":1080,
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```

然后利用这个配置文件启动shadowsocks：

``` 
 sudo ssserver -c /etc/shadowsocks.json -d start
```
 
## 本地配置


在这个[地址](https://github.com/shadowsocks/shadowsocks/tree/master#client)下载对应的客户端。

运行客户端以后，配置上服务器的公网ip以及上面的配置文件中配置的端口以及密码，大功告成，一条梯子已经搭成了。

## 最后

其实这个方案还是有一个缺点的，恩，致命的缺点，购买一台海外服务器将是一笔不菲的费用。

> 金钱其实什么都不能给你，除了自由。——Savio

参考连接及图片版权所有：[http://vc2tea.com/whats-shadowsocks/](http://vc2tea.com/whats-shadowsocks/)

