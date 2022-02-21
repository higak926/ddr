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

``` python:settings.py
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

``` python:models.py
from django.db import models
class Profile(models.Model):
    name = models.CharField(max_length=64)
    email = models.EmailField()
    message = models.CharField(max_length=256)
    created_at = models.DateTimeField(auto_now_add=True)
```

### Step 5. Create Models And Serializers
Let's create Models snd Serializers.

``` python:models.py
from django.db import models
class Profile(models.Model):
    name = models.CharField(max_length=64)
    email = models.EmailField()
    message = models.CharField(max_length=256)
    created_at = models.DateTimeField(auto_now_add=True)
```

``` python:serializers.py
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

``` python:urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('webapi.urls')),
]
```

``` python:urls.py
from django.urls import path
from . import views
urlpatterns = [
    path('api/profile/', views.ProfileListCreate.as_view() ),
]
```

``` python:views.py
from webapi.models import Profile
from webapi.serializers import ProfileSerializer
from rest_framework import generics
class ProfileListCreate(generics.ListCreateAPIView):
    queryset = Profile.objects.all()
    serializer_class = ProfileSerializer
```

### Step 8. Create Test Data
Let's create Test Data.

``` json:profiles.json
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

### Step 9. Check The Works
Let's check The Works.

```
docker container exec -it ddr_api python /code/test_app/manage.py runserver 0:8000
```

## Front
### Step 1. Create React Project
Let's create React Project.

```
docker container exec -it ddr_front npx create-react-app .
docker container exec -it ddr_front npm install axios bulma
```

### Step 2. Create Views
Let's create Views.

``` html:index.html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="shortcut icon" href="%PUBLIC_URL%/favicon.ico" />
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1, shrink-to-fit=no"
    />
    <link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
    <title>React App</title>
  </head>
  <body>
    <nav class="navbar" role="navigation" aria-label="main navigation">
      <div class="container">
        <div class="navbar-brand">
          <a class="navbar-item" href="/">
            MY REACT-DJANGO TEST APP
          </a>

          <a role="button" class="navbar-burger burger" aria-label="menu" aria-expanded="false" data-target="navbarBasicExample">
            <span aria-hidden="true"></span>
            <span aria-hidden="true"></span>
            <span aria-hidden="true"></span>
          </a>
        </div>
      </div>
    </nav>
    <div class="container" id="root"></div>
  </body>
</html>
```

``` js:index.js
import 'bulma/css/bulma.min.css'; # ←追加
```

``` js:App.js
import React, { Component } from 'react';
import './App.css';
import axios from 'axios';

class Form extends React.Component {
  render() {
    return (
      <form id="post-data" onSubmit={this.props.handleSubmit}>
        <div className="field">
          <div className="control">
            <input
              className="input"
              type="text" name="name"
              id="name"
              placeholder="Name"
              value={this.props.name}
              onChange={this.props.handleChange}
            />
          </div>
        </div>
        <div className="field">
          <div className="control">
            <input
              className="input"
              type="text"
              name="email"
              id="email"
              placeholder="Email"
              value={this.props.email}
              onChange={this.props.handleChange}
            />
          </div>
        </div>
        <div className="field">
          <div className="control">
            <textarea
              className="textarea"
              name="message"
              id="body"
              cols="10"
              rows="3"
              placeholder="Message"
              value={this.props.message}
              onChange={this.props.handleChange}>
            </textarea>
          </div>
        </div>
        <input
          className="button is-fullwidth is-primary is-outlined"
          type="submit"
          value="SEND POST"
        />
      </form>
    );
  }
}

class UserList extends React.Component {
  renderRow() {
    const listItems = this.props.users.map((u) =>
      <tr key={u.id}>
        <td>{u.id}</td>
        <td>{u.name}</td>
        <td>{u.email}</td>
        <td>{u.message}</td>
        <td>{u.created_at}</td>
      </tr>
    );
    return(
      listItems
    );
  }
  render() {
    return (
      <table className="table is-fullwidth">
        <thead>
          <tr>
            <th>Id</th>
            <th>Name</th>
            <th>Email</th>
            <th>Message</th>
            <th>Created at</th>
          </tr>
        </thead>
        <tbody>
          {this.renderRow()}
        </tbody>
      </table>
    );
  }
}

class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      users: [],
      usersLength: 0,
      name: "",
      email: "",
      message: "",
    };
    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    if (event.target.name === 'name') {
      this.setState({name: event.target.value});
    }
    else if (event.target.name === 'email') {
      this.setState({email: event.target.value});
    }
    else if (event.target.name === 'message') {
      this.setState({message: event.target.value});
    }
  }

  handleSubmit(event) {
    event.preventDefault();
    const { name, email, message } = this.state;
    const conf = {
      'name': name,
      'email': email,
      'message': message
    };
    axios.post("http://localhost:8000/api/profile/?format=json", conf)
    .then(response => {
      this.state.users.unshift(response.data);
      this.setState({
        users: this.state.users,
        usersLength: this.state.users.length,
      });
    })
    .catch((error) => {
      console.error(error);
    });
  }

  componentDidMount() {
    axios.get('http://localhost:8000/api/profile/?format=json')
    .then(response => {
      this.setState({
        users: response.data.reverse(),
        usersLength: response.data.length,
      });
    })
    .catch((error) => {
      console.log(error);
    });
  }

  render() {
    return (
      <div className="columns is-multiline">
        <div className="column is-6">
          <div className="notification">
            This is my react-django app.
          </div>
        </div>
        <div className="column is-6">
          <Form
            name={this.state.name}
            email={this.state.email}
            message={this.state.message}
            handleChange={this.handleChange}
            handleSubmit={this.handleSubmit}
          />
        </div>
        <div className="column is-12" id="user-table">
          <p>There are {this.state.usersLength} users.</p>
          <UserList
            users={this.state.users}
          />
        </div>
      </div>
    );
  }
}

export default App;
```

``` css:App.css
// ↓ 追加
#user-table {
  overflow: scroll;
}
```

### Step 3. Check The Final Works
Let's check The Final Works.

```
docker container exec -it ddr_api python test_app/manage.py runserver 0:8000
docker container exec -it ddr_front npm start
```

Please access to localhost:3000, and If it works, it's successful🎉🎉🎉

### <font color="Navy">※ reference</font><br>
https://hodalog.com/tutorial-django-rest-framework-and-react/

<br>
Have a nice development !
