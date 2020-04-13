---
layout: post
title: "Windows Terminal 사용해보기"
subtitle: 'Windows에서 통합 Terminal 사용하기'
author: "yoogomja"
header-style: text
tags:
  - windows
  - terminal
  - tools
---

집에서 사용하고 있던 PC를 포맷하면서 윈도우 개발환경을 오랜만에 설정 겸 업데이트하게 되었는데, 
아무래도 맥에서 사용하고 있던 iTerm과 다른 터미널 환경이 영 신경쓰이고 불편했었다. 
그래서 윈도우에서도 기본 제공해주는 별도의 터미널이 있는지 확인해보았는데, 작년에 preview 버전으로 
릴리즈된 Windows Terminal이라는 프로그램이 있었다. 

![윈도우 터미널 실행화면](/img/in-post/202004/terminal.png)*윈도우 터미널 실행화면*

작년 즈음 출시되었다는 이 프로그램은 기존에 다른 터미널 프로그램처럼 tab 기능을 지원하는 뿐만 아니라
각 탭마다 다양한 프로그램을 열 수 있도록 해준다. 기본적으로 `powershell`과 `cmd`을 제공하고 있다. 

설치 [링크](https://www.microsoft.com/store/productId/9N0DX20HK701)에서 설치할 수 있다.

![윈도우 터미널 설치](/img/in-post/202004/window_store.png)*윈도우 터미널 설치*

#### git bash 추가

실제로 작업을 할 떄에 나의 경우에는 대부분 `bash`를 사용하는데, 여기서는 새탭을 열어도 bash가
추가되어 있지 않다. 이 때에는 별도로 추가를 해주어야 한다. 

추가하는 방법은 다음과 같다. 

먼저 `+`버튼 우측에 있는 화살표 버튼을 눌러 `settings` 항목을 클릭해준다. 
이렇게하면 `profiles.json`파일이 열리는데, 여기서 `list`에 배열을 하나 추가해주자. 

```javascript 
{
    // ... 
    "profiles" : 
    {
        "defaults":
        {
            
        },
        "list":[
            // ... 
            // powershell / cmd 설정 내용 

            // BASH 설정 내용
            {
                "guid": "{00000000-0000-0000-ba54-000000000002}",
                "acrylicOpacity" : 0.75,
                "closeOnExit" : true,
                "colorScheme" : "Campbell",
                "commandline" : "\"%PROGRAMFILES%\\git\\usr\\bin\\bash.exe\" -i -l",
                "cursorColor" : "#FFFFFF",
                "cursorShape" : "bar",
                "fontFace" : "Consolas",
                "fontSize" : 10,
                "historySize" : 9001,
                "icon" : "ms-appx:///ProfileIcons/{0caa0dad-35be-5f56-a8ff-afceeeaa6101}.png",
                "name" : "Bash",
                "padding" : "0, 0, 0, 0",
                "snapOnInput" : true,
                // BASH가 실행될 때, 기본 위치
                "startingDirectory" : "d:\\04_DEV\\",
                "useAcrylic" : true
            }
        ]
    }
```

위에서 `bash 설정 내용`을 통째로 추가해주면 된다. 다만 저 항목중에 `startingDirectory`항목은 실제 내가
사용하면서 기본 위치를 변경하기 위해 설정해준 항목이므로, 원하는 위치가 있다면 저 곳에서 변경하면 된다. 

그리고 사용하면서 처음 terminal을 실행했을 때 출력되는 항목이 현재는 `powershell`로 되어있을 것이다. 
이것을 변경하고 싶다면, `defaultProfile`항목에 써있는 `guid`를 `bash`의 `guid`를 입력해주면 된다. 

생각보다 쓰면서 쾌적해서 좋은데, 사용한지 얼마 안되서 당분간은 계속 이것저것 테스트 해봐야겠디.



