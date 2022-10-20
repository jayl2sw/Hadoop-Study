# Apache Spark 기본 실습

## 개요

1. RDD 생성
2. Lazy Evaluation
3. Lineage 확인, Paritition 확인
4. Shuffle by Repartition
5. Partitions for Files
6. Cache
7. Cache Size(Int vs String vs Object)
8. Get Cached RDD by name
9. Word Count



## 1. RDD 생성

Scala용 Spark-Shell 실행, 프로세스 확인,  웹 UI 확인

```bash
$ cd /kikang/spark3
$ ./bin/spark-shell
```

```scala
sc
spark
sc.master
sc.defaultMinPartitions
sc.uiWebUrl
```

### 실습1

드라이버 프로그램 내 데이터를 이용하여 RDD 생성 및 필터링

```scala
val data = 1 to 10000
val distData = sc.parallelize(data) // RDD로 변함
distData.count()
distData.getNumPartitions
val distData2 = distData.filter(_ < 10) // _ 는 레코드 한건 한건
distData2.collect()
distData2.getNumPartitions
```



### 실습2

드라이버 프로그램 내 데이터를 이용하여 RDD 생성 및 필터링 (with rdd.union)

```scala
val distData = sc.parallelize(1 to 10000)
val distData2 = sc.parallelize(10001 to 20000)
val distData3 = sc.parallelize(20001 to 30000)
val distData4 = sc.parallelize(30001 to 40000)
val distData5 = sc.parallelize(40001 to 50000)
val distData6 = distData.union(distData2).union(distData3).union(distData4).union(distData5)
distData6.count()
distData6.getNumPartitions
val distData7 = distData6.filter(_ < 10)
distData7.collect()
distData7.getNumPartitions
```



## 2. Lazy Evaluation

### 실습1

외부 데이터(파일)를 이용하여 RDD 생성 및 Lazy Evaluation 확인 (Job 실행이 Lazy하게 일어남)

```scala
val data = sc.textFile("README.txt") // 없는 파일이어도 아직 실행되지 않았기 때문에 에러가 발생하지 안음
val distData = data.map(r => r + "_map")
distData.count // action이 발생했음, 테스트 파일이 없으므로 에러 발생 

val data = sc.textFile("README.md")
val distData = data.map(r => r + "_map")
distData.count
```

* spark job은 Action이 일어날때 실행된다.

### 실습2

```scala
val rdd_wc = sc.textFile("README.txt").flatMap(_.split(" ")).map((_, 1)).reduceByKey(_ + _) // reduceByKey는 셔플이 일어남
// 파티션의 크기를 어떻게 정할지 정하기 위해 파일을 확인함
// job이 실행되서가 아닌 파일을 확인 해서 에러 발생
val rdd_wc = sc.textFile("README.txt").flatMap(_.split(" ")).map((_, 1)).groupByKey() // 없는 파일이기 때문에 에러 발생

val rdd_wc = sc.textFile("README.md").flatMap(_.split(" ")).map((_, 1)).reduceByKey(_ + _) 
// 있는 파일이더라도 job은 실행되지 않음
rdd_wc.collect.take(10).foreach(println)
```



## 3. Lineage 확인, Partition 개수 확인

> ㄴㅁㅇ

```scala
val rdd = sc.textFile("README.md", 10)  // 병렬화 수준을 설정할 수 있다. 10개의 스레드 사용
val rdd_wc = rdd.flatMap(_.split(" ")).map((_, 1)).reduceByKey(_ + _, 5) // 의도적으로 파티션 개수 지정 가능
rdd_wc.count
rdd.toDebugString
rdd_wc.toDebugString
rdd.getNumPartitions
rdd_wc.getNumPartitions
```

![image-20220906223528662](C:\Users\SSAFY\Desktop\assets\image-20220906223528662.png)

* map, flatmap의 경우 같은 스테이지다. (partition의 변화가 없다.)
  * 들여쓰기가 같으면 partition의 변화가 없다.

* shuffle 할 때, 스테이지가 나뉜다.



## 4. Shuffle by Repartition

### 실습1

> ss

```scala
val data = sc.textFile("README.md")
data.getNumPartitions
val distData = data.map(r => r + "_map")
distData.getNumPartitions
val newData = distData.repartition(10) // 파티셔닝을 다시함 => data의 value값의 hash값에 따라 정렬 
newData.getNumPartitions // 파티션 개수 변경됨 => 비어있는 파티션이라도 파티션 채움
newData.toDebugString
newData.count //action 수행

newData.collect.foreach(println) // action 드라이브로 가져와서 print 

data.saveAsTextFile("dataREADME.md") // 읽자마자 만든 RDD를 TextFile로 만듬 (안섞임)

newData.saveAsTextFile("newData") // 섞여 있음
```

