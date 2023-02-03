# Django 회원가입, 로그인

> 2023년 02월 3일 금요일

## Django 회원가입

---



|                              | urls      | views                                           | templates                                            | 비고                                                         |
| ---------------------------- | --------- | ----------------------------------------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| 회원가입<br />form 요청      | /register | def register()<br />if request.method == 'GET'  | register.html<br />.name, pw, email                  | form  입력정보 전달<br />active, method                      |
| 회원가입정보<br />DB저장요청 | /register | def register()<br />if request.method == 'POST' | 없음                                                 | 로그인 페이지 이동                                           |
| 로그인<br />form요청         | /login    | def login()<br />if request.method == 'GET'     | login.html<br />.name, pw, 로그인버튼,<br />회원가입 | 로그인버튼: POST로<br />성공여부확인요청<br />회원가입:<br />회원가입 form요청 |
| 로그인 성공여부<br />확인    | /login    | def login()                                     | 없음                                                 | 성공시 : 로그인성공페이지<br />(logout)<br />실패시 : 로그인페이지 |
| 로그아웃<br />요청           | /logout   | if request.method == 'POST'                     | 없음                                                 | 로그인페이지Form페이지                                       |



## 1. 환경설정

### (1) models.py

```python
class MyMember(models.Model):
    myname = models.CharField(max_length=100)
    mypassword = models.CharField(max_length=100)
    myemail = models.CharField(max_length=100)

    def __str__(self):
        return str({'myname':self.myname, 'myemail':self.myemail})
```

```python
# migrations/0002_mymember.py 생성
python manage.py makemigrations
# DB에 테이블 생성
python manage.py migrate
```



### (2) admin.py

```python
from . models import MyMember
admin.site.register(MyMember)  
```



> ###### Session 
>
> > 사용자의 브라우저와 웹 서버 간의 상태 관리 기술이다.
>
> > Session은 사용자의 브라우저에서 생성된 데이터를
> > 웹서버에 저장하여, 웹 어플리케이션이 사용자와 관련된 정보를 관리하는데 사용한다.
> > 예를 들어, 사용자의 로그인 정보, 장바구니 내역 등을 저장할 수 있다.
> >
> > ```python
> > #session data 저장
> > request.session['key'] = value
> > 
> > # session data 읽어오기
> > value = request.session.get('key', default)
> > ```



## 2. 회원가입

### (1) index.html

```html
<input type="button" value="회원가입" onclick="registerhref()">
<script>    
    function registerhref(){
        location.href = 'register'
    }
</script>
```

```html
<h3><a href="{% url 'register' %}">회원가입</a></h3>
```



### (2) urls.py

```python
from django.contrib import admin
from django.urls import path
from . import views
urlpatterns = [
    path('register/', views.register , name = 'register'),
]
```

### (3) views.py

```python
def register(request):
    if request.method == 'GET':
        return render(request, 'register.html')
```



### (4) register.html

```html
<body>
    <h1>Register</h1>
    <form action="{% url 'register' %}" method="post">
        {% csrf_token %}        
        NAME : <input type="text" name="myname">
        <br>
        PASSWORD : <input type="password" name="mypassword">
        <br>
        EMAIL : <input type="text" name="myemail">
        <br>
        <input type="submit" value="회원가입">
    </form>    
</body>
```



## 3. 로그인







## 4.업로드/다운로드

django/django-admin startproject updown

### (1) 환경설정

updown/updown/settings.py

```python
INSTALLED_APPS = ['updown']
TEMPLATES = [{'DIRS': [BASE_DIR/'templates'],}]
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR/'media'
```

urls.py

```python
from django.contrib import admin
from django.urls import path
from . import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', views.index, name='index'),
    path('upload/', views.upload_proc, name='upload'),
]
```

views.py

```python
from django.shortcuts import render

def index(request):
    return render(request, 'index.html')

def upload_proc(request):
    pass
```

폴더 생성

updown/templates

updown/media



### (2) fileupload

templates/index.html

```html
    <form action="{% url 'upload' %}" method="post" enctype="multipart/form-data">
        {% csrf_token %}
        <input type="file" name="uploadfile">
        <br>
        <input type="submit" value="업로드">
    </form>
```

views.py

```python
from django.shortcuts import render, redirect
from django.core.files.storage import default_storage
from django.core.files.base import ContentFile

def upload_proc(request):
    upload_file = request.FILES['uploadfile']
    upload = default_storage.save(upload_file.name, 
        ContentFile(upload_file.read()))
    return redirect('index')
    # default_storage : settings.py 에서 설정한 
    #           MEDIA_ROOT = BASE_DIR/'media'
    # upload_file.name : random을 화일명에 더해서 올림(덮여 쓰여지지 않도록)
```

















## 1. Git

### [1]Git 초기 설정

