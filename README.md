# Docker로 마인크래프트 설치하기

## 0. 작업 환경

- Cloud 인스턴스: AWS, Azure, GCP, OCI 등
- OS: Ubuntu 20.04, 22.04 등
- Docker 설치
- 작업 디렉토리: /data/minecraft
- 사전 작업 참고 문서: https://github.com/20eung/docker-portainer-watchtower


## 1. Minecraft 설치

```
$ sudo mkdir -p /data/minecraft/mc

$ sudo vi /data/minecraft/mc/docker-compose.yml

$ cat /data/minecraft/mc/docker-compose.yml

```

```
version: '3.7'

services:
  minecraft:
    image: itzg/minecraft-server:latest
    container_name: minecraft
    hostname: mc
    restart: always
    stdin_open: true
    tty: true
    network_mode: "bridge"
    ports:
      - 25565:25565
    environment:
      - TZ:"Asia/Seoul"
      - PUID:1000
      - PGID:1000
      - EULA=TRUE
    volumes:
      - /etc/timezone:/etc/timezone:ro
      - /usr/share/zoneinfo/Asia/Seoul:/etc/localtime:ro
      - ./data:/data
```

***
## 2. 마인크래프트 도커 실행하기

```
$ cd /data/minecraft/mc

$ sudo docker-compose up -d

```

***
## 3. server.properties 파일 예제

- 커스텀 motd 만들기: https://minecraft.tools/en/motd.php

```
#Minecraft server properties
enable-jmx-monitoring=false
level-seed=
rcon.port=25575
enable-command-block=true
gamemode=survival
enable-query=false
generator-settings={}
enforce-secure-profile=true
level-name=world
motd=\u00A7b\u00A7l1st \u00A7e\u00A7l Minecraft Server\!
query.port=25565
pvp=true
texture-pack=
generate-structures=true
max-chained-neighbor-updates=1000000
difficulty=hard
network-compression-threshold=256
max-tick-time=60000
require-resource-pack=false
max-players=3
use-native-transport=true
online-mode=true
enable-status=true
allow-flight=true
broadcast-rcon-to-ops=true
view-distance=10
max-build-height=256
server-ip=
resource-pack-prompt=
cheat-enabled=false
allow-nether=true
server-port=25565
default-player-permission-level=operator
enable-rcon=true
sync-chunk-writes=true
op-permission-level=4
server-name=\u00A7b\u00A7l1st \u00A7e\u00A7l Minecraft Server\!
prevent-proxy-connections=false
hide-online-players=false
resource-pack=
entity-broadcast-range-percentage=64
simulation-distance=10
player-idle-timeout=0
rcon.password=minecraft
force-gamemode=false
rate-limit=0
hardcore=false
white-list=false
broadcast-console-to-ops=true
allow-cheats=true
spawn-npcs=true
previews-chat=false
spawn-animals=true
snooper-enabled=true
function-permission-level=4
level-type=default
text-filtering-config=
spawn-monsters=true
enforce-whitelist=false
resource-pack-sha1=
spawn-protection=0
max-world-size=29999984
```

***
## 4. setop.sh 명령어

특정 유저에게 op 권한을 부여하는 명령어

```
#!/bin/sh
sudo docker exec mc mc-send-to-console op user1
sudo docker exec mc mc-send-to-console op user2
```

***
## 5. whojoin.sh 스크립트

마인크래프트 서버에 유저가 참가(join)하거나 퇴장(left)하면.  
텔레그램으로 알려주는 스크립트

```
#!/usr/bin/bash
HOME='/data/minecraft/mc'
JOINS=`grep -E "joined | left" ${HOME}/data/logs/latest.log | wc -l`
JOINS_OLD=`cat ${HOME}/data/join.dat`

if [ "${JOINS}" == "${JOINS_OLD}" ]; then
  echo "Equal"
else
  echo "Diff"
  echo ${JOINS} > ${HOME}/data/join.dat

  LAST=`grep -E "joined | left" ${HOME}/data/logs/latest.log | tail -1`
  JOIN='['${HOSTNAME}'-mc1] '${LAST}
  echo ${JOIN}
  ${HOME}/telegram-send.sh "${JOIN}"
fi
```

***
## 6. telegram-send.sh 스크립트

