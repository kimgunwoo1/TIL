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
from django.shortcuts import render, redirect
from .forms import MyMemberForm

def index(request):
    return render(request, 'index.html')

def register(request):
    if request.method == 'GET':
        return render(request, 'register.html', {'form': MyMemberForm()})
    else:
        form = MyMemberForm(request.POST)

        if form.is_valid():
            form.save()  #DB 저장
        return redirect('index')
```

index.html

```html
    <h1>My Member</h1>
    <a href="{% url 'register' %}">회원가입</a>
    <br>
    <a href="{% url 'login' %}">로그인</a>
```

![](https://github.com/kimgunwoo1/TIL/blob/master/image/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-02-08%20%EC%98%A4%ED%9B%84%201.02.16.png?raw=true)

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

![](https://github.com/kimgunwoo1/TIL/blob/master/image/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-02-08%20%EC%98%A4%ED%9B%84%201.03.06.png?raw=true)

## 3. 로그인 로그아웃

urls.py

```python
from django.contrib.auth import views as auth_views #views와 이름이 겹치치 않도록

# login logout
    path('login/', auth_views.LoginView.as_view(template_name = 'login.html'), name='login'),
    # 장고가 회원가입 만들어 놓았듯이, 로그인 로그아웃도 만들어 놓은 것임
    # get : login.html 보여주고, post : 겍체 세션에 저장해서 보내줌
    # settings.py LOGIN_REDIRECT_URL='/result' : 로그인 성공하면 '/result'로 요청
    path('result/', views.result, name='result'),
    
    path('logout/', auth_views.LogoutView.as_view(), name='logout')
    # settings.py LOGOUT_REDIRECT_URL='/' : 로그인 성공하면 '/'로 요청
```

