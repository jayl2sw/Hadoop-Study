# Spark 클러스터 구성

## 개요

1. 다운로드, 설정
2. Standalone 시작/종료
3. Standalone 테스트

### Standalone이란?

Spark에서 제공하는 Cluster Manager (YARN과 비슷)

Standalone에서는 Spark만 구동가능하다 (Map Reduce와 같은 다른 분산 병렬 어플리케이션 구동 불가능)



## 1. 다운로드, 설정 

기존에 실습 때문에 다운로드 되어 있음 > spark3, jdk8 다운로드 

### workers 설정

```bash
$ cp /kikang/spark3/conf/workers.template /kikang/spark3/conf/workers
$ vi /kikang/spark3/conf/workers
```



> /kikang/spark3/conf/workers

```b
spark-worker-01
spark-worker-02
spark-worker-03
```



### spark3 전송

master의 spark3 디렉토리를 worker 01 ~ 03에 전송

```bash
$ scp -r /kikang/spark3 spark@spark-worker-01:/kikang
$ scp -r /kikang/spark3 spark@spark-worker-02:/kikang
$ scp -r /kikang/spark3 spark@spark-worker-03:/kikang
```



## 2. Standalone 시작/종료

### Standalone 시작/종료

```bash
# Standalone 시작
$ /kikang/spark3/sbin/start-all.sh

$ jps
Master

# Standalone 종료
$ /kikang/spark3/sbin/stop-all.sh
```

#### 주의할 점! 

```bash
# hadoop에서의 start-all은 dfs와 yarn을 모두 구동시키는 것이다.
$ /kikang/hadoop3/sbin/start-all.sh
```

- path에 hadoop3의 sbin을 등록해 두었기 때문에 Standalone 시작을 위해선 spark3의 sbin을 사용하여야 한다. 



### spark-env.sh 설정

* hostname, port, cores, memory 설정

> /kikang/spark3/conf/spark-env.sh
>
> Master에는 Master 설정만, Worker에는 Worker 설정만 있어도 된다.

```sh
SPARK_MASTER_PORT=7177			# default: 7077
SPARK_MASTER_WEBUI_PORT=8180	# default: 8080
SPARK_WORKER_PORT=7178			# default: random
SPARK_WORKER_WEBUI_PORT=8181 	# default: 8081
SPARK_WORKER_CORES=8			# default: all available 일부러 뻥튀기
SPARK_WORKER_MEMORY=8G			# default: machine's total RAM minus GiB
SPARK_PUBLCI_DNS=${HOSTNAME}	
```



* worker들한테 spark-env.sh 전송

```bash
$ scp -r /kikang/spark3/conf/spark-env.sh spark@spark-worker-01:/kikang/spark3/conf
$ scp -r /kikang/spark3/conf/spark-env.sh spark@spark-worker-02:/kikang/spark3/conf
$ scp -r /kikang/spark3/conf/spark-env.sh spark@spark-worker-03:/kikang/spark3/conf


scp -r /hadoopeco/spark3/conf/spark-env.sh dokcho@dokcho-worker-01:/hadoopeco/spark3/conf
scp -r /hadoopeco/spark3/conf/spark-env.sh dokcho@dokcho-worker-02:/hadoopeco/spark3/conf
scp -r /hadoopeco/spark3/conf/spark-env.sh dokcho@dokcho-worker-03:/hadoopeco/spark3/conf
scp -r /hadoopeco/spark3/conf/spark-env.sh dokcho@dokcho-worker-04:/hadoopeco/spark3/conf
scp -r /hadoopeco/spark3/conf/spark-env.sh dokcho@dokcho-worker-05:/hadoopeco/spark3/conf
scp -r /hadoopeco/spark3/conf/spark-env.sh dokcho@dokcho-worker-06:/hadoopeco/spark3/conf
scp -r /hadoopeco/spark3/conf/spark-env.sh dokcho@dokcho-worker-07:/hadoopeco/spark3/conf


```



