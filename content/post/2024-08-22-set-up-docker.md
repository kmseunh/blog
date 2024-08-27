+++
title = "Docker에 Nginx 웹 서버를 구축해 보자"
date = "2024-08-22"
description = "Docker 설정"
tags = [
    "2024",
    "server",
    "docker",
    "nginx"
]
categories = [
    "indiv"
]
series = ["Study"]
+++

웹 서버(Web Server)는 웹 페이지와 애플리케이션을 인터넷이나 네트워크를 통해 사용자에게 제공한다.

- 클라이언트(일반적으로 웹 브라우저)로부터의 요청을 처리하고, 해당 요청에 맞는 데이터를 서버에서 찾아 응답으로 반환하는 역할을 한다.

Nginx는 웹 서버 소프트웨어로, 웹 서버, 리버스 프록시, 로드 밸런서, 캐싱 등의 다양한 기능을 제공한다. <br>

- 웹 애플리케이션의 성능, 보안, 효율성을 크게 개선할 수 있다.

도커(Docker)는 컨테이너화 기술을 기반으로 하는 오픈 소스 플랫폼이다. <br>
리눅스에서 애플리케이션을 실행할 수 있는 독립된 환경을 제공하는데 이때, 독립된 환경을 컨테이너라고 한다.

- 컨테이너는 애플리케이션을 실행할 수 있는 환경

<br>

원래는 서버 전역에 Nginx를 설치하여 사용하는 방법을 고려했지만, Docker를 활용하면 Nginx와 각 프로젝트의 환경을 완전히 분리할 수 있다. <br>
또한, Docker를 사용하면 각 프로젝트 내 애플리케이션을 독립적으로 관리할 수 있어, 유지보수 및 확장성 측면에서 이점이 있다. <br>

이러한 이유로 Docker를 사용해 보려고 한다.

<p align="center"><img src="https://github.com/user-attachments/assets/344f7915-4cee-493c-b311-a8382fb18a3c" width="400"></p>

<!--more-->

## docker & docker-compose 설치

```bash
# 필수 패키지 설치
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common

# Docker 공식 GPG 키 추가
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo tee /etc/apt/trusted.gpg.d/docker.asc

# Docker 저장소 추가
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/trusted.gpg.d/docker.asc] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Docker 설치
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Docker 설치 확인
docker --version

# 도커 실행 권한 설정
sudo usermod -aG docker $USER
```

<span style='color: #2D3748; background-color: #fff5b1'>&nbsp;'sudo usermod -aG docker $USER '&nbsp;</span> 명령어는 현재 사용자를 Docker 그룹에 추가하여 사용자가 Docker 명령어를 실행할 때마다 `sudo`를 입력할 필요 없이 더 효율적으로 작업할 수 있게 해준다.

```bash
# docker-compose 최신 버전 정보 가져오기
LATEST_VERSION=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep 'tag_name' | cut -d\" -f4)

# 최신 버전 바이너리 다운로드 및 설치
sudo curl -L "https://github.com/docker/compose/releases/download/${LATEST_VERSION}/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 실행 권한 부여
sudo chmod +x /usr/local/bin/docker-compose

# docker-compose 설치 확인
docker-compose --version
```

## Nginx 웹 서버 구축

Nginx와 Certbot을 사용하여 SSL 인증과 HTTPS 설정을 자동으로 구성하는 웹 서버를 구축하려고 한다.

- Certbot은 SSL/TLS 인증서를 자동으로 발급하고 갱신하는 도구로, `Let’s Encrypt`와 같은 인증 기관(CA)과 연동하여 웹사이트에 HTTPS를 적용하고
보안을 강화하는 데 사용된다.

> 도메인은 <a href="https://www.gabia.com/?utm_source=google&utm_medium=cpc&utm_term=%EA%B0%80%EB%B9%84%EC%95%84&utm_campaign=%EA%B0%80%EB%B9%84%EC%95%84" target="_blank">가비아</a>에서 구매하려고 했는데, 우선적으로 테스트 해보고 싶어서 <a href="https://xn--220b31d95hq8o.xn--3e0b707e/" target="_blank">내도메인.한국</a>에서 발급받았다.

### Nginx 및 Certbot 초기 설정

```bash
# nginx 폴더 생성 후 이동
mkdir nginx
cd ngix

# docker-compose.yml 작성
vi docker-compose.yml
```

```yaml
# docker-compose.yml
version: '3.9'
services:
  nginx:
    image: nginx:latest
    restart: unless-stopped
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./conf/nginx.conf:/etc/nginx/nginx.conf
      - ./data/certbot/conf:/etc/letsencrypt
      - ./data/certbot/www:/var/www/certbot

  certbot:
    image: certbot/certbot
    restart: unless-stopped
    volumes:
      - ./data/certbot/conf:/etc/letsencrypt
      - ./data/certbot/www:/var/www/certbot
```

```bash
# conf 폴더 생성
mkdir conf

# nginx.conf 파일 작성
vi conf/nginx.conf
```

