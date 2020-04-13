---
layout: post
title: "Chocolatey 사용해 보기"
subtitle: 'Windows에서도 패키지 매니저를 사용해보자'
author: "yoogomja"
header-style: text
tags:
  - windows
  - packagemanager
  - tools
  - chocolatey
---

전 포스트와 같이, 이번에 오랜만에 윈도우 개발 환경을 세팅해보는 중이다. 아무래도 윈도우를 사용하면서 
가장 큰 불편함이라고 한다면 쉘과 패키지 매니저라고 생각하는데, 쉘은 이번에 알게된 `Window Terminal`로
조금 해소됐다고 생각한다. 다음 스텝은 패키지 매니저였는데, 다행히 윈도우에도 패키지 매니저가 생겼더라.

사용할 프로그램은 `chocolatey`라는 프로그램이다. business edition이나, pro edition에 대한 별도 가격 
정책이 존재하지만, 기본적으로는 무료로 사용할 수 있는 것으로 보인다. 

이 프로그램을 이용해서 node를 설치한다면 아래와 같은 방식이 된다. 

```console
> choco install nodejs -y
```

이렇게 해주면 몇가지 동의를 거쳐 직접 프로그램을 설치해준다. 사용법 자체는 다른 패키지 매니저 들과 다르지 
않다.


### 시스템 요구 사항

`chocolatey`는 다음과 같은 사양을 필요로 한다. 부족한 항목이 있다면 미리 해결해 두자

- Windows 7+ / Windows Server 2003 + 
- PowerShell v2+ (minimum is v3 for install from this website due to [TLS 1.2 requirement](https://chocolatey.org/blog/remove-support-for-old-tls-versions))
- .NET Framework 4+ (the installation will attempt to install .NET 4.0 if you do not have it installed)

### 설치 순서

별도의 msi 설치 파일같은건 제공하지 않고 직접 커맨드를 입력해 설치하도록 알려주고 있다. 많이 어렵지는 않다. 

#### 보안 관련 

설치에 앞서, 이 프로그램은 오픈소스로 운영이 되는 만큼, 보안에 대해서 설치 단계부터 매우 엄격하고 조심스럽게 
언급하고 있다. 그래서 `powershell`의 [Get-ExecutionPolicy](https://go.microsoft.com/fwlink/?LinkID=135170)라는 것을 먼저 고지해주고, 해당 항목을 `Bypass` 혹은 `AllSigned`로 해달라고 이야기 한다. 

`Bypass`는 패키지 설치 시 별도 확인을 거치지 않도록 하고, `AllSigned`시에는 매번 확인하게 하므로 더 보안 측면에 안전하다는 이야기로 보인다. [원문](https://chocolatey.org/install) 

#### 설치

- [x] `powershell` 실행 

우선 `powershell`을 관리자 모드로 실행해주자. 파일 경로에 직접 접근하는 일이 많아서 그런듯 하다. 

- [x] `Get-ExecutionPolicy` 설정

그리고 나서 위에 언급한 `Get-ExecutionPolicy`설정을 해주어야 하는데, 일단 `powershell`을 실행한 후 `Get-ExecutionPolicy`라는 명령어를 입력해주자. 만약 `Restricted`라는 응답이 출력된다면, `Set-ExecutionPolicy AllSigned` 혹은 `Set-ExecutionPolicy Bypass -Scope Process`라고 입력해주자. 나는 `AllSigned`를 실행해줬다.

그리고 아래의 커맨드를 입력해서 실제 다운로드를 실행해주자. 

```console
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

- [x] 설치 확인 

모두 설치가 끝나고 나면, `choco` 혹은 `choco -?`를 입력해 명령어 내용들이 나오며 설치가 됐음을 확인할 수 있다.


#### 주요 명령어

```bash
$ choco list                # 설치된 항목을 확인합니다
$ choco search keyword      # keyword 항목을 검색합니다.
$ choco install keyword     # keyword 항목을 설치합니다.
$ choco uninstall keyword   # keyword 항목을 제거합니다.
```

