# django - CRUD



 1. 프로젝트 생성

    ```python
    django-admin startproject dbtest
    ```

    

 2. 환경설정
    settings.py 에 INSTALLED_APPS, 'dbtest' 추가

 3. DB생성을 위한 준비
    models.py에 class 작성

    ```python
    from django.db import models
    
    
    
    class Myboard(models.Model):    #myboard table 이 될 것입니다.
    
    ​    myname = models.CharField(max_length=100)
    
    ​    mytitle = models.CharField(max_length=500)
    
    ​    mycontent = models.CharField(max_length=2000)
    
    ​    mydate = models.DateTimeField()
    
    
    
    # myboard table
    
    # 컬럼명 : myname, mytitle, mycontent, mydate
    
    
    
    ​    def __str__(self) :
    
    ​        return str({'myname':self.myname})
    ```

    

 4. DB 생성
    dbtest 이동

    ```python
    python manage.py makemigrations
    python manage.py migrate
    ```

    

 5. 관리자 페이지 만들기
    dbtest > admin.py 작성

    ```python
    from django.contrib import admin #admin을 위해서 만들어준 라이브러리
    from .models import Myboard #우리가 정의한 모델을 import
    
    admin.site.register(Myboard)  #Myboard Table을 볼 수 있다
    ```

    

 6. admin 계정 생성
    python manage.py createsuperuser (admin/1234)

 7. Dbeaver에서 접속하기 위한 설치

    ```pip install mysqlclient```