```nginx
# nginx.conf
server {
     listen 80;

     server_name <구매한 도메인>;

     location /.well-known/acme-challenge/ {
             allow all;
             root /var/www/certbot;
     } 
}
```

```bash
# docker-compose 실행
docker-compose -f docker-compose.yml up -d

# 컨테이너 상태 확인
docker ps

# 인증서 발급 스크립트 다운로드
curl -O https://raw.githubusercontent.com/wmnnd/nginx-certbot/master/init-letsencrypt.sh
chmod +x init-letsencrypt.sh

# 인증서 발급 스크립트 수정
vim init-letsencrypt.sh

domains=(<구매한 도메인> www.<구매한 도메인>)
rsa_key_size=4096
data_path="./[CUSTOM]/certbot"
email="[이메일]" # Adding a valid address is strongly recommended
staging=0 # Set to 1 if you're testing your setup to avoid hitting request limits

# 인증서 발급 스크립트 실행
sudo ./init-letsencrypt.sh
```

### HTTPS 적용 및 자동 갱신 설정

```nginx
# nginx.conf 수정
user nginx;

worker_processes 1;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    server {
        listen 80;
        server_name <구매한 도메인>;
        server_tokens off;

        location /.well-known/acme-challenge/ {
            allow all;
            root /var/www/certbot;
        }

        location / {
            return 301 https://$host$request_uri;
        }
    }

    server {
        listen 443 ssl;
        server_name <구매한 도메인>;
        server_tokens off;

        ssl_certificate /etc/letsencrypt/live/<구매한 도메인>/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/<구매한 도메인>/privkey.pem;
        ssl_protocols TLSv1.2 TLSv1.3;

        include /etc/letsencrypt/options-ssl-nginx.conf;
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

        location / {
            root /usr/share/nginx/html;
            index index.html index.htm;
        }

        error_page 500 502 503 504   /50x.html;
        location = /50x.html {
            root /usr/share/nginx/html;
        }
    }
}
```

```yaml
# docker-compose.yml 수정
version: '3.9'
services:
  nginx:
    image: nginx:latest
    restart: unless-stopped
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./conf/nginx.conf:/etc/nginx/nginx.conf
      - ./data/certbot/conf:/etc/letsencrypt
      - ./data/certbot/www:/var/www/certbot
    # Nginx 웹 서버를 실행하고 주기적으로 Nginx 설정을 다시 로드하는 명령어
    command: '/bin/sh -c ''while :; do sleep 6h & wait $${!}; nginx -s reload; done & nginx -g "daemon off;"'''

  certbot:
    image: certbot/certbot
    restart: unless-stopped
    volumes:
      - ./data/certbot/conf:/etc/letsencrypt
      - ./data/certbot/www:/var/www/certbot
    # 컨테이너가 실행되는 동안 12시간마다 Certbot을 사용해 SSL 인증서를 갱신하는 무한 루프를 실행하는 명령어
    entrypoint: "/bin/sh -c 'trap exit TERM; while :; do certbot renew; sleep 12h & wait $${!}; done;'"
```

```bash
# docker compose 실행
docker-compose -f docker-compose.yml up -d

# 인증서 발급 스크립트 실행
sudo ./init-letsencrypt.sh

# 컨테이너 상태 확인
docker ps
```

#### 서버의 디렉토리 구조

```bash
~/docker/
└── nginx/
    ├── conf/
    │   └── nginx.conf
    ├── data/
    │   └── certbot/
    │       └── www/
    ├── docker-compose.yml
    └── init-letsencrypt.sh
```

## ISSUE 상황

<span style='color: #2D3748; background-color: #fff5b1'>&nbsp;'docker-compose up -d'&nbsp;</span> 명령어로 컨테이너를 생성하면 `nginx:latest` 컨테이너가 `Restarting (127) x seconds ago`의 상태였다. <br>

컨테이너 로그를 확인하면, `ssl: error:80000002:system library::no such file or directory:calling fopen(/etc/letsencrypt/live//fullchain.pem, r)` 였고, SSL/TLS 인증서 파일을 찾지 못해 발생하는 문제였다.

처음 nginx 컨테이너를 올리고 `sudo ./init-letsencrypt.sh` 명령어로 인증서를 발급했는데, 그 이후에 HTTPS 설정을 하고 나서  스크립트를 실행하지 않아서 생긴 에러 같다. <br>
( `~같다`라고 예상한 이유는 다양한 시도를 해도 해결되지 않다가 스크립트를 한 번 더 실행하고 나서 정상작동했기 때문이다.. )

<hr>

글로 보면 10분도 안 돼서 설정할 거 같은데 HTTPS 적용 및 자동 갱신 설정에서 수많은 에러를 맞닥뜨려서 3일 정도를 삽질했다. <br>
docker와 nginx와 같은 서버와 인프라에 대한 지식이 없어서 당연한 거고, 이번 경험을 통해 도커의 구조와 역할, 웹서버에 대해 조금 알게 되었다. <br>

앞으로 험난한 일정이 예상되지만, 책상에 계속 앉아서 고민하고 해결하는 게 재밌다.
