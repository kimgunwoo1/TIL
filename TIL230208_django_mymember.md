# 회원가입



## 1. 환경설정

프로젝트 생성

```
django-admin startproject mymember
```

settings.py

```
INSTALLED_APPS = ['mymember']
```

mymember/mymember/templates 폴더생성

mymember/mymember/forms.py

```python
from django import forms
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User
#from .models import MyMember

class MyMemberForm(UserCreationForm):
    #UserCreationForm 'username, password, lastname'
    email = forms.EmailField()
    first_name = forms.CharField()
    last_name = forms.CharField()

    class Meta:
        model = User  #admin User
        fields = ('username', 'password1', 'password2', 'email','first_name', 'last_name')
```

table 생성

```
python manage.py migrate
```



## 2. 회원가입

urls.py

```python
from django.contrib import admin
from django.urls import path
from . import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', views.index, name='index'), 
    #회원가입 form 보여줘, http://127.0.0.1/
    path('register/', views.register, name='register'),
    #회원가입 정보 DB에 넣어줘 요청, http://127.0.0.1/register
    
]
```

views.py

```python
from django.contrib import admin
from django.urls import path
from . import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', views.index, name='index'), 
    #회원가입 form 보여줘, http://127.0.0.1/
    path('register/', views.register, name='register'),
    #회원가입 정보 DB에 넣어줘 요청, http://127.0.0.1/register
    
]	
```

index.html

```html
    <h1>My Member</h1>
    <a href="{% url 'register' %}">회원가입</a>
    <br>
    <a href="{% url 'login' %}">로그인</a>
```

![스크린샷 2023-02-08 오후 1.02.16](/Users/macpro/TIL/image/스크린샷 2023-02-08 오후 1.02.16.png)

register.html

```django
    <h1>Register</h1>
    <form action="{% url 'register' %}" method='POST'>
    {% csrf_token %}
        <table>
            {{form.as_table}}
        </table>
        <input type="submit" value='회원가입'>
    </form>
```



