 title: Django 中 many-to-many 数据库关系的 migrate 操作
date: 2016-3-29 01:49:38
category: tech
tags: [Django, many-to-many, migrate]
---
公司的项目是用 Django 作为基础架构搭建的，在设计数据库的时候由于我当时对 Django 的一些细节还不是很清楚，导致在设计一组多对多的关系的时候直接使用了系统自带的 many_to_many 关系，之后随着需求的增加和代码的进一步开发，发现在这个多对多的关系上需要增加附属信息，而直接使用系统自带的 many_to_many 是无法完成这一需求的。

举个例子来说就是这样的：
假设我们现在有两份数据，分别是 **乐队** 和 **歌手**，显然乐队和歌手之间应该是一个多对多的关系（一个乐队有多个歌手是显而易见的，而随着时间的推演，一个歌手曾经隶属于多个乐队也是很正常的事情），当我们使用默认的 Django 的多对多关系的时候，我们是可以很容易地表述这一信息的。

之后随着需求的发展，问题变成了这个样子，我们现在还需要记录每个歌手 **加入乐队的时间** 和 **从乐队离队的时间**。首先从直觉上我们会意识到这个其实是对乐队和歌手关系的一个补充描述，所以最合理的添加地方就是在描述乐队和乐手关系的表里面。在这儿我们需要插播一下数据库中是如何实现多对多关系的：

- 对于最简单的一对一关系，只需要在一对一关系的任意一方的表里面添加一列，内容为另外一方的 ID 即可实现。
- 对于一对多的关系，我们只需要在多的一方的表里面添加一列，内容以一的一方的 ID 也就可以实现了。
- 对于多对多的关系，如果我们还是使用上面的两个方法的话，由于我们不知道另外一端到底会有多少个，这样就无法简单通过扩展表的形式来实现了，而且就算实现了，在数据上也是很大的冗余。所以在面对多对多的关系的时候，数据库采用的方法是在新建一张表，这张表里面有两列，分别指向多对对关系里面的双方，这样问题就解决了。

知道了多对多关系是怎么实现的，然后就是迁移的事情了，Django 其实是提供了自动以这张表的能力的，只需要在创建多对多关系的时候指定它们的 through 就好了，这样后续也可以继续扩展这张表提供更多的附加数据，但是默认的情况下直接建立了一张表，这张表里面只有两列，就是多对多关系的双方。

如果我们直接新建 through（其实就是 Django 里面的一个 model class），并修改多对多关系的话，直接使用系统的 makemigtations，migrate 我们将会得到一个异常 (Django migration error :you cannot alter to or from M2M fields, or add or remove through= on M2M fields)，所以这样是行不通的。

这个问题如鲠在喉了好久，Google 上搜到的一些答案也不尽如人意，不过参考那些答案，最终还是把这个问题解决了，基本思路如下：

1. 保留原有的 many_to_many field，并新建一个 many_to_many field，新建的 many_to_many field 里面指定自定义的 through。
2. 执行 migrate，使得新建的 many_to_many field 生效。
3. 执行 data-migration，把旧的 many_to_many field 的数据迁移过来。（这一步可以直接通过写临时脚本来实现，但是由于要考虑 migration 操作的完整性和独立性，所以其实是整个思路里面花时间最长的地方。）
4. 删除旧的 many_to_many field，并修正新 field 里面的相关命名。
5. 排查代码，修改用于增加多对多关系的代码。

代码示例如下：

- 在执行步骤 1 之前，我们有两个对象 Publication2 和 Article2，它们之间是多对多的关系，并且是直接使用 Django 默认的 many_to_many 来处理的，代码如下：

    ```python
    #models.py
    from django.db import models

    class Publication2(models.Model):
        title = models.CharField(max_length=30)

        def __str__(self):              # __unicode__ on Python 2
            return self.title

    class Article2(models.Model):
        headline = models.CharField(max_length=100)
        publications = models.ManyToManyField(Publication2)

        def __str__(self):              # __unicode__ on Python 2
            return self.headline
    ```
- 然后修改 models.py 文件到下面这样，我们增加了 publications_m 用来做中间的中转 field，注意这里添加 related_name 是为了和上面的 publications 区分，否则的话从 Publication2 找 Article2 的时候就无法区分是 publications 关系的还是 publications_m 关系的了，Django 也会报错。

    ```python
    from django.db import models

    class Publication2(models.Model):
        title = models.CharField(max_length=30)

        def __str__(self):              # __unicode__ on Python 2
            return self.title

        class Meta:
            ordering = ('title',)

    class Article2(models.Model):
        headline = models.CharField(max_length=100)
        publications = models.ManyToManyField(Publication2)
        publications_m = models.ManyToManyField(Publication2, related_name="ariticle2_m", through='WriteRelation2')

        def __str__(self):              # __unicode__ on Python 2
            return self.headline

    class WriteRelation2(models.Model):
        publication2 = models.ForeignKey(Publication2, on_delete=models.CASCADE)
        article2 = models.ForeignKey(Article2, on_delete=models.CASCADE)
        write_date = models.DateTimeField(auto_now_add=True)

    ```

