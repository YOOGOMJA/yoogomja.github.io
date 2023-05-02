---
layout: post
title: "NextJS에서 typescript와 함께 i18next으로 다국어 지원 적용하기[1]"
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

- [NextJS에서 typescript와 함께 i18next으로 다국어 지원 적용하기[1]](https://yoogomja.github.io/2023/05/02/[nextjs]-i18next)
- [NextJS에서 typescript와 함께 i18next으로 다국어 지원 적용하기[2]](https://yoogomja.github.io/2023/05/02/[nextjs]-i18next-practices)

## NextJS 13+

지난 해 app directory, 서버 컴포넌트 등의 기능을 포함하는 [NextJS 13 버전이 배포](https://nextjs.org/blog/next-13)되었다. 신규로 시작한 외주 프로젝트에서 NextJS를 사용하기로 하여서 가장 최근 버전을 사용하려고 보니 몇가지 부분에서 변경점이 있었다.  
`app Dir` 기능은 페이지와 페이지에 종속적인 컴포넌트 등을 분리할 수 있어 굉장히 유용할 것으로 기대했는데, 신규 프로젝트의 가장 중요한 요소였던 국제화에 있어서 사용례가 조금 다른지라 이번 스펙에서는 제외하기로 하였다.

| 다만, locize에서 게시한 [i18n with Next.js 13 and app directory (an i18next guide)](https://locize.com/blog/next-13-app-dir-i18n/)에서 해결법을 제시하고 있으나, 내 경우에는 적절하게 작동하지 않았다.

## i18next

기존에 국제화를 지원하기 위해서는 [i18next](https://www.i18next.com/)을 주로 활용한다. 국제화 기능 아이디어 자체는 굉장히 심플하다. 모든 UI 상 표기되는 텍스트를 이미 작성해둔 언어별 문자열로 대치하는 것. 이 과정에서 많이 놓치게되는 부분은 예외 처리인데, `alert`등의 함수로 출력하는 모든 내용 또한 언어별로 작성해두어야 한다. 그 말인 즉슨 대다수의 오류까지도 모두 파악해두어 메시지를 다국어로 작성해두어야 한다는 것이다. 모든 텍스트를 모아두어야 하는 만큼 관리가 주요 관건이다.

오히려 구현은 아주 심플하다. 현재 언어를 선택해두고, 특정 함수(`t`)를 활용하여 원하는 문자열을 자원에서 불러오도록 한다. 실제 공식 홈페이지의 `i18next` 구현의 코드는 아래와 같다.

```js
await i18next.init({
  lng: "en", // if you're using a language detector, do not define the lng option
  debug: true,
  resources: {
    en: {
      translation: {
        key: "hello world",
      },
    },
  },
});
// initialized and ready to go!
document.getElementById("output").innerHTML = i18next.t("key");
```

예제의 `resources`는 `lazy loading`을 지원하기도 하는데 이를 통해 목적에 따라서, 그리고 언어에 따라서 각각 다른 파일로 나누어 저장해두고 불러와 사용할 수 있다. 보통 json 포맷을 활용하는 것으로 보인다.

## NextJS에서는? next-i18next!

`NextJS`에서는 SSR등의 요구사항이 있는만큼 [next-i18next](https://github.com/i18next/next-i18next)를 사용하기도 한다. 굉장히 간편한 설정으로 `i18next`의 기능을 그대로 활용할 수 있다. 확실히 간편했던 점은, 문자열 파일들을 static 파일로서 가지고 있을 수 있다는 점이었는데 구조 또한 굉장히 단순했다. 문서에서 제안하는 구조는 다음과 같았다.

```md
.
└── public
└── locales
    ├── en
    |   └── common.json
    └── de
        └── common.json
```

`json` 파일로 언어 파일을 다루는데, `common.json`이외에 각각 목적에 맞는 다양한 파일들을 만들어둘 수 있다. 도메인 단위가 될 수도 있고, 페이지 단위가 될 수도 있을 것이다.

### Locale

작업하면서 먼저 알아두었던 것은 예제에 있는 `en`, `de`등의 코드였는데, 이는 ISO 표준으로 패키지 내에서 따로 기재해두지는 않는다. 이번 프로젝트에서 필요했던 언어 분류는 중국, 일본, 한국, 미국이었는데 각각의 언어 코드를 알맞게 구분하여 작업하는게 나중에도 용이할 것 같아서 따로 알아두었었다. 이중에 중국어가 조금 특이 했는데 `zh` 코드를 사용하나 간체, 번체 혹은 지역에 따라서 다른 코드를 사용하고는 했다. 금번에는 간체만 지원하기로 하여 `zh`로만 표기하기로 하였다.

## i18next + TypeScript

`next-18next`는 이름이 웃긴 것과 별개로 굉장히 사용이 편리했는데, 일련의 설정 후, 컴포넌트에서 `useTranslation`이라는 훅을 활용하는 것만으로 단어를 가져오는 함수를 얻을 수 있었다. 파일을 여러개로 나누어도 크게 다르지는 않은데 주로 아래와 같은 형태를 띈다.

```tsx
import { useTranslation } from "next-i18next";

export const Footer = ()=>{
  const {t} = useTranslation(['common', 'footer']);

  return (
    <footer>
      <h5>{t("common:meta.product-name")}<h5>
      <p>{t("footer:label.description")}</p>
    </footer>
  );
}

```

`:` 구분자를 활용해서 NameSpace를 구분하고 필요한 항목을 불러와 활용하게 된다. 다만, `TypeScript`를 활용하는 입장에서 `t`함수의 인자로 전달해주는 문자열들에 대한 검증과 자동 완성이 이루어지지 않는 부분은 굉장한 불안 요소라고 느낄 수 밖에 없어 `TypeScript` 설정을 참고하여 프로젝트를 세팅하도록 해보았다. 이 부분은 다음 글에서 다루도록 한다.