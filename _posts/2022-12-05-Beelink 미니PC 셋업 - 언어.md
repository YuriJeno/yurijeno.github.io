---
title: Beelink 미니PC(MINI S N5095) 셋업 (1/4) - 언어
author: yurijeno
date: 2022-12-05 20:30:00 +0900
categories: [Computer, MiniPC]
tags: [Computer, Windows, MiniPC, Beelink]
# toc: false
# pin: false
# math: false
# mermaid: false
img_path: /assets/img/post/2022-12-05-Beelink 미니PC 셋업 - 언어/
image:
  path: Beelinik-MINI-S.jpg
  width: 400
  height: 250
  alt: Image of Beelink MINI S
---

## 배경, 제품정보

젠포트 단톡방에서 물탄찬밥님의 추천으로 할인중이던 MiniPC를 구매하게 되었다. (데스크탑은 100W씩 전기가 나가길래 전기요금을 아끼고 싶었다. 간단한 프로그램 돌릴 계획이다.)

[Beelink](https://www.bee-link.com/) 회사의 모델 중 하나인데 나는 [MINI S N5095](https://www.bee-link.com/catalog/product/index?id=302)를 구매했다
- RAM 8GB에 SSD 256GB로 주문

처음 받은 박스와 까본 모양은 다음과 같다.

![Unboxing-1](Beelink-Unboxing1.jpg)
![Unboxing-2](Beelink-Unboxing2.jpg)

비교할만한 물체를 옆에 두진 않았는데, 왼쪽에 있는 메뉴얼 아래 컴퓨터가 있고 손바닥만한 크기다

부팅은 간단하다. AC 어댑터에 연결하고, 키보드마우스와 모니터에 연결하면 끝. 버튼만 누르면 Beelink 로고가 뜨고 곧 있으면 윈도우 11이 켜지게 된다.


문제는, Beelink가 외국(중국) 업체다 보니 한글윈도우가 아니라 영문윈도우 기본으로 설치되어있다는 점. 그래서 처음에 계정 설정할때 영어로 해야한다. 영어자체는 불편하지 않은데, 언어설정이 한국어가 아니면 국산프로그램(HTS라던가 은행이라던가) 사용할때 걱정이 된다.

## 언어설정 영문 → 한국어

윈도우 키 누르고 language 검색하여서 "Language and Region" 설정에 들어가서 한글 언어팩을 설치해주면 된다.

다음과 같이 들어가서 (원래는 영어로 뜬다. 지금 한글팩 설치한 상태)
![언어설정-1](언어설정-1.png)
1번 누르고 Korean 검색해서 추가해준 다음
2번 누르고 상세 설정 들어가서 언어팩을 다운로드 하면 된다 (원래는 Download 버튼이 뜨는데 지금은 설치된 모습이다...)
![언어설정-2](언어설정-2.png)


그럼 밀린 윈도우 업데이트를 하고 한참을 기다린 후에 언어팩이 다운로드 된다.

그 다음 3번 `Windows Display Language`를 누르고 한국어를 선택해주고, 4번과 5번도 마찬가지로 설정해준다. 지역을 한국어로, 사용지역 언어를 한국어로 바꿔주면 된다.

이래도 한글이 깨져나오는 경우가 있는데 그러면 6번(기본 언어 설정) 들어가서 시스템 로캘을 한국어로 바꾸고 재부팅을 하면 끝!

![언어설정-3](언어설정-3.png)
![언어설정-4](언어설정-4.png)

아니 그런데 하단에 있는 `UTF-8 사용` 저 체크박스는... 윈도우 11오면서 처음 봤다. 나는 적용하지 않았다.

> 추측: 한글윈도우는 EUC-KR과 같은 인코딩을 쓰는데 그걸 UTF-8로 바꿔주는게 아닐까 싶다.
{: .prompt-warning }


## 그 외 추가 설정

- 관리자 권한에서 다음 명령 실행 (최대절전모드 미사용)
	```shell
	> powercfg -h off
	```

- 전원 설정 들어가서 계속 컴퓨터 건드리지 않아도 절전모드랑 디스플레이 종료 없도록 설정

- 개인 정보 및 보안 들어가서 필요없는 권한은 죄다 해제
	- 윈도우 기본 셋업 시 정보 공개 동의 안한 것도 세부설정 들어가면 다시 봐야하는 내용 있음
