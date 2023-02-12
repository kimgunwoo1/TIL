# 설문조사



## 1. 환경설정

프로젝트 생성

```bash
django-admin startproject mypolls
cd mypolls
python manage.py startapp polls
```

settings.py

```
INSTALLED_APPS = ['polls']
```

urls.py

```python
from django.contrib import admin
from django.urls import path, include
from django.views.generic.base import RedirectView

urlpatterns = [
    path('admin/', admin.site.urls),
    path('polls/', include('polls.urls')),
  	# http://127.0.0.1:8000/polls/~~
    path('', RedirectView.as_view(url='polls/') ),
    # 첫페이지에서 에러나지 않게 리다이렉트
]

```

polls/urls.py 

````python
from django.urls import path
from . import views

app_name='polls'
# http://127.0.0.1:8000/polls/~~~
urlpatterns = [
    path('', views.index, name='index'),
    # http://127.0.0.1:8000/polls/~~~ : 질문리스트
    path('<int:question_id>/', views.detail, name='detail'),
    # http://127.0.0.1:8000/polls/숫자 : 초이스리스트 form
    path('<int:question_id>/vote/', views.vote, name='vote'),
    # http://127.0.0.1:8000/polls/숫자/vote/ : 선택한 것을 DB에 저장
    path('<int:question_id>/result', views.result, name='result'),
    # http://127.0.0.1:8000/polls/숫자/result/ : 결과 보여 주기
]
````

polls/models.py

```python
from django.db import models

# Create your models here.
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date=models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.question_text
        #object의 메모리 번지 이름

class Choice(models.Model):
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
    question_id = models.ForeignKey(Question, on_delete=models.CASCADE)
    #Question의 질문이 삭제되면 연결된 choice_text들도 자동 삭제
    def __str__(self):
        return self.choice_text    
```

```bash
python manage.py makemigrations
python manage.py migrate
```



## 2. 질문리스트

views.py

```python
from django.shortcuts import render, get_object_or_404
from .models import Question, Choice
from django.http import HttpResponseRedirect
from django.urls import reverse
# Create your views here.

def index(request):
    question_list = Question.objects.all().order_by('-pub_date')
    return render(request, 'polls/index.html', {'question_list': question_list})
```

templates/polls/index.html

```django
{% if question_list %}
 <ul>
     {% for question in question_list %}
         <li>
             <a href="{% url 'polls:detail' question.id %}">{{question.question_text}}</a>
             {% comment %} http://127.0.0.1:8000/polls/숫자 {% endcomment %}
         </li>
     {% endfor %}
 </ul>
{% else %}
    <strong>투표항목이 존재하지 않습니다.</strong>
{% endif %}
```



## 3. 초이스 리스트

views.py

```python
def detail(request, question_id):
    question = get_object_or_404(Question, id=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```

detail.html

```django
    <h1>{{question.question_text}}</h1>

    {% if error_message %}
        <p><strong>{{error_message}}</strong></p>
    {% endif %}
    <form action="{% url 'polls:vote' question.id %}" method="post">
        {% csrf_token %}
        {% for choice in question.choice_set.all %}
            <input type='radio' name='choice' value={{choice.id}}
                id = 'choice{{forloop.counter}}'>
            <label for='choice{{forloop.counter}}'>{{choice.choice_text}}</label>
            <br>
        {% endfor %}
        <br><br>
        <input type="submit" value='투표'>
    </form>
    {% comment %} 
        question.choice_set.all
        . question의 정보에서 foregin에 연결된 table choice table에 넣어져 있는 집합
        . 그 안에 있는 것을 모두 가지고 옴(all)
        . choice는 연결된 고정명 (소문자)
        . _set은 고정
    {% endcomment %}
```



## 4. 선택한 것을 DB에 저장

views.py

```python
def vote(request, question_id):
    #어떤 질문인지 가지고 옴
    question = get_object_or_404(Question, id=question_id)

    #radio button, default 값이 없기 때문에 오류가 날 수 있어요.
    try:
        select_choice = question.choice_set.get(id=request.POST['choice'])
        # 질문 객체에 연결된 항목객체 중 get 조건에 맞는 것을 가져 옴

    except(KeyError, Choice.DoesNotExist):
        return render(request, 'polls/detail.html', {'question':question, 'error_message':'아무것도 입력하지 않았습니다.'})
    else:
        select_choice.votes += 1
        select_choice.save() #DB값 반영
    return HttpResponseRedirect(reverse('polls:result', args=(question_id,)))
```



## 5. 결과 보여주기

views.py

```python
def result(request, question_id):
    question = get_object_or_404(Question, id=question_id)
    return render(request, 'polls/result.html', {'question': question})
```

result.html

```django
	    <h1>{{question.question_text}}</h1>
    <ul>
        {% for choice in question.choice_set.all %}
            <li>{{choice.choice_text}} - {{choice.votes}} 표</li>
        {% endfor %}
    </ul>
```



## 6. shell 에서 DB에 자료 넣기

```python
# shell 실행
python manage.py shell

# 모델 import
from polls.model import Question
# 모델로 객체를 생성하고 지정한 필드에 값을 넣어준다.
question = Question(question_text='선호하는 스마트폰은?')
# 데이터를 정장한다.
question.save()

# 저장된 데이터를 확인한다.
question id
question.question_text

```



### tip 1. sqlite

```bash
# Database 보기
.databases

# table 보기
.tables

# update
update blog set description='my google blog' where id=2;

#sqlite 종료
.quit
```



### tip 2. vscode에서 html template과 django template 동시에 사용하기

> 파일 > 기본설정 > 설정
>
> 검색 : emmet:include
>
> 항목추가 클릭
>
> 항목 : django-html
>
> 값에 : html