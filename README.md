pjax
可能大家听说过 Ajax，没听说过 pjax，这个技术实际上就是利用 Ajax 技术实现了局部页面刷新，既可以实现 URL 的更换，又可以做到无刷新加载。

要开启这个功能需要先将 pjax 功能开启，然后安装对应的 pjax 依赖库，首先修改_config.yml 修改如下：

pjax: true

然后安装依赖库，切换到 next 主题下，然后安装依赖库：

$ cd themes/next
$ git clone https://github.com/theme-next/theme-next-pjax source/lib/pjax
这样 pjax 就开启了，页面就可以实现无刷新加载了


为便于同步源码而创建的hexo分支。
