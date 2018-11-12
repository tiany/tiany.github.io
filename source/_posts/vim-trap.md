 title: Vim 踩坑一例
date: 2015-12-20 23:38:41
category: tech
tags: vim
---
从大学开始使用 Vim 到现在算起来也将近 10 年了，10 年的使用过程里面，除了初期的学习成本之外，vim 基本上没有给我造成过大的麻烦，直到最近的一次。

事情是这样的，前一段时间我受一些文章的影响 vim 下多文件的编辑从以前习惯的多 tab 方式切换到多 buffer 的模式了，此类的文章如：[Vim Tab Madness. Buffers vs Tabs](https://joshldavis.com/2014/04/05/vim-tab-madness-buffers-vs-tabs/)，当然，这并没有问题，因为对多文件的编辑使用 buffer 的形式本身就是 Vim 的设定。多 buffer 下切换 buffer ( 我使用了 minibufferexplorer 插件 ) 的时候是需要先写出下当前 buffer 的 (`:w`), 这点和在多 tab 编辑下的习惯不太一样，所以我在自己的本地环境也设置了 `set hidden`, 这样就能避免切换到新的 buffer 的时候就必须写出当前 buffer ，然而比较作死的是，我有一部分代码是直接在服务器上写的，而在服务器的 vimrc 中我并没有设置 `set hidden`，而且当时也并没有觉得会出什么大问题，直到有一天，我打算完成如下的一个任务：
> 从 a 文件中分离出来一部分代码到 b 文件中

我当时的的操作流程如下：

> 1. 新建 b 文件
> 2. vim 同时打开 a，b 文件，用 `shift v` 选中 a 里需要的代码，按 `d` 删除（**这里是犯的第一个错误**），`:w` 保存文件
> 3. 跳转到 b 文件，这个时候出了一个小失误，我失手按了 `yy` 键，复制了某个空行（**这里是第二个错误**），发现在 b 里面无法粘贴原来的代码
> 4. 退回到 a 文件，企图用 `u` 撤销操作失败，`:undolist` 输出为空…… 这个时候我意识到了问题的存在，保留进程未动，企图恢复文件
> 5. 企图通过复制 .swp 文件恢复，复制 .swp 文件到其他目录，然后建立同名文件，打开，按照提示选择恢复，发现已经是 `:w` 之后的文件了
> 6. 企图通过 vim 的自动备份文件来恢复，发现没有备份文件，虽然我以前很讨厌 vim 的备份文件，但是此刻我真的好希望有备份文件，我记得以前备份文件是需要手动关闭的，但是检查了下我的 vimrc 没发现关闭自动备份的地方，感觉像是某个版本后 vim 就默认关闭自动备份了 , 只打开了 writebackup（**这里是第三个错误**）
> 7. 本来想在通过文件系统试下找回的，但是想想也就几百行代码，还是直接重写吧，于是作罢。

这个过程的三个简单总结下：

> 1. 第一个错误，此处应该用复制，待粘贴好之后再执行 delete
> 2. 第二个错误，使用的是默认寄存器时，一定要随时小心寄存器被覆盖
> 3. 第三个错误，备份任何时候都不是多余的。  
 **冗余不做，日子甭过，备份不做，十恶不赦** &emsp;&emsp;&emsp; -- 摘自某 Linux 前辈签名

问题发生了，但是我坚信 vim 一定有办法解决这个问题的，于是开始请教了一堆人（这里感叹下，vim 的 freenode irc 真的好冷清），并做了各种尝试，最终结论如下，与君共飨：

1. 首先是造成这个问题的原因

    >   When unloading a buffer Vim normally destroys the tree of undos created for that buffer.

    以上文字来源于 `:help persist-undo`，( 多谢 [百合喵](http://lilydjwg.is-programmer.com/)), 也就是默认的 buffer 设置下，`:w` 然后切换到其他 buffer 的时候，当前 buffer 的 `undo tree` 就会被销毁了，这也是造成这次事故的根本原因。

2. 解决方案，也是最简单的，设置 `set hidden`， 这个可以保证你切换到其他 buffer 的时候当前的 buffer 只是 hidden 而不是 unload，所以 undo tree 就不会被销毁了，但是使用这个选项的时候一定要特别小心地使用 `:q!` `:qall!` 这样的命令，否则容易丢文件

3. 设置 persist-undo，详细的设置见 `:help persist-undo`, 推荐搭配 [undo-tree](https://github.com/mbbill/undotree) 使用 , 参考设置
    ```vim
    nnoremap <F5> :UndotreeToggle<cr>
    if has("persistent_undo")
        set undodir=~/.undodir/
        set undofile
    endif
    ```
4. 设置 backup，详细设置可以看 `:help backup`, 参考设置：
    ```vim
    set backupdir=~/.vim_backdir/
    set backup
    ```

如上配置之后基本上就不用担心 vim 丢文件了，当然，配合上好用的版本控制工具，效果就更好了。
