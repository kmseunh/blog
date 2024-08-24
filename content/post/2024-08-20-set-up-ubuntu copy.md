+++
title = "Ubuntu 세팅을 해보자"
date = "2024-08-20"
description = "우분투 설정"
tags = [
    "2024",
    "server",
    "ubuntu"
]
categories = [
    "indiv"
]
series = ["Study"]
+++

나만의 서버가 생겼으니까 세팅을 해보려고 한다. <br>
세팅을 어렵지 않고 간단했지만 중간에 SSH 설정하면서 삽질을 좀 해가지고 시간이 오래걸렸다.

<!--more-->

## 기본 패키지 설치

```bash
sudo apt update
sudo apt install net-tools
```

- `net-tools` 패키지를 설치하면 여러 유용한 네트워크 관련 명령어를 사용할 수 있게 된다.

## vim 설정

```vim
set number                  " 행 번호 표시
set showcmd                 " 명령어 입력 시 명령어 표시
set showmatch               " 매칭되는 괄호 표시
set cursorline              " 커서가 있는 줄 강조
set hlsearch                " 검색 내용 하이라이트
set wildmenu                " 명령어 자동 완성 시 메뉴 표시
set wmnu                    " tab 자동완성 목록 표시
set nowrap                  " 명령어 자동 완성 시 메뉴 표시
set tabstop=4               " 탭을 4칸으로 설정
set shiftwidth=4            " 자동 들여쓰기 시 4칸 이동
set expandtab               " 탭을 스페이스로 변환
set autoindent              " 자동 들여쓰기
set clipboard=unnamedplus   " 시스템 클립보드 사용
set ignorecase              " 대소문자 구별 X
syntax on                   " 구문 강조 활성화
```

vim이 기본으로 설치되어 있어서 설정만 추가했다.

## SSH 접속 포트 변경

```bash
# SSH 설정 편집
sudo vim /etc/ssh/sshd_config

# 15번 라인: 새로운 포트 지정
Port 22
Port 10002

# 58번 라인: ssh key 파일 대신 password 입력 방식
PasswordAuthentication yes

# 59번 라인 암호 없는 유저 로그인 x
PermitEmptyPasswords no
```

58번과 59번 라인이 주석이 되어 있어서 둘 다 주석을 해제했다.

```bash
# SSH 설정 적용
sudo systemctl restart ssh

# 현재 실행 중인 SSH 서버 프로세스의 네트워크 상태 확인
sudo netstat -tnlp | grep sshd

# 현재 설정된 포트 확인
cat /etc/ssh/sshd_config | egrep ^\#?Port
```

## ufw 방화벽 설정

```bash
# 방화벽 상태 확인
sudo ufw status

# 방화벽 실행
ufw enable

# 추가한 포트 열기
sudo ufw allow 10002/tcp

# ssh 접속 확인
ssh <username>@<server_ip> -p 2222
```

## sudo 권한 계정 추가

```bash
# 사용자 계정 생성
sudo adduser <계정 이름>

# 계정 생성 확인
grep /bin/bash /etc/passwd | cut -f1 -d:

# sudo 권한 추가
sudo usermod -aG sudo <계정 이름>

# 생성된 계정 접속
su <계정 이름>

# sudo 권한 확인
groups

# 새로 생성한 계정 SSH 접속
ssh <계정 이름>@<server_ip> -p 2222
```

특정 사용자에게 관리자 권한을 부여하여 시스템의 설정을 변경하거나 프로그램을 설치할 수 있게 해주는 과정이다.

## SSH 사용자 접근 제한

```bash
# SSH 설정 변경
sudo vim /etc/ssh/sshd_config

# 14번 라인: 새로 추가한 계정 혀용, 기본값(ubuntu, opc)은 차단
AllowUsers <계정 이름>
DenyUsers ubuntu, opc

# SSH 재시작
sudo systemctl restart ssh
```

특정 사용자만 SSH 접속을 허용하거나 특정 사용자의 SSH 접속을 차단하여 보안을 강화할 수 있다고 한다.

## 타임존(Time Zone) 설정

```bash
# 현재 타임존 확인
timedatectl


# 타임존(Time Zone) 설정
sudo timedatectl set-timezone Asia/Seoul
```

## Swap 메모리 설정

스왑 메모리(Swap Memory)는 시스템의 물리적 메모리(RAM)가 부족할 때 디스크 공간을 임시 메모리로 사용하는 기능으로 메모리가 초과되었을 때, 메모리 부족 현상을 해결할 수 있다.

```bash
# Swap 메모리 상태 확인
free -h

# Swap 파일 생성
sudo falocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile

# Swap 파일 활성화
sudo swapon /swapfile

# Swap 파일 자동 마운트 설정
sudo vim /etc/fstab

# 맨 마지막 줄에 추가
/swapfile swap swap defaults 0 0
```

## fail2ban 설치

fail2ban은 서버에서 SSH, FTP, HTTP 등의 서비스에 대해 비정상적인 접근 시도를 감지하고 차단하는 도구이다.

```bash
# fail2ban 설치
sudo apt install fail2ban

# 기본 설정 파일 복사
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# jail.local 파일 수정
[DEFAULT]
bantime  = 10m      # 접속 차단 시간
findtime  = 10m     # 공격 시도로 간주되는 시간
maxretry  = 5       # 최대 로그인 시도 횟수
backend   = auto

[sshd]
enabled   = true
port      = ssh
logpath   = %(sshd_log)s

# fail2ban 재시작
sudo systemctl restart fail2ban

# fail2ban 상태 확인
sudo systemctl status fail2ban

# checkban 명령 alias 추가
```bash
# Bash 설정 파일 수정
vim ~/.bashrc

# 맨 아래줄에 추가
alias checkban="sudo tail -f /var/log/fail2ban.log"

# 설정 파일 적용
source ~/.bashrc
```

<span style='color: #2D3748; background-color: #fff5b1'>&nbsp;'sudo tail -f /var/log/fail2ban.log'&nbsp;</span>은 fail2ban 로그 파일에서 발생하는 모든 실시간 활동을 볼 수 있는 명령어이다.

## ISSUE 상황

기존 포트 22가 아닌 다른 포트로 SSH 접속을 설정하기 위해 `sshd_config`에 포트를 추가하고 오라클 인스턴스의 수신 규칙 목록에 등록도 했는데 방화벽 포트를 열어두지 않고 로그아웃을 해버렸다. 22 포트를 지워버린 상태라 접속할 수 있는 방법이 사라져버린 것이다..

진짜 너무 당황스러워서 GPT와 구글링으로 해결 방법을 검색하고 시도해 봤는데, 오라클 인스턴스에 Shell로 접속이 가능하다는 것을 알았다. OCI에 접속하고 인스턴스 페이지에 들어가면 리소스 항목에 콘솔 접속이라는 메뉴가 있다. 콘솔 접속에 들어가서 Cloud Shell 접속 실행을 하면 접속이 가능했다.

인스턴스를 삭제하고 다시 생성해야 하나 고민을 했지만 방법이 있어서 다행이었다..

<p align="center"><img src="https://github.com/user-attachments/assets/63ce090b-b23a-4920-9e74-d11ee5156403" width="400"></p>