* 이후 추가적인 port open 필요함(8180, 8181	)

![image-20220904230142408](C:\Users\SSAFY\Desktop\assets\image-20220904230142408.png)



### Standalone 시작/종료 옵션

#### Master 

> on spark-master-01
>
> Master만 켜지고 Worker들은 켜지지 않음

```bash
$ /kikang/spark3/sbin/start-master.sh --port 7177 -webui-port 8180 --host ${HOSTNAME}
$ /kikang/spark3/sbin/stop-master.sh 
```

#### Worker

> on spark-worker-01 ~ 03
>
> 단일 Worker 하나만 켜짐

```bash
$ /kikang/spark3/sbin/start-worker.sh spark://spark-master-01:7177 --cores 8 --memory 8G --host ${HOSTNAME}
$ /kikang/spark3/sbin/stop-worker.sh 
```

* 하나하나 서버를 띄우면 나중에 master가 변했을때 그곳으로 worker들을 띄울 수 있다.

* 하나씩 끄면 worker의 status가 DEAD로 변함



## 3. Sparkalone 테스트

```bash
$ bin/spark-shell --master spark://spark-master-01:7177
```

> exectutor 확인

![image-20220905004936514](C:\Users\SSAFY\Desktop\assets\image-20220905004936514.png)

* default값으로 가용한 모든 코어를 할당

* spark-shell 이후에 다른 어플리케이션을 실행하게 되면 해당 APP을 위한 코어가 남지 않는다.

  * 따라서 설정을 통해 모든 코어를 할당하지 않도록 변경

    ```bash
    $ spark-shell --help
    # executor 메모리 설정
    --executor-memory NUM G
    # executor core 설정
    --executor-cores NUM
    
    # executor 개수 설정
    
    # on Standalone
    --total-executor-cores NUM
    
    # on YARN and Kubernetes 
    --num-excutors NUM
    
    $ ./bin/spark-shell --master spark://spark-master-01:7177 --executor-memory 2G --executor-cores 2 --total-executor-cores 12
    ```

    

  

![image-20220905005120959](C:\Users\SSAFY\Desktop\assets\image-20220905005120959.png)

* Running application에 코어는 다 주고 Memory는 1기가만 준다.



![image-20220905005324482](C:\Users\SSAFY\Desktop\assets\image-20220905005324482.png)

* Executor는 반드시 하나의 application을 위해서 동작
* 여러 Application 구동 불가능

![image-20220905010147673](C:\Users\SSAFY\Desktop\assets\image-20220905010147673.png)





![image-20220905010420799](C:\Users\SSAFY\Desktop\assets\image-20220905010420799.png)



### spark-env.sh 추가 설정

> /kikang/spark3/conf/spark-env.sh

```bash
# 특별한 요청이 없다면 Total Cores 5개를 할당하겠다.
SPARK_MASTER_OPTS="-Dspark.deploy.defaultCores=5" 
```

* 이후 모든 Worker로 보냄

#### 결과

![image-20220905011448724](C:\Users\SSAFY\Desktop\assets\image-20220905011448724.png)

```sh
# rdd를 디스크에 캐시하거나 shuffle data를 기록하기 위한 경로 지정
SPARK_LOCAL_DIRS=/kikang/spark3/local (-> default: "/tmp" 위치를 바꿔서 shuffle data, cached rdd block을 쉽게 할 수 있게 한다.)
```



#### Word count Test

```bash
# spark-shell 재실행
$ spark-shell --master spark://spark-master-01:7177
```

```scala
val rdd_wc = sc.textFile("README.md").flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_)
rdd_wc.collect.take(10).foreach(println)	
```

![image-20220905171143646](C:\Users\SSAFY\Desktop\assets\image-20220905171143646.png)

​	

```scala
// RDD Cache 되었을 때, (on disk) 확인
rdd_wc.persist(org.apache.spark.storage.StorageLevel.DISK_ONLY)
// persist의 경우 모아두었다가 Action이 발생할 때 한번에 적용
 
```

