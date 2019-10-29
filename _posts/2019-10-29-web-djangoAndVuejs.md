---
title: Django와 Vue.js 연동하기
categories:
- web
tag:
- vue
- django
---

## Build Django Project 

```console
$ django-admin startproject config
$ mv config django-vue
```

우선 장고 프로젝트를 config로 생성 후 최상위 프로젝트명을 django-vue로 변경해준다.

```console
$ cd django-vue
$ tree .
django-vue
├── config
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── manage.py
```

다음과 같이 기본 장고프로젝트 구조를 갖게된다. 
프로젝트 구현시에 장고에서는 API만을 구현하고, Vue.js로 frontend 작업을 모두할 것이므로 장고 앱 생성명을 api로 해준다.

```console
$ ./manage.py startapp api
```
```
django-vue
├── api
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── migrations
│   │   └── __init__.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── config
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── manage.py
```

migration후에 서버를 실행해보면 다음과 같이 시작페이지가 뜨는 것을 확인할 수 있다.

```console
$ ./manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying sessions.0001_initial... OK
$ ./manage.py runserver
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
October 29, 2019 - 06:17:41
Django version 2.2.6, using settings 'config.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
[29/Oct/2019 06:17:42] "GET / HTTP/1.1" 200 16348
[29/Oct/2019 06:17:42] "GET /static/admin/css/fonts.css HTTP/1.1" 304 0
[29/Oct/2019 06:17:42] "GET /static/admin/fonts/Roboto-Bold-webfont.woff HTTP/1.1" 304 0
[29/Oct/2019 06:17:42] "GET /static/admin/fonts/Roboto-Regular-webfont.woff HTTP/1.1" 304 0
[29/Oct/2019 06:17:42] "GET /static/admin/fonts/Roboto-Light-webfont.woff HTTP/1.1" 304 0
```
![]({{ site.url }}/assets/images/django_start_page.png)

## Build Vue.js Project

```console
$ vue create .

Vue CLI v3.11.0
? Generate project in current directory? Yes
```

현재 장고프로젝트에 Vue Project를 생성하면 다음과 같은 구조를 갖게된다.

```
django-vue
├── api
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── migrations
│   │   └── __init__.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── config
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── manage.py
├── package-lock.json
├── package.json
├── public
│   └── static
│   │  └── favicon.ico
│   └── index.html
└── src
    ├── App.vue
    ├── assets
    │   └── logo.png
    ├── components
    │   └── HelloWorld.vue
    ├── main.js
    └── store
        └── index.js
```

여기서 `public/` 디렉터리 안에 `static/` 디렉터리를 생성해주고, 이미지를 옮겨준다.

```console
$ npm run serve
 DONE  Compiled successfully in 2766ms                                                                                          15:30:26

  App running at:
  - Local:   http://localhost:8080/

  Note that the development build is not optimized.
  To create a production build, run npm run build.
```
vue 프로젝트 서버를 실행한후 `http://localhost:8080`에 들어가면 다음과 같이 정상적으로 설정한 것을 확인할 수 있다.

![]({{ site.url }}/assets/images/vue-start-page.png)

## Connecting Vue.js and Django: Webpack

webpack을 이용해 Django와 Vue.js 프로젝트를 연동할 것이다.

### django-webpack-loader([owais/django-webpack-loader](https://github.com/owais/django-webpack-loader))

```console
$ pip install django-webpack-loader
$ pip list
Package               Version
--------------------- -------
Django                2.2.6
django-webpack-loader 0.6.0
pip                   19.3.1
pytz                  2019.3
setuptools            40.6.2
six                   1.12.0
sqlparse              0.3.0
```

`django-webpack-loader` 를 설치해준다.

#### settings.py

settings.py에 webpack loader와 관련된 설정과 vue 프로젝트 경로 설정을 해줄 것이다.

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'webpack_loader',
    'api',
]
```
`webpack_loader` 를 INSTALLED_APPS에 추가해준다.

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'public')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```
Vue 프로젝트 생성된 public 디렉토리를 Template DIRS로 설정해준다.

```python
WEBPACK_LOADER = {
    'DEFAULT': {
        'CACHE': DEBUG,
        'BUNDLE_DIR_NAME': '/bundles/',  # must end with slash
        'STATS_FILE': os.path.join(BASE_DIR, 'webpack-stats.json'),
    }
}
```
다음과 같이 WEBPACK_LOADER와 관련된 설정을 추가해준다.

#### urls.py

`config/urls.py` 파일에 api app url 설정을 추가해준다.

```python
from django.contrib import admin
from django.urls import path
from . import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', views.HomeView.as_view(), name='home'),
]
```

#### views.py

`config/` 하위에 views.py를 새로 생성해준뒤 HomeView 추가해준다.

```python
from django.shortcuts import render
from django.views.generic import TemplateView

# Create your views here.
class HomeView(TemplateView):
  template_name = 'index.html'
```

#### public/index.html

```html
{% load render_bundle from webpack_loader %}
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <link rel="icon" href="<%= BASE_URL %>favicon.ico">
    <title>django-vue</title>
  </head>
  <body>
    <noscript>
      <strong>We're sorry but django-vue doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
    </noscript>
    <div id="app"></div>
    {% render_bundle 'index' %}
    <!-- built files will be auto injected -->
  </body>
</html>
```

`{% load render_bundle from webpack_loader %}` 와 `{% render_bundle 'index' %}`를 `index.html` 에 추가해준다.

### webpack-bundle-tracker([owais/webpack-bundle-tracker](https://github.com/owais/webpack-bundle-tracker))

Vue.js에서는 `webpack-bundle-tracker`에 관련된 설정을 해줘야한다.

#### install

```console
$ npm install --save-dev webpack-bundle-tracker
```
webpack-bundle-tracker를 설치해준다. 

#### vue.config.js

최상위폴더(django-vue/) 하위에 `vue.config.js` 파일을 생성한다.

```js
// vue.config.js
const BundleTracker = require('webpack-bundle-tracker');

module.exports = {
  publicPath: 'http://127.0.0.1:8080/',
  outputDir: './dist/',

  chainWebpack: config => {
    config.optimization.splitChunks(false)

    config.plugin('BundleTracker').use(BundleTracker, [{filename: './webpack-stats.json'}])

    config.devServer.public('http://0.0.0.0:8080').host('0.0.0.0').port(8080).https(false).headers({"Access-Control-Allow-Origin":["\*"]})
  },

  pages: {
    index: 'src/main.js'
  }
}
```
다음과 같이 webpack과 관련된 설정을 해주면 된다.

## Run Server

```console
$ npm run serve
$ ./manage.py runserver
```
Django 프로젝트와 Vue 프로젝트를 각각 서버를 실행한 후 `http://localhost:8000`(장고서버) 에 접속하면 Vue프로젝트 실행 첫화면이 뜨는 것을 확인할 수 있다.

![]({{ site.url }}/assets/images/vue-start-page.png)

## 참조

- [Vue.js and Django: Modern Multi-Page Website](https://blog.gundammc.com/vue-js-django/)
- [gtalarico/django-vue-template](https://github.com/gtalarico/django-vue-template)