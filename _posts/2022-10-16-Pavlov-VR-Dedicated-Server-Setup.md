---
title: Pavlov VR Dedicated Server Setup (파블로브 VR 전용서버 설정하기)
author: yurijeno
date: 2022-10-16 16:15:00 +0900
categories: [Server, Game]
tags: [Linux, Server, Pavlov VR]
# toc: false
# pin: false
# math: false
# mermaid: false
# img_path: /img/path/
image:
  path: /assets/img/post/2022-10-16-Pavlov-VR-Dedicated-Server-Setup/header.jpg
  width: 800
  height: 500
  alt: Game Banner
---


## Summary

- [Pavlov VR](https://store.steampowered.com/app/555160/Pavlov_VR/) 게임의 전용서버를 리눅스서버에 셋업하는 과정 정리

- ~~공식 위키문서~~에 필요한 내용이 다 있지만, 설치과정순이 아닌 기능순으로 서술되어있어 설치과정 위주로 따로 정리함
  - 수정, 문서링크는 http://wiki.pavlov-vr.com/index.php?title=Dedicated_server 였는데 현재는 죽어있는 상황 (2023-11-05 기준)

## Intro

Pavlov VR은 카운터 스트라이크와 같은 FPS 게임을 VR 환경으로 만든 게임이고, **주로 멀티플레이 경쟁전**으로 플레이된다.

멀티플레이 FPS는 당연히 서버 환경과 핑에 영향을 받을수밖에 없다. 그런데 Pavlov에서 기본 제공하는 호스팅 방식으로하면 게임 환경이 좋지 않다.
- 내 컴퓨터가 아니라 Pavlov에서 제공하는 공식 서버를 사용하는데,
- Pavlov에서 제공하는 Asia Pacific 서버를 사용하더라도 핑이 200ms가 넘는다
  - FPS는 핑 150ms만 넘어도 게임하다가 짜증이 날수있다.

그러다보니 게임하는 사람들끼리 클라우드서버를 마련하고 그 위에 Pavlov Dedicated Server(전용서버)를 올리게 되었다. 아무래도 VR 게임이다 보니 개발자와 같은 얼리어답터가 많아서 쉽게 된 것 같다. 이 글은 제공받은 Rocky Linux 서버에 파블로브 전용서버를 올리는 방법이다.

> 서버란 단어가 여러의미로 되는데, 정리하면
> - '전용서버'는 파블로브 게임에서의 서버를 제공하는 프로그램이고,
> - '리눅스서버'는 전용서버 프로그램이 실행되는 서버 컴퓨터이다
{: .prompt-tip }

사용자가 직접 파블로브 전용서버를 돌려서 얻는 장점은 다음과 같다.
1. 공식서버보다 더 많은 플레이어 참여 가능
  - 공식서버는 10명까지만, 전용서버는 24명까지 지원됨 (5대5와 12대12 게임은 전혀 다르다)
  - 24명까지 가능하지만 21명이 넘으면 렉이 심해서 보통 20명으로 10대10을 자주 즐김
2. 팀킬러 등 비매너 플레이어 영구 추방(Ban) 가능
  - 공식서버는 일일이 투표를 통한 Kick을 해야함
3. 파블로브에서 제공하는 서버 지원 도구로 인게임에서 맵과 게임모드 변경 가능
  - 공식서버는 일일이 로비 갔다가 다시 게임 시작해야함


## 전용서버 설치

매우 세세하게 잘 정리된 ~~문서~~를 그대로 따라하면 된다. 하드웨어 스펙도 적혀있고, 친절하게 우분투와 로키 리눅스에서 잘 실행된다고도 적혀있다.
- 수정, 문서링크는 http://wiki.pavlov-vr.com/index.php?title=Dedicated_server 였는데 죽어있음... (2023-11-05 기준)

> 유의: 메뉴얼에 적혀있는 설치경로나 계정명은 그냥 그대로 쓰는게 정신건강에 편하다. bash 스크립트 등에 계정명과 설치경로가 하드코딩 되어있다.
{: .prompt-danger }

설치하기전에 서버 저장소 용량은 여유롭게 잡는게 좋다. 파블로브 게임 맵(Map)은 보통 개당 2GB 이상이라 10개만 받아도 20GB 이상 먹는다. (추정상 텍스쳐 등등이 전부 포함되서 그런것같다)

### 전용서버 설치 환경

- Rocky Linux 9
- RAM 4GB 이상
- Storage 25GB 이상

### 설치 과정

#### root 권한으로 실행

다음 명령을 root 권한에서 bash에서 실행. 복사 후 쉘에 붙여넣기 하면 됨

```bash
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/g' /etc/selinux/config
sudo setenforce 0
# 위 두 명령은 SELINUX 보안수준을 해제하는 것
# 마음에 안들긴하지만...이걸 안하면 나중에 골치 아파지니 꼭 할 것
# - 게임서버를 시스템 서비스로 등록해서 실행할때 권한 에러 발생한다!

sudo yum install gdb curl glibc.i686 libstdc++.i686 libstdc++-devel.i686 libstdc++-devel.x86_64 unzip wget -y
wget https://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/l/libcxx-3.8.0-3.el7.x86_64.rpm
sudo rpm -i libcxx-3.8.0-3.el7.x86_64.rpm

sudo adduser -m steam
# 위키에서는 useradd를 쓰라고 되어있는데 난 adduser 사용함
# 우분투계열에서 useradd보다 adduser가 더 깔끔한 개인적인 경험이 있음


# 새로 만든 계정의 비밀번호 설정
sudo passwd steam


# Pavlov에서 사용하는 포트번호를 열어준다
sudo firewall-cmd --zone=public --add-port=7777/tcp
sudo firewall-cmd --zone=public --add-port=7777/udp
sudo firewall-cmd --zone=public --add-port=8177/tcp
sudo firewall-cmd --zone=public --add-port=8177/udp
sudo firewall-cmd --zone=public --add-port=9100/udp
sudo firewall-cmd --zone=public --add-port=9100/tcp
sudo firewall-cmd --list-ports
firewall-cmd --runtime-to-permanent


# root 권한에서 일반 유저인 steam 계정으로 변경한다
su steam
```

#### steam 계정에서 실행

쉘에서 현재 계정이 `steam`인 것을 확인하고 다음 명령을 bash에서 실행. 복사 후 쉘에 붙여넣기 하면 됨

```bash
mkdir ~/Steam && cd ~/Steam && curl -sqL "https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz" | tar zxvf -

~/Steam/steamcmd.sh +force_install_dir /home/steam/pavlovserver +login anonymous +app_update 622970 +exit
# 이건 어떤 버전을 돌린것인지에 따라 다른데 PC 서버 기준
# Oculus Shack 서버를 돌릴거면 위키 페이지 참고

~/Steam/steamcmd.sh +login anonymous +app_update 1007 +quit
mkdir -p ~/.steam/sdk64
cp ~/Steam/steamapps/common/Steamworks\ SDK\ Redist/linux64/steamclient.so ~/.steam/sdk64/steamclient.so
cp ~/Steam/steamapps/common/Steamworks\ SDK\ Redist/linux64/steamclient.so ~/pavlovserver/Pavlov/Binaries/Linux/steamclient.so

chmod +x ~/pavlovserver/PavlovServer.sh

# 서버에서 사용하는 설정 파일 준비
mkdir -p /home/steam/pavlovserver/Pavlov/Saved/Logs
mkdir -p /home/steam/pavlovserver/Pavlov/Saved/Config/LinuxServer
mkdir -p /home/steam/pavlovserver/Pavlov/Saved/maps

touch /home/steam/pavlovserver/Pavlov/Saved/Config/mods.txt
touch /home/steam/pavlovserver/Pavlov/Saved/Config/blacklist.txt
touch /home/steam/pavlovserver/Pavlov/Saved/Config/whitelist.txt
```

그리고 `nano` 도구를 사용해서 `Game.ini`를 생성하고 작성한다
```bash
nano /home/steam/pavlovserver/Pavlov/Saved/Config/LinuxServer/Game.ini
```

`Game.ini`는 다음과 같이 설정한다.
```
[/Script/Pavlov.DedicatedServer]
bEnabled=true
ServerName="My_private_idaho"
MaxPlayers=10     #Set this to 10 for Shack. 24 is the max for PC, setting it higher will not allow players to join.
ApiKey="ABC123FALSEKEYDONTUSEME"
bSecured=true
bCustomServer=true
bVerboseLogging=false
bCompetitive=false #This only works for SND
bWhitelist=false
RefreshListTime=120
LimitedAmmoType=0
TickRate=90
TimeLimit=60
#Password=0000
#BalanceTableURL="vankruptgames/BalancingTable/main"
MapRotation=(MapId="UGC1758245796", GameMode="GUN")
MapRotation=(MapId="datacenter", GameMode="SND")
MapRotation=(MapId="sand", GameMode="DM")
```
- `ApiKey`는 https://pavlov-ms.vankrupt.com/servers/v1/key 서 받으면 된다
- 킬 피드, 라운드 종료 후 스탯 정보를 받고싶다면 `bVerboseLogging`를 `true`로 설정해주자
- 그 외 자세한 설정은 위키페이지 참고
- 다 작성하고 난 뒤에는 Ctrl-O, Enter, Ctrl-X 순으로 누르면 저장하고 `nano` 에디터를 종료한다

여기까지 했으면 **서버 설정이 완료되었다.** `~/pavlovserver/PavlovServer.sh` 를 실행하면 파블로브 전용서버가 실행된다. 그런데 이렇게 실행하면 내가 터미널을 종료하면 전용서버도 따라서 종료된다는 문제가 있다. 그리고 파블로브 서버는 종종 업데이트 되기 때문에 주기적인 업데이트 체크도 해야한다. 따라서 이제 업데이트 자동일정 등록과 전용서버 백그라운드 자동 실행을 설정한다.

## 전용서버 자동업데이트 & 자동실행

### 자동 업데이트 설정

다음 명령을 `steam` 사용자인채로 bash에 실행한다

```bash
cat << 'EOF' > $HOME/pavlov_daily_update_and_restart.sh
#!/bin/bash

USER="steam"
SERVICENAME="pavlovserver.service"
INSTALLDIRNAME="pavlovserver"
USERHOME="/home/$USER"

echo -e "Beginning Pavlov VR update run on $(date)\n\n"

systemctl stop "$SERVICENAME"

sleep 5
sudo -iu "$USER" "$USERHOME/Steam/steamcmd.sh" +login anonymous +force_install_dir "$USERHOME/$INSTALLDIRNAME" +app_update 622970 +exit
sudo -iu "$USER" "$USERHOME/Steam/steamcmd.sh" +login anonymous +app_update 1007 +quit
sudo -iu "$USER" cp "$USERHOME/Steam/steamapps/common/Steamworks SDK Redist/linux64/steamclient.so" "$USERHOME/.steam/sdk64/steamclient.so"
sudo -iu "$USER" cp "$USERHOME/Steam/steamapps/common/Steamworks SDK Redist/linux64/steamclient.so" "$USERHOME/pavlovserver/Pavlov/Binaries/Linux/steamclient.so"

systemctl start "$SERVICENAME"

echo -e "Ending Pavlov VR update run on $(date)\n\n"

EOF

chmod +x $HOME/pavlov_daily_update_and_restart.sh
mkdir $HOME/pavlov_update_logs && touch $HOME/pavlov_update_logs/pavlov_daily_update_and_restart.sh.log
```
- cat 부터 ~ EOF까지는 `pavlov_daily_update_and_restart.sh` 를 작성하는 명령이다. nano로 수동으로 입력해도 된다


그리고 다음 명령을 `root` 권한으로 실행해 crontab 자동일정을 등록한다

```bash
CRONLINE="00 4 * * * /home/steam/pavlov_daily_update_and_restart.sh >> /home/steam/pavlov_update_logs/pavlov_daily_update_and_restart.sh.log 2>&1"
# 한국시각 새벽 2시면 한창 게임할때라 4시부터 실행하도록 함
# 00 4 의 의미는 리눅스 서버 시각 기준으로 4:00 AM 에 실행하라는 것
(sudo crontab -u root -l; echo "$CRONLINE" ) | sudo crontab -u root -
unset CRONLINE
```
- `crontab -l` 을 실행하면 등록된 자동실행 `cron` 작업 항목이 보인다

### 전용서버 백그라운드 자동실행

위에서 `pavlovserver.service`라는 서비스가 등장하는데 이제 이를 만들어서 설정을 완료한다. `root` 권한으로 다음과 같이 `nano` 도구로 서비스 설정 파일 작성한다

```bash
sudo nano /etc/systemd/system/pavlovserver.service
```

`pavlovserver.service` 파일은 아래와 같이 작성한다
```
[Unit]
Description=Pavlov VR dedicated server

[Service]
Type=simple
WorkingDirectory=/home/steam/pavlovserver
ExecStart=/home/steam/pavlovserver/PavlovServer.sh

RestartSec=1
Restart=always
User=steam
Group=steam

[Install]
WantedBy = multi-user.target
```

파일 저장 후 다음 명령을 실행한다
```bash
# 부팅 시 자동으로 실행되도록 등록
systemctl enable pavlovserver.service

# 전용서버 실행
systemctl start pavlovserver.service
```

이제 전용서버 실행이 완료되었다. 게임에서 서버에 접속 가능하다. https://pavlovhorde.com/pcServers/ 에서 서버명을 검색해서 잘 떠있는지 확인할 수 있다.


## 전용서버 제어 도구 Rcon 설정

위 내용까지 다 마쳤으면 파블로브 전용 서버 설정이 완료된것이고 게임 플레이 가능하다. 이제 리눅스서버 터미널을 종료해도 자동으로 매일 새벽 4시에 업데이트 체크 작업을 수행하고 백그라운드로 전용서버가 실행된다.

여기서 더 나아가서 전용서버의 Rcon 기능을 활성화할 수 있다. Rcon은 전용서버 관리자 제어 도구로서 다음 기능을 제공한다
- 접속자 정보 확인
- 맵과 게임모드 변경
- 특정 플레이어 팀 변경
- 플레이어 Kick
- 플레이어 Ban
- 치트 기능
  - 플레이어에게 아이템/캐쉬 등 부여
- 그 외 등등


`steam` 사용자 계정으로 다음과 같이 실행하고 설정파일을 작성한다
```bash
nano /home/steam/pavlovserver/Pavlov/Saved/Config/RconSettings.txt
```

```
Password=ChangeThisPassword
Port=9100
```
- 당연하지만, `ChangeThisPassword` 는 본인이 사용할 패스워드로 수정

그 후 `root` 계정으로 서비스를 중단했다 다시 시작한다
```bash
systemctl stop  pavlovserver.service
systemctl start pavlovserver.service
```

이제 정말로 모든 전용서버 설정 과정을 완료했다

### Rcon 사용법

다음과 같이 https://pavlovhorde.com/pcServers/ 에서 내 서버를 찾은 다음 Connect를 누르고,

![PC Server list in horde.com](/assets/img/post/2022-10-16-Pavlov-VR-Dedicated-Server-Setup/pcServerlist.png)

패스워드 칸에 `ChangeThisPassword` 부분을 입력하면 된다. 그러면 접속되며 웹 인터페이스로 내 서버를 제어할 수 있다

![PC Server Rcon menu](/assets/img/post/2022-10-16-Pavlov-VR-Dedicated-Server-Setup/pavlovRconStatus.png)
- 현재 맵, 게임 모드를 알 수 있고 지금은 플레이어 0명으로 비어있는 것을 알 수 있다.
- 아래에는 Rcon 명령을 실행할 수 있는 버튼들이 있다


## 참고 명령


```bash
cat /etc/os-release # os 정보 확인

sudo systemctl start pavlovserver   # 시작
sudo systemctl restart pavlovserver # 재시작
sudo systemctl stop pavlovserver    # 종료
sudo systemctl status pavlovserver  # 현재상태 확인
sudo systemctl enable pavlovserver  # 부팅 시 시작 등록

sudo journalctl -u pavlovserver     # 서버 로그 전체 확인

# to live-tail the logs
sudo journalctl -u pavlovserver -f  # 서버 로그 하단 확인
```


## 나중에 조사할 만한 내용

- Rcon 기능을 연동한 Discord Bot 구현. https://github.com/makupi/pavlov-bot 참고해서 개발

