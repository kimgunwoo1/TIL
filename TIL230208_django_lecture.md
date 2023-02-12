# LECTURE



## 1. 환경설정

```bash
django-admin startproject lecture
cd lecture
python manage.py startapp bbs
python manage.py startapp users
```

settings.py

```python
INSTALLED_APPS = [
     'bbs',
    'users',
]
TEMPLATES = [
        'DIRS': [BASE_DIR/'templates'],
]
STATIC_URL = 'static/'
STATICFILES_DIRS=[BASE_DIR/'static']

MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR/'media'

AUTH_USER_MODEL = 'users.Member'
# 인증에 사용할 class를 지정 default : auth_user -> 우리가 만들 users.Member로 사용

AUTHENTIFICATION_BACKENDS = [
    'django.contib.auth.backends.ModelBackend',
    # 정해져 있음 대소문자 구분 됨.
]
```

lecture/users/models.py

```python
from django.db import models
from django.conf import settings
from django.contrib.auth.models import AbstractUser

# Create your models here.

class Member(AbstractUser):
    mobile = models.CharField(max_length=20)
    image = models.ImageField(upload_to=settings.MEDIA_ROOT, blank=True, null=True)
```

```
python manage.py makemigrations
python manage.py migrate
```





lecture/users/admin.py

```python
from django.contrib import admin
from .models import Member
# Register your models here.

admin.site.register(Member)
```



lecture/lecture/urls.py

```python
from django.contrib import admin
from django.urls import path, include
from django.views.generic.base import TemplateView
from django.conf.urls.static import static
from django.conf import settings

urlpatterns = [
    path('admin/', admin.site.urls),
    path('users/', include('users.urls')),
    path('', TemplateView.as_view(template_name='index.html'), name='home'),
    # http://127.0.0.1
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
		# js, css, image 정적 파일 관리 (Django가 webserver 역할을 함)

```

lecture/users/urls.py

```python
from django.urls import path, include
from django.conf.urls.static import static
from django.conf import settings
from . import views

app_name = 'users'
urlpatterns = [
    path('login', views.login, name='login' ),
    
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

lecture/users/views.py

```python
from django.shortcuts import render

# Create your views here.
def login(request):
    pass
```



디렉토리 생성

/lecture/static/css

/lecture/static/js

<img src = "https://github.com/kimgunwoo1/TIL/blob/master/image/directory.png?raw=true" width="240px">

부트스트랩

```
pip install django-bootstrap4
```

## 2. 초기화면



## 3. 회원가입

## 4. 로그인 / 로그아웃

## 5. 게시판

### 1. 게시판 목록

### 2. 새글작성

### 3. 글 상세보기

### 4. 수정/삭제/좋아요

### 5. 댓글

