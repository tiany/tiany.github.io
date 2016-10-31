﻿title: Vim踩坑一例 
date: 2015-12-20 23:38:41
category: tech
tags: vim
---
从大学开始使用Vim到现在算起来也将近10年了，10年的使用过程里面，除了初期的学习成本之外，vim基本上没有给我造成过大的麻烦，直到最近的一次。

事情是这样的，前一段时间我受一些文章的影响vim下多文件的编辑从以前习惯的多tab方式切换到多buffer的模式了，此类的文章如：[Vim Tab Madness. Buffers vs Tabs](https://joshldavis.com/2014/04/05/vim-tab-madness-buffers-vs-tabs/)，当然，这并没有问题，因为对多文件的编辑使用buffer的形式本身就是Vim的设定。多buffer下切换buffer(我使用了minibufferexplorer插件)的时候是需要先写出下当前buffer的(`:w`),这点和在多tab编辑下的习惯不太一样，所以我在自己的本地环境也设置了`set hidden`, 这样就能避免切换到新的buffer的时候就必须写出当前buffer，然而比较作死的是，我有一部分代码是直接在服务器上写的，而在服务器的vimrc中我并没有设置`set hidden`，而且当时也并没有觉得会出什么大问题，指导有一天，我打算完成如下的一个任务：
>   从a文件中分离出来一部分代码到b文件中

我当时的的操作流程如下：

>   1.  新建b文件
>   2.  vim同时打开a，b文件，用`shift v`选中a里需要的代码，按`d`删除（**这里是犯的第一个错误**），`:w` 保存文件
>   3.  跳转到b文件，这个时候出了一个小失误，我失手按了`yy`键，复制了某个空行（**这里是第二个错误**），发现在b里面无法粘贴原来的代码
>   4.  退回到a文件，企图用`u`撤销操作失败，`:undolist`输出为空…… 这个时候我意识到了问题的存在，保留进程未动，企图恢复文件
>   5.  企图通过复制.swp文件恢复，复制.swp文件到其他目录，然后建立同名文件，打开，按照提示选择恢复，发现已经是`:w`之后的文件了
>   6.  企图通过vim的自动备份文件来恢复，发现没有备份文件，虽然我以前很讨厌vim的备份文件，但是此刻我真的好希望有备份文件，我记得以前备份文件是需要手动关闭的，但是检查了下我的vimrc没发现关闭自动备份的地方，感觉像是某个版本后vim就默认关闭自动备份了,只打开了writebackup（**这里是第三个错误**）
>   7.  本来想在通过文件系统试下找回的，但是想想也就几百行代码，还是直接重写吧，于是作罢。

这个过程的三个简单总结下：
>   1. 第一个错误，此处应该用复制，待粘贴好之后再执行delete
>   2. 第二个错误，使用的是默认寄存器时，一定要随时小心寄存器被覆盖
>   3. 第三个错误，备份任何时候都不是多余的。 
**冗余不做，日子甭过，备份不做，十恶不赦** &emsp;&emsp;&emsp; --摘自某Linux前辈签名

问题发生了，但是我坚信vim一定有版本解决这个问题的，于是开始请教了一堆人（这里感叹下，vim的freenode irc真的好冷清），并做了各种尝试，最终结论如下，与君共飨：

1. 首先是造成这个问题的原因

    >   When unloading a buffer Vim normally destroys the tree of undos created for that buffer.
    
    以上文字来源于`:help persist-undo`，(多谢 [百合喵](http://lilydjwg.is-programmer.com/)), 也就是默认的buffer设置下，`:w`然后切换到其他buffer的时候，当前buffer的`undo tree`就会被销毁了，这也是造成这次事故的根本原因。

2.  解决方案，也是最简单的，设置`set hidden`， 这个可以保证你切换到其他buffer的时候当前的buffer只是hidden而不是unload，所以undo tree就不会被销毁了，但是使用这个选项的时候一定要特别小心地使用`:q!` `:qall!` 这样的命令，否则容易丢文件

3.  设置persist-undo，详细的设置见`:help persist-undo`, 推荐搭配 [undo-tree](https://github.com/mbbill/undotree) 使用, 参考设置
    ```
    nnoremap <F5> :UndotreeToggle<cr>
    if has("persistent_undo")
        set undodir=~/.undodir/
        set undofile
    endif
    ```
4. 设置backup，详细设置可以看 `:help backup`, 参考设置：
    ```
    set backupdir=~/.vim_backdir/
    set backup
    ```
    
如上配置之后基本上就不用担心vim丢文件了，当然，配合上好用的版本控制工具，效果就更好了。