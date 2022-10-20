# Sqoop

### Sqoop import

```bash
$ bin/sqoop import --connect jdbc:mysql://j7e201.p.ssafy.io:3306/dokcho --username root --password 0000 --query 'SELECT e.emp_no, e.birth_date, e.first_name, e.last_name, e.gender, e.hire_date, d.dept_no FROM employees e, dept_emp d WHERE (e.emp_no = d.emp_no) AND $CONDITIONS' --target-dir /user/j7e201/sqoop/employees --split-by e.emp_no
```

![image-20220916152701434](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220916152701434.png)



> Apache commons-lang 설치 해주어야 함 

```bash
$ wget https://dlcdn.apache.org//commons/lang/binaries/commons-lang-2.6-bin.tar.gz
```



```bash
# 테이블 통채로 가져옴
$ bin/sqoop import --connect jdbc:mysql://j7e201.p.ssafy.io:3306/employees --username root --password 0000 --table employees --target-dir /user/fastcampus/sqoop/employees1

# query를 실행해서 가져옴
$ bin/sqoop import --connect jdbc:mysql://j7e201.p.ssafy.io:3306/employees --username root --password 0000 --query 'SELECT e.emp_no, e.birth_date, e.first_name, e.last_name, e.gender, e.hire_date, d.dept_no FROM employees e, dept_emp d WHERE (e.emp_no = d.emp_no) AND $CONDITIONS' --target-dir /user/j7e201/sqoop/employees2 -m 1
# mapper 한개	
```

```shell

```





### Sqoop list-databases

* 데이터베이스 받아옴

* ```bash
  $ bin/sqoop list-databases --connect jdbc:mysql://j7e201.p.ssafy.io:3306/dokcho --username root --password 0000
  ```



### Sqoop list-tables

* 데이터베이스 내의 테이블 정보 받아옴

* ```bash
  $ bin/sqoop list-tables --connect jdbc:mysql://j7e201.p.ssafy.io:3306/dokcho --username root --password aoj29tj293092852tknwiog2395gw9023t
  ```



### Sqoop eval

* sql query를 실행해서 결과를 받아옴

* ```bash
  $ bin/sqoop eval --connect jdbc:mysql://j7e201.p.ssafy.io:3306/dokcho --username root --password aoj29tj293092852tknwiog2395gw9023t --query "select * from monster"
  ```



### Sqoop import

1. 테이블 통채로 들고와서 import	

   ```bash
   $ bin/sqoop import --connect jdbc:mysql://j7e201.p.ssafy.io:3306/dokcho --username root --password "aoj29tj293092852tknwiog2395gw9023t" --table monster --target-dir /user/fastcampus/sqoop/dokcho/monster
   ```

2. 쿼리문 실행해서 해당 결과값을 import

   ```bash
   $ bin/sqoop import --connect jdbc:mysql://j7e201.p.ssafy.io:3306/dokcho --username root --password "aoj29tj293092852tknwiog2395gw9023t" --query 'SELECT * FROM monster WHERE (type=1) AND $CONDITIONS' --target-dir /user/j7e201/sqoop/dokcho/yakcho -m 1 # mapper의 개수 설정 =파일의 개수
   ```

3. mapper가 여러개일 때 어떤 컬럼을 기준으로 나눌지 결정 (split-by)

   ```bash
   $ bin/sqoop import --connect jdbc:mysql://j7e201.p.ssafy.io:3306/dokcho --username root --password 0000 --query 'SELECT * FROM monster WHERE $CONDITIONS' --target-dir /user/j7e201/sqoop/dokcho2 -m 3 --split-by type # mapper의 개수 설정 =파일의 개수
   ```

   

### Sqoop export

```bash
$ bin/sqoop export --connect jdbc://mysql://j7e201.p.ssafy.io:3306/dokcho --username root --password 0000 --table {table_name} --export-dir {hdfs_path}
```



## 실제 코드

```sh
now=$(date +"%y%m%d_%H")
/hadoopeco/sqoop/bin/sqoop import --connect jdbc:mysql://j7e201.p.ssafy.io:3306/dokcho --username root --password aoj29tj293092852tknwiog2395gw9023t --table users --target-dir /user/dokcho/backups/$now/users
/hadoopeco/sqoop/bin/sqoop import --connect jdbc:mysql://j7e201.p.ssafy.io:3306/dokcho --username root --password aoj29tj293092852tknwiog2395gw9023t --table user_monster --target-dir /user/dokcho/backups/$now/usermonster
```

```bash
now=$(date +"%y%m%d_%H")
/eco/sqoop/bin/sqoop import --connect jdbc:mysql://j7e201.p.ssafy.io:3306/dokcho --username root --password aoj29tj293092852tknwiog2395gw9023t --table users --target-dir /user/dokcho/backups/users/$now
/eco/sqoop/bin/sqoop import --connect jdbc:mysql://j7e201.p.ssafy.io:3306/dokcho --username root --password aoj29tj293092852tknwiog2395gw9023t --table user_monster --target-dir /user/dokcho/backups/usermonster$now
```

