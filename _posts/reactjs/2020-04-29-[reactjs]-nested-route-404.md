---
layout: post
title: "hooks와 react-router의 중첩 Route와 404 페이지 설정하기"
subtitle: "typescript, hooks와 함께 디테일한 react-router 다루기!"
author: "yoogomja"
header-style: text
tags:
    - nodejs
    - react
    - react-router
    - typescript
    - react-hooks
    - hooks
---

이번에 별도로 진행하는 프로젝트에서 `react`와 `react-router`를 사용하고 있다. 실제로 개발을 하다보니 이해가 부족했다 싶은 부분이 있었는데, RESTful 하게 주소를 디자인 하고 404 페이지를 만드는 부분이었다. 그래서 헷갈렸던 부분을 해결하고 정리해본다. (이 포스트는 `typescript`와 `react-router v5`를 기준으로한다.)

### Not Found 페이지 만들기

Not found 페이지를 추가 하는 걸 먼저 진행했는데, 생각보다 염두에 두어야 하는 구조가 있었다.

```javascript
// index.tsx
// ...
ReactDOM.render(
    <React.StrictMode>
        <Router>
            <SideMenu />
            <Switch>
                <Route exact path='/' component={MainScene} />
                <Route path='/users' component={UserListScene} />
                <Route path='*' component={NotFoundScene} />
            </Switch>
        </Router>
    </React.StrictMode>,
    document.getElementById("root")
);
```

위 코드는 지금 프로젝트의 초안이었던 라우터 구성이다. `SideMenu` 컴포넌트는 `Link` 컴포넌트들을 포함하는 `navbar`역할의 컴포넌트이고 나머지는 실제 페이지에 해당하는 부분이다. 먼저 여기서 내가 놓쳤던 부분은 다음과 같다.

#### 1. 라우터의 우선순위

라우팅을 할 때, `Route` 컴포넌트의 자식들을 위에서부터 훑어 내려오게 된다. 가장 먼저 불러오는 것은 `<Route exact path='/' component={MainScene} />` 이 항목이고 그 다음부터 순차적으로 match되는 항목을 불러오는 식. 그렇기 떄문에 404 페이지를 만든다면 `<Route path="*"/>`와 같이 `path="*"` props를 가진 컴포넌트를 가장 맨 아래에 두면된다. 

이렇게 하면 위에 선언된 `Route`항목들을 모두 조회하고도 결과가 없다면 나머지 모든(\*) 경로에 대하여 특정 컴포넌트(404 페이지)를 출력하게 하는 것. 여기까진 쉬웠다.

#### 2. Switch 컴포넌트

위의 방법으로 진행한다면 왠지 최하단에 404 페이지에 대한 컴포넌트가 모든 경로에서 출력된다. 이 이런 경우는 보통 `exact` 키워드로 정확하게 입력된 주소와 매치되는 경로에만 출력되도록 설정하는데, 404 페이지 같은 경우는 경로를 커버하도록 하기때문에 `exact`키워드는 쓸 수가 없다. 그래서 `Switch` 컴포넌트를 써준다. `Route` 컴포넌트 하위에 본 컴포넌트를 넣어두면, `Switch` 컴포넌트의 하위에 있는 `Route` 컴포넌트들 중 하나씩만 출력되게 된다. 원래는 매칭되는 항목이 있으면 한페이지에서 여러 컴포넌트가 출력될 수도 있지만, 그것을 방지 하도록 사용하는 것. 한 줄만에 전혀 다른 결과를 출력하게 된다.

### 중첩된 Route 만들기

404 페이지를 만든 후에는 특정 주소 형태를 만들 필요가 있었다. 유저 관련 기능을 만들면서 생긴 요구사항이다. `/users/`에서는 유저의 목록을 출력하고 `/users/:user_name`형태의 주소에서는 해당 `user_name`을 가진 사용자의 상세 정보를 보여주는 것.

그러기 위해서 `Route` 컴포넌트를 중첩하기로 했다. 소스는 아래와 같다.

```javascript
// ...
ReactDOM.render(
    <React.StrictMode>
        <Router>
            <SideMenu />
            <Switch>
                <Route exact path='/' component={MainScene} />
                <Route path='/register' component={RegisterUserScene} />
                <Route path='/users'>
                    <Switch>
                        {/* 목록 페이지 */}
                        <Route exact path='/users/' component={UserListScene} />
                        {/* 유저 404 페이지 */}
                        <Route
                            exact
                            path='/users/404-not-found'
                            component={UserNotFoundScene}
                        />
                        {/* 유저별 상세 페이지  */}
                        <Route
                            path='/users/:user_name'
                            component={UserDetailScene}
                        />
                    </Switch>
                </Route>
                <Route path='*' component={NotFoundScene} />
            </Switch>
        </Router>
    </React.StrictMode>,
    document.getElementById("root")
);
```

