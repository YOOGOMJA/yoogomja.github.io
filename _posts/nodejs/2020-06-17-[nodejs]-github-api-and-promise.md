---
layout: post
title: "Express에서 Github API를 사용하기"
subtitle: "Github API를 쉽게 사용하기 위한 패키지와 응용법을 알아보자 "
author: "yoogomja"
header-style: text
tags:
    - nodejs
    - expres
    - github-api
    - promise
---

진행 중인 사이드 프로젝트에서 `github REST api`를 사용중에 있다. 기본적인 조회부터 꽤 다양한 항목들을 수행할 수 있는데 이번 프로젝트에서는 주로 조회에 관련된 기능들을 이용해 작업중에 있다. 

[Github API Document](https://developer.github.com/v3/)

사용하기에 앞서 요청 가능량을 알아둘 필요가 있는데, `github REST API`에서 인증된 요청에 한해서는 한 시간당 5000번의 요청이 가능하고 인증되지 않은 요청에서는 한시간당 60건의 요청이 가능하다.([원문](https://developer.github.com/v3/#rate-limiting)) 정기적으로 정보를 크롤링해오는 서비스 특성상 필요한 요청만 하더라도 인증이 필요하다. 이때 개인의 인증된 요청은 `github` 계정에서 `Personal Access Token`을 생성하고 해당 토큰을 넘겨주면 된다. 

[Personal Access Token 관련 링크 ](https://www.openshift.com/blog/private-git-repositories-part-3-personal-access-tokens) 

# 1. nodejs에서 github REST API 요청하기

`github REST API`의 주소형태는 기본적으로 아래와 같은 형태를 띈다

```sh
curl -i https://api.github.com/users/octocat
HTTP/1.1 200 OK
Date: Mon, 01 Jul 2013 17:27:06 GMT
Status: 200 OK
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 56
X-RateLimit-Reset: 1372700873
```

호스트는 `https://api.github.com`이고, 그 뒤에 필요한 항목들이 위치하게 되는데, 내가 가져오려고 하는 항목들과 주소는 다음과 같다

- 특정 사용자의 이벤트 조회 : `https://api.github.com/users/USER_NAME/events`
- 특정 사용자의 정보 : `https://api.github.com/users/USER_NAME`
- 특정 저장소의 정보 : `https://api.github.com/repos/REPOSITORY_NAME`
- 특정 저장소의 언어 정보 : `https://api.github.com/respos/REPOSITORY_NAME/languages`

대다수의 조회 결과들은 한번에 모든 결과를 반환하지 않고 페이징 기능을 제공하고 있다. 이때 페이지당 아이템 갯수는 조정할 수 없다. (이벤트를 기준으로 페이지 당 10개의 항목을 조회하게 된다.) 그렇기 떄문에, 모든 정보를 조회하기 위해서는 쿼리로 조회할 페이지를 같이 넘겨주어 요청해야한다. 

하지만 이런 복잡한 기능들을 매번 문자열을 조작해 요청하는 것은 조금 번거로운 일이기 때문에, 미리 만들어진 패키지를 이용해 작업하기로 했다.

# 2. octonode 사용하기 

## 1. 기본 사용 법 

`github` 측에서 제공하는 `nodejs`패키지 명은 `octonode`이다. `yarn add octonode` 혹은 `npm install octonode`로 설치할 수 있다. 

[octonode 저장소](https://github.com/pksunkara/octonode)

해당 패키지를 사용하는 예제 코드는 아래와 같다. 

```javascript
import github from 'octonode';

const user_name = "YOOGOMJA"; // 조회할 사용자 이름을 넣어줍니다.
const page = 1;

// 사용자의 이벤트를 조회하기 
github
.client(YOUR_ACCESS_TOKEN) // 토큰이 있는 경우 넘겨줍니다.
.get(
    `/users/${user_name}/events`,   // 요청 주소
    { page : page },                // 페이징 등 요청에 데이터를 추가로 
                                    // 넘겨주어야 하는 경우 입력합니다
    (err,status, body)=>{           // 반환 함수 입니다.
        if(!err){
            // 오류가 없는 경우
        }
        else{
            // 오류가 있는 경우
        }
    }
);
```

`client`라는 함수로 클라이언트 관련 정보를 먼저 초기화 해주고 난 뒤, 요청할 수 있다. 첫번째 파라미터인 요청 주소는 host까지 입력하지 않고 뒤에 요청할 정보에 해당하는 부분만 넘겨주면 된다. 두번째 파라미터는 요청에 필요한 정보를 추가로 넘겨줄 경우 사용하는데 보통은 비워두고, 페이징을 사용하는 경우등에 사용하게 된다. 세번째 파라미터에는 callback함수를 넘겨주게되며 상태와 오류정보, 그리고 조회 결과를 파라미터로 넘겨준다. 다만, callback 함수 형태기 때문에 `Promise`형태로 구현할때에는 조금 불편함이 따르게 된다. 해당함수를 `Promise` 형태로 전환하면 다음과 같다. 


```javascript
import github from 'octonode';

/**
 * @description github API를 이용해 사용자의 이벤트를 불러옵니다
 * @param {string} user_name github API에서 이벤트를 불러올 사용자의 login입니다
 * @param {number} page 데이터를 가져올 페이지를 입력합니다. 기본값은 1입니다.
 * @returns {Promise} 결과를 담은 프로미스 객체를 반환합니다
 */
const fetchEvents = (user_name, page = 1) =>{
    return new Promise((resolve, reject)=>{
        const _APIClient = github.client(config.github.api_token);
        _APIClient.get(
            `/users/${user_name}/events`,{ page : page}, 
            (err,status, body)=>{
            if(!err){
                resolve({ data : body });
            }
            else{
                reject({ error : err , status : status});
            }
        });
    });
}

// 실행 예제
const result = await fetchEvents("YOOGOMJA" , 1);
```

위와 같이 `Promise`로 감싸주면, 대부분의 요청을 `async / await` 키워드와 함께 편하게 사용할 수 있다. 아직 `octonode`측에서 제공하지 않고 있어 조금 불편하지만 일단은 저정도로도 사용하는데에 큰 어려움은 없다. 
