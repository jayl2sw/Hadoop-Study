# Hadoop 클러스터 구성

## 개요

1. 다운로드, 설정

2. HDFS 시작/종료
3. HDFS 테스트
4. Data 다운로드 및 PUT
5. YARN 시작/종료
   * YARN : 클러스터 매니저
6. YARN 테스트



### 1. 다운로드, 설정

#### 하둡 다운로드

```bash
$ wget https://archive.apache.org/dist/hadoop/common/hadoop-3.3.2/hadoop-3.3.2.tar.gz
$ tar xvfz hadoop-3.3.2.tar.gz
```



#### 하둡 설정

> ##### hadoop-env.sh 

```sh
export JAVA_HOME=/kikang/jdk8
```



> ##### core-site.xml
>
> ​	hdfs와 Yarn이 모두 사용

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://spark-master-01:9000</value>
    </property>
</configuration>
```



> ##### hdfs-site.xml
>
> ​	namenode가 구동중인 서버의 메타 데이터가 저장되는 위치
>
> ​	datanode 서버의 대용량 데이터가 저장되는 위치
>
> ​	namenode가 체크포인트를 쓰는 곳

```xml
<configuration>
	<property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///kikang/hadoop3/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.name.dir</name>
        <value>file:///kikang/hadoop3/dfs/data</value>
    </property>
    <property>
        <name>dfs.namenode.checkpoint.dir</name>
        <value>file:///kikang/hadoop3/dfs/namesecondary</value>
    </property>
</configuration>
```



> ##### yarn-site.xml

```xml
<configuration>
    <!-- host name 잡아줌 --> 
	<property>
    	<name>yarn.resourcemanager.hostname</name>
        <value>spark-master-01</value>
    </property>
    <!-- port 변경이 필요할 경우, 공격을 막기 위해--> 
    <property>
    	<name>yarn.resourcemanager.webapp.address</name>
        <!-- 기본 포트-->
        <!-- 
			<value>spark-master-01:8088</value>
        -->
		<value>spark-master-01:8188</value>
    </property>
</configuration>
```



> ##### workers

```
spark-worker-01
spark-worker-02
spark-worker-03
```



##### - copy 'jdk8', 'hadoop3' from master to workers

```
scp -r /hadoopeco/jdk8 dokcho@dokcho-worker-01:/hadoopeco/
scp -r /hadoopeco/jdk8 dokcho@dokcho-worker-02:/hadoopeco/
scp -r /hadoopeco/jdk8 dokcho@dokcho-worker-03:/hadoopeco/
scp -r /hadoopeco/jdk8 dokcho@dokcho-worker-04:/hadoopeco/
scp -r /hadoopeco/jdk8 dokcho@dokcho-worker-05:/hadoopeco/
scp -r /hadoopeco/jdk8 dokcho@dokcho-worker-06:/hadoopeco/
scp -r /hadoopeco/jdk8 dokcho@dokcho-worker-07:/hadoopeco/
scp -r /hadoopeco/hadoop3/etc/hadoop/ dokcho@dokcho-worker-01:/hadoopeco/hadoop3/etc/hadoop/
scp -r /hadoopeco/hadoop3/etc/hadoop/ dokcho@dokcho-worker-02:/hadoopeco/hadoop3/etc/hadoop/
scp -r /hadoopeco/hadoop3/etc/hadoop/ dokcho@dokcho-worker-03:/hadoopeco/hadoop3/etc/hadoop/
scp -r /hadoopeco/hadoop3/etc/hadoop/ dokcho@dokcho-worker-04:/hadoopeco/hadoop3/etc/hadoop/
scp -r /hadoopeco/hadoop3/etc/hadoop/ dokcho@dokcho-worker-05:/hadoopeco/hadoop3/etc/hadoop/
scp -r /hadoopeco/hadoop3/etc/hadoop/ dokcho@dokcho-worker-06:/hadoopeco/hadoop3/etc/hadoop/
scp -r /hadoopeco/hadoop3/etc/hadoop/ dokcho@dokcho-worker-07:/hadoopeco/hadoop3/etc/hadoop/
scp -r /hadoopeco/hadoop3 dokcho@dokcho-worker-01:/hadoopeco/
scp -r /hadoopeco/hadoop3 dokcho@dokcho-worker-02:/hadoopeco/
scp -r /hadoopeco/hadoop3 dokcho@dokcho-worker-03:/hadoopeco/
scp -r /hadoopeco/hadoop3 dokcho@dokcho-worker-04:/hadoopeco/
scp -r /hadoopeco/hadoop3 dokcho@dokcho-worker-05:/hadoopeco/
scp -r /hadoopeco/hadoop3 dokcho@dokcho-worker-06:/hadoopeco/
scp -r /hadoopeco/hadoop3 dokcho@dokcho-worker-07:/hadoopeco/
scp -r ~/.profile dokcho@dokcho-worker-01:~/
scp -r ~/.profile dokcho@dokcho-worker-02:~/
scp -r ~/.profile dokcho@dokcho-worker-03:~/
scp -r ~/.profile dokcho@dokcho-worker-04:~/
scp -r ~/.profile dokcho@dokcho-worker-05:~/
scp -r ~/.profile dokcho@dokcho-worker-06:~/
scp -r ~/.profile dokcho@dokcho-worker-07:~/

