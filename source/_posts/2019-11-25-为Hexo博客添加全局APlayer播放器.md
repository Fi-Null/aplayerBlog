---
title: 为Hexo博客添加全局APlayer播放器
date: 2019-11-25 21:57:48
tags:
- Hexo
- 音乐
- 前端
---

{% note info %}
如果你只是想要在自己站点的左下角加一个极其Stable的音乐播放器，而不想听我废话自己踩的坑，请直接跳转：  
[“Show me the CODE!”](/2019/11/25/为Hexo博客添加全局APlayer播放器/#Show-me-the-CODE)
{% endnote %}
<!-- more -->
# 早期尝试

## 网易云iframe

今天，我试图解决一个历史遗留问题。

之前我博客上的音乐播放器一直都是网易云的外链iframe，就这个样式的：

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=1384318412&auto=0&height=66"></iframe>

我一直把这个玩意放在sidebar里。但是问题是，移动版视图下sidebar是不展示的，只有header能够显示出来，而且还需要用toggle展开。我使用的主题是`NexT`，试验了一下，需要修改的地方是：

`\themes\next\layout\_partials\header\index.swig`

由于toggle展开的是`site-nav`，将源码修改如下：

```html
{{ partial('_partials/header/brand.swig', {}, {cache: theme.cache.enable}) }}

<nav class="site-nav">
  {{ partial('_partials/header/menu.swig', {}, {cache: theme.cache.enable}) }}
  <!--网易云iframe-->
  <iframe frameborder="no" border="0" marginwidth="0" 
        marginheight="0" width=330 height=86 
        src="//music.163.com/outchain/player?type=2&id=1384318412&auto=0&height=66">
  </iframe>
  <!--网易云iframe end-->
</nav>

{%- if theme.algolia_search.enable or theme.local_search.enable %}
  <div class="site-search">
    {{ partial('_partials/search/index.swig', {}, {cache: theme.cache.enable}) }}
  </div>
{%- endif %}

{{- next_inject('header') }}
```

![site-nav中的iframe.jpg](https://i.loli.net/2020/01/12/Dmhzxi8XFIQK7np.jpg)

这个效果属实🍔🍔。并且在桌面版视图下还会把sidebar压下去，不太理想。

## 明月浩空音乐

有没有悬浮式的播放器呢？有的。随便查一查就看到了这篇文章：[一款炫丽的网页播放器插件](https://leafjame.github.io/posts/2636059693.html)，这里介绍了`绚丽彩虹播放器`，确实是够绚丽的，点进去一看还是个东方二次元，连域名都叫`badapple.top`，爱了。

一阵操作之后，预览的效果确实挺理想的，可是到我网站上就只能播放第一首曲子了，和上面那篇文章的评论区里一位老哥说的一模一样：

![出现了绚丽的问题](https://i.loli.net/2020/01/12/Sb8tgk2FZLrnu3i.jpg)

在我试图解决这个问题的时候，`badapple.top`突然就503了，回想起上面最后的更新日期是2018年，我就感到一阵不妙。不一会儿我就发现了和这绚丽播放器几乎`  完 全 一 致  `的替代品[AIP播放器](https://music.ain19.com/)，并且找出了前面问题的解决方案。可是，这两个东西相似度怎么能够这么高？？

原来它们都是从[明月浩空音乐](https://player.lmih.cn/)改来的，而这个原版是需要收费的。这明月浩空又是何许人也？传闻其名乃“李明浩”，这里有他不知真假的黑历史：[关于Colorful2.6(明月浩空模板)后门剖析](https://www.chinasuc.cn/?thread-151.htm)。

本来如果我不太在意这个插件的安全性，折腾播放器的经历到这里就应该结束了。可是AIP播放器它不能自由修改主题色，而默认的颜色偏偏还是绿的，强迫症让我不能忍。看了看它的加载方式，是用`\theme\next\layout\_layout.swig`中插入的JS脚本加载远端服务器上的CSS，JS、CSS、播放列表都在远端，这可没法随便改了。

当然，也许我可以自己写一个CSS（在NexT里用的是Stylus）来覆盖它的设定——可惜我不会。

那咋办嘛，只能再看看有没有其他选择了。如此“简单”的需求难道会没有对应的开源软件吗？

# APlayer

在寻找替代品的过程中，我曾看到过这篇文章：[可能是目前最好用的两个 Hexo 播放器插件](https://diygod.me/best-players-in-hexo/)，讲的是开源的[hexo-tag-aplayer](https://github.com/MoePlayer/hexo-tag-aplayer)，这是一个给hexo使用的[APlayer](https://github.com/MoePlayer/APlayer)。

然而在看了它的文档之后，我没有第一时间get到它的点，特别是不知道要咋用。README.md里关于用法仅有一句：

```
  {% aplayer title author url [picture_url, narrow, autoplay, width:xxx, lrc:xxx] %}
```

这是个啥？当时我虽然知道Hexo的`Tag Plugins`，准确地说是NexT带来的[Bootstrap Callout](https://theme-next.org/docs/tag-plugins/note)——不得不说，`Tag Plugins`这个名字起得真是沙雕，它太容易和`tags`混淆了——但是我因为没什么机会用到，并没有细看过它的格式，所以也并不明白这<code>{&#37; &#37;}</code>是个什么操作。

现在必须得找其他方法的话，那就只好研究一下了。看了些Issues才搞明白原来是写在md文件里的，这与我的想法背道而驰——我希望能放在所有页面的固定位置，如果能做到网易云音乐网页版那样跳转页面而播放进度不变就再好不过了。

经过一段时间的测试之后，我发现如果使用`hexo-tag-aplayer`最好的解决方案就是如这个[issue](https://github.com/MoePlayer/hexo-tag-aplayer/issues/58)所言，在md里插入播放器后，再把生成的html文件里对应部分复制到header的`index.swig`里。可是这不就是个威力加强版的网易云iframe了吗？！

其实如果不是因为另一个问题，也还勉强能用。它的`fixed`模式（文档里翻译为“固定模式”）总让人感觉不太对劲，播放列表竟然是向上展开，如果放在header里，本来位置就很靠上，向上展开之后有一部分会伸出页面之外。虽然有一个默认值为`340px`的参数`listmaxheight`表示播放列表展开的高度，但我将其调为负值时，不但没有变成向下展开，而且还把播放列表倒过来了……

{% blockquote 沃兹·吉硕德 %}
当你感到迷惑，就应该去查Issues，大概率会有人和你遇到同样的迷惑问题。
{% endblockquote %}

所以我又查到一个[issue](https://github.com/MoePlayer/hexo-tag-aplayer/issues/79)，什么？`吸底`？这正是我想要的——这说的不就是**原版**APlayer吗？！

![APlayer吸底模式](https://i.loli.net/2020/01/12/Rm4tpK1sBMrdcyT.jpg)

在APlayer的文档里，对`fixed`模式的描述是`吸底`，这才是我所期望的功能，前面被“固定模式”这个沙雕翻译给整蒙了。那么，原版APlayer又是怎么用的呢？

![APlayer入门](https://i.loli.net/2020/01/12/CBPWNdEnRqrUoHs.jpg)

看到这里，我的迷惑更上一层楼，有一定前端知识的人应该很快就能看懂这里在说什么，但是我显然不是……

好在这时看到这篇文章：[添加全站 APlayer 播放器](https://cloud.tencent.com/developer/article/1157669)，看起来把这段代码加到`_layout.swig`的body里就OK了：

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/aplayer@1.7.0/dist/APlayer.min.css">
<div class="aplayer" data-id="534542404" data-server="netease" data-type="song"></div>
<script src="https://cdn.jsdelivr.net/npm/aplayer@1.7.0/dist/APlayer.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/meting@1.1.0/dist/Meting.min.js"></script>
```

但是依然不行：

> 这里的外部调用链接没有包含版本号，默认加载最新 release 版本；正常来说不会出什么问题，但是如果需要 100% 稳定运行，建议自行加上版本号。

事后发现这里加载的最新版本MetingJS是`2.0.0`，但实际上这个版本和APlayer根本不兼容，而MetingJS的文档却给人一种稳得一批的感觉：

![虚假的MetingJS文档](https://i.loli.net/2020/01/12/tmUfeSh5KFILic7.jpg)

（都不Supported你贴个啥APLAYER啊！！）

好，那就自己加上版本号。这里从`jsDelivr`引用了三个文件，APlayer的CSS和JS，Meting的JS。最新的（截至2019年11月25日）稳定版本使用如下，你也可以去`jsDelivr`自行查找。

# Show me the CODE!

如果你和我一样使用`NexT`主题，请复制以下代码，定位到`\theme\next\layout\_layout.swig`，将其粘贴到body体内即可。如果是其他情况，**大同小异！大同小异！**

```html
<!-- 引用依赖 -->
<link rel="stylesheet" 
  href="https://cdn.jsdelivr.net/npm/aplayer@1.10.1/dist/APlayer.min.css">
<script src="https://cdn.jsdelivr.net/npm/aplayer@1.10.1/dist/APlayer.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/meting@1.2.0/dist/Meting.min.js"></script>

<!-- 我使用的APlayer本体 -->
<div class="aplayer" 
  data-id="2465890057" 
  data-server="netease" 
  data-type="playlist" 
  data-fixed="true" 
  data-autoplay="true" 
  data-order="random" 
  data-volume="0.55" 
  data-theme="#cc543a" 
  date-preload="auto" >
  </div>
<!--如果将本体放在body里面导致页面加载出现问题，请尝试放到body体后面-->
  ```

  APlayer的属性设置，请查阅[APlayer中文文档-参数](https://aplayer.js.org/#/zh-Hans/?id=%E5%8F%82%E6%95%B0)以及能让你使用音乐平台的[MetingJS](https://github.com/metowolf/MetingJS)。

  > **注意：** 如果你使用上述代码，请以`data-xxx`格式书写参数名！

  # Bonus：使用pjax，页面跳转不重新播放

**最后一步！**

[网易云音乐网页版](https://music.163.com/)底部的播放器，无论在网易云里如何跳转页面，只要还在当前标签页都是不重新加载的，这样就可以使正在播放中的音乐不随页面跳转中断。怎样实现这个功能呢？

[知乎-网易云音乐网页版底部的播放器是如何实现在切换网页时不停止播放音乐的？](https://www.zhihu.com/question/23720144)

这篇文章讲述了pjax的使用：[迷你音乐播放器及实现全站pjax无刷新加载](https://cloud.tencent.com/developer/article/1096366)

虽然不是专讲APlayer，但是大同小异，大同小异！我作为对前端没什么了解的人，就跟着这个路子踩坑吧。

需要添加pjax和`1.7.0`以上版本的jquery，我这里依然从`jsDelivr`引用，最好放在body体结束前：

```html
<!-- require JQuery -->
<script src="https://cdn.jsdelivr.net/npm/jquery@3.4.1/dist/jquery.min.js"></script>
<!-- require pjax -->
<script src="https://cdn.jsdelivr.net/npm/pjax@0.2.8/index.min.js"></script>
```
然后按照以下方法使用pjax:

![使用pjax](https://i.loli.net/2020/01/12/gsN2MBVqIzCW9Uf.jpg)

**然而，全部木大！** 该跳还是跳了，也许是我姿势不对。

我转念一想，`NexT`这种历史悠久的主题，`pjax`这么好用的技术，岂有不内置的道理？鬼使神差之下，我打开主题的`_config.yml`一搜，还真的有：

```yml
# Easily enable fast Ajax navigation on your website.
# Dependencies: https://github.com/theme-next/theme-next-pjax
# For moreinformation: https://github.com/MoOx/pjax
pjax: true
```

仅仅直接在这里把选项置为`true`是不行的，因为虽然配置文件里已经写了选项，但其实并没有安装对应的插件。关于如何使用，[Dependencies](https://github.com/theme-next/theme-next-pjax)的文档里说得极为清楚，在此就不赘述了。

开启之后，全站都实现了无刷新加载，接下来你就随便戳吧，音乐始终会在左下角播放的。

至此，一整天的折腾之旅才算是终于告一段落了。