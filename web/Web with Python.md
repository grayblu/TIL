# Web with Python

### 1. Client - Server

### 2. Flask

Flask 실행 방법(Bash)

기본적으로 **FLASK_APP=파일명.py flask run** 명령어를 사용하여 서버를 구동시킨다

여기서 아래와 같이 환경변수를 추가하면 다음 실행 시 간편한 명령어만으로 앱을 실행할 수 있다.

```bash
$ export flask_app=app.py
$ flask run
```

웹서버 프로그램 구현 시 항상 고정으로 입력하는 코드는 아래와 같다.

```python
from flask import Flask
app = Flask(__name__)
```

여기서 app은 flask 내장 함수이므로 다른 변수명으로 하지 않도록 한다. 만약 다른 변수명으로 FLASK 인스턴스를 생성할 경우 다른 설정이 필요하다.

URL 설정은 route()를 이용한다.

```python
@app.route('/')
def index():
    return 'Index Page'
```

Variable

```python
@app.route('/user/<int:username>') # 타입은 string, int, float, path, uuid ...
def show_user_profile(username):
    # show the user profile for that user
    return f'User {username}'
```



### 3. Django

MTV 구조와 동작 메커니즘

Model: 데이터 관리

Template: 사용자가 보는 화면

View: 중간 관리자(MVC에서 Controller 역할)

**browser(req) -> urls -> views -> template ->views-> browser(res)**

bash에서 프로젝트 생성시

```bash
$ django-admin startproject intro .
```

그러면 프로젝트 디렉토리 내 manage.py가 생성된다. 그리고 서버 실행 시 명령어는 다음과 같다.

```bash
$ python manage.py runserver
```

App 등록 시 settings.py에서  아래와 같이 추가하며, django.xxx는 내장 Apps이다. 애플리케이션 추가 시 각 성격에 맞는 위치(주석)에 입력한다.

```bash
$ python manage.py startapp 앱이름
```



```python
INSTALLED_APPS = [
    
    # local apps
    'pages.apps.PagesConfig',
    # third party apps
    # django apps
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

```

Django는 한글 지원이 가능한 프레임워크이며, settings.py에서 아래와 같은 설정을 통해 한글화 가능하다.

```python
# Internationalization
# https://docs.djangoproject.com/en/2.2/topics/i18n/

LANGUAGE_CODE = 'ko-kr'

TIME_ZONE = 'Aisa/Seoul' 
```

local app 추가 후 intro의 urls.py에서 urlpatterns에서 local path를 추가하며, 이때 작성 순서를 아래에서 위로 작성한다.(업데이트 시 기존 코드 위에 추가)

```python
urlpatterns = [
    path('pages/', include('pages.urls')),
    # 고정
    path('admin/', admin.site.urls),
]
```

예로 pages라는 애플리케이션을 만들고, 해당되는 path를 입력한다. 이 때 urls는 자동으로 만들어지지 않으므로 pages 디렉토리에 urls.py를 만든다. 해당 설정을 통해 url 요청에 따라 urlpatterns가 경로를 인식하고 해당 애플리케이션 디렉토리에 있는 urls.py로 요청한다.

pages 디렉토리에서 urlpatterns 코드를 작성하고, template을 요청할 수 있도록 경로를 아래와 같이 설정한다.

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index),
]
```

해당 설정으로 인해, /pages 요청에 대하여 인식하고 views.py에 있는 index()함수를 호출한다. index()

### 구현 중에 겪은 예외 상황: `django.urls.exceptions.NoReverseMatch:`

url 구성이 잘못된 경우 => url 입력하는 코드를 살핀다. 또는 template 에서 받는 인자 값을 확인 것.(variable routing인지 object인지)

```html
<!-- <a href="{% url 'movies:detail' movieCd %}"><button type="button" class="btn btn-primary">상세보기</button></a> -->

