title: Django中的静态文件处理
date: 2016-4-30 20:24:00
category: tech
tags: [Django, static_files]
---
刚开始使用Django的时候，为了弄清楚其中关于静态文件处理的相关知识，走了不少的弯路，文档也看了好几遍，所以打算在这儿简单记录一下，防止自己将来忘记，也希望可以帮到别人。

我们将利用django的开发分为三个阶段，在不太的阶段，django对静态文件的处理是不太一样的：

1.  初始开发阶段
    &emsp;&emsp;在这个阶段，我们基本上就是直接使用django进行开发，要查看结果的时候也就是通过`python manage.py runserver`来直接查看的，如果你所有的静态资源文件都是放在自己的各个应用下面的，那么没有问题(注意需要将你的app增加到`INSTALLED_APPS`里），但是如果你有公共的静态资源，放在其他目录下的，就需要关注一个settings.py中可以指定的参数`STATICFILES_DIRS`,在它里面增加自己的自定义文件夹，这样跑起来的时候系统就会去找了。比如我们在工程的根目录有一个static目录里面放了大家共用的静态资源文件，我们就可以这样定义`STATICFILES_DIRS`：
    
    ```python
    STATICFILES_DIRS = (
        os.path.join(BASE_DIR, "static"),
    )
    ```
2. 搭配uwsgi，nginx的调试阶段
    &emsp;&emsp;这一步我们假设已经部署好了uwsgi，nginx，但是暂时还不想完全把静态文件提出来让nginx管理。由于这一步的时候我们的python环境已经是由uwsgi提供的了，而不是`python manage.py runserver`,所以我们需要在`urls.py`中增加如下配置，来保证django会帮我们渲染静态文件

    ```python
    from django.contrib.staticfiles.urls import staticfiles_urlpatterns
    # ... the rest of your URLconf here ...
    urlpatterns += staticfiles_urlpatterns()
    ```

    &emsp;&emsp;值得注意的是，这个方法渲染效率其实是很低的，而且是不安全的，所以请务必之在调试环境下使用，第一步里面的`python manage.py runserver`其实也是用了这个方法，而且此方法也旨在`DEBUG=True`的时候才生效。
3. 搭配uwsgi，nginx的生产阶段
    &emsp;&emsp;在生产阶段，需要保证静态文件的渲染足够快，所以一定不会用上面的方法了，这个时候我们需要把专业的事情交给专业的人来做，也就是把他们都交给nginx和apache这样的web_server来做。
    &emsp;&emsp;在交给这样的nginx，apache这样的web_server之前，我们需要把自己所有的静态文件都集中到一个地方，这样才能方便nginx，apache这些服务来处理，需要使用django提供的`python manage.py collectstatic`命令，在使用这个命令之前需要在settings.py中指定MEDIA_ROOT的值，也就是最终把所有的静态文件收集到什么目录，然后在nginx中，再将静态url都指定到这个目录就好了。
    &emsp;&emsp;考虑到网站加速，我们有时候会使用oss或者cdn之类的服务，在使用这些服务的时候我们需要将上一步中我们通过collectstatic收集起来的文件上传到oss或者cdn上面，然后在`settings.py`中指定`STATIC_URL`为指定的访问前缀即可。
    &emsp;&emsp;另外，推荐下ManifestStaticFilesStorage，这个Storage在collectstatic的时候会将所有的文件都算一个hash作为文件后缀，这样再上传到cdn，oss上面的时候就可以设置缓存的时间为永久了，因为一旦更新的话，对应的hash就会发生变化了。详细的情况看[这里](https://docs.djangoproject.com/en/1.9/ref/contrib/staticfiles/#manifeststaticfilesstorage)

另外本文没有提到的一点，就是在template的html里面引用static文件的时候一定要使用
```python
{% load static from staticfiles %}
<img src="{% static "images/hi.jpg" %}" alt="Hi!" />
```
这样的形式，才能保证你的项目尽可能灵活。 但是这个会造成前端无法单独开发，建议构建一个虚拟环境，会接受templates的文件名，然后渲染对应的template，然后经template和static目录共享给前端的开发人员，这样他们就可以比较容易地开发了。

_这个月又是在最后的时间赶文涨，说不仓促是骗人的，下个月一定要努力，希望不用这样赶文章了。_