- 执行  `./manage.py makemigrations m2m3` 和 `./manage.py migrate`, 其中 m2m3 是我为这个工程取的名字（第三个 m2m 测试），执行后上面 models.py 中的修改就会生效了，生成的 migrate file 内容如下：

    ```python
    # 0002_****.py

    # -*- coding: utf-8 -*-
    from __future__ import unicode_literals

    from django.db import migrations, models

    class Migration(migrations.Migration):

        dependencies = [
            ('m2m3', '0001_initial'),
        ]

        operations = [
            migrations.CreateModel(
                name='WriteRelation2',
                fields=[
                    ('id', models.AutoField(serialize=False, primary_key=True, verbose_name='ID', auto_created=True)),
                    ('write_date', models.DateTimeField(auto_now_add=True)),
                    ('article2', models.ForeignKey(to='m2m3.Article2')),
                    ('publication2', models.ForeignKey(to='m2m3.Publication2')),
                ],
            ),
            migrations.AddField(
                model_name='article2',
                name='publications_m',
                field=models.ManyToManyField(related_name='ariticle2_m', to='m2m3.Publication2', through='m2m3.WriteRelation2'),
            ),
        ]
    ```

- 也是最关键的一步，执行 data migration，具体步骤先执行 `./manage.py makemigrations --empty m2m3` 这一步会生成一个空的 migrate file 用于填写具体的迁移脚本，填写后再执行 `./manage.py migrate` 即可完成 data migration，可参考 Django 的[官方文档](https://docs.djangoproject.com/en/1.9/topics/migrations/#data-migrations)。我写的 data migration 的脚本如下：

    ```python
    # 0003_****.py

    # -*- coding: utf-8 -*-
    from __future__ import unicode_literals

    from django.db import migrations, models

    def forwards_func(apps, schema_editor):
        Article2 = apps.get_model("m2m3", "Article2")
        WriteRelation2 = apps.get_model("m2m3", "WriteRelation2")

        for t in Article2.publications.through.objects.all():
            WriteRelation2.objects.create(article2=t.article2, publication2=t.publication2)

    def reverse_func(apps, schema_editor):
        Article2 = apps.get_model("m2m3", "Article2")
        WriteRelation2 = apps.get_model("m2m3", "WriteRelation2")

        for t in WriteRelation2.objects.all():
            Article2.publications.through.objects.create(article2=t.article2, publication2=t.publication2)
            t.delete()
            #WriteRelation2.objects.create(article2=t.article2, publication2=t.publication2)
        #pass

    class Migration(migrations.Migration):

        dependencies = [
            ('m2m3', '0002_auto_20160311_0849'),
        ]

        operations = [
            migrations.RunPython(forwards_func, reverse_func),
        ]
    ```

- 剩下的就是清理工作了，要分两步进行，先删除旧的 many_to_many field， 将新的 many_to_many field 修改名称到原来的样子。完全修改好之后的 models.py 和对应的两个 migrate file 的内容如下：

    ```python
    #  0004_***.py
    # -*- coding: utf-8 -*-
    from __future__ import unicode_literals

    from django.db import migrations, models

    class Migration(migrations.Migration):

        dependencies = [
            ('m2m3', '0003_auto_20160311_0849'),
        ]

        operations = [
            migrations.RemoveField(
                model_name='article2',
                name='publications',
            ),
        ]

    #  0005_***.py
    # -*- coding: utf-8 -*-
    from __future__ import unicode_literals

    from django.db import migrations, models

    class Migration(migrations.Migration):

        dependencies = [
            ('m2m3', '0004_remove_article2_publications'),
        ]

        operations = [
            migrations.RemoveField(
                model_name='article2',
                name='publications_m',
            ),
            migrations.AddField(
                model_name='article2',
                name='publications',
                field=models.ManyToManyField(to='m2m3.Publication2', through='m2m3.WriteRelation2'),
            ),
        ]

    #  models.py
    from django.db import models

    class Publication2(models.Model):
        title = models.CharField(max_length=30)

        def __str__(self):              # __unicode__ on Python 2
            return self.title

    class Article2(models.Model):
        headline = models.CharField(max_length=100)
        publications = models.ManyToManyField(Publication2, through='WriteRelation2')

        def __str__(self):              # __unicode__ on Python 2
            return self.headline

    class WriteRelation2(models.Model):
        publication2 = models.ForeignKey(Publication2, on_delete=models.CASCADE)
        article2 = models.ForeignKey(Article2, on_delete=models.CASCADE)
        write_date = models.DateTimeField(auto_now_add=True)

    ```
- 最后就是找到代码里面类似如下的地方：

    ```python
    a1 = Article2.objects.create(headline='A1')
    p1 = Publication2.objects.create(title='p1')
    p2 = Publication2.objects.create(title='p2')
    a1.publications.add(p1)
    a1.publications.add(p2)
    ```

    改为

    ```python
    a1 = Article2.objects.create(headline='A1')
    p1 = Publication2.objects.create(title='p1')
    p2 = Publication2.objects.create(title='p2')
    WriteRelation2.objects.create(article2=a1, publication2=p1)
    WriteRelation2.objects.create(article2=a1, publication2=p2)
    ```

The End.
