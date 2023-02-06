# photo - project

## 1. 환경설정

### (1) 프로젝트 생성

```
django-admin startproject myphoto
cd myphoto
python manage.py startapp photo
```

### (2) settings.py

```python
INSTALLED_APPS = ['photo',]
TIME_ZONE = 'Asia/Seoul'
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'myphoto',   #Database 이름
        'USER' : 'root',    #ID / PW
        'PASSWORD' : '1234',
        'PORT' : '3306',
        'HOST' : 'localhost',
    }
}
```

### (3) DB 생성
models.py

   ```python
   class Photo(models.Model):
       title = models.CharField(max_length=50)
       autho = models.CharField(max_length=50)
       image = models.CharField(max_length=200)
       description = models.TextField()
       price = models.IntegerField()
   
       def __str__(self):
           return str(self.title)
   ```

   ```
   python manage.py makemigrations
   python manage.py migrate
   ```

### (4) 관리자계정  admin.py
   ```python
   from django.contrib import admin
   from .models import Photo
   # Register your models here.
   
   admin.site.register(Photo)
   ```

   ```python
   python manage.py createsuperuser
   ```

## 2. 첫페이지 만들기

### (1) 경로 설정 myphoto/urls.py

   ```python
   from django.contrib import admin
   from django.urls import path, include
   
   urlpatterns = [
       path('admin/', admin.site.urls),
       path('', include('photo.urls')),
       #http://127.0.0.1/~~~ ==> photo.urls
   ]
   ```

   photo/urls.py

   ```python
   from django.urls import path, include
   from . import views
   urlpatterns = [
       path('', views.photo_list, name='photo_list'),
       path('photo/<int:pk>/', views.photo_detail, name='photo_detail'),
       path('photo/new/', views.photo_post, name='photo_post'),
       path('photo/<int:pk>/edit/', views.photo_edit, name = 'photo_edit'),
   ]
   ```

### (2) view설정   myphoto/photo/views.py
   ```python
   from django.shortcuts import render, get_object_or_404, redirect
   from .models import Photo
   
   # Create your views here.
   def photo_list(request):
       photo = Photo.objects.all()
       return render(request, 'photo/photo_list.html', {'photos':photo})
   
   def photo_detail(request):
       pass
   
   def photo_post(request):
       pass
   
   def photo_edit(request):
       pass
   ```

   

### (3) template 페이지  photo/templates/photo/photo_list.html
   ```python
   <h1>사진목록페이지</h1>
       <h3><a href="{% url 'photo_post' %}">New Photo</a></h3>
       <section>
           {% for photo in photos %}
           <div>
               <h2><a href="{% url 'photo_detail' pk=photo.id %}">{{photo.title}}</a></h2>
               <img src="{{photo.image}}" alt="{{photo.title}}" width="300px">
               <p>{{photo.author}}, {{photo.price}}</p>
           </div>
           {% endfor %}
       </section>
   ```

> 사진 입력
>   http://127.0.0.1:8000/admin
>   사진 link를 입력한다.

## 3. Detail 페이지 만들기

### (1) 경로설정

```python
from django.urls import path, include
from . import views
urlpatterns = [
    path('photo/<int:pk>/', views.photo_detail, name='photo_detail'),
]
```



### (2) View 설정

```python
from django.shortcuts import render, get_object_or_404, redirect
from .models import Photo
from .forms import PhotoForm

def photo_detail(request, pk):
    photo = get_object_or_404(Photo, pk=pk)
    return render(request, 'photo/photo_detail.html', {'photo':photo})
```



### (3) template 페이지 photo_detail.html

```html
    <h1><a href="{% url 'photo_list' %}">홈으로 돌아가기</a></h1>
    <h1>{{photo.title}}</h1>

    <h3><a href="{% url 'photo_edit' pk=photo.id %}">Edit Photo</a></h3>
    <section>
        <div>
            <img src="{{photo.image}}" alt="{{photo.title}}" width="300px">
            <p>{{photo.description}}</p>
            <p>{{photo.author}}, {{photo.price}}</p>
        </div>
    </section>
```



## 4. Edit 페이지

### (1) 경로설정

```python
from django.urls import path, include
from . import views
urlpatterns = [
    path('photo/<int:pk>/edit/', views.photo_edit, name = 'photo_edit'),
]
```

### (2) View 설정

```python
from django.shortcuts import render, get_object_or_404, redirect
# get_object_or_404 : 에러처리까지 한다.
from .models import Photo
from .forms import PhotoForm
def photo_edit(request,pk):
    photo = get_object_or_404(Photo, pk=pk)
    if request.method == 'GET':
        form = PhotoForm(instance=photo)
    elif request.method == 'POST':
        form = PhotoForm(request.POST, instance=photo)
        if form.is_valid():
            photo = form.save(commit=False)
            photo.save()
            return redirect('photo_detail', pk=photo.id)

    return render(request, 'photo/photo_post.html', {'form': form})
```



## 5. 이미지 업로드

(1) 환경설정

settings.py

```python
INSTALLED_APPS = [    
    'photo',
]
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR/'media'
```

index.html(template)

```html
<form action="{% url 'upload' %}" method="POST" enctype="multipart/form-data">
        {% csrf_token %}
        <input type="file" name="uploadfile">
        <br>
        <input type="submit" value="업로드">
    </form>
```

views.py

```python
def upload_proc(request):
    upload_file = request.FILES['uploadfile']
    upload = default_storage.save(upload_file.name,
        ContentFile(upload_file.read()))
    # return redirect('index')
    return render(request,'download.html',{'filename':upload})
    # default_storage : settings.py에서 설정한 MEDIA_ROOT = BASE_DIR/'media'
    # upload_file.name : random을 화일명에 더해서 올림 (덮어쓰여지지 않도록)


def download_proc(request,filename):
    return HttpResponse(default_storage.open(filename).read(),
        content_type='application/force-downlaod')

        #content_type='application/force-downlaod' => 다운로드 할수 있도록 하는 타입
```



(2) forms.py -> imagefile 추가, image 삭제

```python
from django import forms
from .models import Photo

class PhotoForm(forms.ModelForm):
    # image -> imagefile 변경
    class Meta:
        model = Photo
        fields = ('title','author' ,
        'imagefile',
        'description','price')
```



