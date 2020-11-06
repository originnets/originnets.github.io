---
layout: post
title: 激活
categories: 激活
description: 激活
keywords: 激活
---


激活


日前在国外科技论坛有大神发布名为HWIDGEN激活工具，该激活工具几乎秒杀所有版本Windows 10系统。

我们知道Windows 10现在激活后会带有数字权利，数字权利可以在我们重装系统后自动激活无需再次激活。

而HWIDGEN激活工具正是直接获取数字激活权利进行永久激活，此方法激活后用户下次安装同样无需激活。
![](https://img.lancdn.com/landian/2018/05/47882-6.png)
Windows 10数字权利获取工具HWIDGEN介绍及详细使用说明

先介绍使用再介绍原理：

下载由远景论坛网友[黯然 KING](http://bbs.pcbeta.com/viewthread-1786788-1-1.html)编写的简化脚本包：[https://dl.lancdn.com/landian/software/HWIDGEN/](https://dl.lancdn.com/landian/software/HWIDGEN/)

由于借助该网友制作的脚本激活非常简单因此蓝点网不再赘述，具体直接以下面几张图片介绍整个激活流程。

**注意事项 : 激活前电脑必须已经能够联网并且必须未禁用Windows Update服务，如已禁用请先开启该服务。**

![](https://img.lancdn.com/landian/2018/05/47882-1.png)

完整解压压缩包后右键点击Activation.CMD文件并选择使用管理员身份运行，然后会提示工具支持的版本。

接下来有几个选项默认情况下我们直接填写数字1来激活本机系统，输入后脚本自动工作联网获取数字权利。

最后执行完毕后脚本会提示你系统已经激活，至此激活完毕, 若前往系统激活选项会看到已经获得数字权利。
![](https://img.lancdn.com/landian/2018/05/47882-2.png)
![](https://img.lancdn.com/landian/2018/05/47882-3.png)
![](https://img.lancdn.com/landian/2018/05/47882-4.png)
![](https://img.lancdn.com/landian/2018/05/47882-5.png)
![](https://img.lancdn.com/landian/2018/05/47882-7.png)

HWIDGEN获得数字权利的激活原理：

说到激活原理自然先得继续介绍Windows 10系统的数字权利，所谓数字权利即与已系统绑定的激活许可证。

默认情况下当Windows 10被激活后会自动生成与硬件ID对应的许可证，该许可证会存储到微软的服务器上。

当系统重新安装时自动将硬件ID提交给微软检索对应的许可证，若许可证符合则系统自动激活无需用户操作。

至于 HWIDGEN 是如何通过修改系统内核数据来激活系统就是技术问题了，有兴趣的请看[GitHub上的脚本](https://github.com/vyvojar/slshim)。

在激活系统后同时连接微软将硬件ID对应的许可证上传，最终对于用户来说系统在激活的瞬间就有数字权利。

本方法激活的系统没有任何副作用，如果你登录微软账号的话就会自动将数字许可证绑定到你的微软账号上。

当然不论是否登录账号都不会影响数字许可证，即下次重装系统输入对应版本的激活密钥后系统将自动激活。
