---
layout: post
title: "@emotion + nextjs + typescript에서 Theme 타입 선언 하기"
subtitle: "emotion.d.ts를 활용한 Theme 선언"
author: "yoogomja"
header-style: text
tags:
  - javascript
  - typescript
  - emotion
  - nextjs
  - nx
---

> [Emotion Typescript](https://emotion.sh/docs/typescript)문서를 참고하여 작성되었습니다.

`emotion`을 활용하면서 타입스크립트를 활용하다보면 `ThemeProvider`를 사용하게 되는데 이 때, `Theme` 객체는 `emotion`에서 제공하는 일반 타입을 따라가게 된다.

이렇게 되면 테마에서 제공하고자 하는 요소를 타이핑할 수 없게되는데, 이를 override하기 위해서 `emotion.d.ts`파일을 활용하게 된다.

## 생성하기

```ts
import "@emotion/react";
import { default as CustomTheme } from "<YOUR_TYPE_PATH>";

declare module "@emotion/react" {
  // eslint-disable-next-line @typescript-eslint/no-empty-interface
  export interface Theme extends CustomTheme {}
}
```

위의 형태의 코드를 작성하여 루트 폴더 혹은 `@types` 폴더를 생성하여 추가해두면 `Theme`를 덮어쓰게 된다.
이 경우 조금 특이한 점이라면, eslint의 no-empty-interface 옵션을 활성화 해둔다는 점이다.

`Theme`자체가 `emotion` 안에서 `interface`로 생성되어 있어서 그런지, `type`으로 선언하게 되면 덮어쓰지 않는 문제가 있었다.
이 때문에, 외부 파일에서 따로 선언해둔 타입을 확장하여 덮어쓰도록 하려면 interface가 비어있게 된다. 이 부분은 lint에서 오류 혹은 경고 표기 하기 떄문에 옵션을 표기해두는 편이 좋다.

이렇게 선언해두고나면, `ThemeProvider` 하위에서 참조하게되는 `emotion` 컴포넌트들로부터 해당 테마 타입을 넘겨받을 수 있게된다.

### with nx

`nx`를 사용한다면, UI 라이브러리 등을 구성하게 되는 경우가 있다. 이 경우에는 해당 라이브러리 경로에도 `emotion.d.ts`를 추가해주고, 각각 app의 루트 경로에도 `emotion.d.ts`를 추가해주어야 한다. 한쪽에만 추가해주는 경우에는 앱 등지에서 사용할 수 없게된다.

놀랍게도 빌드 레벨에서부터 오류가 발생하니 반드시 확인해두는게 좋을 것 같다.
