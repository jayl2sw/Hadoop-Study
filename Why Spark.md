# Why Spark?

## 과제 

### 과제 1. 2008년 비행기록 데이터에서 월(month)별 비행 횟수, 



```python
import pandas as pd
import time
df = pd.read_csv("/kikang/data/airline_on_time/2008.csv")
df.groupby([df.Year, df.Month]).agg({'FlightNum': 'size', 'ArrDelay':{'sum', 'mean', 'max', 'min'}})
elapsed_time = time.time() - start_time
print(time.strftime("%H:%M:%S", time.gmttime(elapsed_time)))
```

* pandasql으로는 메모리 부족으로 killed

* PySpark 실습 (DataFrame API)  --> 처음 초기화 오버헤드 존재

  ```python
  import pyspark.sql.functions as fn
  import time
  
  start_time = time.time()
  df = spark.read.option("header", True).option("inferSchema", False).csv("/kikang/data/airline_on_time/2008.csv")
  df.groupBy(df.Year, df.Month).agg(fn.count(df.FlightNum), fn.sum(df.ArrDelay), fn.avg(df.ArrDelay), fn.max(df.ArrDelay.cast("double")), fn.min(df.ArrDelay.cast("double"))).orderBy(df.Year.cast("integer").asc(), df.Month.cast("integer").asc()).toDF("Year", "Month", "FlightNum_Cnt", "ArrDelay_Sum", "ArrDelay_Avg", "ArrDelay_Max", "ArrDelay_Min").show(1000)
  elapsed_time = time.time() - start_time
  print(time.strftime("%H:%M:%S", time.gmttime(elapsed_time)))
  ```

  

* PySpark 실습2 (Spark SQL)

  ```python
  import time
  
  ```



### 과제 2. 모든 년도의 비행기록 데이터에서 집계

```python
import pyspark.sql.functions as fn
import time

start_time = time.time()
df = spark.read.option("header", True).option("inferSchema", False).csv("/kikang/data/airline_on_time/*.csv")
df.groupBy(df.Year, df.Month).agg(fn.count(df.FlightNum), fn.sum(df.ArrDelay), fn.avg(df.ArrDelay), fn.max(df.ArrDelay.cast("double")), fn.min(df.ArrDelay.cast("double"))).orderBy(df.Year.cast("integer").asc(), df.Month.cast("integer").asc()).toDF("Year", "Month", "FlightNum_Cnt", "ArrDelay_Sum", "ArrDelay_Avg", "ArrDelay_Max", "ArrDelay_Min").show(1000)
elapsed_time = time.time() - start_time
print(time.strftime("%H:%M:%S", time.gmttime(elapsed_time)))
```



* Spark는 분석하고자 하는 모든 데이터를 메모리에 올리지 않는다. 

* 데이터 건 바이 건으로 메모리에 올리기 때문에 그정도의 메모리만 있으면 연산이 가능하다.
* 속도가 느리면 CPU를 더 많이 할당하면 한번에 더 많은 연산이 동시에 가능하다.





## 스칼라 다운로드

```bash
$ wget https://downloads.lightbend.com/scala/2.12.15/scala-2.12.15.tgz -O /kikang/scala-2.12.15.tgz
$ tar xvfz /kikang/scala-2.12.15.tgz
```

> ~/.profile  path 등록

```
export SCALA_HOME=/kikang/scala-2.12.15
export PATH=$SCALA_HOME/bin:$PATH	
```



## 과제

### 과제 1. README.md 파일 내 단어별 개수를 구하시오

```bash
$ mkdir -p /kikang/projects/spark/src/lab/core/wordcount
$ vi /kikang/projects/spark/src/lab/core/wordcount/WordCountSpark.scala
```

 

> /kikang/projects/spark/src/lab/core/wordcount/WordCountSpark.scala

