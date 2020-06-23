---
layout: post
title: "[정원사 프로젝트] 5. 테스트와 리팩토링 하기"
subtitle: "정원사 프로젝트 회고, 사용자 테스트와 개선, 리팩토링하기"
author: "yoogomja"
header-style: text
tags:
    - project
    - study
---

### 프로젝트 회고 글타래 

- [1. 토이 프로젝트 기획과 기술 정하기](https://yoogomja.github.io/2020/06/19/git-farm-project-1/)
- [2. 개발 환경 구성과 백엔드 개발하기(1) : mongoose + github API](https://yoogomja.github.io/2020/06/20/git-farm-project-2/)
- [3. 백엔드 개발하기 (2) : crawling + scheduler](https://yoogomja.github.io/2020/06/20/git-farm-project-3/)
- [4. 프론트 개발하기](https://yoogomja.github.io/2020/06/21/git-farm-project-4/)
- [5. 테스트와 리팩토링하기](https://yoogomja.github.io/2020/06/22/git-farm-project-5/)
- [6. 정리](https://yoogomja.github.io/2020/06/23/git-farm-project-6/)

# 1. 문서화

1차 완성 직후, 백엔드 서버 쪽의 정리를 진행했었다. 프로젝트의 목적과 구조, 설치를 위한 필수 파일과 API에 대한 목록을 정리해두었었다. 이 작업의 목적은 동아리에 정원사 프로젝트를 제공하기 위한 면도 있겠지만, 추후 필요하다면 개인도 다른 단체도 백엔드 코드를 그대로 클론해서 재 사용할 수 있도록 하는 것이 목적이었다. 나 이외의 사람들도 유지보수 혹은 필요한 기능을 추가할 수 있게 하려면 프로젝트에 대한 문서화가 필요했다. 그래서 1차적으로 간단한 내용과 API목록을 문서화 해두었었다. 

다만 이 문서화한 내용들은 추후에 리팩토링을 거치면서 대거 수정된 부분이 있었고, 설치에 선행되어야 하는 부분 또한 추가되었었다. 그렇기에 문서 부분도 대거 수정되어야 했으므로 개발이 완료된 지금에도 꽤 많은 잔 작업들이 남아있는 상황이다. 

현재의 코드는 `react`코드와 백엔드 `express`서버의 코드가 분리되어있으므로 현재 상태로 사용한다면 필요한 사람들은 목적에 따라 화면은 별도로 구성하고 API부분은 그대로 사용할 수 있을 것이라 예상된다. 목적만큼 많은 사람들이 두루 사용하게 될지는 알 수 없지만, 일단 개발을 마친데에 의의를 두기로한다. 

이후로는 API 상세 목록의 문서화가 남아있다. 개인적으로는 `postman`을 이용해서 계속 API의 형태를 확인해왔기 때문에, 보통 대부분의 API의 형태를 기억하고 있지만, 정확히 어떤 필드들이 넘어오는지는 아직 나도 전부 기억하고 있는 것이 아니기 때문에 응답 내용과 파라미터들을 정리해둘 필요가 있었다. 다만 이런 항목은 기존에 대부분의 API 응답을 `interface`로 미리 선언해두었기 때문에, 큰 참조가 되어 금방 작성할 수 있을 것이라고 생각한다. 

# 2. 정리하며 

토이 프로젝트로 가볍게 시작한 일이었지만, 1차 리팩토링 이후에도 좋은 피드백들이 더러 나와서 이후로도 업데이트를 이어갈 수 있을 것으로 보인다. 꽤 가벼운 마음으로 시작했으나, 리팩토링 이후에도 `react`와 `nodejs`, `express`, `sass`같은 기술들에 대해서 더 알아보고 보완할 부분을 많이 발견하게 되어 이후 작업에 큰 도움이 될 수 있을 것이라 생각한다. 이 프로젝트는 이후에 `gcp`의 `compute engine`으로 업로드 해두었고, 당분간 계속 유지될 계획이다. 

[완성 프로젝트 링크](http://34.64.243.31/)
[백엔드 소스 깃허브](https://github.com/YOOGOMJA/github_garden_mern)
[프론트엔드 소스 깃허브](https://github.com/YOOGOMJA/github_garden_mern_client)
[프론트엔드 소스 v2 깃허브](https://github.com/YOOGOMJA/github_garden_mern_client.v2)