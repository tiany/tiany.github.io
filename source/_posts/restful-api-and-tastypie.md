title: RESTfulAPI与Tastypie
date: 2016-2-29 00:08:38
category: tech
tags: [RESTful-api, Tastypie, Python]
---
RESTful API 是一个基于 HTTP 定义的通用接口，可以让后台比较容易地数据暴露给前台使用，实在是在现在这个多前台并行开发后端往往支持不力的情况下后端 <del> 偷懒的绝佳法门 </del> 统一接口的一个非常好的方式。

RESTful API 在不同的语言和框架下都有这自己不同的实现，由于我开发的时候使用了 Python + Django 的组合，而在实现 RESTful API 的时候为了方便起见，我使用了自己之前稍微有过接触的 Tastypie，其实在 Tastypie 之后出现了很多比较优秀的 RESTful api 方案，诸如：[django-rest-framework](http://www.django-rest-framework.org/) 此处不展开。

所有的 RESTful API 由于涉及到前后端的数据交互，所以一定要指定一个数据格式，这个格式目前比较主流的就是 json 了，而 TastyPie 相对于其他的实现比较好的一点就是这个从后端数据到 json 的序列化过程可以自动生成，不需要自己去写。

RESTful API 基本可以分为以下几个部分：

1. **元**
    这个部分一般用于查询接口说明，可以从中获取到服务提供的具体接口和调用形式，使用 HTTP 的 GET/OPTION 方法获取。
2. **增**
    一般使用 HTTP 的 POST 方法完成，用于向数据源增加数据。
3. **删**
    一般使用 HTTP 的 DELETE 方法完成，用于删除已经存在的数据。
4. **查**
    一般使用 HTTP 的 GET 方法获取，用于获取数据的具体内容。
5. **改**
    一般使用 HTTP 的 PUT/PATCH 方法完成，用于修改已经存在的数据。

用于前端交互一般都是通过 js 完成的，下面给出一段通过 js 访问相关接口的示例如下：

- **POST**
  to get the new created object resource_uri, you should specify the CORS_EXPOSE_HEADERS = ('Location',) in django-cors-header if this is a cors request.

  ```javascript
  $.ajax({
    url: url,
    data: JSON.stringify({'address':'TEST', 'name':'TEST'}),
    method: 'POST',
    dataType: 'json',
    contentType: 'application/json',
    complete: function (xhr, textStatus) {
      console.log("complete");
      console.log(textStatus);
      console.log(xhr.status);
      console.log(xhr.getResponseHeader('Location')); //resource uri.
    }
  });
  ```

- **DELETE**

  ```javascript
  $.ajax({
    url: url,
    method: 'DELETE',
    dataType: 'json',
    contentType: 'application/json',
    complete: function (xhr, textStatus) {
      console.log("complete");
      console.log(textStatus);
      console.log(xhr.status);
    }
  });
  ```

- **GET**

  ```javascript
  $.ajax({
    url: url,
    method: 'GET',
    dataType: 'json',
    contentType: 'application/json',
    complete: function (xhr, textStatus) {
      console.log(JSON.stringify(xhr.responseJSON));
    }
  });
  ```

- **PUT/PATCH**

  ```javascript
  $.ajax({
    url: url,
    method: 'PATCH',
    data: JSON.stringify({'build_count':10}),
    dataType: 'json',
    contentType: 'application/json',
    complete: function (xhr, textStatus) {
      console.log("complete");
      console.log(textStatus);
      console.log(xhr.status);
    }
  });
  ```

使用 Tastypie 构建的 RESTful API 需要注意以下点：

- Tastypie 在上传文件方面目前没有比较官方的实现，目前网上给出的方案一般都是 hack 源码的，如果不想 hack 源码的话，可以考虑单独使用普通的 API 来实现，但是一致性会比较差，需要取舍。
- RESTful API 如果是跨域调用的话，需要注意设置跨域请求的参数，可以考虑使用[django-cors-headers](https://github.com/ottoyiu/django-cors-headers)
- 使用 RESTful API 请务必注意安全，由于此 API 将后台的数据暴露了出去，所以对于调用方身份一定要做严格的校验，避免出现安全问题。

关于 RESTful API 基本就是这些东西，详细的知识建议翻阅[官方文档](http://www.restapitutorial.com/)，Tastypie 的[官方文档](https://django-tastypie.readthedocs.org/en/latest/) 更要好好翻，我当时因为没有好好翻文档吃了很多亏，谨记，谨记。

<del> 幸好这个月有 29 号，不然真的就要连续两个月断更了，打算三月份多写一篇来弥补下一月份缺的文章，暂定的两篇文章，一篇是 django 的简单开发总结，一篇应该会写一个游戏的求解程序，希望不会跳票。</del>