```scala
package lab.core.wordcount

import scala.io.Source
import java.io.PrintWriter

object WordCountScala {
  def main(args: Array[String]): Unit = {
    println(">>>> WordCountScala...")

    var input = "/kikang/spark3/README.md"
    var output = "/kikang/spark3/README.wordcount_scala"
    var delimiter = " "

    val ar = Source.fromFile(input).getLines.toArray
    println(">>>> Line Count : " + arr.count(line => true))

    val arr2 = arr.flatMap(line => line.split(delimiter))

    val arr3 = arr2.map(word => (word, 1))

    val arr4 = arr3.groupBy(tuple => tuple._1).toArray

    val arr5 = arr4.map(tuple_grouped => (tuple_grouped._1, tuple_grouped._2.map(tuple => tuple._2)))

    val arr6 = arr5.map(tuple_grouped => (tuple_grouped._1, tuple_grouped._2.reduce((v1, v2) => v1 + v2)))

    arr6.foreach(word_count => println(">>>>> word count : " + word_count))

    // --개수 내림차순 정렬
    val arr7 = arr6.sortBy(tuple => -tuple._2)

    arr7.foreach(word_count => println(">>>>> word count (count desc) : " + word_count))

    // --정렬 기준 정의
    val ordering = new Ordering[(String, Int)] {
      override def compare(a: (String, Int), b: (String, Int)) = {
        if ((a._2 compare b._2) == 0) (a._1 compare b._1) else -(a._2 compare b._2)
      }
    }
    // --개수 내림차순, 단어 오름차순
    val arr8 = arr6.sortBy(tuple => tuple)(ordering)

    arr8.foreach(word_count => println(">>>>> word count (count desc, word asc) : " + word_count))

    new PrintWriter(output) { write(arr8.mkString("\n")); close }

  }
}
```



>  컴파일

```bash
$ mkdir -p /kikang/projects/spark/target
$ cd /kikang/projects/spark
$ scalac -d target/ src/lab/core/wordcount/WordCountScala.scala
```



> Jar archieve

```bash
$ cd /kikang/projects/spark/target
$ jar cvf ../wordcount.jar ./lab/core/wordcount/*
```



> 실행

```bash
$ scala -classpath "/kikang/projects/spark/wordcount.jar" lab.core.wordcount.WordCountScala
$ cat /kikang/spark3/README.wordcount_scala | more
```





> /kikang/projects/spark/src/lab/core/wordcount/WordCountSpark.scala

```scala
package lab.core.wordcount

import org.apache.spark.SparkContext

object WordCountSpark {

  def main(args: Array[String]): Unit = {
    println(">>>>>>> WordCountSpark.....")

    var input = "/kikang/spark3/README.md"
    var output = "/kikang/spark3/README.wordcount_spark"
    var delimiter = " "

    var sc = new SparkContext()

    val rdd = sc.textFile(input)
    println(">>>>>>> Line Count : " + rdd.count())

    val rdd2 = rdd.flatMap(line => line.split(delimiter))

    val rdd3 = rdd2.map(word => (word, 1))

    val rdd4 = rdd3.groupBy(tuple => tuple._1)

    val rdd5 = rdd4.map(tuple_grouped => (tuple_grouped._1, tuple_grouped._2.map(tuple => tuple._2)))

    val rdd6 = rdd5.map(tuple_grouped => (tuple_grouped._1, tuple_grouped._2.reduce((v1, v2) => v1 + v2)))

    rdd6.foreach(word_count => println(">>>>> word count : " + word_count))
	
    // --개수 내림차순 정렬
    val rdd7 = rdd6.sortBy(tuple => -tuple._2)

    rdd7.foreach(word_count => println(">>>>> word count (count desc) : " + word_count))
	
    // --정렬 기준 정의
    val ordering = new Ordering[(String, Int)] {
      override def compare(a: (String, Int), b: (String, Int)) = {
        if ((a._2 compare b._2) == 0) (a._1 compare b._1) else -(a._2 compare b._2)
      }
    }
	// --개수 내림차순, 단어 오름차순
    val rdd8 = arr6.sortBy(tuple => tuple)(ordering, implicitly[scala.reflect.ClassTag[(String, Int)]])

    rdd8.foreach(word_count => println(">>>>> word count (count desc, word asc) : " + word_count))

    rdd8.saveAsTextFile(output)

    sc.stop()
  }
}
```



> 컴파일

```bash
$ scalac -classpath "kikang/spark3/jars/spark-core_2.12-3.2.1.jar" -d target/ src/lab/core/wordcount/WordCountSpark.scala
```



> jar archieve

```bash
$ cd /kikang/projects/spark/target
$ jar cvf ../wordcount.jar ./lab/core/wordcount/*
```



> 실행

```bash
$ /kikang/spark3/bin/spark-submit --class lab.core.wordcount/WordCountSpark /kikang/projects/spark/wordcount.jar # on local
$ cat /kikang/spark3/README.wordcount_spark/part-* | more
$ ls -al /kikang/spark3/README.wordcount_spark/
```