```





```bash
# jdk 복사
$ scp -r /hadoopeco/jdk8 dokcho@dokcho-worker-01:/hadoopeco/
$ scp -r /kikang/jdk8 spark@spark-worker-02:/kikang/
$ scp -r /kikang/jdk8 spark@spark-worker-03:/kikang/

# hadoop 복사
$ scp -r /kikang/hadoop3 spark@spark-worker-01:/kikang/
$ scp -r /kikang/hadoop3 spark@spark-worker-02:/kikang/
$ scp -r /kikang/hadoop3 spark@spark-worker-03:/kikang/

# path를 위한 .profile 복사
$ scp -r ~/.profile spark@spark-worker-01:~/
$ scp -r ~/.profile spark@spark-worker-02:~/
$ scp -r ~/.profile spark@spark-worker-03:~/
```



### 2. HDFS 시작, 종료

#### HDFS 포맷 (최초 1회)

```bash
$ /eco/hadoop3/bin/hdfs namenode -format
```



#### HDFS 시작

```bash
$ /eco/hadoop3/sbin/start-dfs.sh
```

![image-20220903231236105](C:\Users\SSAFY\Desktop\assets\image-20220903231236105.png)



웹 UI 확인(방화벽 Open 필요)

* http://spark-master-01:9870



만약 WebUI에서 DataNode가 보이지 않는다면? (필요시 방화벽 오픈)

* Name Node와 Data Node의 통신이 불안정 함을 의미 (서버간 통신)

* Data Node 로그 확인(on spark-worker-01~03)

  ```bash
  $ tail -f /hadoopeco/hadoop3/logs/hadoop-spark-dokcho-dokcho-worker-01.log
  ```

  * retry 중이면 방화벽 열면 성공적으로 connect 된다.





### 3. HDFS 테스트 

> HDFS WebUI 사용 or CLI 사용



#### 디렉토리 리스트

* HDFS 웹 UI > Utilities > Browse the File System

#### 디렉토리 만들기

* Create Directory > Write File Name > CREATE

  ![image-20220903235116558](C:\Users\SSAFY\Desktop\assets\image-20220903235116558.png)

  * dr.who 계정으로 되어 있음 -> spark 계정으로 바꾸어야 함

  * core-site.xml에 static user 추가

    ```xml
    <property>
    	<name>hadoop.http.staticuser.user</name>
        <value>dokcho</value>
    </property>
    ```

  *  모든 worker (1, 2, 3)에 다 보내주어야 함

    ```bash
    scp /hadoopeco/hadoop3/etc/hadoop/core-site.xml dokcho-worker-01:/hadoopeco/hadoop3/etc/hadoop/
    scp /hadoopeco/hadoop3/etc/hadoop/core-site.xml dokcho-worker-02:/hadoopeco/hadoop3/etc/hadoop/
    scp /hadoopeco/hadoop3/etc/hadoop/core-site.xml dokcho-worker-03:/hadoopeco/hadoop3/etc/hadoop/
    scp /hadoopeco/hadoop3/etc/hadoop/core-site.xml dokcho-worker-04:/hadoopeco/hadoop3/etc/hadoop/
    scp /hadoopeco/hadoop3/etc/hadoop/core-site.xml dokcho-worker-05:/hadoopeco/hadoop3/etc/hadoop/
    scp /hadoopeco/hadoop3/etc/hadoop/core-site.xml dokcho-worker-06:/hadoopeco/hadoop3/etc/hadoop/
    scp /hadoopeco/hadoop3/etc/hadoop/core-site.xml dokcho-worker-07:/hadoopeco/hadoop3/etc/hadoop/
    
    
    
    ```

* hdfs 재기동

  ```bash
  # dfs 중지
  $ /hadoopeco/hadoop3/sbin/stop-dfs.sh
  
  # dfs 기동
  $ /kikang/hadoop3/sbin/start-dfs.sh
  ```

  ​	![image-20220903235910047](C:\Users\SSAFY\Desktop\assets\image-20220903235910047.png)

  

* 이미지 업로드
  * 9864 port 열어야 worker들과 local이 통신할 수 있음



#### in CLI

```bash
$ hdfs dfs -ls
$ hdfs dfs -mkdir -p /test/cli
$ hdfs dfs -put -f {local_path} {hdfs_path} # -f 덮어쓰기
$ hdfs dfs -put -f /kikang/jdk8 /test/cli 