```
#!/usr/bin/bash

GROUP_ID='그룹ID 찾는 법은 구글링해서 찾으시길...'
BOT_TOKEN='봇 토큰 찾는 법도 구글링해서 찾으시길...'

DATE="$( date "+%Y-%m-%d %H:%M")"

# this 3 checks (if) are not necessary but should be convenient
if [ "$1" == "-h" ]; then
  echo "Usage: `basename $0` \"text message\""
  exit 0
fi

if [ -z "$1" ]
  then
    echo "Add message text as second arguments"
    exit 0
fi

if [ "$#" -ne 1 ]; then
    echo "You can pass only one argument. For string with spaces put it on quotes"
    exit 0
fi

TXT="$DATE%0A$1"

curl -s \
  --data parse_mode=HTML \
  --data "text=$TXT" \
  --data "chat_id=$GROUP_ID" \
  'https://api.telegram.org/bot'$BOT_TOKEN'/sendMessage' > /dev/null

```

***
## 7. 반복 수행을 하기 위한 crontab 설정

```
$ sudo crontab -e

$ sudo crontab -l

# m h  dom mon dow   command
# │ │  │   │   └── day of week(0-7, 0=7=sunday)
# │ │  │   └── month(1-12)
# │ │  └── dates of month(1-31)
# │ └── hours(0-23)
# └── minutes(0-59)
#
# 매분마다 실행
* * * * * /data/minecraft/mc/whojoin.sh
```


***
## 8. 디렉토리 구조

