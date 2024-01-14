---
title: Beelink 미니PC(MINI S N5095) 셋업 정리, 요약
author: yurijeno
date: 2023-02-12 11:30:00 +0900
categories: [Computer, MiniPC]
tags: [Computer, Windows, MiniPC, BIOS, Autologin, Beelink, Mobile]
# toc: false
# pin: false
# math: false
# mermaid: false
img_path: /assets/img/post/2023-02-12-Beelink 미니PC 셋업 - 정리/
image:
  path: Beelinik-MINI-S.jpg
  width: 400
  height: 250
  alt: Image of Beelink MINI S
---

미니PC(Beelink MINI S N5059) 제품을 한글 윈도우에서 젠트레이더 오토스타터를 자동 실행하도록 셋업하는 과정을 정리한 내용 요약. 시리즈는 다음과 같이 총 4편으로 작성하였음


[1편]({% post_url 2022-12-05-Beelink 미니PC 셋업 - 언어 %})
- 영문 윈도우가 설치된 미니PC에서 한글 언어팩을 설치하고 언어 설정을 완료하였음

[2편]({% post_url 2023-01-08-Beelink 미니PC 셋업 - 자동부팅 %})
- 미니PC BIOS 제공 기능을 설정하여 정해진 시간에 자동부팅이 이루어지도록 함
- 정전 대비하여 전원이 재 공급되면 부팅이 되도록 하였음

[3편]({% post_url 2023-01-29-Beelink 미니PC 셋업 - 윈도우 자동로그인 %})
- BIOS에 의해 컴퓨터 부팅이 자동으로 시작되면 윈도우가 뜰것임
- 윈도우 자동로그인을 설정하여 키보드/마우스 없이도 로그온이 되도록 함

[4편]({% post_url 2023-02-05-Beelink 미니PC 셋업 - 작업스케줄러 %})
- 작업스케줄러를 사용하여 자동 로그온이 되면 젠트레이더 오토스타터를 실행되도록 함
- 또한 정해진 시간(20:00)에 컴퓨터를 자동으로 종료하여 매일 재부팅이 이루어지도록 함

한번 이렇게 미니PC 세팅을 마쳐두면 모니터, 키보드, 마우스 없이 인터넷선만 연결해두면 자기가 알아서 작동하게 된다.

