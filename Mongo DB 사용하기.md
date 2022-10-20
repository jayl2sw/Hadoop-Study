# Mongo DB 사용하기

```bash
$ docker pull mongo
$ docker run --name mongodb -d -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=dokcho -e MONGO_INITDB_ROOT_PASSWORD="dkdlvhs14tkrhtlvek" mongo
$ docker exec -it mongodb /bin/bash
$ mongosh -u dokcho -p "dkdlvhs14tkrhtlvek"

use admin
db.createUser({ user: 'dokcho', pwd: "dkdlvhs14tkrhtlvek", roles: ['root'] })
```



## Collection 생성, 제거

```shell
# 옵션 설정하고 Collection 만들 때
> db.createCollection({Collection_name}, {
... capped: true,
... autoIndex: true,
... size: 6142800,
... max: 10000
... })

# Collection 제거
db.{Collection_name}.drop()
```



## Document 생성, 제거

```shell
# 옵션 설정안하고 Collection 자동으로 만들어짐
db.{collection_name}.insert({key: value})
db.{collection_name}.remove(criteria, justOne)
```



## 테이블, 컬렉션 조회

```shell
# db 조회
show dbs

# 사용할 DB 설정
use {database_name}

# 컬렉션 조회
show collections
```



## Document 조회

```shell
# 컬랙션 내 모든 데이터 조회
db.{collection_name}.find({})

# 컬렉션내 조건에 맞는 데이터 조회
db.{collection_name}.find({key: value})
```