$ hdfs dfs -get /test/cli /kikang/
$ hdfs dfs -cat /test/cli/jdk8/THIRD_PARTY_README
$ hdfs dfs -head /test/cli/jdk8/THIRD_PARTY_README
$ hdfs dfs -tail /test/cli/jdk8/THIRD_PARTY_README

$ hdfs dfs -mv {path} {new_path} 
$ hdfs dfs -rm {path}

```



### 4. Data 다운로드 및 PUT

#### Data download

```bash
# download data from data source
wget https://dataverse.harvard.edu/api/access/datafile/:persistentId?persistentId=doi:10.7910/DVN/HG7NV7/RUGDRW -O /kikang/data/aot/1997.csv.bz2
wget https://dataverse.harvard.edu/api/access/datafile/:persistentId?persistentId=doi:10.7910/DVN/HG7NV7/H07RX8 -O /kikang/data/aot/1998.csv.bz2
wget https://dataverse.harvard.edu/api/access/datafile/:persistentId?persistentId=doi:10.7910/DVN/HG7NV7/IP6BL3 -O /kikang/data/aot/1999.csv.bz2
wget https://dataverse.harvard.edu/api/access/datafile/:persistentId?persistentId=doi:10.7910/DVN/HG7NV7/YGU3TD -O /kikang/data/aot/2000.csv.bz2
wget https://dataverse.harvard.edu/api/access/datafile/:persistentId?persistentId=doi:10.7910/DVN/HG7NV7/CI5CEM -O /kikang/data/aot/2001.csv.bz2
wget https://dataverse.harvard.edu/api/access/datafile/:persistentId?persistentId=doi:10.7910/DVN/HG7NV7/OWJXH3 -O /kikang/data/aot/2002.csv.bz2
wget https://dataverse.harvard.edu/api/access/datafile/:persistentId?persistentId=doi:10.7910/DVN/HG7NV7/KM2QOA -O /kikang/data/aot/2003.csv.bz2
wget https://dataverse.harvard.edu/api/access/datafile/:persistentId?persistentId=doi:10.7910/DVN/HG7NV7/CCAZGT -O /kikang/data/aot/2004.csv.bz2
wget https://dataverse.harvard.edu/api/access/datafile/:persistentId?persistentId=doi:10.7910/DVN/HG7NV7/JTFT25 -O /kikang/data/aot/2005.csv.bz2
wget https://dataverse.harvard.edu/api/access/datafile/:persistentId?persistentId=doi:10.7910/DVN/HG7NV7/EPIFFT -O /kikang/data/aot/2006.csv.bz2
wget https://dataverse.harvard.edu/api/access/datafile/:persistentId?persistentId=doi:10.7910/DVN/HG7NV7/2BHLWK -O /kikang/data/aot/2007.csv.bz2
wget https://dataverse.harvard.edu/api/access/datafile/:persistentId?persistentId=doi:10.7910/DVN/HG7NV7/EIR0RA -O /kikang/data/aot/2008.csv.bz2
```



```bash
# unzip data
bzip2 -d /kikang/data/aot/1997.csv.bz2
bzip2 -d /kikang/data/aot/1998.csv.bz2
bzip2 -d /kikang/data/aot/1999.csv.bz2
bzip2 -d /kikang/data/aot/2000.csv.bz2
bzip2 -d /kikang/data/aot/2001.csv.bz2
bzip2 -d /kikang/data/aot/2002.csv.bz2
bzip2 -d /kikang/data/aot/2003.csv.bz2
bzip2 -d /kikang/data/aot/2004.csv.bz2
bzip2 -d /kikang/data/aot/2005.csv.bz2
bzip2 -d /kikang/data/aot/2006.csv.bz2
bzip2 -d /kikang/data/aot/2007.csv.bz2
bzip2 -d /kikang/data/aot/2008.csv.bz2
```



```bash
# 확인
$ ls -alh /kikang/data/airline_on_time
$ du -sh /kikang/data/airline_on_time
```



#### Put data on HDFS

```bash
# 디렉토리 생성
$ /kikang/hadoop3/bin/hdfs dfs -mkdir -p /kikang

