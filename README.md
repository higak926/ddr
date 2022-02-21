# README

This branch has an example.
Check it out!!

## API

### Step 1. Create Django Project
Let's create a Django Project.

```
docker container exec -it ddr_api django-admin startproject test_app
```

### Step 2. Create API Template
Let's create API Template.

```
docker container exec -it ddr_api mkdir test_app/webapi
docker container exec -it ddr_api django-admin startapp webapi test_app/webapi
```

### Step 3. Environment Settings
Let's set an Environment.

```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'webapi',#←追加
    'rest_framework',#←追加
    'corsheaders',#←追加
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'corsheaders.middleware.CorsMiddleware',#←追加
]

.....

# ↓settings.pyの末尾に追加
CORS_ORIGIN_WHITELIST = (
    'localhost:3000/',
    'localhost:3000',
)

# DB設定
DATABASES = {
    'default': {
    'ENGINE': 'django.db.backends.postgresql',
    'NAME': 'test_app',
    'USER': 'postgres',
    'HOST': 'db',
    'PORT': 5432,
  }
}

# アクセス許可
ALLOWED_HOSTS = ['*']
```

### Step 4. Create Models
Let's create Models.

```python:webapi/models.py
from django.db import models
class Profile(models.Model):
    name = models.CharField(max_length=64)
    email = models.EmailField()
    message = models.CharField(max_length=256)
    created_at = models.DateTimeField(auto_now_add=True)
```

### Step 5. Create Models And Serializers
Let's create Models snd Serializers.

```python:webapi/models.py
from django.db import models
class Profile(models.Model):
    name = models.CharField(max_length=64)
    email = models.EmailField()
    message = models.CharField(max_length=256)
    created_at = models.DateTimeField(auto_now_add=True)
```

```python:webapi/serializers.py
from rest_framework import serializers
from webapi.models import Profile
class ProfileSerializer(serializers.ModelSerializer):
    class Meta:
        model = Profile
        fields = ('id', 'name', 'email', 'message', 'created_at')
```

### Step 6. Migrate
Let's make migrations, migrate and createsuperuser.

```
docker container exec -it ddr_api python test_app/manage.py makemigrations
docker container exec -it ddr_api python test_app/manage.py migrate
docker container exec -it ddr_api python test_app/manage.py createsuperuser
```

### Step 7. Routing
Let's set Routing.

```python:test_app/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('webapi.urls')),
]
```

```python:webapi/urls.py
from django.urls import path
from . import views
urlpatterns = [
    path('api/profile/', views.ProfileListCreate.as_view() ),
]
```

```python:webapi/views.py
from webapi.models import Profile
from webapi.serializers import ProfileSerializer
from rest_framework import generics
class ProfileListCreate(generics.ListCreateAPIView):
    queryset = Profile.objects.all()
    serializer_class = ProfileSerializer
```

### Step 8. Create Test Data
Let's create Test Data.

```json:webapi/fixtures/profiles.json
[
    {
        "model": "webapi.profile",
        "pk": 1,
        "fields": {
            "name": "A子",
            "email": "a-ko@gmail.com",
            "message": "A子でーす",
            "created_at": "2019-01-01 00:00:00"
        }
    },
    {
        "model": "webapi.profile",
        "pk": 2,
        "fields": {
            "name": "B子",
            "email": "b-ko@gmail.com",
            "message": "B子でーす",
            "created_at": "2019-02-01 00:00:00"
        }
    },
    {
        "model": "webapi.profile",
        "pk": 3,
        "fields": {
            "name": "C子",
            "email": "c-ko@gmail.com",
            "message": "C子でーす",
            "created_at": "2019-03-01 00:00:00"
        }
    }
]
```

and insert data.
```
docker container exec -it ddr_api python test_app/manage.py loaddata profiles
```

### Step 9. Check The Operation
Let's check The Operation.
```
docker container exec -it ddr_api python /code/test_app/manage.py runserver 0:8000
```

### <font color="Navy">※ reference</font><br>
https://hodalog.com/tutorial-django-rest-framework-and-react/

<br>
Have a nice development !
