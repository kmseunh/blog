+++
title = "CI/CD를 구축해보자"
date = "2024-08-28"
description = "Jenkins, Github Action로 CI/CD 파이프라인 구축"
tags = [
    "2024",
    "github",
    "jenkins",
    "ci/cd"
]
categories = [
    "indiv"
]
series = ["Study"]
+++

서비스를 배포하려면 개발, 테스트, 빌드, 배포 등 여러 단계를 거쳐야 하며, 코드 변경 시마다 이 과정을 반복하는 것은 굉장히 번거롭다. <br>
CI/CD는 이러한 반복 과정을 자동화하여 애플리케이션을 빠르고 안정적으로 배포하며, 개발자가 빌드나 배포에 소요하는 시간을 줄여 개발에 더 집중할 수 있게 도와준다고 한다.

- CI (Continuous Integration): 빌드, 테스트의 자동화
- CD (Continuous Delivery/Deployment): 배포의 자동화

다양한 CI/CD 도구 중에서 Jenkins와 GitHub Actions을 고민하고 있었다. <br>
고민하던 중 두 가지 도구를 함께 사용하는 방법이 효과적이라는 것을 알게 되었고, 이를 각각 CI와 CD에 활용해 보고 싶다는 생각이 들었다.

> Jenkins와 GitHub Actions을 각각 CI와 CD에 활용하면, Jenkins는 복잡한 빌드와 테스트 과정을 효과적으로 관리하고, GitHub Actions은 간편하게 배포를 자동화하여 전체 CI/CD 프로세스가 더 효율적이고 안정적으로 진행된다고 한다.

<p align="center"><img src="https://github.com/user-attachments/assets/8dccbcfb-8862-4110-9cfe-6401fc94fbeb" width="450"></p>

<!--more-->

## GitHub, Docker Hub 및 Jenkins 연동 설정

### GitHub Token 생성 및 Jenkins에 등록

Jenkins에서 GitHub와의 통합을 위해 GitHub Personal Access Token을 생성하고 Jenkins에 등록해야 한다.

```md
1. GitHub 프로필의 Settings 탭으로 이동한다.
2. Developer settings > Personal access tokens > Tokens (classic)을 클릭한다.
3. Generate new token > `repo`와 `admin:repo_hook` 체크 > Generate token을 클릭한다.
4. 생성된 토큰을 복사하여 안전한 장소에 보관한다.

# Jenkins Credentials 생성
1. Jenkins에 접속하고 Jenkins 관리 > Credentials로 이동한다.
2. Domains 밑에 있는 (global) > Add Credentials을 클릭한다.
3. Username는 Github ID, Password는 생성한 Github Token을 입력하고 Create 버튼을 클릭한다.
  • ID는 식별 가능한 ID를 입력한다.
```

### Github Repository Webhook 생성

GitHub Webhook을 설정하여 레포지토리의 커밋 이벤트를 Jenkins에서 자동으로 감지하고 빌드하도록 설정해야 한다.

```md
1. GitHub 레포지토리의 Settings 탭으로 이동한다.
2. Webhooks 섹션에서 Add webhook을 클릭한다.
3. Payload URL에 `Jenkins URL/github-webhook/` 를 입력한다. 
  • ex. `http://your-jenkins-server.com/github-webhook/`
4. Content type은 `application/json`, 이벤트는 `Just the push event`로 설정한다.
5. Add webhook 버튼을 클릭하여 Webhook을 생성한다.
```

## Docker와 Jenkins 설정

### Docker Hub 연결

Jenkins에서 Docker 이미지를 빌드하고 Docker Hub로 Push 할 수 있도록 Docker Hub 계정과 Jenkins를 연동해야 한다.

```md
1. Docker Hub에 로그인 한다.
2. Account Settings로 이동하여 Security > New Access Token을 클릭한다.
3. Access Token Description 입력 > Generate 클릭하고 생성된 Token을 복사한다.
4. Jenkins의 대시보드에서 Jenkins 관리 -> Credentials로 이동한다.
5. Domains 밑에 있는 (global) > Add Credentials을 클릭한다.
6. Username는 Docker Hub Username, Password는 복사한 Access Token을 입력하고 Create 버튼을 클릭한다.
```

### Jenkins에 Node.js 및 Docker Pipeline 플러그인 설치

`Node.js`와 `Docker` 관련 작업을 수행하기 위해 필요한 플러그인을 설치해야 한다.

```md
1. Jenkins의 대시보드에서 Jenkins 관리 -> Plugins로 이동한다.
2. Available 탭에서 다음 플러그인을 검색하고 설치한다.
  • NodeJS
  • Docker Pipeline
3. Jenkins 관리 > Global Tool Configuration > Nodejs 항목 > Add NodeJs를 클릭한다.
  • Install automatically 체크
  • Name: pipeline에서 사용할 이름
  • Version: Dockerfile Node.js 버전
