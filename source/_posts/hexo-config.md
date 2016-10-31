title: hexo-config
date: 2015-12-08 21:34:25
category: tech
tags: [hexo, mathjax]
---

想写技术博客的时间其实有很久了，但是一直都没有实施起来过，怎么说呢，拖延症确实还是太严重了，希望这次会是一个好的开始，目前的计划是每月保证一更，希望能坚持下来。

把技术博客从wp迁移到github pages了，一个主要的原因是我开始喜欢上Markdown这样的写作方式了，另外也是想体验下github pages的博客功能。前期在搭建的技术选型中在jekyll和hexo中犹豫了好久，不过还是更喜欢hexo这个名字， 你看，hex是16进制，hexo和hero又只有一字之差，而且读着也蛮朗朗上口的，还有一个私心，是想在用hexo的过程中了解下node.js, 这么多好处就由不得我不去选hexo了啊……

选好了最基础的框架之后，就开始动手了，这方面网上有很多教程，基本上随便找找就可以搞定了，我是参考这两篇完成的：

- [使用GitHub和Hexo搭建免费静态Blog](http://wsgzao.github.io/post/hexo-guide/)
- [Setting up a Github Pages blog with Hexo](http://jdpaton.github.io/2012/11/05/setup-hexo/)

建立好基础的架构只是第一步，下面需要找到自己心仪主题，否则千篇一律的blog总归是很难让人赏心悦目的，主题可以在官方给出的参考里面去找.
[官网主题列表](https://hexo.io/themes/)

我在万般纠结之后选择了这个主题: [maupassant-hexo](https://github.com/tufu9441/maupassant-hexo)，总体上还是很满意的，不过后来测试发了篇文章，发现了两个问题：

1. 不支持数学公式显示
2. 引用的样式不喜欢

虽然我可能几百年也用不了一次数学公式显示，而引用的使用机会也不是很高，但是知道了问题的存在而不去解决实在是太让人难受了，于是去翻了下主题的代码，发现用到了jade和sass，这两个都是我之前没有接触过的技术（不得不感叹前端技术日新月异啊）。但是问题还是要解决的，下面简单记录下两个问题的解决过程，希望能帮到有需要的人。

1. 不支持数学公式显示
    1. 网上搜索得到的方案是使用mathjax，在网上找了一段js代码研究jade的语法终于正确嵌入后，像[这样](http://hujiaweibujidao.github.io/blog/2014/04/15/flying-on-mac/)，发现在显示的时候末尾会有一个小竖线，如前面链接里面显示的，看着非常难受。
    2. 为了解决小竖线的问题，看是看mathjax的[官网](https://www.mathjax.org/)，发现虽然他们官网的例子里面是没有小竖线的，但是翻他们官网的页面源码，把mathjax的相关配置抄过来之后，发现在我的博客上显示还是有问题，遂放弃。
    3. 后来想到在Cmd Markdown和StackEdit上面数学公式渲染后都是没有小竖线的，于是从Cmd Markdown的页面里面借鉴了相关mathjax的配置，问题解决，相关修改可以参考[这里](https://github.com/tiany/maupassant-hexo/commit/fd8319fbf9a5f0936ad3348132e41d5c2d5a8215)
2. 引用的样式不喜欢
    1. 可能是习惯了markdown那样的用一条小竖线来表示引用，对于这个主题里面用一个很大的双引号来表示引用实在无法喜欢起来，显示找到源码中关于blockquote的相关css配置，修改后发现左边会很宽。
    2. 用chrome的开发工具调试后发现是由于浏览器自带的css属性没有reset导致的，后来把相关的属性重写后就好了(blockquote的margin-left属性，chrome下默认是40px），最终的修改参考[这里](https://github.com/tiany/maupassant-hexo/commit/c3453553343a9072425350c1d13bdef1fd2c18ef)

两个修改后，这个主题基本符合我的需求了，修改后的效果如下：

- 数学公式
$$ E = mc^2 $$

- 引用样式
> 这是一段引用

第一篇博文，就写到这里吧，希望能够坚持下来。
