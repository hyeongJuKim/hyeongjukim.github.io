---
layout: post
title:  React + Django Project 만들기
author: "HyeongJu"
tags: [Python, React]
comments: true
description: react를 frontend로 django를 backend로 사용하기 예제입니다.
---

## React + Django Project 만들기
> react를 frontend로 django를 backend로 사용하기 예제입니다.

## required
- pipenv
- pyhon3
- django

## info
- directory name: `react-django`
- project name: `backend`
- app name: `oneLine`
- frontend app name: `frontend`
  
## install pipenv
```bash
$ brew install pipenv # Mac OS 기준
```
Python에서 권장하는 python 가상환경 설정 패키지를 설치한다.

## install django
```bash
$ cd <프로젝트 디렉토리명>
$ pipenv --three
$ pipenv shell
```
해당 디렉토리로 이동 후에 python3 가상 환경을 만들고 shell 명령어로 실행합니다.

```bash
(react-django)$ pipenv install django
(react-django)$ django-admin startproject backend .
```
pipenv를 통해 django를 설치하고, `backend`라는 이름으로 django app을 생성합니다


```bash
(react-django)$ python manage.py startapp oneLine
```
`onLine`라는 이름의 django 앱을 하나 생성합니다.  
    

```python
# backend/settings.py 설정 변경
INSTALLED_APPS = [
    # 만든 앱을 추가해주세요.
    'oneLine.apps.OnelineConfig'
]

LANGUAGE_CODE = 'ko-kr'

TIME_ZONE = 'Asia/Seoul'
```

## setting django
```bash
(react-django)$ python manage.py migrate
(react-django)$ python manage.py createsuperuser
```
super-user 생성

```bash
(react-django)$ python manage.py runserver
```
`localhost:8000/admin`에서 정상적으로 접속되는지 확인하기.


## 모델과 어드민 파일 다루기

```python
# oneLine/models.py

from django.db import models

class WiseSaying(models.Model):
    text = models.TextField()

    def __str__(self):
        return self.text
```
`wisesaying` 클래스 추가하기.

```python
# oneLine/admin.py

from django.contrib import admin
from .models import WiseSaying

admin.site.register(WiseSaying)

```
admin에 추가하기.  

```bash
(react-django)$ python manage.py makemigrations
(react-django)$ python manage.py migrate
```
다시 한번 마이그레이트를 한다.

## install rest_framework 
```bash
(react-django)$ pipenv install djangorestframework
```
djangp rest framework app 설치

```python
# backend/settings.py
INSTALLED_APPS = [
    'rest_framework',
]
```
설치한 엡 적용하기.

```python
# oneLine/serializers.py

from rest_framework import serializers
from .models import WiseSaying

# 직렬화
class WiseSayingSerializer(serializers.ModelSerializer):
    class Meta:
        model = WiseSaying
        fields = '__all__'
```
파일 생성해서 직렬화를 하자.  

```python
# oneLine/views.py에 추가
from django.shortcuts import render
from rest_framework import viewsets
from .serializers import WiseSayingSerializer
from .models import WiseSaying


class WiseSayingView(viewsets.ModelViewSet):
    serializer_class = WiseSayingSerializer
    queryset = WiseSaying.objects.all()
```

```python
# backend/urls.py에 rest를 확인 할 수 있도록 url을 추가하자.
from django.contrib import admin
from django.urls import path
from django.urls import path, include
from rest_framework import routers
from oneLine import views

router = routers.DefaultRouter()
router.register('wisesaying', views.WiseSayingView, 'wisesaying')

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include(router.urls))
]

```

## react app 설치하기
```bash
(react-django)$ npx create-react-app frontend
```

```json
,"proxy": "http://localhost:8000"
```
frontend/package.json에서 port 변경하기

## cors 다루기
```bash
(react-django)pipenv install django-cors-headers
```

```python
# backend/settings.py 

INSTALLED_APPS = [
    # 나머지 부분은 같습니다.
    'corsheaders',
]

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    # 맨 위에 넣어주세요. 나머지는 같습니다.
]    

CORS_ORIGIN_WHITELIST = (
    'http://localhost:3000',
    )
```

## index 페이지 보여주기
```python
# backend/settings.py

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [
            # 추가된 부분입니다.
            os.path.join(BASE_DIR, 'frontend', 'build'),
        ],
      # 나머지는 같습니다.
    }
] 

STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'frontend', 'build', 'static')
]
```
(frontend/build 폴더에서 찾는 것)  
또 STATICFILES_DIRS에 적어준 값에서 static 폴더를 찾아서 적용시킵니다.

```bash
(react-django)$ npx yarn build
```
빌드를 하면 frontend/build 폴더에 static 파일들이 생성된다.  


## axios를 통한 데이터 처리
axios를 통해서 http get,post, delete 요청을 할 수 있다.
```bash
 (react-django)$ npm install axios
```

```react
// src/TextItem.js

import React from "react";

const TextItem = ({ text, id, handleClick }) => {
  return (
    <div>
      <p>{text}</p>
      <button onClick={() => handleClick(id)}>삭제</button>
    </div>
  );
};
```
간단하게 데이터를 입력 할 수 있는 `src/TextItem.js`를 만들자.

```react
// App.js

import React, { Component } from "react";
import "./App.css";
import TextItem from "./TextItem";
import axios from "axios";

// 장고의 템플릿에서 폼 데이터를 만들 때 {%raw%}{% csrf token %}{%endraw%}을 대체하는 코드
axios.defaults.xsrfCookieName = "csrftoken";
axios.defaults.xsrfHeaderName = "X-CSRFToken";


class App extends Component {
  state = {
    value: "",
    textList: []
  };

  componentDidMount() {
    this._renderText();
  }
  render() {
    const { textList } = this.state;
    console.log(textList);
    return (
      <div className="App">
        <h1>OneLine App</h1>
        <div>
          <label>
            Text:
            <input
              type="text"
              value={this.state.value}
              onChange={this._handleChange}
            />
          </label>
          <button onClick={this._handleSubmit}>submit</button>
        </div>
        <h2>Long Text</h2>
        {textList.map((text, index) => {
          return (
            <TextItem
              text={text.text}
              key={index}
              id={text.id}
              handleClick={this._deleteText}
            />
          );
        })}
      </div>
    );
  }
  _handleChange = event => {
    this.setState({ value: event.target.value });
  };
  _handleSubmit = () => {
    const { value } = this.state;
    axios
      .post("/api/wisesaying/", { text: value })
      .then(res => this._renderText());
  };
  _renderText = () => {
    axios
      .get("/api/wisesaying/")
      .then(res => this.setState({ textList: res.data }))
      .catch(err => console.log(err));
  };
  _deleteText = id => {
    axios.delete(`/api/wisesaying/${id}`).then(res => this._renderText());
  };
}

export default App;
```

App.js의 코드를 수정하자.  


## 참고사이트
이 글은 [justmakeyourself blog](https://justmakeyourself.tistory.com/entry/django-connect-react){: target="_blank"} 을 참고해서 작성했습니다.