```

### Jenkins 컨테이너에 Docker 설치

Jenkins가 Docker 명령어를 실행할 수 있도록 Jenkins 컨테이너에 Docker를 설치해야 한다.

```bash
# jenkins 컨테이너 접속
docker exec -it jenkins-container-name bash

# Docker 설치
## 패키지 업데이트 및 필수 패키지 설치
apt update
apt install ca-certificates curl gnupg lsb-release

## Docker GPG 키링 설정
mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg

## Docker 리포지토리 추가(Debian 용)
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian bookworm stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

## 패키지 목록 업데이트 및 docker 설치
apt update
apt install docker-ce docker-ce-cli containerd.io
```

## Jenkins CI 파이프라인 구축

Jenkins에서 CI 파이프라인을 설정해야 한다.

```md
1. Jenkins 대시보드에서 New Item > pipeline을 선택한 후 이름을 지정한다.
2. GitHub project를 클릭하고 Project url에 레포지토리 URL을 입력한다.
3. GitHub hook trigger for GITScm polling을 클릭한다.
3. Build Triggers에서 GitHub hook trigger for GITScm polling을 선택한다.
4. Pipeline > Pipeline script를 선택하고 Script를 작성한다.
```

```yml
# Pipeline Script
pipeline {
    agent any
    
    tools {
        nodejs 'nodejs-18'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github-token', url: '<http://your-jenkins-server.com>'
            }
        }
        
        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                    sh 'npm run build'
                    script {
                        def frontendImage = docker.build('your-docker-hub-username/your-docker-hub-repository')
                        docker.withRegistry('https://index.docker.io/v1/', 'docker-hub') {
                            frontendImage.push('latest')
                        }
                    }
                }
            }
        }
    }
}
```

현재는 frontend 디렉토리에서 빌드 후 Docker 이미지를 생성하여 Push하는 방식으로 설정되어 있다. <br>
앞으로 frontend와 backend를 각각 독립적으로 빌드 하기 위해, backend 디렉토리도 별도로 분리하여 빌드 할 예정이며, 이를 위해 유사한 방식으로 별도의 stage를 추가할 계획이다.

## GitHub Actions으로 CD 구축

### 레포지토리에 Secret Key 생성

GitHub Actions에서 필요한 비밀 키를 GitHub 레포지토리에 설정해야 한다.

```md
1. GitHub 레포지토리의 Settings -> Secrets로 이동한다.
2. New repository secret를 클릭하고 다음 항목들을 추가한다.
  • SERVER_IP: 서버 IP 주소
  • SERVER_USER: 서버 사용자 이름
  • SERVER_SSH_KEY: 서버로 접속하기 위한 SSH 개인 키
```

 `.github/workflows/frontend-deploy.yml` 작성

```yml
name: frontend-deploy

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Pull Docker image from Docker Hub
        run: |
          docker pull your-docker-hub-username/your-docker-hub-repository

      - name: Set up SSH key
        uses: webfactory/ssh-agent@v0.6.0
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

      - name: Deploy to server
        run: |
          ssh -o StrictHostKeyChecking=no -p 22 <username>@<ip-address> '
            if [ $(docker ps -q -f name=<your-docker-container-name>) ]; then
              docker stop <your-docker-container-name>
              docker rm <your-docker-container-name>
            fi

            docker pull your-docker-hub-username/your-docker-hub-repository

            docker run -d --name your-docker-hub-repository -p 3000:3000 your-docker-hub-username/your-docker-hub-repository
          '
```

`deploy.yml`을 생성하면 자동으로 Actions 탭에서 workflow가 생성된다.

## ISSUE 상황

도커 컨테이너에서 Jenkins를 관리하고 있어 서버에는 Node.js가 설치되어 있지 않았다. <br>
그래서 Jenkins 빌드 시 Node.js가 컨테이너 내에서 설치될 수 있도록 Jenkins 플러그인에서 Node.js를 설치하도록 설정했다.

또한 Github Action에서 `deploy.yml` 파일을 생성했는데 Action이 활성화되지 않는 이슈가 발생했다. <br>
파일을 생성하면 자동으로 활성화된다고 했는데 이슈가 발생해서 굉장히 당황스러웠다. <br>
알고보니 파일 이름을 수정할 때 workflows 폴더를 workflow로 수정해버려서 생긴 이슈였다.. 허허

<hr>

드디어 원하는 서버 세팅이 완료되었다. <br>
또 다른 프로젝트들을 생성하고 도메인을 추가로 설정할 때 또 다른 어려움이 있을 것 같아 걱정되긴 하지만, 지금은 마음 편하게 공부하며 프로젝트를 진행하려고 한다. <br>
( 터미널 바라만 봐도 좋아.. )