![image-20220906224703587](C:\Users\SSAFY\Desktop\assets\image-20220906224703587.png)

![image-20220906225049384](C:\Users\SSAFY\Desktop\assets\image-20220906225049384.png)

* DAG의 동그라미가 RDD이다. 	



![image-20220906230014874](C:\Users\SSAFY\Desktop\assets\image-20220906230014874.png)

* 동일한 RDD는 앞쪽 스테이지는 이미 해놨으니까 skip한다.

  * spark는 항상 변하지 않는 데이터를 처리한다고 생각하고 있기 때문에 스킵이 가능하다.

    

### 실습2 (숫자)

```scala
val intRDD2 = sc.parallelize(1 to 20)
intRDD2.saveAsTextFile("intRDD2")

val intRDD5 = intRDD.repartition(5) // 데이터를 섞는다.
intRDD5.saveAsTextFile("intRDD5") 
```



## 5. Partitions for Files

### 실습1

```scala
val rdd = sc.textFile("README.md")
rdd.getNumPartitions
val rdd2 = sc.textFile("NOTICE")
rdd2.getNumPartitions
val rdd3 = sc.textFile("README.md, NOTICE")
rdd3.getNumPartitions
val rdd4 = sc.textFile("LI*,NO*,RE*")
rdd.saveAsTextFile("rdd")
rdd.saveAsTextFile("rdd2")
rdd.saveAsTextFile("rdd3")
rdd.saveAsTextFile("rdd4")
```

* Task = Partition 개수 확인
* rdd4 => partition별 데이터 unbalance, 특정 partition으로 데이터가 치우칠 수 있음



### 실습2

```scala
val rdd = sc.parallelize(1 to 1000)
rdd.getNumPartitions
val rdd2 = sc.parallelize(1 to 10)
rdd2.getNumPartitions
val rdd3 = sc.parallelize(1 to 10000)
rdd3.getNumPartitions
val rdd4 = sc.parallelize(1 to 2)
rdd4.getNumPartitions
val rdd5 = rdd.union(rdd2).union(rdd2).union(rdd3).union(rdd4)  
rdd5.getNumPartitions  // union 하면 rdd의 partition 개수가 다 더해진다.

val rdd6 = rdd5.map(x => { Thread.sleep(1); x*x }) // 그냥 로직이 복잡하다고 생각하고 Thread.sleep1 줌
rdd6.foreachPartition(iterator => { println(s">>>>partition index: ${org.apache.spark.TaskContext.get.partitionId} partition data size: ${iterator.size}....")})
```

![image-20220907003225669](C:\Users\SSAFY\Desktop\assets\image-20220907003225669.png)

* WebUI 확인

  ![image-20220907003328797](C:\Users\SSAFY\Desktop\assets\image-20220907003328797.png)

  ![image-20220907003422601](C:\Users\SSAFY\Desktop\assets\image-20220907003422601.png)



![image-20220907003553967](C:\Users\SSAFY\Desktop\assets\image-20220907003553967.png)

* unbalance 하면 어느 태스크는 먼저 끝나고 어느 태스크는 늦게 끝날 수 있다.

  

```scala
val rdd7 = rdd5.repartition(8)
rdd7.getNumPartitions

val rdd8 = rdd7.map(x => { Thread.sleep(1); x*x })
rdd8.foreachPartition(iterator => { println(s">>>>partition index: ${org.apache.spark.TaskContext.get.partitionId} partition data size: ${iterator.size}....")})
// 웹 UI 확인 ... RDD repartition으로 데이터 치우침 해소... But, 한번에 실행할 수 있는 task(1개) 한계로 별 효과 없음
```

![image-20220907004036546](C:\Users\SSAFY\Desktop\assets\image-20220907004036546.png)



### 실습3 고의적 unbalance 생성

```scala
val rddd = sc.parallelize(1 to 10000, 1)
rddd.getNumPartitions
val rddd2 = sc.parallelize(1 to 10, 1)
rddd2.getNumPartitions
val rddd3 = rddd.union(rddd2) // union은 데이터를 섞지 않고 partition을 그대로 붙이기만 함 
rddd.getNumPartitions
val rddd4 = rddd3.map(x => { Thread.sleep(1); x*x })
rddd4.foreachPartition(iterator => { println(s">>>>partition index: ${org.apache.spark.TaskContext.get.partitionId} partition data size: ${iterator.size}....")})

```

* 10개짜리가 끝나고 10000개짜리가 끝날 때 까지 기다려야함

* 이런 경우 repartition을 진행함

