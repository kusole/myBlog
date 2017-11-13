---
title: django_openstack_auth认证与集成方法
date: 2017-11-13 16:07:37
tags: dashborad
categories: openstack学习
---
## 开始准备
### 安装django开发环境
1.安装python3-pip和django1.11
```shell
sudo apt install python3-pip
sudo python3 -m pip install django
```
<!--more-->

2.建立Django工程
进入需要创建工程的目录，用如下命令创建工程：
```python
django-admin startproject netsec
```
netsec目录下的文件结构如下：
```
netsec\
    manage.py
    netsec\
        __init__.py
        settings.py
        urls.py
        wsgi.py
```
3.建立Django应用
进入需要创建工程的目录，用如下命令创建应用：
```python
django-admin startapp openstack_netsec
```
netsec目录下openstack_netsec目录的文件结构如下：
```python
openstack_netsec\
    __init__.py
    admin.py
    apps.py
    migrations\
        __init__.py
    models.py
    tests.py
    views.py
```

### 安装并配置django_openstack_auth
安装非常的简单：
1.运行 `python3 -m pip install django_openstack_auth`
2.添加 `openstack_auth` to `settings.INSTALLED_APPS`
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'openstack_auth',
    'openstack_netsec'
]
```
3.添加openstack_auth.backend.KeystoneBackend到settings.AUTHENTICATION_BACKENDS
```python
AUTHENTICATION_BACKENDS = ('openstack_auth.backend.KeystoneBackend',)
```
4.配置 API endpoint(s) in settings.py:
```python
OPENSTACK_HOST = "192.168.89.11"
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
```
5.包含'openstack_auth.urls'到urls.py文件.
```python
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^auth/', include('openstack_auth.urls')),
]
```
## 编写keystone应用
### 向导urls编写
在netsec/urls.py中添加如下：
```python
from openstack_auth import utils
from openstack_netsec import views

urlpatterns = [
    url(r'^$',views.splash, name='splash'),
    url(r'^admin/', admin.site.urls),
    url(r'^auth/', include('openstack_auth.urls')),
    url(r'^index/',views.index, name='index'),
]
```
`r'^$'`表示匹配到空项，即匹配`127.0.0.1:8000`时执行视图函数`views.splash`  
`r'^index/'`表示匹配到`127.0.0.1:8000/index/`时执行视图函数`views.index`

### views视图函数编写
views.splash函数的编写
```python
def splash(request):
    if not request.user.is_authenticated():
        return shortcuts.redirect('/index')
    response = shortcuts.redirect('/index')
    if 'logout_reason' in request.COOKIES:
        response.delete_cookie('logout_reason')
    if 'logout_status' in request.COOKIES:
        response.delete_cookie('logout_status')

    return response
```
当匹配到空白时，交由splash函数处理，首先判断用户是否已经登录过，如果没有登录过跳转到登录选择登录界面，当已经登录过，直接显示已经登录

### template编写
index.html
```html
<!DOCTYPE html>
<html lang="zh-cn">
<head>
    <meta charset="utf-8">
    <meta http-equiv="x-ua-compatible" content="ie=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>首页</title>
    <link rel="stylesheet" href="https://unpkg.com/mobi.css/dist/mobi.min.css">
</head>
<body>
<div class="flex-center">
    <div class="container">
        <div>
            <h1 class="logo"><a href="{% url 'index' %}">keystone 接入</a></h1>
            {% if user.is_authenticated %}
                <p>你已登录，欢迎你：<a href="#">{{ user.username }}</a></p>
                <button class="btn btn-default"><a href="{% url 'logout' %}?next={{ request.path }}">注销登录</a></button>
            {% else %}
                <p>你还没有登录，请
                    <button class="btn btn-default"><a href="{% url 'login' %}?next={{ request.path }}">登录</a></button>
                </p>
            {% endif %}
        </div>
    </div>
</div>
</body>
</html>
```
login.html
```html
<!DOCTYPE html>
<html lang="zh-cn">
<head>
    <meta charset="utf-8">
    <meta http-equiv="x-ua-compatible" content="ie=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>登录</title>
    <link rel="stylesheet" href="https://unpkg.com/mobi.css/dist/mobi.min.css">
    <style>
        .errorlist {
            color: red;
        }
    </style>
</head>
<body>
<div class="flex-center">
    <div class="container">
        <div class="flex-center">
            <div class="unit-1-2 unit-1-on-mobile">
                <h3>登录</h3>
                <form class="form" action="{% url 'login' %}" method="post">
                    {% csrf_token %}
                    {% for field in form.visible_fields %}
                            {{ field.label_tag }}
                            {{ field }}
                            {{ field.errors }}
                    {% endfor %}
                    <button type="submit" class="btn btn-primary btn-block">登录</button>
                    <input type="hidden" name="next" value="{{ next }}"/>
                </form>
            </div>
        </div>
    </div>
</div>
</body>
</html>
```
当用户已经登录时，显示登录成功和注销登录界面，当用户没有登录时，显示登录按键，当用户点击登录按钮后，进入登录界面，登录界面由django form表格生成，action为django的login，此时要输入的由用户名和密码，当用户填写好表格后，点击登录，form自动将表单提交给django_openstack_auth中的login视图函数来处理，之后交由controller的keystone的认证机制来进行认真，认证完成后，返回给用户一个凭据token,之后的操作都需要这个凭据来进行验证。

之后的效果如下：
![](http://otl4ekdmf.bkt.clouddn.com/openstackauth1.png)
![](http://otl4ekdmf.bkt.clouddn.com/openstackauth2.png)
![](http://otl4ekdmf.bkt.clouddn.com/openstackauth3.png)
