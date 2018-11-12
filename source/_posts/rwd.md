title: 响应式布局
date: 2016-05-15 23:42:00
category: tech
tags: [frontend, rwd]
---

最近看了一些 RWD 的东西，本月就偷个懒，用这篇整理的 RWD 的东西来发一篇 blog 吧，总算不用等到月末再发了 , 由于内容主要是看到的一些东西一些摘要，所以上下文并不是很丰富，权做纪念吧。:-)

响应式布局是为了让一套布局方案能够适应多种多样的显示需求而出现的，解决了传统方案下需要针对不同的设备而专门撰写不同的站点的问题。

流式布局的关键技术主要有以下几点：

- viewport

    ```html
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    ```

    上面的 `width=device-width` 可以让页面宽度随着设备而改变，这样才可以进行依据宽度的布局，`initial-scale=1` 则是为了优化用户体验，在屏幕比较小的设备上，像素密度会比普通的显示器大很多，为了显示效果，这些设备一般会把字体等缩放到一个设备无关的大小 (device-independent pixels)，设置这个参数可以让 css 文件中的 px 和 device-independent pixels 对应上。

    基本上主流的浏览器都支持，对于 IE < IE9 的可以使用如下的 js 来解决 :

    ```javascript
    <!--[if lt IE 9]>
        <script src="http://css3-mediaqueries-js.googlecode.com/svn/trunk/css3-mediaqueries.js"></script>
    <![endif]-->
    ```

- fluid layout
    流式布局主要基于 css 的 float 属性来实现，主要的设计形式如下：
  - Mostly fluid
    ![Mostly fluid](http://ww4.sinaimg.cn/large/4d1782b1gw1f3g2w0qdbej20dw0a3wem.jpg)
    [示例代码](https://developers.google.com/web/fundamentals/design-and-ui/responsive/patterns/mostly-fluid?hl=en)
  - Column drop
    ![Column drop](http://ww3.sinaimg.cn/large/4d1782b1gw1f3g2ytwrqsj20dw0a374f.jpg)
    [示例代码](https://developers.google.com/web/fundamentals/design-and-ui/responsive/patterns/column-drop?hl=en)
  - Layout shifter
    ![Layout shifter](http://ww4.sinaimg.cn/large/4d1782b1gw1f3g31mblxij20dw06tdfx.jpg)
    [示例代码](https://developers.google.com/web/fundamentals/design-and-ui/responsive/patterns/layout-shifter?hl=en)
  - Tiny tweaks
    ![Tiny tweaks](http://ww2.sinaimg.cn/large/4d1782b1gw1f3g3738whdj20dw03ut8o.jpg)
    这个里面主要是对字体等等做一些微调，布局上没有大的变化。
    [示例代码](https://developers.google.com/web/fundamentals/design-and-ui/responsive/patterns/tiny-tweaks?hl=en)
  - Off canvas
    ![Off canvas](http://ww2.sinaimg.cn/large/4d1782b1gw1f3g392bhwmj20dw081wem.jpg)
    [示例代码](https://developers.google.com/web/fundamentals/design-and-ui/responsive/patterns/off-canvas?hl=en)

- media-query
  - css3 引入的新心功能，可以根据屏幕的宽度加载不同的 css 文件

    ```html
    <!-- 400px 宽度下加载此 css -->
    <link rel="stylesheet" type="text/css"
    　　media="screen and (max-device-width: 400px)"
    　　href="tinyScreen.css" />

    <!-- 400-600px 加载此 css -->
    <link rel="stylesheet" type="text/css"
    　　media="screen and (min-width: 400px) and (max-device-width: 600px)"
    　　href="smallScreen.css" />
    ```

    OR

    ```css
    /* css 中引入其他的 css 文件 */
    @import url("tinyScreen.css") screen and (max-device-width: 400px);
    ```

  - @media

    也可以在 css 中直接使用 @media 规则

    ```css
    /* 如下指定了一组在屏幕宽度小于 400 时的队则 */
    @media screen and (max-device-width: 400px) {
        .column {
            float: none;
            width:auto;
        }
        #sidebar {
            display:none;
        }
    }
    ```

- misc：
  - 使用相对宽度
  - 使用相对大小字体，em，rem，etc.
  - mobile first，优先设置移动页面，再一步步扩展为桌面版本，这个可以让你的布局更加灵活，而反过来的时候很多更改都会变得很麻烦。

最后，css 控制前端的布局确实是很辛苦的一件事情，不过做了一段时间之后就会稍微有点感觉了，另外，[flex-box](http://flexboxin5.com/) 是个很好的东西，解决了很多 css 布局的问题，强烈推荐了解，上面的示例代码里面，很多也都用到了。最近还在看一本响应式布局的书，后面如果有新的收获，应该还会再更新的。
