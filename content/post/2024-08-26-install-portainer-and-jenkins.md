+++
title = "Portainer와 Jenkins를 설치해보자"
date = "2024-08-26"
description = "Portainer와 Jenkins 설치 후 Https 프록시 설정"
tags = [
    "2024",
    "server",
    "docker",
    "jenkins"
]
categories = [
    "indiv"
]
series = ["Study"]
+++

웹 서버를 구축하고 4일 동안 정말 고통스러운 시간이었다. <br>
`Portainer`와 `Jenkins`을 설치하고 Https 프록시 설정을 하고 싶었는데 쉴 틈 없이 쏟아지는 에러들로 하염없이 시간을 보내버렸다. <br>
Portainer는 오래 걸리지 않았는데, Jenkins을 설치할 때 정말 많은 시간이 걸렸다.<br>
( 프로젝트 진행한 다음에 설치해도 되는데 왜 그랬을까... )

Portainer는 Docker 환경을 관리하기 위한 웹 기반 관리 도구로, Docker 컨테이너, 이미지, 볼륨 등을 직관적인 사용자 인터페이스를 통해 쉽게 관리할 수 있다. <br>

Jenkins는 CI/CD(지속적 통합 및 지속적 배포)를 자동화하는 도구로, 코드 변경 사항을 자동으로 빌드, 테스트, 배포하는 파이프라인을 구축할 수 있다.

특히 Jenkins를 활용하여 개발 및 배포 프로세스를 자동화하고, GitHub와 연동하여 파이프라인을 구축해 보고 싶다.

<p align="center"><img src="https://github.com/user-attachments/assets/e3d93ca0-9e74-429a-b368-47d5786069de" width="450"></p>

<!--more-->

## Portainer & Jenkins 설치

```yaml
# docker-compose.yml에 Portainer & Jenkins 서비스 추가
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: unless-stopped
    ports:
      - 9000:9000
    volumes:
      - '/var/run/docker.sock:/var/run/docker.sock'
      - './portainer_data:/data'

  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    restart: unless-stopped
    ports:
      - 8080:8080
      - 50000:50000
    volumes:
      - './jenkins_home:/var/jenkins_home'
      - '/var/run/docker.sock:/var/run/docker.sock'
    environment:
      - JENKINS_OPTS=--prefix=/jenkins
    user: root

volumes:
  portainer_data:
  jenkins_home:
```

Jenkins와 Portainer 데이터는 로컬 볼륨에 저장되므로 컨테이너가 재시작되더라도 데이터가 유지된다.

## HTTPS 프록시 설정

```nginx
# nginx.conf에 Portainer & Jenkins 프록시 설정 추가
    location /jenkins/ {
        proxy_pass http://jenkins:8080/jenkins/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /portainer/ {
        proxy_pass http://portainer:9000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location / {
        try_files $uri $uri/ =404;
    }
```

리버스 프록시를 설정하여 `/jenkins/`와 `/portainer/` 경로를 통해 각 서비스에 접근할 수 있다.

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
    ├── init-letsencrypt.sh
    ├── jenkins_home
    └── portainer_data
```

## ISSUE 상황

이 작업이 예상보다 4일이나 걸린 이유를 곰곰이 생각해 보았다. <br>
작업 중 주요하게 발생한 에러는 `403 Forbidden`과 `404 Not Found`였는데, `redirect.js` 및 login 페이지가 `/usr/share/nginx/html`에 존재함에도 불구하고 해당 파일을 찾지 못했다. <br>
또한 Jenkins 관련 디렉토리와 파일의 권한을 모두 일치시켰음에도 불구하고 동일한 에러가 반복해서 발생했다.

문제를 해결하기 위해 `docker-compose.yml`에서 `Jenkins` 서비스를 설정할 때,

```yaml
    environment:
      - JENKINS_OPTS=--prefix=/jenkins
    user: root
```

를 추가한 결과, 문제가 해결되었다.

<span style='color: #2D3748; background-color: #fff5b1'>&nbsp;'JENKINS_OPTS=--prefix=/jenkins'&nbsp;</span> 는 Jenkins가 `/jenkins` 경로 아래에서 실행되도록 하고, <span style='color: #2D3748; background-color: #fff5b1'>&nbsp;'user: root'&nbsp;</span> 는 Jenkins를 루트 사용자로 실행함으로써, 파일 시스템의 접근 권한 문제를 해결해 준다고 한다.

<hr>

생각보다 오래 걸리고 해결되지 않아 막막했지만, 결국 여차저차 성공하게 되어 다행이다. <br>
100번의 에러에 짜증 나도 1번 성공할 때의 그 도파민 어쩔껀데~

이제 Git으로 프로젝트를 내려받고, Docker를 이용해 이미지를 생성하여 Docker Hub에 push 해보려 한다.
