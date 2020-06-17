---
layout: post
title: "React + Django 환경설정 하기"
subtitle: 'React 프로젝트를 빌드하고 Django에 연동해보자'
author: "yoogomja"
header-style: text
tags:
  - django
  - react
  - deploy
  - infrastructure
---

`react`를 개발하는 내용을 보면 보통 `create-react-app`을 이용해 앱을 만들고 개발서버를 실행하는 과정이 많이 나오는데, 그 이후 배포에 관해서는 많이 다루고 있지 않다.  
굳이 이유를 생각보자면 `react`는 클라이언트 프레임워크일 뿐이기에 배포될 서버 환경은 다양해서 그러리라 생각된다.   

내가 토이 프로젝트로 개발하고 있는 환경에서는 `react`와 `react-router`를 사용하고 있다. 이 경우에 `react` 개발 서버에서 자동으로 주소에 맞는 파일을 보여주는데,
실제 배포될 때에는 저 개발 서버를 이용해서 배포하면 안되고, 빌드한 후에 배포해주어야 한다. 배포를 위해 빌드하면 흩어져있던 javascript 파일들을 하나로 합쳐준다. 이를 번들링이라고 한다. 
    
보통 이렇게 빌드를 하면 해당 프로젝트에서 `/build`라는 경로에 번들링된 javascript파일들과 css등등의 파일들이 있는데, 배포를 할때는 이 파일들을 불러오도록 해주어야 한다.  
장고에서는 프로젝트의 템플릿을 해당 파일로 연결해주고, 따로 설정되지 않은 모든 주소에 대해서 리액트 파일로 접근하도록 설정하는 식이다.

이 프로젝트에서 장고 서버는 API 서버 역할을 하도록 만들어졌다. 그래서 모든 api 관련 항목들은 `/api`라는 주소 아래에 연결시켜두었다.    
`/api` 경로를 제외한 다른곳에서는 리액트 파일을 불러오도록 하는 것이다. 진행 과정은 다음과 같다.

- [x] 리액트 빌드 파일을 옮기기
- [x] 리액트 빌드 파일의 경로를 template으로 설정 
- [x] static 경로 연결
- [x] url 연결

### 1. 리액트 빌드 파일을 옮기기 

`/build` 경로에 생성된 파일들은 장고 프로젝트에 `/client`라는 이름의 폴더를 만들어 그쪽으로 복사해준다. 앞으로 모든 `react` 파일들은 빌드 후 해당 폴더로 옮겨 담아야 한다.

### 2. 리액트 빌드 파일의 경로를 template으로 설정 

`settings.py`를 수정해야한다. 해당 파일안에 TEMPLATE 항목을 수정해야한다. 

```python
    #settings.py

    TEMPLATE = [
        {
            'BACKEND': 'django.template.backends.django.DjangoTemplates',
            # 템플릿 위치를 client 폴더로 고정 ! 
            'DIRS': ['client'],
            'APP_DIRS': True,
            'OPTIONS': {
                # ...
            },
        }
    ]
```

### 3. static 경로 설정

리액트에서 가지고 있는 이미지 파일, favicon등을 사용하기 위해서는 `staticfiles_dir`에 클라이언트 측 static 경로를 추가해주어야 한다.   
사실 상 api서버에서는 별도의 이미지 파일을 들고 있을 필요가 없으니, 해당 폴더만 static 폴더가 되는 셈. 방법은 다음과 같다. 

```python
    # settings.py

    STATICFILES_DIRS = [
        # 실제 static 파일은 모두 client 측에서 소유 
        os.path.join(PROJECT_ROOT, 'client/static')
    ]
```

### 4. url 설정

이제 연결된 템플릿의 내용을 주소에 매칭시켜 주어야한다. 프로젝트의 `urls.py`를 수정해야한다. 

```python
    # urls.py
    
    from django.views.generic import TemplateView
    
    urlpatterns = [
        # 모든 주소를 우선 client 쪽으로 연결 시킴
        url(r'^$', TemplateView.as_view(template_name='index.html'),name='index'),
        # ... 
    ]
```

위의 순서로 연결해주면, 실제 리액트의 배포가 이루어지게 된다.   

실제 배포가 된다면, 개발서버와 장고서버가 제각기 다른 서버가 아니라 한 서버에서 동작하는 것이 되므로, 리액트에서 주소를 요청할 때
배포 상황인지, 개발 서버인지 확인 후 주소를 때에 맞춰 호스트를 변경해주도록 설정하는 것이 좋다.   

이것은 리액트 측에 개발환경과 배포환경에서의 환경변수를 각각 작성해주어야 한다. 리액트의 루트 폴더에 `.env.development` 파일과 `.env.production` 파일을 만들어준다. 
그리고 `.env.development`파일안에는 `REACT_APP_DB_HOST=개발서버_주소`, `.env.production`에서는 `REACT_APP_DB_HOST=배포서버_주소`형태로 입력해준다.

위 파일들을 만들어주고 설정해주면 주소가 환경변수로 등록되어, 상황에 따라 맞는 주소로 요청을 보내게 된다. 
리액트내에서 저 주소를 불러올 때에는 아래와 같이 해주면된다. 

```javascript
    // host에 주소를 붙여서 사용하자
    const host = process.env.REACT_APP_DB_HOST;
```

이렇게 해주면 배포할 준비가 모두 끝나게 된다.