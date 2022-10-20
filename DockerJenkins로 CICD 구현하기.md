# Docker/Jenkins로 CI/CD 구현하기

## 1. Docker 설치

```shell
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



## 2. Jenkins 설치

```shell
# Docker 이미지 다운로드
$ docker pull jenkins/jenkins:lts-jdk11

# 이미지 실행
$ docker run -d -p 20001:8080 -p 50000:50000 -v /var/jenkins:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock --name jenkins -u root jenkins/jenkins:lts-jdk11

# 현재 실행되어 있는 container 확인
$ docker ps

# 컨테이너 내부 접근
$ docker exec -it <container_id> bash
```



## 3. Jenkins 내부에 Docker 설치



## 4. Jenkins 설정

j7e201.p.ssafy.io:20001로 들어가서 jenkins 웹 접근

<img src="https://velog.velcdn.com/images%2Fhanif%2Fpost%2F52217ab0-1162-45e6-b911-568e5d6d486e%2Fimage.png" alt="img" style="zoom:67%;" />

```shell
# 비밀번호는 아래 코드를 통해 확인 가능
$ docker logs 7b2d11e63f5b
```



## 5. Plugin 설치

### ![image-20220926140312303](C:\Users\SSAFY\Desktop\assets\image-20220926140312303.png)

Jenkins 관리 > 플러그인 관리 > 설치 가능에서

`GitLab Plugin` 과 `Generic Webhook Trigger` 설치

![image-20220926140432932](C:\Users\SSAFY\Desktop\assets\image-20220926140432932.png)

## 6. Item 만들기

### 1. 새로운 아이템

![image-20220926140128667](C:\Users\SSAFY\Desktop\assets\image-20220926140128667.png)

### 2. Freestyle Project

![image-20220926140200576](C:\Users\SSAFY\Desktop\assets\image-20220926140200576.png)



### 3. 소스코드 관리 > 빌드 유발

![image-20220926140501479](C:\Users\SSAFY\Desktop\assets\image-20220926140501479.png)

![image-20220926140530846](C:\Users\SSAFY\Desktop\assets\image-20220926140530846.png)

Build when ... 을 누르면 고급 옵션을 눌러서 제일 밑에 Security token이 있다. 이는 나중에 Gitlab과 연동할 때 사용



### 4. Dockerfile 작성

dockerfile은 로컬 데이터를 컨테이너에 넣고 이미지화 할 때 사용되는 로직을 작성하는 파일이다

```dockerfile
FROM openjdk:8-jdk-alpine

# 빌드한 jar 파일을 컨테이너에 넣음 
ADD build/libs/DockChoDoGam-0.0.1-SNAPSHOT.jar app.jar

# jarfile 실행
ENTRYPOINT [ "java", "-jar","-Dspring.profiles.active=gcp", "/app.jar"]

# 컨테이너의 해당 포트 열어줌
EXPOSE 8081
```



### 5. Execute Shell

![image-20220926140642387](C:\Users\SSAFY\Desktop\assets\image-20220926140642387.png)

```shell
# 배포할 폴더로 이동 (해당 폴더에 dockerfile 위치)
cd Backend
cd DockChoDoGam

# gradlew 권한 획득
chmod +x ./gradlew

# jar파일 빌드
./gradlew build

# 기존에 backend라는 이름의 컨테이너가 존재한다면 멈춤
docker ps -f name=backend -q | xargs --no-run-if-empty docker container stop
# 해당 컨테이너 삭제
docker container ls -a -f name=backend -q | xargs -r docker container rm

# 도커 빌드 (dockerfile을 이용)
docker build -t backend .

# 만약 backend라는거 있으면 지움
docker ps -q --filter "name=backend" | grep -q . && docker stop backend && docker rm backend | true

# backend 컨테이너 시작 
docker run -p 8080:8081 -d -e TZ=Asia/Seoul --name=backend backend
docker rmi -f $(docker images -f "dangling=true" -q) || true
```



### 6. GitLab Webhook 설정 

repository > settings > webhook