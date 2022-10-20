# Server

## 접속

J07E201.pem 파일을 다운로드 받고 해당경로에서

```bash
$ ssh -i J07E201.pem ubuntu@j07e201.p.ssafy.io
```



## 기본 설정

```bash
$ sudo apt upgrade
$ sudo apt update

# Java 설치
$ sudo apt install openjdk-8-jdk

# JAVA_HOME 설정
# ~/.bashrc에 export 추가
$ sudo vi ~/.bashrc
export JAVA_HOME=$(readlink -f /usr/bin/java | sed "s:bin/java::")

# 적용
$ source ~/.bashrc

# 방화벽 설정
$ sudo ufw allow 22
# 22 port를 열어놓지 않으면 ssh로 접근 불가능하므로 꼭 열어야 한다.
$ sudo ufw enable
# 상태 확인
$ sudo ufw status
```



## Docker 다운로드

```bash
$ sudo apt update

# http 패키지 설치
    $ sudo apt-get install -y ca-certificates \ 
    curl \
    software-properties-common \
    apt-transport-https \
    gnupg \
    lsb-release
    
# 레포지토리 설정
# Docker의 Official GPG Key 를 등록
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
# stable repository 를 등록
$ echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
# docker 엔진 설치
$ sudo apt install docker-ce docker-ce-cli containerd.io

# docker 그룹에 사용자 추가
$ sudo usermod -aG docker ubuntu
```



## Docker를 이용하여 MySQL 다운로드

```bash
# Docker 이미지 다운로드
$ docker pull mysql

# Docker 이미지 확인
$ docker images

# MySQL Docker 컨테이너 생성 및 실행
$ docker run --name mysql-container -e MUSQL_ROOT_PASSWORD={password} -d -p 3306:3306 mysql:latest

# 도커 컨테이너 리스트 출력
$ docker ps -a

# MySQL Docker 컨테이너 접속
$ docker exec -it mysql-container bash
```



## Docker를 이용하여 MongoDB 다운로드 

```bash
# 몽고 Dcoker 이미지 다운로드
$ docker pull mongo

# Mongo db Docker 컨테이너 생성 및 실행
$ docker run --name mongodb -d -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=dokcho -e MONGO_INITDB_ROOT_PASSWORD="dkdlvhs14tkrhtlvek" mongo
```



## Jenkins 설치 + CI/CD

### 1. Jenkins 설치

```bash
# jenkins Docker 이미지 다운로드
$ docker pull jenkins/jenkins

# Docker 컨테이너 생성 및 실행
$ docker run --name jenkins -d -p 20001:8080 -p 50000:50000 jenkins/jenkins
```



### 2. Jenkins 설정

j7e201.p.ssafy.io:20001로 들어가서 jenkins 웹 접근

![img](https://velog.velcdn.com/images%2Fhanif%2Fpost%2F52217ab0-1162-45e6-b911-568e5d6d486e%2Fimage.png)

```bash
# 비밀번호는 아래 코드를 통해 확인 가능
$ docker logs 7b2d11e63f5b

# 계정 정보
dokcho
ehrch12@l
```

