# Docker

## CLI

```bash
# 전체 컨테이너 확인
$ docker ps -a 

# 이미지 리스트 확인 
$ docker images

# 이미지 설치
$ docker pull {image_name})

# 컨테이너 생성
$ docker run --name {container_name} [OPTIONS] -p port:port IMAGE
```

### Docker 컨테이너 시작/중지/재시작

```bash
# Docker 컨테이너 시작
$ docker start {container_name}

# Docker 컨테이너 중지
$ docker stop {container_name}

# Docker 컨테이너 재시작
$ docker restart {container_name}
```



### Docker 컨테이너 접속

```bash
$ docker exec {container_name} basx`h
```