#### Route 중첩하기

계획된 형태로 출력하기 위해서 `Route`를 중첩하기로 했다. `/users` 경로를 설정해둔 부분이 그것인데, 여기서 부모 `Route`에서 `/users`라고 굳이 path를 써두고 안쪽에 똑같은 경로로 `Route`를 작성한게 의아할 수 있다. 직접 작성해보니 부모 `Route`에서 우선적으로 path를 작성해두지 않으면, 본 `Route`보다 하단에 있는 내용들이 출력되지 않게 되는 문제가 있었다. 아마도 path가 없는 `Route`는 자동으로 `path="\*"` 속성을 갖는게 아닐까 싶다.

그리고 `<Route exact path='/users/' component={UserListScene} />`이 항목은 `/users` 라는 주소에 정확하게 접근했을 때, List 관련 컴포넌트를 줄력하기 위해 한번 더 작성해 주었는데, `exact` 속성을 제외하면 모든 페이지에서 list 관련 컴포넌트가 출력되게 된다. 반드시 추가해 줄 것. `Switch` 컴포넌트도 마찬가지.

#### 하위 404 페이지 만들기

등록된 유저에 한해서만 페이지를 보여주어야 하는데, 특정 이름은 존재하지 않는 경우가 있다. 아마 대다수가 없는 이름인 것이 맞다고 봐야할텐데, 이때 실제 디비상에 존재하는지 아닌지 판단을 하기 위해서는 일단 디비에서 접속을 하는 과정이 필요하다. 그러기 위해서 일단은 주소 형태가 맞다면, `UserDetailScene`이라는 컴포넌트에 우선 접근하고 디비에 확인해본 후 없는 사용자라는 출력을 해주어야 한다. 그러기 위해 이번에는 404 페이지에 특정 주소를 주고, `Route` 상 최하단에 두는 것이 아니라 상단에 두었다. 이렇게 해둔 이유는, 만약, `/users/yoogomja`라는 주소로 접근했을 때, `yoogomja`라는 회원이 없다면 주소창에 `/users/yoogomja`라는 형태를 남겨두면 안되는 것 아닌가 하는 생각 때문이다. 그렇기에 사용자가 존재하지 않으면, `/users/404-not-found`라는 주소로 redirect시키게 된다. 이 코드는 컴포넌트 안에서 직접 작성 해주어야 한다.

이 기능을 구현하기 위해서는 `/users/:user_name`로 작성된 `Route`의 파라미터인 `user_name`을 가져오는 것 부터 시작해야된다. 그 과정은 아래와 같다.

### Route의 Parameter를 hooks와 함께 사용하기

우선 위 코드에서 사용된 `UserDetailScene` 컴포넌트의 간략한 내용은 다음과 같다.

```javascript
import React, { useEffect, useState } from "react";
import { useParams, useHistory } from "react-router";
import { CommonStyles } from "../../styles";

const UserDetailScene = () => {
    // 파라미터 가져오기
    const { user_name } = useParams();
    // 라우터의 history 항목 가져오기
    const history = useHistory();

    useEffect(() => {
        // 테스트를 위해 user_name이 yoogomja인 것만 사용자가 존재하는 것으로 판단
        if (user_name === "yoogomja") {
            console.log("matched");
        } else {
            // 사용자가 존재하지 않으면 404 페이지로 redirect한다
            history.push("./404-not-found");
        }
    }, []);

    return (
        <div style={CommonStyles.container}>
            <p style={CommonStyles.text}>detail</p>
        </div>
    );
};

export default UserDetailScene;
```

우선 여기서는 `useHistory`와 `useParams` 훅을 사용한다. `useHistory`는 리다이렉트를 하기 위해서 사용하고, `useParams`는 `Route`에서 path 속성에 `/users/:user_name` 이라고 작성했던 것 중 `user_name` 항목을 재사용하기 위해서 사용한다. `useParams()`를 실행하면 코드에서 처럼 경로에서 봤던 `user_name`이라는 이름의 값을 자동으로 불러오게 된다.

이렇게 저장된 값으로 유저가 존재하는지 서버에서 판단한 후, 존재하지 않는다면, `history.push`함수를 사용해 404페이지로 리다이렉트 시키는 것. 훅은 너무 편해서 좋다

위의 기능들을 작성하면 기본적인 페이지 정도는 무난하게 만들 수 있다. 그 밖에 API에 대한 document는 [링크](https://reacttraining.com/react-router/web/api/Hooks/useparams)에서 확인 할 수 있다.
