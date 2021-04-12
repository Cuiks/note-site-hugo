---
title: "Django富文本编辑器django-quill-editor"
date: 2021-04-08T21:31:44+08:00
tags: ["django", "quill"]
draft: true
---

## 前言

最近在做一个web项目，用到了富文本编辑器，主要目的是为了方便粘贴图片，类似截图后可以直接`ctrl+v`把粘贴板里的图片粘贴到编辑框中。项目使用了python3+Django，所以也就是使用一个Django的富文本编辑器。Django常用的富文本编辑器[django-ckeditor](https://github.com/django-ckeditor/django-ckeditor)，[django-tinymce](https://github.com/jazzband/django-tinymce)，[django-quill](https://github.com/coremke/django-quill)，[django-quill-editor](https://github.com/LeeHanYeong/django-quill-editor)等。本文主要介绍`django-quill-editor`的使用，因为在开发过程中发现粘贴进去的图片不能修改大小，非常不方便，于是通过Quill扩展的方式添加了图片添加大小的扩展，遂写此文章记录。:full_moon_with_face:

demo projec: [Cuiks/django-quill-editor](https://github.com/Cuiks/django-quill-editor)。该项目fork自[kensnyder/quill-image-resize-module](https://github.com/kensnyder/quill-image-resize-module)。添加了修改图片大小功能。

## 文章结构

- `django-quill-editor`在Django中的配置及使用
- `django-quill-editor`图片插件的配置及使用
- 遇到的问题

### 1、`django-quill-editor`在Django中的配置及使用

[官方文档](https://django-quill-editor.readthedocs.io/en/latest/)介绍的简单明了，但是也基本够用:neutral_face:

1. 安装

   `pip install django-quill-editor`

2. Django配置

   ```python
   # 添加到Django
   INSTALLED_APPS = [
       'django.contrib.admin',
       ...
       'django_quill',
   ]
   ```

3. Django model配置 && 数据迁移

   ```python
   # models.py
   from django.db import models
   from django_quill.fields import QuillField
   
   class QuillPost(models.Model):
       content = QuillField()
   ```

4. Django admin注册

   ```python
   from django.contrib import admin
   from .models import QuillPost
   
   @admin.register(QuillPost)
   class QuillPostAdmin(admin.ModelAdmin):
       pass
   ```

   注册完成后看到如下

   ![django-quill-editor-admin](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210411222619-django-quill-admin.png)

5. Form表单使用

   ```html
   # 在前端界面使用该方法引入
   <head>
       {{ form.media }}
   </head>
   # 或使用该方法引入
   <head>
       <!-- django-quill-editor Media -->
       {% include 'django_quill/media.html' %}
   </head>
   ```

   ```python
   # 表单中配置
   # forms.py
   from django import forms
   from .models import QuillPost
   
   class QuillPostForm(forms.ModelForm):
       class Meta:
           model = QuillPost
           fields = (
               'content',
           )
   
   # 返回的view使用表单
   # views.py
   from django.shortcuts import render
   from .forms import QuillPostForm
   
   def model_form_view(request):
       return render(request, 'form_view.html', {'form': QuillPostForm()})
   ```

   ```html
   # 前端页面展示表单
   <!-- form_view.html -->
   <form action="" method="POST">{% csrf_token %}
       {{ form.content }}
   </form>
   ```

   注：文档分别介绍了`Using as form`和`Using as ModelForm`。本文只介绍后者使用。

6. 自定义配置

   ```python
   # 修改Django的配置文件settings.py。添加
   QUILL_CONFIGS = {
       'default':{
           'theme': 'snow',
           'modules': {
               'syntax': True,
               'toolbar': [
                   [
                       {'font': []},
                       {'header': []},
                       {'align': []},
                       'bold', 'italic', 'underline', 'strike', 'blockquote',
                       {'color': []},
                       {'background': []},
                   ],
                   ['code-block', 'link'],
                   ['clean'],
               ]
           }
       }
   }
   ```

   关于配置项可参考：[Quill Configuration](https://quilljs.com/docs/configuration/)

7. 富文本编辑框的前端回显展示(Django模板语言)

   ```html
   # html页面展示
   <div class="form-control ql-editor ql-container ql-snow">{{ object.content.html|safe }}</div>
   ```

   使用`conten.html`属性，可以获取该字段html格式，添加`safe`控制，可在前端进行展示。

### 2、`django-quill-editor`图片插件的配置及使用

配置完成后，粘贴图片进去会发现往往图片充满了整个编辑框，看起来很不方便。通过查阅[Quill文档](https://quilljs.com/docs/modules/)。会发现Quill的Modules支持自定义扩展，所以确定方向：寻找(让我写，我也不会啊:new_moon_with_face:)Quill调整图片大小的扩展。[QuillJS](https://github.com/quilljs/quill)是一个比较热门的项目，很像web框架都开发了响应的扩展。在网上搜索发现项目[quill-image-resize-module](https://github.com/kensnyder/quill-image-resize-module)就是专门为Quill开发的扩展用于调整图片大小。到此我们就找到了Quill调节图片大小的方案。

接下来介绍怎么在`django-quill-editor`中使用自定义Modules。

1. 下载[Modules](https://github.com/kensnyder/quill-image-resize-module/blob/master/image-resize.min.js) 或 使用CDN引入Modules。此处我使用后者。google上找的[CDN地址](https://cdn.jsdelivr.net/npm/quill-image-resize-module@3.0.0/image-resize.min.js)，也不知道稳不稳:new_moon:

2. 项目中引入。此处需要注意，Modules必须在Quill后面引入，否则会报错。

   ```html
   <head>
       {% include 'django_quill/media.html' %}
       <!-- Quill resize-image.js -->
       <script src="https://cdn.jsdelivr.net/npm/quill-image-resize-module@3.0.0/image-resize.min.js"></script>
   </head>
   ```

   

3. 修改Django项目`settings.py`，引入`imageResize`

   ```python
   # Quill config
   QUILL_CONFIGS = {
       "default": {
           'theme': 'snow',
           'modules': {
               'syntax': True,
               'toolbar': [
                   [
                       {'font': []},
                       {'header': []},
                       {'align': []},
                       'bold', 'italic', 'underline', 'strike', 'blockquote',
                       {'color': []},
                       {'background': []},
                   ],
                   ['code-block', 'link', 'image', 'video'],
                   ['clean'],
               ],
               'imageResize': {  # open imageResize
                   'displaySize': True
               }
           }
       }
   }
   ```

4. 实现效果

   ![效果](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20210411230306-django_quill_editor_result.png)

## 3、遇到的问题

上面已经强调过，Quill的Modules必须要在Quill之后引入，否则就会报找不到的错误，如下：

```javascript
Uncaught TypeError: Cannot read property 'imports' of undefined
    at Object.<anonymous> (image-resize.min.js:1)
    at e (image-resize.min.js:1)
    at Object.<anonymous> (image-resize.min.js:1)
    at e (image-resize.min.js:1)
    at image-resize.min.js:1
    at image-resize.min.js:1
    at image-resize.min.js:1
    at image-resize.min.js:1

quill.min.js:7 quill Cannot import modules/imageResize. Are you sure it was registered?

quill.min.js:7 quill Cannot load imageResize module. Are you sure you registered it?

quill.min.js:7 quill Cannot import modules/imageResize. Are you sure it was registered?

quill.min.js:7 Uncaught TypeError: e is not a constructor
    at e.value (quill.min.js:7)
    at e.value (quill.min.js:7)
    at quill.min.js:7
    at Array.forEach (<anonymous>)
    at e.value (quill.min.js:7)
    at new t (quill.min.js:7)
    at new QuillWrapper (VM153 django_quill.js:9)
    at (index):157
    at (index):159

django_quill.js:1 Uncaught SyntaxError: Identifier 'QuillWrapper' has already been declared
```

如果使用框架应该就没有引入顺序这些麻烦了吧:neutral_face:

## 结束语

OK。本文就介绍在这里，有什么问题欢迎在评论区与我讨论。实现/修改代码在仓库[Cuiks/django-quill-editor](https://github.com/Cuiks/django-quill-editor)。

:end: