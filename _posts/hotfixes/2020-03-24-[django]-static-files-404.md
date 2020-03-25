---
layout: post
title: "[django] 장고 nginx 배포 시 static 경로의 404에러"
subtitle: '배포시 conf 파일 수정하기'
author: "yoogomja"
header-style: text
tags:
  - Django
  - nginx
  - deploy
  - docker
  - hofixes
---

이번에 사이드 프로젝트로 django + restframework로 서버 배포를 앞두고 있다. 이때, 환경 설정을 담당하는 친구는 docker와 nginx를 이용해서 배포를 준비하고 있는데, static 파일들을 정상적으로 불러오지 못하고 있었다.    
   
확인한 과정은 STATIC_ROOT와 STATIC_DIR, STATIC_URL이 정상적으로 등록되어 있는지 확인을 했고, 문제는 발생하지 않았다. 개발 과정에서는 문제가 없었으니 미칠 노릇.    

그래서 확인해보니 nginx.conf 파일에 문제가 있었다. 서버의 경로를 지정해 주는 부분이 문제였는데, 내용은 이러했다. 

```python
server {
    # 서버에서 static 경로
    location /static/ {
        # 이 부분이 문제였다.
        alias /app/static;
    }
    ...
}
```

위 부분에서 alias에 있는 주소는 나중에 /static/ 경로로 들어오는 주소들의 나머지 부분들을 /app/static/의 경로로 연결해주는데, `localhost:8000/static/img.png`여야할 경로가 `localhost:8000/staticimg.png`가 되어버린 상황이 되어버리는 것. 그래서 아래와 같이 수정했다.

```python
server {
    # 서버에서 static 경로
    location /static/ {
        # / 하나가 만든 참사.. 
        alias /app/static/;
    }
    ...
}
```
위 문제 덕분에 환경설정을 담당하는 친구와 새벽에 한동안 고생했는데, 참으로 의미없다 의미없어.
