---
layout: post
title: "countDocuments와 페이징 구현"
subtitle: "mongoose에서 페이징을 구현해보자"
author: "yoogomja"
header-style: text
tags:
    - nodejs
    - express
    - mongodb
    - mongoose
---

이번 사이드 프로젝트에서 `mongodb`와 `mongoose`를 다루고 있는데, 화면을 구성하면서 자연스럽게 페이징 기능을 구현해야하는 요구 사항이 생겼다. 
`mysql`에서야 자주 만들어본 쿼리지만, `mongoose`에서는 다뤄본 적이 없기 때문에 정리해둔다. 

## 1. 모든 아이템의 갯수를 얻기

페이징을 구성하면서 조회할 수 있는 마지막 페이지를 계산하기 위해서는 모든 컬럼(도큐먼트)의 갯수를 알야두어야 한다. 쉽게 갯수를 파악하기 위해 `mongoose`에서 제공하는 함수가 있는데 그 함수가 `countDocuments`함수이다. 사용법은 아래와 같다.

```javascript
// filter 옵션을 파라미터로 받고, 갯수를 number형태로 반환한다. 
// 비동기 요청을 발생시키게 된다. 
// __model은 mongoose 모델이라고 가정
__model.countDocuments(
    {
        // find 조건 
    }
)

// 실제 사용
// Models.Event는 mongoose 모델이라고 가정 
const cnt = await Models.Event.countDocuments({ actor: currentUser._id });

```

위와 같이 `countDocuments`함수를 사용하면, 현재 조건에 맞는 컬럼(도큐먼트)의 갯수를 빠르게 가져올 수 있다.

## 2. 페이징 

`mongoose`에서 조회하는 방법은 여러가지가 있다. `find`와 `aggregate`가 그것인데, 페이징을 구현할 때 키워드는 동일하지만 구현하는 방법에 조금 차이가 있다. 

### 2.1. find와 페이징 

`find`함수는 기본적으로 `promise`형태의 `mongoose`객체를 반환한다. 그렇기 때문에 다양한 함수를 연속으로 실행해 원하는 결과를 얻을 수 있는데, 여기서 페이징을 구현하는 함수는 `skip`함수와 `limit`함수이다. 다음 예제를 보자

```javascript
// ITEMS_PER_PAGE : 10
// CURRENT_PAGE : 1 보다 큰 자연수

__model.find()
        .skip(ITEMS_PER_PAGE * (CURRENT_PAGE - 1))
        .limit(ITEMS_PER_PAGE)
        .sort( { 'field1' : -1 } )  // field1을 desc 순서로 조회
        .exec(function(err, data){  
            // 콜백 함수 
            // 내용이 없어도 상관없습니다
        });
```

기존의 `Promise` 방식을 따르면 위와 같이 구현할 수 있다. `CURRENT_PAGE`에서 1을 뺀 이유는 페이지는 1부터 시작하나, 1을 그대로 페이지 값에 넣을 경우 1페이지에서도 `ITEMS_PER_PAGE`만큼의 결과를 건너뛰게 되기 때문이다. 페이지에서 정렬이 필요한 경우에는 위와 같이 `sort`함수와 같이 사용하면 된다. `promise`가 아닌 `async/await` 형태로 사용하는 경우도 크게 다르지 않다.

```javascript
const result = await __model.find()
                            .skip(ITEMS_PER_PAGE * (CURRENT_PAGE - 1))
                            .limit(ITEMS_PER_PAGE)
                            .sort( { 'field1' : -1 } )  // field1을 desc 순서로 조회
                            .exec();

```

기본적으로 모두 `Promise`형태의 객체를 반환하기 떄문에, `await`키워드를 붙여주기만 하면 문제 없이 작동시킬 수 있다. 


### 2.2. aggregate와 페이징 

`$group`등을 사용하기 위해 `aggregate`함수를 사용하기도 하는데, 여기서 사용하는 방법도 위와 거의 유사하다. 방법은 아래와 같다.

```javascript
const result = await __model.aggregate([
    // filter 부분
    { $match : { field_1 : some_value_you_want } },
    // 정렬
    { $sort : { create_at : -1 } },
    // 페이징 관련 
    { $skip : (ITEMS_PER_PAGE * (CURRENT_PAGE - 1)) },
    { $limit : ITEMS_PER_PAGE }
]);

```

`aggregate`에서는 `$skip`과 `$limit`키워드를 사용하게 된다. `find`와 다른 점은 사용순서에 있다. 실제 사용해보니 `find`와는 다르게 `aggregate`는 조건의 배치 순서에 따라서 결과가 달라지곤 한다. 일반 SQL을 작성할 때와 비슷하게 생각하면 될 것 같다. 그렇기 때문에, 페이징 보다 `$sort`를 먼저 위치 시켜야 정렬에 오류 없이 결과를 조회할 수 있다. 