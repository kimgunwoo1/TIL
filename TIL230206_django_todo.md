# Todo 목록 웹서비스

## 1. 환경설정

```python
python manage.py startproject mytodo
cd mytodo
python manage.py startapp todo
```

settings.py

```python
INSTALLED_APPS=[todo]
TIME_ZONE = 'Asia/Seoul'
```

models.py

```python
from django.db import models

# Create your models here.

class Todo(models.Model):
    title = models.CharField(max_length=100)
    description = models.TextField(blank=True)
    # table 상에 빈 값으로 들어 갈 수 있다.
    create = models.DateTimeField(auto_now_add=True)
    complete = models.BooleanField(default=False)
    important = models.BooleanField(default=False)
    #완료 여부, 중요한 일 여부

    def __str__(self):
        return self.title
```

forms.py

```python
from django import forms
from .models import Todo

class TodoForm(forms.ModelForm):
    class Meta:
        model = Todo
        fields = ('title', 'description','important','complete')
```

mytodo/urls.py

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('todo.urls')),
]
```

todo/urls.py

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.todo_list, name='todo_list'),                # 메인 리스트
    path('post/', views.todo_post, name='todo_post'),           # 신규 할인 등록
    path('<int:pk>/', views.todo_detail, name='todo_detail'),   # 할일 상세
    path('<int:pk>/edit/', views.todo_edit, name='todo_edit'),  # 할일 수정
    path('doen/', views.done_list, name = 'done_list'),         # 완료한 목록 리스트
    path('done/<int:pk>/',views.todo_done, name='todo_done'),   # 할일 완료 표기
]

```

admin.py

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('todo.urls')),
]
```

```python
python manage.py createsuperuser
```

## 2. 메인 리스트

urls.py

```python
urlpatterns = [
    path('', views.todo_list, name='todo_list'),]                # 메인 리스트
```

views.py

```python
from django.shortcuts import render
from .models import Todo

def todo_list(request):
    todos = Todo.objects.filter(complete=False)
    return render(request, 'todo/todo_list.html', {'todos': todos})
```

todo_list.html

```html
  <div class="container">
    <h1>TODO 목록 앱</h1>
    <p>
      <a href="{% url 'todo:todo_post' %}"><i class="bi-plus"></i>Add Todo</a>
      <a href="{% url 'todo:done_list' %}" class="" style="float:right">완료한 TODO 목록</a>
    </p>
    <ul class="">
      <!-- for문활용하여 할일보여주기 -->
      {% for todo in todos %}
      <li class="">
        <a href="{% url 'todo:todo_detail' pk=todo.pk %}">{{todo.title}}</a>
          <!-- 중요한 일이라면 -->
            {% if todo.important %}
                <span>!</span>
            {% endif %}
        <div style="float:right">
          <a href="{% url 'todo:todo_done' pk=todo.pk %}" class="">완료</a>
          <a href="{% url 'todo:todo_edit' pk=todo.pk %}" class="">수정하기</a>
          <a href="{% url 'todo:todo_delete' pk=todo.pk %}" class="">삭제하기</a>
        </div>
      </li>
      {% endfor %}

  </div>
```



## 3. 신규 할 일 등록

urls.py

```python
from django.urls import path
from . import views
app_name = 'todo'
urlpatterns = [
    path('post/', views.todo_post, name='todo_post'),           # 신규 할 일 등록
]
```

views.py

```python
from django.shortcuts import render, redirect
from .models import Todo
from .forms import TodoForm

def todo_post(request):
    if request.method == 'POST':
        form = TodoForm(request.POST)
        if form.is_valid():
            todo = form.save(commit=False)
            todo.save()
            return redirect('todo:todo_list')
    else:
        form = TodoForm()
    return render(request, 'todo/todo_post.html', {'form':form})
```

todo_post.html

```html
<form method="POST">
  {% csrf_token %} {{ form.as_p }}
  <button type="submit" class="btn btn-primary">등록</button>
</form>
```



## 4. 할 일 상세

urls.py

```python
from django.urls import path
from . import views
app_name = 'todo'
urlpatterns = [
    path('<int:pk>/', views.todo_detail, name='todo_detail'),   # 할일 상세
]
```

views.py

```python
def todo_detail(request, pk):
    todo = Todo.objects.get(id=pk)
    return render(request, 'todo/todo_detail.html', {'todo':todo})
