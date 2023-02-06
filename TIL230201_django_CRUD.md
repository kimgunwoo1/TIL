# django - CRUD



 1. 프로젝트 생성

    ```python
    django-admin startproject MyBoard
    ```

    

 2. 환경설정
    settings.py 에 INSTALLED_APPS, 'MyBoard' 추가
    DATABASES를 sqlite 에서 mariadb 사용할 수 있도록 환경설정

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',å
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'MyBoard',
]

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'myboard',   #Database 이름
        'USER' : 'root',    #ID / PW
        'PASSWORD' : '1234',
        'PORT' : '3306',
        'HOST' : 'localhost',
    }
}
```



 1. DB생성을 위한 준비
    models.py에 class 작성

    ```python
    from django.db import models
    
    class Myboard(models.Model):    #myboard table 이 될 것입니다.
        myname = models.CharField(max_length=100)
        mytitle = models.CharField(max_length=500)
        mycontent = models.CharField(max_length=2000)
        mydate = models.DateTimeField()
    
    # myboard table
    # 컬럼명 : myname, mytitle, mycontent, mydate
    # MyBoard의 object(row)를 출력할 때 메모리 출력대신에 정의된 것이 출력
    # 메모리에 들어가 있는 오브젝트를 대표할 이름을 지정
    # 프린트 할 때 확인이 용이하여 필수적인 것은 아니지만 사용한다.
        def __str__(self) :
            return str({'mytitle':self.mytitle, 'mydate':self.mydate})
    ```

    

 2. DB 생성
    workspace/django/MyBoard 이동

    ```python
    python manage.py makemigrations MyBoard
    # migrations/0001_initial.py생성된다.
    python manage.py migrate
    # DB에 table이 생성된다.
    ```

    

 3. 관리자 페이지 만들기
    workspace/django/MyBoard/MyBoard admin.py 작성

    ```python
    from django.contrib import admin #admin을 위해서 만들어준 라이브러리
    from . models import Myboard #우리가 정의한 모델을 import
    
    admin.site.register(Myboard)  #Myboard Table을 볼 수 있다
    ```

    

 4. admin 계정 생성
    python manage.py createsuperuser (admin/1234)

 5. Dbeaver에서 접속하기 위한 설치
    ```pip install mysqlclient```



게시판 CRUD

 1. 첫페이지
    index
    urls.py

    ```python
    from django.contrib import admin
    from django.urls import path
    from . import views
    
    urlpatterns = [
        path('admin/', admin.site.urls),
        path('', views.index, name = 'index'),
    ]
    ```

    views.py

    ```python
    from django.shortcuts import render
    from .models import Myboard
    
    def index(request):
        myboard_all = Myboard.objects.all().order_by('-id')
        return render(request, 'index.html', {'board_all':myboard_all})
    ```

    templates/index.html

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Document</title>
    </head>
    <body>
        <h1>Hello, Django</h1>
    
        <table border="1">
            <col width="50">
            <col width="100">
            <col width="500">
            <col width="150">
            <tr>
                <th>번호</th>
                <th>작성자</th>
                <th>제목</th>
                <th>작성일</th>
            </tr>
            {% if not board_all %}
                <tr>
                    <th colspan="4">-------작성된 글이 없습니다-----------</th>
                </tr>
            {% else %}
                {% for data in board_all %}
                    <tr>
                        <td>{{data.id}}</td>
                        <td>{{data.myname}}</td>
                        <td><a href="detail/{{data.id}}">{{data.mytitle}}</a></td>
                        <!-- http://127.0.0.1/detail/id -->
                        <td>{{data.mydate}}</td>
                    </tr>
                {% endfor %}
            {% endif %}
            <tr>
                <td colspan="4" align="right">
                    <input type="button" value="글작성" onclick="locationhref()">
                </td>
            </tr>
    
        </table>
    </body>
    </html>
    <script>
        function locationhref(){
            location.href = '/insert_form/'
        }
    </script>
    ```

    

    
    
    
    
 2. 게시물 삽입

    urls.py

    ```python
    from django.contrib import admin
    from django.urls import path
    from . import views
    
    urlpatterns = [
        path('admin/', admin.site.urls),
        path('', views.index, name = 'index'),
        path('insert_form/', views.insert_form, name='insertform'),
        path('insertres/', views.insert_proc, name='insertres')
    ]
    ```

    views.py

    ```python
    from django.contrib import admin
    from django.urls import path
    from . import views
    
    urlpatterns = [
        path('admin/', admin.site.urls),
        path('', views.index, name = 'index'),
        path('insert_form/', views.insert_form, name='insertform'),
        path('insertres/', views.insert_proc, name='insertres')
    ]
    ```

    templates/insert_form.html

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Document</title>
    </head>
    <body>
        <h1>Insert</h1>
        <!-- DB에 넣을 때는 (insert) form, post, action -->
    
        <form action="/insertres/" method="post">
            {% csrf_token %}
            <table border="1">
                <input type="hidden" name="id" >
                <tr>
                    <th>작성자</th>
                    <td><input type="text" name="myname" ></td>
                </tr>
                <tr>
                    <th>제목</th>
                    <td><input type="text" name="mytitle" ></td>
                </tr>
                <tr>
                    <th>내용</th>
                    <td>
                        <textarea name="mycontent" cols="60" rows="10"></textarea>
                    </td>
                </tr>
                <tr>
                    <td colspan="2" style="text-align: right">
                        <input type="button" value="취소">
                        <input type="submit" value="제출">
                    </td>
                </tr>
    
            </table>
        </form>
    </body>
    </html>
    ```

    

    

    

 3. 게시물확인

    

    Pangin을 위한 100개 게시물 생성

    

    python manage.py shell

    from django.utils import timezone

    from MyBoard.models import Myboard

    



