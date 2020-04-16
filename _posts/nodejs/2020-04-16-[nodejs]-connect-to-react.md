---
layout: post
title: "[nodejs] React + NodeJS X Express 연결하기"
subtitle: 'React 프로젝트를 빌드하고 NodeJS X Express에 연결해보자'
author: "yoogomja"
header-style: text
tags:
  - nodejs
  - react
  - deploy
  - express
---

지금 진행 중인 토이 프로젝트에서 `react`를 프론트로, `nodejs` + `express`를 백엔드 서버로 사용 중 이다. 이전에 다른 프로젝트에서도 
사용해본 경험이 있는데, 연결하는 부분은 자꾸 잊어먹게 되서 남겨둔다. 

기본적으로 `react` 개발서버를 그대로 사용하지 않고, 빌드해서 사용하는 것은 동일하다. 이떄 `create-react-app`을 사용해 프로젝트를 생성했다면,
`build` 키워드를 사용해서 빌드하고 있을 것이다. 지금 내 프로젝트의 구조는 아래와 같다.

```
|-- client
|    |
|    -- src ... 
|
|-- server
|   |
|   -- index.js...
```

위 구조에서 `client/build`경로에 빌드 파일이 위치하게 된다. 나는 이것을 `server/client/` 경로로 옮기도록 작업을 먼저 했다. 리액트의 `package.js` 
를 먼저 수정해보자.

```javascript
// ...
"scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    //  기존에 존재하던 파일을 삭제하고 빌드 후 파일을 옮긴다.
    "build_to_server": "rm -rf ../server/client && react-scripts build && mv build ../server/client",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
```

위와 같이 파일을 수정해주면, `/server/client` 경로에 빌드파일이 위치하게 된다. 이때 실행 명령어는 `yarn run build_to_server`가 된다. 

그 이후 백엔드 측에서도 준비가 되어야 하는데, 백엔드에서는 `app.js`파일을 수정해야 한다. 라우팅을 지정한 코드 아래쪽에 다음과 같은 코드들을 추가해준다. 

```javascript
    // 리액트 파일을 static 경로로 추가
    app.use(express.static(path.resolve(__dirname, './client')));

    // 이제 모든 주소는 리액트로 보냄
    app.get('*', function(request, response) {
    response.sendFile(path.resolve(__dirname, './client', 'index.html'));
    });
```

우선 static 파일의 경로로 빌드 파일이 존재하는 경로를 추가해준 후, 먼저 등록해둔 routing 항목들을 제외한 모든 주소를 `react` 측으로 보내준다. 
물론 이것은 `react-router`를 사용하는 것을 전제로 작업했다. 사용하지 않아도 크게 상관은 없겠지만. 

