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

