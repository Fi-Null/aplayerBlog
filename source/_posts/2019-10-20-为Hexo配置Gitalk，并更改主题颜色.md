---
title: 为Hexo配置Gitalk，并更改主题颜色
date: 2019-10-20 21:01:35
category:
- 前端
tags:
- Hexo
---

# 配置Gitalk
在重新搭建这个站点的过程中，碰到一个“小”问题：以前我的网站使用的是WordPress，内建了评论系统，而现在在Hexo站点中该如何引入评论系统？

<!-- more -->

当然，这根本就不是问题，Hexo最著名的主题NexT已经在配置文件中给出了多种可供选择的评论系统。Google搜索到的大部分人使用的是Disqus，然而当我按照他们的流程一通操作后，执行`hexo g`却永远都是：

    next disqus TypeError: Cannot read property 'replace' of null

在我感到阵阵窒息之后，我看到了这样的issue：

![不支持Disqus.jpg](https://i.loli.net/2020/01/12/fkE8JSxeo4pjmu7.jpg)

我人傻了，3月份就不支持了咋现在还在配置文件里？

> 显然，当时匆匆扫过的我并没明白 `暂时的解决方案是 count: false ` 是什么意思。

于是我转而尝试其他的评论系统，好么，国内的一个赛一个麻烦，又是实名认证又是人脸信息的，还是别了。而国外的又时刻有被河蟹的风险。

终于我看到了Gitalk，它基于GitHub的issue，只要有GitHub账号就可以评论，看来同性交友网站不倒，这玩意就能用下去。

配置起来不算太麻烦，在NexT的配置文件里**找到comments块，将gitalk给active上**（这一步很重要），再去下面一点的gitalk块里按照文档一顿填，基本就完事了。

给出我的配置以供参考：

![gitalk块配置](https://i.loli.net/2020/01/12/jPw6frglHAW52ME.jpg)

到这里，理论上你基本就能看到和gitalk官网上一样的效果了。

然而，坑爹的事情不会就此停止，如果你在本地`hexo s`，然后打开一篇文章，你应该确实会看到“Gitalk加载中”，然后 * 噔  噔  咚 * ，出现了这样的场景：

![未找到相关的Issues](https://i.loli.net/2020/01/12/VmbrWfpDMNute5O.jpg)

你会试着点击`使用GitHub登录`，然而什么也没有发生，这大概是因为GitHub的登陆结果传不回本机的Hexo server。不必奇怪，尽管`hexo d`将它部署上去，访问你的站点再次点击它就好了。

需要注意，每篇文章下面的Gitalk都需要你戳上一下才能开始运行，这是因为刚刚说过的，Gitalk的原理是issue，你得以管理员的身份在repo里给每篇文章开上一个对应的issue，其他人才能在下面留言。

举个栗子，这是我用于测试Gitalk的几个留言在站点上的样子：

![gitalk在站点上](https://i.loli.net/2020/01/12/62GCSo1yZp3ixRW.jpg)

而这是repo里对应的issue：

![gitalk在issue中](https://i.loli.net/2020/01/12/l7pDt8vaf1gKdmk.jpg)

注意到红线标出的这一行，Gitalk就是用这两个标签来对应issue和文章的。

# 更改Gitalk的主题颜色
如果你足够细心，也许会注意到官方的gitalk是蓝色的，而我站点中的却是与页头一致的绯色。

这两种颜色差别太大，我想要保持网页配色的一致性，于是免不了多折腾两下。

要更改颜色，首先就得知道这是什么颜色。

F12，审查元素，可以看到默认的色号是`#6190E8`。

![默认配色](https://i.loli.net/2020/01/12/mjACFTl8et5hfwp.jpg)
既然如此，我们去Gitalk的repo，找找这个色号出现在哪些地方。

![repo中的位置](https://i.loli.net/2020/01/12/EAkaJT7sGz8jcC1.jpg)

可以看到，这个色号出现在`dist/gitalk.css`和`src/style/index.styl`中。而真正决定Gitalk颜色的，应当是`dist`（Distribution，分发）中的`gitalk.css`。

明白了这一点，我们就可以整活了。首先将这个repo给fork过来，再将`dist/gitalk.css`中的色号全都改成想要的样子。我使用的绯色`#CC543A`取自[NIPPON COLORS - 日本の伝統色](http://nipponcolors.com)。

康康初音浅葱色？

![浅葱](https://i.loli.net/2020/01/12/rzUKvqA5SIT7tPG.jpg)

这套色彩也是锤子的坚果手机“文青版”那几个骚气配色名称：远州鼠、落栗、苏芳、石竹、枯草、柳煤竹茶、锖青磁、鸠羽紫的来源。

![坚果手机文青版](https://i.loli.net/2020/01/12/7p1J9MOtAChDqyi.jpg)

当然，这还不算完，我们得让NexT主题用上我们修改过的`gitalk.css`。

我们可以在next文件夹里搜索gitalk，看看它是如何被加载的。

![gtalk搜索结果](https://i.loli.net/2020/01/12/UiRWDco3Tr6IAls.jpg)

显然，第2、4行的`gitalk_css_uri`和`gitalk_js_uri`就是关键。它说明gitalk的文件可以从两个地方加载：一个是`theme.vendors`，也就是主题配置文件`_config.yml`的`vendors`块中的记录，一个是`jsdelivr`这个CDN。

![gitalk vendors](https://i.loli.net/2020/01/12/FpBQKzJZhqjdtmY.jpg)

以上两张截图里的CDN已经被我改成了我修改后的分支中对应的文件。关于这个CDN地址的格式，可以参看[jsDelivr官网](https://www.jsdelivr.com)的说明。

地址修改完成后，`hexo g -d`走起，你的配色应该已经成了想要的样子，庆贺吧！