```scala
val rddd5 = rddd3.repartition(2)
rddd5.getNumPartitions

val rddd6 = rddd5.map(x => { Thread.sleep(1); x*x })
rddd6.foreachPartition(iterator => { println(s">>>>partition index: ${org.apache.spark.TaskContext.get.partitionId} partition data size: ${iterator.size}....")})
// 2-core에서는 두개가 동시에 진행 됨 => 실행속도 1/2됨
// 해당 함수를 두번 실행하게 되면 parallelize와 union stage는  skip
```





## 6. Cache

### 실습1

```scala
val data = sc.textFile("README.md")
val distData = data.map(r => r + "_map")
distData.name = "distData" // 캐시된 데이터가 많으면 구별하기 어렵기 때문에 이름을 달아줌
distData.cache // 캐시가 이루어지지 않음 (액션이 실행될 때 캐시 됨)
distData.take(5) // 부분만 처리하는 로직이기 때문에 100% 모든 데이터가 cache 되지 않는다.
```

* Storage 탭에서 Fraction Cached 부분이 100%가 아님을 확인
  * Task도 1개만 실행되었음

```scala
distData.collect
```

* Storage탭에서 Fraction Cached 부분이 100%
  * Task도 모두 실행되었음

```scala
distData.getStorageLevel // 캐시 스토리지 레벨 확인  StorageLevel(memory, deserialized, 1 replicas) 
// memory에 직렬화 되지 않은 상태로 복사본 없이 저장되어 있음
distData.unpersist() // 캐시 삭제
```



## 7. Cache Size (Type, Serialize)

* 동일한 데이터를 타입만 다르게 캐시데이터로 저장할 때 크기 비교
* 직렬화 되었을 때와 안되었을 때, 비교
  * 튜닝 포인트가 될 수 있다.

### 실습 1. Int vs String vs Class

```scala
val intRdd = sc.parallelize(1 to 10000)
intRdd.name = "intRdd"
intRdd.cache
intRdd.count
val strRdd = intRdd.map(_.toString)
strRdd.name = "strRdd"
strRdd.cache
strRdd.count
case class StrCase (str:String)
val strCaseRdd = intRdd.map(x => StrCase(x.toString))
strCaseRdd.name="strCaseRdd"
strCaseRdd.cache
strCaseRdd.count
```

![image-20220907180128117](C:\Users\SSAFY\Desktop\assets\image-20220907180128117.png)

* Object (Header 16 bytes), String (Overhead 40 bytes, 2bytes/char)

```scala
intRdd.unpersist() // 캐시 삭제
strRdd.unpersist()
strCaseRdd.unpersist()
```



### 실습 2. Serialized vs Deserialized

```scala
// Deserialized
val data = (1 to 10000).map(_.toString)
val strRdd = sc.parallelize(data)
strRdd.name = "strRdd"
strRdd.cache 
strRdd.count

// 같은 Data로 구성된 아이디
val strRdd2 = sc.makeRDD(data)
strRdd2.setName("strRdd2")
strRdd2.persist(org.apache.spark.storage.StorageLevel.MEMORY_ONLY)
strRdd2.persist(org.apache.spark.storage.StorageLevel.MEMORY_ONLY_SER) // serialized 에러 발생
strRdd2.unpersist() // 취소하고 다시 해야함

strRdd2.persist(org.apache.spark.storage.StorageLevel.MEMORY_ONLY_SER)
strRdd2.count
```

![image-20220908003005030](C:\Users\SSAFY\Desktop\assets\image-20220908003005030.png)

* Serialize 하면 메모리 사용이 최소화 된다. 

* Serialized Data는 메모리 사용은 적지만 처리하는데 CPU가 많이 사용된다.
  * 어차피 처리를 위해서는 Deserialize 해야함
  * CPU를 많이 쓰더라도 죽지 않지만 메모리는 많이 사용하면 죽음



## 8. Get Cached RDD by Name

```scala
val data = sc.textFile("./README.md")
val distData = data.map(r => r + "_map")
data.setName("data")
distData.name = "distData"
data.cache
distData.cache
sc.getPersistentRDDs // rdd_id -> actual rdd
sc.getPersistentRDDs.foreach(println)

distData.collect

// data라는 이름을 가지는 rdd unpersist x._1 = rdd_id,  x._2 = actual_rdd
sc.getPersistentRDDs.filter(x => x._2.name.equals("data")).foreach(x => x._2.unpersist()) 
sc.getPersistentRDDs.foreach(x => x._2.unpersist()) // 모든 캐시 제거

distData.collect // 캐시가 제거되었기 때문에 스토리지에 생성 x
```



## 9. WordCount

