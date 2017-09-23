---
title: 终于配置好Hexo了
categories: Leisure
visible: false
date: 2017-09-23 16:57:31
---

搞了快一天，终于配置好Hexo了……未来几个月都不折腾了:joy:

这次更新配置了非常多的东西，稍微记录一下吧。


## 主题Repo

之前是把主题官方Repo当成一个submodule，然后另外存储主题配置文件。但是如果要修改主题，这样做就不够了。
所以我fork了官方的主题Repo，自己管理。


## 将个人纪录类的文章和技术类的文章分开展示

当初为了逼格，文章都用英文写，然后回不了头了……但是有时候也想用母语写点东西啊:joy:，所以就有
把两种文章分开的需求了~

1. 按[这里](http://forwardkth.github.io/2016/05/08/next-theme-post-visibility/)的步骤修改了主题源文件。
2. 用1中的方法修改了`archive`的布局。
3. 修改主题的配置文件，把`Leisure`这个类别放到主页里。
4. 在`scaffolds`中添加`leisure.md`，默认`visible = false`且`categories: Leisure`。

最后的效果是：在`Home`和`Archive`中隐藏所有`visible === false`的文章，这些文章只会在`Leisure`分类下展示。
这个分类可以从主页的`Categories`和`Leisure`进入。


## 不蒜子、Disqus

增加了不蒜子阅读计数，修复了Disqus评论系统。


## Emoji支持

配置一下`hexo-filter-github-emojis`就好了。


## 总结

总之，这次折腾算是结束了:joy:

然后，继续回去看paper……