# put
$ hdfs dfs -put -f /kikang/data /kikang/

# HDFS 확인
# 원본 6.4 GB, Replica 19.3 GB => Repl Factor = 3
$ hdfs dfs -ls -h /kikang/data/airline_on_time
$ hdfs dfs -du -s -h /kikang/data/airline_on_time

```



### 5. Yarn 시작, 종료

```bash
# yarn 시작
$ yarn-start.sh

# Resource Manager WebUI (포트 열기)
# 서버들의 CPU, Memory 관리
http://spark-master-01:8188/

# Node Manager WebUI (포트 열기)
```



### 6. Yarn 테스트

```bash
# yarn을 이용하여 스파크 쉘 실행
$ spark-shell --master yarn

# Exception in thread "main" org.apache.spark.SparkException: When running with master 'yarn' either HADOOP_CONF_DIR or YARN_CONF_DIR must be set in the environment.
HADOOP_CONF_DIR과 YARN_CONF_DIR을 잡아줘야 한다.

# /kikang/spark3/conf2 생성
$ mkdir -p /kikang/spark3/conf2
$ vi /kikang/spark3/conf2/core-site.xml
$ vi /kikang/spark3/conf2/yarn-site.xml

$ YARN_CONF_DIR=/kikang/spark3/conf2 ./bin/spark-shell --master yarn
```



> core-site.xml 설정

```xml
<configuration>
	<property>
        <name>fs.defaultFS</name>
        <value>hdfs://spark-master-01:9000/</value>
    </property>
</configuration>
```



> yarn-site.xml 설정

```xml
<configuration>
	<property>
        <name>yarn.resourcemanager.hostname</name>
        <value>spark-master-01</value>
    </property>
</configuration>
```



```bash
$ YARN_CONF_DIR=/kikang/spark3/conf2 ./bin/spark-shell --master yarn
```

##### Free Tier에서는 불가능!

* yarn의 default spark executor container (1024MB + 512 overhead)가 Free Tier의 기본
* the default one could be too small to launch a default spark executor container ( 1024MB + 512 overhead).



4040 => 8088로 redirect

8188으로 port 번호를 바꾸었으므로 default값을 바꾸어줘야 함

> conf2 > yarn-site.xml

```xml
<property>
	<name>yarn.resourcemanager.webapp.address</name>
    <value>spark-master-01:8188</value>
</property>
```

