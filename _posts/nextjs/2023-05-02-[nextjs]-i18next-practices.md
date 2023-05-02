---
layout: post
title: "NextJS에서 typescript와 함께 i18next으로 다국어 지원 적용하기[2]"
subtitle: "Vercel로 배포한 NextJS 13버전과 typescript, i18next으로 다국어 지원 실패와 성공"
author: "yoogomja"
header-style: text
tags:
  - nextjs
  - react
  - i18n
  - vercel
  - i18next
---

| 이 글은 [next-i18next](https://github.com/i18next/next-i18next),[next-i18next/examples/simple](https://github.com/i18next/next-i18next/tree/master/examples/simple) 문서를 참고하여 작성되었습니다.

## 기본 설정하기

```sh
yarn add next-i18next react-i18next i18next
```

설치는 Installation의 내용을 따라 위의 항목들을 모두 설치한다. 실제 언어 파일들은 이전의 글에서 언급하였듯이 다음과 같은 구조를 띈다.

```md
.
└── public
└── locales
├── en
| └── common.json
└── de
└── common.json
```

이를 사용하기 위해서 `next.config.js`에 `i18n` 내용을 추가하여 하는데, 이는 `next-i18next.config.js` 라는 파일에 따로 분리해둔다. 위 파일은 루트 경로에 배치해두었다.

```js
module.exports = {
  i18n: {
    // 기본 언어
    defaultLocale: "en",
    // 지원 언어
    locales: ["en", "de"],
  },
};
```

여기서 언어 코드가 많이 활용되는 만큼 언어코드를 알아둘 필요가 있었다. 이후, `next.config.js`에는 다음과 같이 위 내용을 불러와 추가해준다.

```js
const { i18n } = require(".next-i18next.config");

module.exports = {
  //...
  i18n,
};
```

다음으로 `_app.tsx`에서 App을 export하는 마지막 줄에 다음과 같은 처리를 해준다. 

```tsx
import { appWithTranslation } from 'next-i18next';

// ... 

export default appWithTranslation(App);
```

이후, 기본적인 사용법은 SSR의 경우, `serverSideTranslations`함수를 활용하고, 컴포넌트 단위로는 `useTranslation`훅을 활용하여 언어를 불러오는 식이다.

## TypeScript와 세팅하기

`TypeScript` 활용을 위해서는 몇 단계 더 나아갈 필요가 있다. 먼저 `tsconfig.json`을 수정해주어야 한다.

```json
{
  "compilerOptions": {
    "resolveJsonModule": true
    // ...
  }
  // ...
}
```

이후, 루트 경로에 `@types`폴더를 생성하여 `i18next.d.ts`를 만들어주고 다음과 같은 내용을 추가해준다.

```ts
/**
 * If you want to enable locale keys typechecking and enhance IDE experience.
 *
 * Requires `resolveJsonModule:true` in your tsconfig.json.
 *
 * @link https://www.i18next.com/overview/typescript
 */
import "i18next";

// 환경과 상황에 따라 다르게 불러오도록 합니다.
import type common from "../public/locales/ko/common.json";
import type auth from '../public/locales/ko/auth.json";


interface I18nNamespaces {
  common: typeof common;
  auth: typeof auth;
}

declare module "i18next" {
  interface CustomTypeOptions {
    // 따로 Namespace를 전달하지 않는 경우 가져오는 기본값입니다.
    defaultNS: "common";
    resources: I18nNamespaces;
  }
}

```

이렇게 하면 IDE에서 타입 체크와 자동 완성까지 지원할 수 있게된다.

### 완성된 주소 구조 

이렇게 작업하여 배포하게되면 로컬 개발 환경 기준으로 `http://localhost:4000/ko`와 같은 형식을 띄게 된다. 가장 첫 라우팅이 `locale`을 표기하게 되는 식이다. FALLBACK language를 설정해두면, 아무것도 입력되지 않는 상태(`http://localhost:4000`)에 자동으로 언어를 선택해준다. 이는 `next-i18next.config.js`파일에서 다음과 같은 줄을 추가하면 된다.

```js
module.exports = {
  // ...
  // 기본 언어 코드
  fallbackLng: 'ko',
```

### Vercel 배포

이후 프로젝트는 Vercel을 통해 배포하도록 하였는데, 이 경우 `locales` 폴더의 경로 설정에 문제가 발생하게 된다고하여 `next-i18next.config.js`에 다음과 같은 내용을 추가했다

```js
module.exports = {
  //...,
  localePath:
    typeof window === "undefined"
      ? require("path").resolve("./public/locales")
      : "/locales",
}
```

## 생각해 볼 만한 것들

이미 좋은 예제가 있는 만큼 적용에는 문제가 없었으나, 몇 가지 우려되는 사항은 존재했다.

### 특정 언어의 파일만을 참고하는 타입

`I18nNamespaces`인터페이스를 선언하는 곳에서도 알 수 있지만 특정 언어의 파일만을 불러와 타입을 생성하게 된다. 그런만큼 파일간의 정합성이 굉장히 중요한데, 이를 수동으로 관리하면 결국 누락은 생길 수 있을 것으로 보인다.

물론 이런 불편함을 이미 잘 알고 있는지, [locize](https://locize.com/)에서 이런 것들을 일괄적으로 적용할 수 있도록 툴을 제공하고 있다. 물론 유로로 제공 하고 있어서 이번에는 활용하지 않았다.

### DRY!

애매한 컴포넌트의 범위를 정하는 것 만큼, 여기서도 범위를 정하는데 굉장히 애로사항이 꽃피는 부분이 있었는데 다음과 같았다.

1. 공통적으로 활용하는 에러 문구의 경우
2. 서버 에러등의 경우
3. Id, Password등 모든 도메인에서 사용하는 단어이면서 동시에 다른 문장에 결합하여 사용하여야 하는 경우

1번의 경우, `common.json`에 instruction이라는 묶음으로 나누어 두면서 어느정도 해결은 되었으나 점점 커질 경우를 대비할 필요가 있다는 생각이 들었다.

2번의 경우는, 서버 에러로 직접 메시지를 표기하지 않고, 통상적인 예처럼 에러 코드를 활용하기 시작했다. 예를 들면 `ERR_MAIL_DUPLICATE` 같은 형식이다. 이 에러코드를 언어 함수에 넘기면 적절한 에러 메시지를 출력하도록 하였다.

3번의 경우는, `common.json`에 `dictionary` 항목을 두어 결합하여 활용하기로 하였는데, 모든 곳에서 활용되는 단어들은 단어만 모여있는 사전 영역을 별개로 선언해두고, `i18next`의 [interpolation](https://www.i18next.com/translation-function/interpolation)을 사용해서 해결했다.