- 최초 한 번만 설정합니다.
- 누가 커밋 기록을 남겼는지 확인할 수 있도록 이름과 이메일을 설정합니다.

```bash
git config --global user.name "이름"
git config --global user.email "메일 주소"
```

- 작성자가 올바르게 설정되었는지 확인 가능 합니다.

```bash
git config --global -l
또는
git config --global --list
```

### [2]Git 기본 명령어

#### (0) 로컬 저장소

- Working Directory (=Working Tree) : 사용자의 일반적인 작업이 일어나는 곳
- Staging Area(= Index) : 커밋을 위한 파일 및 폴더가 추가되는 곳
- Repository : staging area에 있던 파일 및 폴더의 변경사항(커밋)을 저장하는 곳
- Git은 Working Directory -> Staging Area -> Repository 의 과정으로 버전 관리를 수행합니다.


#### (1) git init

```bash
git init
```

- 현재 작업중인 디렉토리를 Git으로 관리한다는 명령어
- .git 이라는 숨김 폴더를 생성하고, 터미널에는 (master) 라고 표기됩니다.

! 주의 사항

1. 이미 Git 저장소인 폴더 내에 또 다른 Git 저장소를 만들지 않습니다.(중첩 금지) 즉, 터미널에 이미 (master)가 있다면, git init을 절대 입력하면 안됩니다.
2. 절대로 홈 디렉토리에서 git init을 하지 않습니다. 터미널의 경로가 ~ 인지 확인합니다.


#### (2) git status

- Working Directory와 Staging Area에 있는 파일의 현재 상태를 알려주는 명령어

#### (3) git add

```bash
# 파일
$ git add a.txt
# 폴더
$ git add my_folder/
# 현재 디렉토리에 속한 파일/폴더 전부
$ git add .
```

- Untracked, Modified -> Staged 로 상태를 변경합니다.

#### (4) git commit

```bash
$ git commit -m "first commit"
```

- Staging Area에 올라온 파일의 변경 사항을 하나의 버전(커밋)으로 저장하는 명령어

#### (5) git log

```bash
$ git log
```

- 커밋의 내역 (ID, 작성자, 시간, 메시지 등)을 조회할 수 있는 명령어
- 옵션
  - --online : 한 줄로 축약해서 보여줍니다.
  - --graph : 브랜치와 머지 내역을 그래프로 보여줍니다.
  - --all : 현재 브랜치를 포함한 모든 브랜치의 내역을 보여줍니다.
  - --reverse : 커밋 내역의 순서를 반대로 보여줍니다.(최신이 가장 아래)
  - -p : 파일의 변경 내용도 같이 보여줍니다.
  - -2 : 원하는 갯수 만큼 내역을 보여줍니다. (2말고 임의의 숫자 사용 가능)

#### (6) 한눈에 보는 Git 명령어

![git_command](https://github.com/kimgunwoo1/TIL/blob/master/image/git.png?raw=true)

## 2. Github

### [1] 원격 저장소 (Remote Repository)

> Github의 원격 저장소를 이용해 내 컴퓨터의 로컬 저장소를 다른 사람과 공유할 수 있습니다.

#### (1) Github에서 원격 저장소 생성

- New Repository (화면 오른쪽 상단 더하기+ 버튼 클릭)

### (2) 로컬 저장소와 원격 저장소 연결

- 원격 저장소의 주소 복사
- 해당 디렉토리로 가서 vscode 실행
- git init을 통해 해당 폴더를 로컬 저장소로 만들어 준다.

```bash
$ git init
```

- git remote

  - 로컬 저장소에 원격 저장소를 등록, 조회, 삭제 할 수 있는 명령어

  1. 등록
     git remote add <이름> <주소> 형식으로 작성합니다.

     ```bash
     $ git remote add origin https://github.com/kimgunwoo1/TIL.git
     ```

  2. 조회

    ```bash
  git remote -v
    ```

  3. 삭제

### (3) 원격 저장소에 업로드

`git remote rm <이름>` 혹은 `git remote remove <이름>` 으로 작성합니다.  

>로컬과 원격 저장소의 연결을 끊는 것이지, 원격 저장소 자체를 삭제하는 게 아닙니다.

```bash
$ git remote rm origin
$ git remote remove origin
```

### (3) 원격 저장소에 업로드

- 커밋을 업로드
- 먼저 로컬 저장소에서 커밋을 생성해야 원격 저장소에 업로드 할 수 있습니다.

1. 로컬 저장소에서 커밋 생성

```bash
# 현재 상태 확인
$ git status

$ git add day1.md

$ git commit -m "upload TIL Day1"

# 커밋 확인
$ git log --online
```

2. git push

   - 로컬 저장소의 커밋을 원격 저장소에 업로드하는 명령어
   - git push <저장소 이름> <브랜치 이름> 형식으로 작성합니다.

   ```bash
   $ git push origin master
   ```


### [2] 실습

- README.md 파일 push 해보기