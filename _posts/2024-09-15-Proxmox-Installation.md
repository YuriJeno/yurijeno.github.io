---
title: Type-1 Hypervisor Proxmox 설치하기
author: yurijeno
date: 2024-09-15 20:00:00 +0900
categories: [Computer, Hypervisor]
tags: [Computer, MiniPC, Proxmox, Hypervisor, VM, Virtualization, Server]
# toc: false
# pin: true
# math: false
# mermaid: false
img_path: /assets/img/post/2024-09-15-Proxmox-Installation/
image:
  path: proxmox_Logo.png
  alt: Proxmox logo
#  width: 400
#  height: 250

---

## Overview

Type-1 Hypervisor인 Proxmox를 설치하는 단계 정리한 내용
- Beelink 등등의 MiniPC 활용하는 용도로 Proxmox를 설치한 내용

## Background

### 설치 이유
2024년 4월 중국산 MiniPC안에 악성코드가 들어있다는 내용을 접하면서 Proxmox 설치함
- 그냥 윈도우 초기화를 하면 그 악성코드는 그대로 남아있음
- 아예 윈도우 복구 파티션까지 다 날리면서 하드초기화를 하는게 정답인데,
- Proxmox 성능 평이 좋아서 아예 뒤집어 엎는겸 Hypervisor를 설치하기로 결정함

### Proxmox 의 장점 : 내장 원격제어
- Windows OS에서 증권사 API나 MTS를 설치하면 꼭 보안프로그램~~악성코드~~가 같이 설치되며 원격제어가 끊긴다
  - 몇몇 보안 프로그램은 Windows RDP 프로토콜을 차단한다
  - 나 같은 경우 미니PC에 원격제어 활성화하고 모니터, 키보드, 마우스를 떼어놓는데,
  - 꼭 이렇게 RDP가 끊기면 다시 온갖 부품 연결하고 노가다를 또 해야한다
- 반면 Proxmox GuestVM에서 이런일이 터지면, Proxmox 제어 콘솔로 들어가버리면 그만이다
  - Windows RDP 말고도 Proxmox에서 제공하는 원격제어가 가능!


### Type-1 Hypervisor 간략소개

윈도우 운영체제 위에서 설치하는 VMWare나 Virtualbox는 Type-2 Hypervisor 이다. 가장 쉽고 흔하게 접할 수 있는 형태지만, Hardware - HostOS - Hypervisor - GuestOS 형태로 실제 Hardware와 GuestOS 사이에 몇겹이나 거치게되어서 성능 하락이 크다는 단점이 있다.

Type-1은 VMWare ESX-i와 비슷한 형태인데, Hardware 위에 바로 OS 처럼 설치된다. 그래서 Hardware - Hypervisor - GuestOS 형태로, 성능하락이 Type-2 보다는 덜하다.

Type-1의 단점을 굳이 적자면, 기존 OS인 윈도우를 날려버려야 한다는 것이 있겠음.

### 기존 기기(미니PC) 드라이버 등 백업하기

Proxmox를 미니PC 같은데 설치한다면 미니PC를 아예 날려버리기 전에 안에있는 드라이버를 백업해두는게 좋음. 제조사 드라이버는 인터넷에서 구하기 어려울 수 있다. 그 자세한 방법은 [유튜브 링크](https://www.youtube.com/watch?v=yyN8eE-kN0A)를 참조하면 되겠음

Proxmox가 잘 돌아간다면 이 드라이버를 쓸 일이 없지만, 만약 호스트 윈도우로 복귀한다면 필요하다.


## Installation Steps

다음 설명글은 Beelink 등 제조사에서 나온 MiniPC에 Proxmox를 설치하는 기준
- Proxmox Server에 접속할때는 그냥 내부 Private IP Address를 사용
{: .prompt-info }



### Prepare Bootable USB

[Proxmox Virtual Environment](https://www.proxmox.com/en/downloads) ISO 이미지를 다운로드 하고 부팅USB를 준비한다. [Rufus](https://rufus.ie/)를 사용해도 좋다. 자세한 방법은 이 글에서는 생략
- 하나만 남기자면, `DD Image Mode`를 써야한다

### Boot with Proxmox Installer

첫 화면에서는 단순하게 "Install Proxmox VE (Graphical)" 을 선택하고 넘어가면 된다

![step-1-entry-menu](step-1-entry-menu.png)

그러면 다음 화면이 나오는데 조금만 기다리면 바로 라이센스 묻는 창이 다시 뜬다

![step-1-before-eula](step-1-before-eula.png)


### Accept EULA

엔드 유저 라이센스를 동의하고 넘어가면 된다

![step-2-accept-eula](step-2-accept-eula.png)


### Select Target Storage

Proxmox를 설치할 저장소를 선택하면 된다. 미니PC라면 보통 SSD 저장소 하나만 있어서 고민할 필요가 없음. 그냥 next 누르면 된다.

![step-3-select-storage](step-3-select-storage.png)

> 위 사진에서 저장소 이름이 VMWare Virtual로 나오는데,
> - 이건 임시로 스크린샷을 캡처하느라 저렇게 뜬거고,
> - 실제로는 SSD 모델명이 정상적으로 출력된다.
{: .prompt-info }



### Location and Timezone

위치와 시간대를 설정한다

![step-4-after-input](step-4-after-input.png)

### Password and Email Address

원격 웹 콘솔 로그인에 사용할 패스워드를 입력하고 이메일 주소를 입력한다
- 패스워드는 계속 사용되고, 이메일 주소는 내 체감상 잘 안쓰였음

![step-5-password-and-email](step-5-password-and-email.png)

### Network Configuration


여기가 제일 중요하다. 네트워크 설정

![step-6-network-config](step-6-network-config.png)

미니PC 모델에 LAN 포트가 2개면 첫번째에 선택하는 인터페이스가 2개로 뜰텐데 잘 골라야한다.
- 공유기와 LAN 선을 연결할때 그 포트로 연결해야만 Proxmox 접속이 가능하다!
- 잘못 골랐으면 다른 포트로 랜선 옮겨 연결하면 된다
- WIFI 인터페이스(wlan)는 불안정하니 권장하지 않음

IP주소는 미니PC가 공유기 사설 네트워크 아래서 사용할 IP 주소 연결
- 쉽게 말하면 메인PC에서 Proxmox로 접속할 IP를 지정하면 됨

Gateway는 보통 공유기 주소를 넣으면 되고, DNS Server는 진리의 구글 DNS 서버인 8.8.8.8을 넣어주면 된다

> 공유기 DDNS 설정 사용해서 미니PC가 여기 입력한 IP를 받도록 고정시키면 더욱 안정적
{: .prompt-tip }

> 위 설정의 네트워크 구성도 기준:
> - 메인PC가 하나있고 미니PC에 Proxmox를 올려서 봇용으로 돌릴텐데
> - 메인PC와 미니PC가 같은 공유기에 묶어 있고, 메인PC에서 미니PC 제어하는 구조
{: .prompt-info }

### Summary

최종 요약을 확인하고 설치를 눌러준다

![step-7-check-summary](step-7-check-summary.png)

그러면 미니피시에 Proxmox가 설치되고 재부팅될텐데, 이 미니PC를 공유기와 랜선으로 연결해주고 IP주소:8006로 접속 테스트해보면 끝
- 임의 인증서를 써서 인증서 에러가 날텐데, 그냥 무시하고 접속하면 됨


![step-8-proxmox-connect-check](step-8-proxmox-connect-check.png)


## Proxmox Initial Configuration

Proxmox 설치 후에는 필수세팅을 마쳐두는게 좋은데 그건 다음 글에 정리