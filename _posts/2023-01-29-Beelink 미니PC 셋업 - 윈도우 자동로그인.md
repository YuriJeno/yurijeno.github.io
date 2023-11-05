---
title: Beelink 미니PC(MINI S N5095) 셋업 (3/4) - 윈도우 자동로그인
author: yurijeno
date: 2023-01-29 16:45:00 +0900
categories: [Computer, MiniPC]
tags: [Computer, Windows, MiniPC, AutoLogon, Beelink]
# toc: false
# pin: false
# math: false
# mermaid: false
img_path: /assets/img/post/2023-01-29-Beelink 미니PC 셋업 - 윈도우 자동로그인/
image:
  path: Beelinik-MINI-S.jpg
  width: 400
  height: 250
  # alt: Responsive rendering of Chirpy theme on multiple devices.
---

## 지난 글 정리

[지난글]({% post_url 2023-01-08-Beelink 미니PC 셋업 - 자동부팅 %})에서는 영어 윈도우 미니PC 자동부팅을 설정했다; 전원재연결시와 특정시간대 자동 부팅

원하는 방향은 다음과 같다
- 미니피시는 전원선과 인터넷선만 연결
  - 하고싶다면 인터넷선도 없이 가능하다
  - MINI S N5095 모델은 WiFi가 있음
- **미니피시는 키보드, 마우스, 모니터 없다**
- 자동으로 부팅되고 **자동으로 로그인되고, 자동으로 트레이더 키고 컴퓨터 종료해야함**

이것을 다 자동화하고 싶기 때문에 윈도우 자동로그인을 등록해야한다

## 윈도우 자동로그인 설정하는 3가지 방법

자동로그인 설정하는 여러 방법이 있는데 그중에서 가장 추천하는 방법은 Sysinternals의 [AutoLogon](https://learn.microsoft.com/en-us/sysinternals/downloads/autologon) 도구를 사용하는 것이다.
그 다음으로 netplwiz를 사용한 설정, 마지막에 레지스트리를 사용한 등록방법 순으로 사용하는 것이 좋다.

### Sysinternals - AugoLogon 사용하는 방법

#### 설정법

1. [AutoLogon](https://learn.microsoft.com/en-us/sysinternals/downloads/autologon) 페이지에서 도구를 다운로드 한다
  - 잘 모르겠으면 Autologon64.exe 실행 (요즘 대부분 컴퓨터는 64비트 운영체제)
  - 32비트 운영체제면 Autologon.exe 실행
  - 64비트 ARM 운영체제면 Autologon64a.exe 실행
2. 아주 간단한 창이 하나 뜨는데 여기에 Username과 Password 입력 후 Enable 누르면 설정 완료
  - ![Autologon.png](Autologon.png)

### netplwiz(사용자 계정 설정)을 사용하는 방법

이 방법을 추천하지 않는 이유는... 최근 윈도우의 경우 대부분은 아래 설정법을 따라하기 전에 추가적인 조취를 취해야하기 때문이다. 보통 3.번 과정에서 체크박스가 뜨지 않음. AutoLogon 도구를 쓰는게 간단하다.

#### 설정법

1. Win + R 키를 눌러서 실행창을 띄운다
2. `netplwiz.exe` 혹은 `control userpasswords2` 를 입력하고 실행
3. 그러면 다음과 같은 창이 뜨는데 여기서 "사용자 이름과 암호를 입력해야 이 컴퓨터를 사용할 수 있음" 체크를 해제
  - 사진에서 "Users must enter a user name and password to use this computer" 라고 적힌 부분
    ![netplwiz](netplwiz.png)
4. 확인을 누르고 패스워드 입력창이 뜨면 패스워드 입력

#### 체크박스가 없을 경우

netplwiz에서 위와 같은 체크박스가 보이지 않으면 레지스트리를 수정하거나 로그인 설정에서 Windows Hello를 해제해야한다. 별로 효용성이 높지 않으므로 더 자세히 조사는 하지 않음

### 레지스트리 등록하는 방법

모든 방법 중 **제일 보안적으로 취약**하고 추천하지 않는 방법. 설정 방법이 쉽지도 않음

#### 설정법

1. 레지스트리 편집기 실행(Win + R, regedit.exe 실행)
2. 편집기에서 `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon` 경로 탐색
3. `편집` → `새로 만들기` → `문자열 값` 클릭하고 이름은 `DefaultUserName` 로 설정, 값은 로그인하고자 하는 계정명 입력
  - 이미 `DefaultUserName` 가 있으면 새로 생성하지 않아도 됨
4. 동일한 방법으로 문자열 값 생성하고 이름은 `DefaultPassword` 로 설정, 값은 계정의 비밀번호 입력
  - 이미 `DefaultPassword` 가 있으면 새로 생성하지 않아도 됨
5. 동일한 방법으로 문자열 값 생성하고 이름은 `AutoAdminLogon`로 설정, 값은 1로 입력

#### 위험성

위에서 바로 감이 왔겠지만, `DefaultPassword` 에 내 비밀번호가 그대로 저장된다. 암호화처리나 해쉬처리가 전혀 안된채로 그대로 저장되기 때문에 매우 위험
- 레지스트리 `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`에 접근만 가능하다면 내 계정의 비밀번호가 그대로 털리게 된다.
- 참고로 해당 서브키에 대해서 마우스 우클릭 후 사용권한을 확인하면 나오지만 일반 유저 계정도 읽기 권한이 있는 영역이다
- 실수로 악성코드 하나라도 잘못 실행하면 내 비밀번호가 바로 털릴 수 있게 됨

보다 자세한 사항은 [참고](https://learn.microsoft.com/en-us/troubleshoot/windows-server/user-profiles-and-logon/turn-on-automatic-logon)
- MS 문서에서도 레지스트리를 수정하느니 그냥 AutoLogon 도구를 사용할것을 추천함

AutoLogon 도구의 경우 암호화하여 비밀번호를 저장한다. 다만 악성코드가 관리자 권한까지 취득해버리면 이 경우도 비밀번호가 털릴 수 있으나 레지스트리 방식에 비하면 훨씬 더 안전하다

## 참고문서
- [3 Ways to Auto Logon to Windows without Typing Your Password](https://www.raymond.cc/blog/auto-login-windows-xp-without-typing-password/)