```
$ tree /data/minecraft/
/data/minecraft/
└── mc
    ├── data
    │   ├── banned-ips.json
    │   ├── banned-players.json
    │   ├── eula.txt
    │   ├── join.dat
    │   ├── libraries
    │   │   ├── com
    │   │   │   ├── github
    │   │   │   │   └── oshi
    │   │   │   │       └── oshi-core
    │   │   │   │           └── 5.8.5
    │   │   │   │               └── oshi-core-5.8.5.jar
    │   │   │   ├── google
    │   │   │   │   ├── code
    │   │   │   │   │   └── gson
    │   │   │   │   │       └── gson
    │   │   │   │   │           └── 2.8.9
    │   │   │   │   │               └── gson-2.8.9.jar
    │   │   │   │   └── guava
    │   │   │   │       ├── failureaccess
    │   │   │   │       │   └── 1.0.1
    │   │   │   │       │       └── failureaccess-1.0.1.jar
    │   │   │   │       └── guava
    │   │   │   │           └── 31.0.1-jre
    │   │   │   │               └── guava-31.0.1-jre.jar
    │   │   │   └── mojang
    │   │   │       ├── authlib
    │   │   │       │   └── 3.11.49
    │   │   │       │       └── authlib-3.11.49.jar
    │   │   │       ├── brigadier
    │   │   │       │   └── 1.0.18
    │   │   │       │       └── brigadier-1.0.18.jar
    │   │   │       ├── datafixerupper
    │   │   │       │   └── 5.0.28
    │   │   │       │       └── datafixerupper-5.0.28.jar
    │   │   │       ├── javabridge
    │   │   │       │   └── 1.2.24
    │   │   │       │       └── javabridge-1.2.24.jar
    │   │   │       └── logging
    │   │   │           └── 1.0.0
    │   │   │               └── logging-1.0.0.jar
    │   │   ├── commons-io
    │   │   │   └── commons-io
    │   │   │       └── 2.11.0
    │   │   │           └── commons-io-2.11.0.jar
    │   │   ├── io
    │   │   │   └── netty
    │   │   │       ├── netty-buffer
    │   │   │       │   └── 4.1.77.Final
    │   │   │       │       └── netty-buffer-4.1.77.Final.jar
    │   │   │       ├── netty-codec
    │   │   │       │   └── 4.1.77.Final
    │   │   │       │       └── netty-codec-4.1.77.Final.jar
    │   │   │       ├── netty-common
    │   │   │       │   └── 4.1.77.Final
    │   │   │       │       └── netty-common-4.1.77.Final.jar
    │   │   │       ├── netty-handler
    │   │   │       │   └── 4.1.77.Final
    │   │   │       │       └── netty-handler-4.1.77.Final.jar
    │   │   │       ├── netty-resolver
    │   │   │       │   └── 4.1.77.Final
    │   │   │       │       └── netty-resolver-4.1.77.Final.jar
    │   │   │       ├── netty-transport
    │   │   │       │   └── 4.1.77.Final
    │   │   │       │       └── netty-transport-4.1.77.Final.jar
    │   │   │       ├── netty-transport-classes-epoll
    │   │   │       │   └── 4.1.77.Final
    │   │   │       │       └── netty-transport-classes-epoll-4.1.77.Final.jar
    │   │   │       ├── netty-transport-native-epoll
    │   │   │       │   └── 4.1.77.Final
    │   │   │       │       ├── netty-transport-native-epoll-4.1.77.Final-linux-aarch_64.jar
    │   │   │       │       └── netty-transport-native-epoll-4.1.77.Final-linux-x86_64.jar
    │   │   │       └── netty-transport-native-unix-common
    │   │   │           └── 4.1.77.Final
    │   │   │               └── netty-transport-native-unix-common-4.1.77.Final.jar
    │   │   ├── it
    │   │   │   └── unimi
    │   │   │       └── dsi
    │   │   │           └── fastutil
    │   │   │               └── 8.5.6
    │   │   │                   └── fastutil-8.5.6.jar
    │   │   ├── net
    │   │   │   ├── java
    │   │   │   │   └── dev
    │   │   │   │       └── jna
    │   │   │   │           ├── jna
    │   │   │   │           │   └── 5.10.0
    │   │   │   │           │       └── jna-5.10.0.jar
    │   │   │   │           └── jna-platform
    │   │   │   │               └── 5.10.0
    │   │   │   │                   └── jna-platform-5.10.0.jar
    │   │   │   └── sf
    │   │   │       └── jopt-simple
    │   │   │           └── jopt-simple
    │   │   │               └── 5.0.4
    │   │   │                   └── jopt-simple-5.0.4.jar
    │   │   └── org
    │   │       ├── apache
    │   │       │   ├── commons
    │   │       │   │   └── commons-lang3
    │   │       │   │       └── 3.12.0
    │   │       │   │           └── commons-lang3-3.12.0.jar
    │   │       │   └── logging
    │   │       │       └── log4j
    │   │       │           ├── log4j-api
    │   │       │           │   └── 2.17.0
    │   │       │           │       └── log4j-api-2.17.0.jar
    │   │       │           ├── log4j-core
    │   │       │           │   └── 2.17.0
    │   │       │           │       └── log4j-core-2.17.0.jar
    │   │       │           └── log4j-slf4j18-impl
    │   │       │               └── 2.17.0
    │   │       │                   └── log4j-slf4j18-impl-2.17.0.jar
    │   │       └── slf4j
    │   │           └── slf4j-api
    │   │               └── 1.8.0-beta4
    │   │                   └── slf4j-api-1.8.0-beta4.jar
    │   ├── logs
    │   │   ├── 2022-11-26-1.log.gz
    │   │   ├── 2022-11-26-2.log.gz
    │   │   └── latest.log
    │   ├── minecraft_server.1.19.2.jar
    │   ├── ops.json
    │   ├── server.properties
    │   ├── usercache.json
    │   ├── versions
    │   │   └── 1.19.2
    │   │       └── server-1.19.2.jar
    │   ├── whitelist.json
    │   └── world
    │       ├── DIM-1
    │       │   └── data
    │       │       └── raids.dat
    │       ├── DIM1
    │       │   └── data
    │       │       └── raids_end.dat
    │       ├── data
    │       │   └── raids.dat
    │       ├── datapacks
    │       ├── entities
    │       │   ├── r.-1.-1.mca
    │       │   ├── r.-1.0.mca
    │       │   ├── r.0.-1.mca
    │       │   └── r.0.0.mca
    │       ├── level.dat
    │       ├── level.dat_old
    │       ├── playerdata
    │       ├── poi
    │       │   ├── r.-1.0.mca
    │       │   ├── r.0.-1.mca
    │       │   └── r.0.0.mca
    │       ├── region
    │       │   ├── r.-1.-1.mca
    │       │   ├── r.-1.0.mca
    │       │   ├── r.0.-1.mca
    │       │   └── r.0.0.mca
    │       └── session.lock
    ├── docker-compose.yml
    ├── setop.sh
    ├── telegramsend.sh
    └── whojoin.sh
```


***
## 참고 URL

- 10초만에 마인크래프트 서버 만들기: https://hibuz.com/minecraft_bedrock_docker_server/
- 마인크래프트 서버 설치 (Ubuntu 18.04): https://www.clien.net/service/board/cm_linux/15430979