```

Django Template 에서 주석을 하더라도 Template Pattern을 읽기 때문에 NoReverseMatch 에러 발생할 수 있음.

이 경우에는 `{# comment #}`를 이용할 것!

해당 이슈를 다루는 내용을 참조한다

[https://www.pythoncircle.com/post/424/solving-django-error-noreversematch-at-url-with-arguments-and-keyword-arguments-not-found/]()



※ Django는 각 특징마다 위->아래, 아래->위 순으로 코드 작성해야 하는 규칙이 있다.



settings.py 내 MIDDLEWARE는 url 패턴을 인식하기 전에 코드를 확인하는 역할을 한다. 일종의 필터 기능과 같다.



**static 파일**

**app_name/static/** 경로 안의 파일들만 인식 가능하다. 예를 들어 /static 내 새롭게 images 디렉토리를 추가하고 그 안에 이미지를 불러올 때 아래와 같이 입력하여 브라우저에서 이미지 처리가 가능하다.

```python
{% extends 'boards/base.html' %}
{% load static %}

{%  block content %}
    <h1>static 파일 실습 </h1>
    <img src="{% static 'images/sample_img.jpg' %}" alt="sample img">
{% endblock %}
```

`{% load static %}`은 항상 extends 하단에 배치한다.



**장고에서 관리자 등록**

```bash
$ python manage.py createsuperuser
```

admin.py에서 모델을 등록하면 관리자 페이지에서 모델 객체를 확인할 수 있다.

```python
from django.contrib import admin
from .models import Board

# Register your models here.
admin.site.register(Board)

```

관리자페이지에 Baord라는 모델을 확인할 때 컬럼 헤드를 변경할 수 있다. 이때, BoardAdmin 클래스를 추가한다.

```python
class BoardAdmin(admin.ModelAdmin):
    list_display = ['id', 'title', 'content', 'created_at', 'updated_at']

admin.site.register(Board, BoardAdmin)
```



Django의 확장 기능을 설치할 수도 있으며, 설치 후 settings.py 의 INSTALLED_APPS에 추가한다. 해당 모듈은 서드 파티이므로 Local 앱과 기본 설치된 django.contrib.xxx 사이에 추가한다.

```bash
$ pip install django-extensions
```

```
INSTALLED_APPS = (
    ...
    'django_extensions',
)
```

확장 모듈은 많은 기능이 있지만 그 중 이를 통해 shell_plus를 사용하면  모델을 자동으로 임포트한다.

```bash
$ python manage.py shell_plus
```



### DB 초기화 방법

1. migrations 에서 000$ 번호 붙은 파일들만 지우기
2. db.splite3 파일 지우기(서버 꺼져 있어야 함)



### Static 경로 설정 및 모델에 image 추가

```python
# 새로운 static 경로 추가, Django에서 해당 경로 자동 인식
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'crud', 'assets', 'images'),
]

# 실제 파일이 저장되는 경로와 이미지 URL 경로 설정
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
MEDIA_URL = '/media/'
```







```python
image = models.ImageField(blank=True)
```

새로운 컬럼 추가 시 테이블 가장 오른쪽에 배치됨, `blank=True`를 통해 이미지 없이도 게시 가능

models.py 수정 후 `makemigrations`, `migrate`명령을 통해 ORM 세팅을 완료한다.

이후 sqlite3에서 테이블에 컬럼이 추가 되었는지 확인한다.

```sqlite
sqlite> .schema boards_board
CREATE TABLE IF NOT EXISTS "boards_board" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "title" varchar(10) NOT NULL, "content" text NOT NULL, "created_at" datetime NOT NULL
, "updated_at" datetime NOT NULL, "image" varchar(100) NOT NULL);

```

### VS CODE에서 Django 프로젝트

```bash
student@M90311 MINGW64 ~/Desktop/hyunho/django/myform
$ python -m venv form-venv

student@M90311 MINGW64 ~/Desktop/hyunho/django/myform
$ ls
form-venv/

student@M90311 MINGW64 ~/Desktop/hyunho/django/myform
$ source form-venv/Scripts/activate

```

가상환경 실행 명령어는 `$ source form-venv/Scripts/activate`

이후 Django 모듈과 pip update명령을 통해 프로젝트 시작 준비를 마친다.

이후 bash에서 `startproject` , `startapp`, `runserver`를 통해 서버 실행 되는지 확인하며, 이후 서버 및 페이지 구현은 Django 가이드라인에 따라 작성한다.



[링크: Django에서 bootstrap4 적용](https://django-bootstrap4.readthedocs.io/)

### Django 서버 구축 방식

FBV - Function Based View

CBV - Class Based View

### Celery를 이용한 백그라운드 작업 수행

celery는 비동기 작업을 수행할 수 있는 파이썬의 패키지이다. 브로커가 필요하며 메시지를 통해 작업이 할당된다.

브로커는 Redis를 선택했으며, celery와 함께 설치할 수 있다.

```bash
pip install -U 'celery[redis]'
```

```
Celery is a project with minimal funding, so we don’t support Microsoft Windows. Please don’t open any issues related to that platform.
```



### 4. Requests

파이썬에서 URL 요청을 하기 위해 사용되는 모듈이다. requests 패키지에 대한 설명이 있는 공식 문서를 참고하여 URL 요청을 어떻게 하는지 확인했다.

[링크바로가기](https://2.python-requests.org//en/master/user/quickstart/)

```python
import requests

res = requests.get('https://www.naver.com')
```

위와 같은 요청에 대한 응답을 통해 html, json 등 요청한 URL에 해당되는 응답 내용을 확인할 수 있다. `res`는 Response 객체이며, 이를 통해 필요한 정보를 얻을 수 있다.

##### Query String을 통해 특정 데이터를 요청 시

```python
payload = {'key1': 'value1', 'key2': 'value2'}
res = requests.get('https://httpbin.org/get', params=payload)

# res.url 출력 시 https://httpbin.org/get?key2=value2&key1=value1
```

requests.get() 메소드의 argument 로 params를 사용하여 파라미터 접근이 가능하다.

