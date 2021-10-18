# 如何优雅地让多个版本的nodejs共存？

```

```



author: 星光

weixi:xingguangzfs



我们都知道，近几年 nodejs 大流行，曾经有一段时间，甚至有 JavaScript 一统天下的趋势，当然现在这种趋势在下降，我认为导致这种结果最主要的原因是还是性能。



简单的讲，nodejs 是基于 chrome v8 引擎的 JavaScript 运行环境，有了它，Javascript 即可以开发前端程序，又可以开发后端程序，这就产生了一种局面，就是会 Javascript ，就可以一个人完成一个项目，这对于小公司来说，是一种巨大的恩惠。



在用 nodejs 开发项目的时候，通常不同的项目使用的 nodejs 版本是不一样的，而且通常如果版本跨度比较大的话，往往导致版本之间兼容性差，甚至不兼容的情况产生，那么如何优雅的在多个 nodejs 版本之间切换呢？这时就要用到下面要介绍的工具 nvm。



nvm 全称 node.js version management ， 是开源的 nodejs 版本管理工具，windows 版本的 nvm 可以在下面地址下载：

https://github.com/coreybutler/nvm-windows/releases



如果下载不了，也可以找我要，微信：xingguangzfs



**（1）安装nvm**

下载好 nvm 后，首先就是安装，我下载的是 exe 安装包，安装很简单，双击打开后，一直 “下一步” ，直到安装完成即可。



安装完后，在 windows 开始菜单中找到 “命令提示符”，最好以管理员方式打开，然后输入命令：nvm  version

显示出 nvm 版本信息说明安装成功，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/GxHp5MAY3ZBdRlLibsThMt55IwesWeZibJSMJw6N5brgZPm1r5opUAksypsorCJ6EzNk9fyoQoeBhJ8EpTgVyicSA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



接着输入命令：nvm ls

查看已经安装的 nodejs 版本列表，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/GxHp5MAY3ZBdRlLibsThMt55IwesWeZibJ6Uq5b7cA2O1xegRpI2EeLKyRA2VLbzcpz6AlmWEy6xmy7yA5GUwKug/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

因为是新安装，所以没有任何版本。



**（2）安装 nodejs**

接下来安装 nodejs ，安装时必须指定版本号，比如安装8.12.0版本输入：

nvm install 8.12.0

安装会先下载，再安装，需要一点时间，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/GxHp5MAY3ZBdRlLibsThMt55IwesWeZibJdp7xx2YWEFz74CTFBgcwQ6QKy5icyVOibvUTuicpjn5h0CnMgG1WgHObg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



此时再输入查询命令：nvm ls

显示如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/GxHp5MAY3ZBdRlLibsThMt55IwesWeZibJQdJt5bIcrUB5DaB0h9wGuZarGC3QESW0GS5g28K3SdHgllfZbjOFvg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**（3）指定要使用的 nodejs 版本**

通过输入：nvm ls 查看已经安装的 nodjs 版本列表，

然后输入：nvm use 8.12.0 指定当前使用的 nodejs 版本，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/GxHp5MAY3ZBdRlLibsThMt55IwesWeZibJjaTDQxc7nJByibFxpnbdCTiavzeNETuTTZ31UYMc1FlaA7TZrdUknl8g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



接着可以输入：node -v 查看当前使用的 nodejs 版本。



**（4）总结**

当需要管理多个 nodejs 版本时，可以使用 nvm 工具非常方便优雅的进行管理，管理过程大致分为三步：

第一步：安装 nvm，直接用 nvm-setup.exe 安装包;

第二步：用 nvm 安装指定的 nodejs  版本，命令：nvm install 8.12.0

第三步：用 nvm 指定当前使用的 nodejs 版本，命令：nvm use 8.12.0



**（5）附录**

常用 nvm 命令表：

[1] 安装指定的 nodejs 版本：nvm install 8.12.0

[2] 查看已经安装的 nodejs 列表：nvm ls

[3] 指定当前使用的 nodejs 版本：nvm use 8.12.0

[4] 查看当前使用的 nodejs 版本：node -v