```

todo_detail.html

```html
<h5 class="">{{todo.title}}</h5>
<p class="card-text">{{todo.description}}</p>
<a href="{% url 'todo:todo_list' %}" class="">목록으로</a>
```



## 5. 할 일 수정

urls.py

```python
path('<int:pk>/edit/', views.todo_edit, name='todo_edit'),  # 할일 수정
```

views.py

```python
def todo_edit(request, pk):
    todo = Todo.objects.get(id=pk)
    if request.method == 'POST':
        form = TodoForm(request.POST, instance=todo)
        if form.is_valid():
            todo = form.save(commit=False)
            todo.save()
            return redirect('todo:todo_list')
    else:
        form = TodoForm(instance=todo)

    return render(request, 'todo/todo_post.html', {'form':form})
```



## 6. 완료한 목록 리스트

urls.py

```python
path('done/', views.done_list, name = 'done_list'),         # 완료한 목록 리스트
```

views.py

```python
def done_list(request):
    todos = Todo.objects.filter(complete=True)
    return render(request, 'todo/done_list.html', {'todos': todos})
```

done_list.html

```html
<ul>
   {% for todo in todos %}
   <li class="list-group-item">
     <a href="{% url 'todo:todo_detail' pk=todo.pk %}">{{todo.title}}</a>
       <!-- 중요한 할일이라면 -->
       {% if todo.important %}
         <span class="badge badge-danger">!</span>
       {% endif %}
   </li>
   {% endfor %}
</ul>

```



## 7. 할일 완료 표기

urls.py

```python
path('done/<int:pk>/',views.todo_done, name='todo_done'),   # 할일 완료 표기
```

views.py

```python
def todo_done(request, pk):
    todo = Todo.objects.get(id=pk)
    todo.complete = True
    todo.save()
    return redirect('todo:todo_list')
```



## 8. 지우기

urls.py

```python
path('<int:pk>/delete/', views.todo_delete, name='todo_delete'), # 지우기
```

views.py

```python
def todo_delete(request, pk):
    todo = Todo.objects.get(id=pk)
    todo.delete()

    return redirect('todo:todo_list')
```



## 9. 파일업로드

settings.py

```python
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR/'media' #물리적 위치 -> 디렉토리 생성하기
```

models.py

```python
imagefile = models.ImageField(upload_to=settings.MEDIA_ROOT,blank=True, null=True)
```

```python
python manage.py makemigrations
python manage.py migrate
```

forms.py

```python
from django import forms
from .models import Todo

class TodoForm(forms.ModelForm):
    class Meta:
        model = Todo
        fields = ('title', 'description','important','complete', 'imagefile')
```

views.py

```python
from django.core.files.storage import default_storage
from django.core.files.base import ContentFile

def todo_post(request):
    if request.method == 'POST':
        form = TodoForm(request.POST)
        if form.is_valid():
            todo = form.save(commit=False)
            todo.save()

            upload_file = request.FILES['imagefile']
            upload = default_storage.save(upload_file.name, 
            ContentFile(upload_file.read()))
            # default_storage = /media, 화일 업로드 기능

            Todo.objects.filter(id=todo.id).update(imagefile=upload)
            #신규로 저장한 todo의 id를 참조해서 imagefile의 값을 update함
            return redirect('todo:todo_list')
    else:
        form = TodoForm()
    return render(request, 'todo/todo_post.html', {'form':form})
```



## 10. 페이지네이션



```python
python manage.py shell
# 임의로 100페이지 생성코드
from todo.models import Todo

for i in range(1,100):
     temp = Todo(title=i, description=i, complete=False, important=False)
     temp.save()
```

views.py

```python
	def todo_list(request):
    todos = Todo.objects.filter(complete=False)
    paginator = Paginator(todos,5) #5개씩
    page_num = request.GET.get('page','1') #page 값이 없으면 default 1

    #페이지에 맞는 모델
    page_obj = paginator.get_page(page_num)

    return render(request, 'todo/todo_list.html', {'todos': page_obj})
```

todo_list.html

```html
    <!-- 처음으로 -->
    <a href="?page=1">처음</a>
    
    <!-- 이전페이지  -->
      {% if todos.has_previous %}
        <a href="?page={{todos.previous_page_number}}">이전</a>
      {% else %}
        이전
      {% endif %}

		<!-- 페이징 -->
    {% for i in todos.paginator.page_range %}
      {% if todos.number >= i|add:-5 and todos.number <= i|add:+5 %}
      <!-- django template filter https://cocook.tistory.com/61 -->
        {% if todos.number == i %}
            <span style="color:red">{{i}}</span>
        {% else %}
        <a href="?page={{i}}">{{i}}</a>
        {% endif %}
      {% endif %}
    {% endfor %}

    <!-- 다음페이지  -->
      {% if todos.has_next %}
        <a href="?page={{todos.next_page_number}}">다음</a>
      {% else %}
        다음
      {% endif %}

    <!-- 끝으로 -->
    <a href="?page={{todos.paginator.num_pages}}">끝</a> 
```

