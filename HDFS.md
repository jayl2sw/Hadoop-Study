### Hadoop File System (SW Platform)

* 빅 데이터 파일을 여러 대의 컴퓨터에 나누어서 저장함

* 각 파일은 여러개의 순차적인 블록으로 저장함

* 하나의 파일의 각각의 블록은 폴트 톨러런스(fault tolerance)를 위해서 여러 개로 복사되어 여러 머신의 여기저기 저장 됨

  * `Fault tolerance` : 시스템을 구성하는 부품의 일부에서 결함(fault)이 발생하여도 정상적 혹은 부분적으로 기능을 수행할 수 있는 것을 말함 (ex 앙상블 기법)

* 빅데이터를 수천대의 값싼 컴퓨터에 병렬 처리하기 위해 분산함

* **주요 구성 요소**

  * MapReduce - 소프트웨어의 수행을 분산함
  * Hadoop Distribtued File System(HDFS) - 데이터를 분산함

* 한 개의 Namenode(master)와 여러개의 Datanode(slaves)

  * Namenode
    * 파일시스템을 관리하고 클라이언트가 file에 접근할 수 있게 함

  * Datanode 
    * 컴퓨터에 들어있는 데이터를 접근할 수 있게 함

* 자바로 맵리듀스 알고리즘 구현





### Map Reduce (분산 처리 프레임워크)

* 데이터 중심 프로세싱
  * 한대의 컴퓨팅 파워로 처리가 어려움
  * 여러 컴퓨터를 묶어서 처리
* 빅데이터를 이용한 효율적인 계산이 가능한 첫번째 프로그래밍 모델
  * 기존에 존재하는 여러가지 다른 병렬 컴퓨팅 방법에서는 프로그래머가 낮은 레벨의 시스템 세부 내용까지 아주 잘 알고 많은 시간을 쏟아야만 함

* 함수형 프로그래밍 (Fucntional programming) 언어의 형태
* 3가지 함수를 구현해서 제공해야함
  * Main
  * Map
  * Reduce

* 각각의 레코드 or 튜플을 키 - 밸류 쌍으로 표현
* 맵 리듀스 프레임워크는 메인 함수를 한 개의 마스터 머신(master machine)에서 수행하는데 이 머신은 맵함수를 수행하기 전에 전처리를 하거나 리듀스 함수의 결과를 후처리 하는데 사용될 수 있다.
* 컴퓨팅은 맵과 리듀스라는 유저가 정의한 함수 한쌍으로 이루어진 맵리듀스 페이지를 한번 수행하거나 여러번 반복해서 수행할 수 있다.

#### MapReduce Phase

* 맵(Map) 페이즈 (머신)
  * 제일 먼저 수행되며 데이터의 여러 파티션에 병렬 분산으로 호출되어 수행
  * 각 머신마다 수행된 Mapper라는 맵 함수가 입력 데이터의 한줄마다 Map함수를 호출한다.
  * K-V 쌍 형태로 결과를 출력 => 여러 머신에 나누어 보냄
* 셔플링(Shuffling) 페이즈 (머신) 
  * 모든 머신에서 맵 페이즈가 끝나면 시작된다.
  * 맵 페이즈에서 각각의 머신으로 보내진 K-V쌍을 Key를 이용해서 정렬 후 각 Key마다 같은 Key를 가진 KV 쌍을 모아서 Value_List를 만든다음에 Key에 따라서 여러 머신에 분산해서 보낸다.
* 리듀스(Reduce) 페이즈
  * 모든 머신에서 셔플링 페이즈가 다 끝나면 각 머신마다 리듀스 페이즈가 시작된다.
  * 각각의 머신에서는 셔플링 페이즈에서 해당 머신으로 보내진 각각의 (key, value_list) 쌍에 리듀스 함수가 호출된다.



### MapReduce의 함수

* 맵 함수 
  * Mapper 클래스를 상속받아서 맵 메소드를 수정
  * 입력 텍스트 파일에서 라인 단위로 호출되고 입력은 KV 형태
  * Key는 입력 텍스트 파일에서 맨 앞문자를 기준으로 맵 함수가 호출된 해당 라인의 첫 번째 문자까지의 오프셋(offset)
  * VALUE는 텍스트의 해당 라인 전체가 들어있다.
* 리듀스 함수
  * Reducer 클래스를 상속 받아서 reduce 메소드 수정
  * 셔플링 페이즈의 출력을 입력으로 받는데 KV리스트의 형태
  * Value-List 맵 함수의 출력에서 KV쌍들의 Value 리스트
* 컴바인 함수
  * 리듀스 함수와 유사한 함수인데 각 머신에서 맵 페이즈에서 맵 함수의 출력 크기를 줄여서 셔플링 페이즈와 리듀스 페이즈의 비용을 줄여주는데 사용
  * 맵의 아웃풋을 컴바인을 통해 셔플링 페이즈에 들어가는 입력 사이즈를 줄여주는 역할 
    * 셔플링 페이즈를 통해 aggregate



### MapReduce를 이용한 Wording Count

![image-20220822115744838](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220822115744838.png)

![image-20220822115759524](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220822115759524.png)



### Overview of MapReduce

* Mapper and Reducer
  * 각 머신에서 독립적으로 수행된다.
  * Mappter는 Map함수를, Reducer는 Reduce 함수를 각각 수행한다.
* Combine functions
  * 각 머신에서 Map 함수가 끝난 다음에 Reduce 함수가 하는 일을 부분적으로 수행한다.
  * 셔플링 비용과 네트웍 트래픽 비용을 감소 시킨다.
* Mapper와 Reducer는 필요하다면 setup() and cleanup()을 수행할 수 있다.
  * setup() : 첫 Map함수나 Reducer 함수가 호출되기 전에 맨 먼저 수행
  * cleanup(): 마지막 Map 함수나 Reduce 함수가 끝나고 나면 수행한다.
* 한개의 MapReduce job을 수행할 때에 Map 페이즈만 수행하고 중단할 수 있다.



#### 실습

* 우분투 환경에서 

  ```bash
  $ wget http://kdd.snu.ac.kr/~kddlab/Project.tar.gz
  $ tar zxf Project.tar.gz
  $ sudo chown -R hadoop:hadoop Project
  $ cd Project
  $ sudo mv hadoop-3.2.2 /usr/local/hadoop
  $ sudo apt update
  $ sudo apt isnatll ssh openjdk-8-jdk-ant -y
  $ ./set_hadoop_env.sh
  $ source ~/.bashrc
  
  # password를 안치고도 돌릴 수 있게 해주는 기능
  $ ssh-keygen -t rsa -P ""
  $ cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys
  $ ssh localhost
  
  # Name node format
  # Disk format과 같은 개념
  $ hadoop namenode -format
  
  #Dfs damon startstar
  $ start-dfs.sh
  
  $ start-mapred.sh
  ```

  

![w](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220822144504896.png)

* 맵리듀스 코드 컴파일

  ```bash
  $ ant # 모든 수정이 끝나고 난뒤 해당 명령어를 실행해야 수정이 반영됨 (빌드)
  ```

* 테스트 데이터를 HDFS에 넣어야 함

  ```bash
  $ cd /home/hadoop/Project
  
  # 하둡의 HDFS에 wordcount_test 디렉토리를 생성
  $ hdfs dfs -mkdir /user
  $ hdfs dfs -mkdir /user/hadoop
  # hdfs내에 input데이터를 저장할 path 생성
  $ hdfs dfs -mkdir wordcount_test
  
  # Linux의 data 디렉토리에 있는 wordcount-data.txt 파일을 하둡의 HDFS의 wordcount_test 디렉토리에 보냄
  # ubuntu local에서 hdfs로 inputdata 전송
  $ hdfs dfs -put data/wordcount-data.txt wordcount_test
  ```

  

* 반드시 맵 리듀스 프로그램이 결과를 저장할 디렉토리를 삭제한 후 프로그램을 실행 해야함

  ```bash
  $ hdfs dfs -rm -r wordcount_test_out
  ```



* Wordcount MapReduce 알고리즘 코드 실행

  ```bash
  # Driver.java에 표시한대로 wordcount를 써서 Wordcount 맵 리듀스 코드를 수행
  # Wordcount_test 디렉토리에 들어있는 모든 파일을 맵 함수의 입력으로 사용함
  $ hadoop jar ssafy.jar wordcount wordcount_test wordcount_test_out
  
  # Hadoop의 실행방법
  $ hadoop jar [jar file] [program name] <input arguments...>
  
  # Reducer 개수를 2개 사용하면 아래와 같은 출력 파일 2개가 생성됨
  $ hdfs dfs -cat wordcount_test_out/part-r-00000 | more
  
  $ ant를 해야 빌드가 됨
  ```



#### 맵 리듀스 입출력에 사용가능한 디폴트(Default) 클래스

* 맵리듀스의 입출력에 사용하는 타입들은 셔플링 페이즈에서 정렬하는데 필요한 비교 함수 등 여러 함수가 이미 정의되어 있다. 
  * Text string
  * intWritable int
  * LongWritable long
  * FloatWritable float
  * DoubleWritable double 
* 새로운 클래스 정의시 필요한 여러 함수도 같이 정의해야 함



### Wordcount.java

![image-20220822165845299](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220822165845299.png)

![image-20220822172719384](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220822172719384.png)



#### Wordcount.java 수정한 다음 실행

* Project 디렉토리에서 ant 실행

* 수행결과 보기

  ```bash
  $ hdfs dfs -rm -r wordcount_test_out # 리듀스 함수 출력 디렉토리를 삭제
  $ hadoop jar ssafy.jar wordcount wordcount_test wordcount_test_out
  
  # reducer를 2개 사용해서 결과 파일이 두개)
  $ hdfs dfs -cat wordcount_test_out/part-r-00000 | more 
  $ hdfs dfs -cat wordcount_test_out/part-r-00001 | more 
  ```

  

![image-20220824123519755](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220824123519755.png)

* combine  함수 사용시![image-20220824123641788](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220824123641788.png)



### Partitioner Class 변경

* Map 함수의 출력인 KV 쌍이 Key에 의해서 어느 Reducer(머신)으로 보내질 것인지를 정해지는데 이러한 결ㅈ어을 정의하는 Class
* 하둡의 기본 타입은 Hash함수가 Default로 제공되고 있어서 Key에 대한 해시 값에 따라 어느 Reducer(머신)으로 보낼지를 결정한다. 
  * 잘 분산 되도록 유사 랜덤으로 설정되어 있는데 이를 바꿀 수 있다.
* 하둡의 기본 타입
  * Text
  * IntWritable
  * LongWritable
  * FloatWritable
  * DoubleWritable

* Map 함수의 출력인 KV 쌍이 Key는 IntWritable 타입이고 VALUE는 Text타입일 때 Partitioner를 수정하여 아래와 같이 각 reducer에 가게 하려면 Partitioner class를 수정해야함
  * Partitioner를 수정해서 Key가 1~30이면 Reducer 1로 나머지는 Reducer 2로 보낸다.

![image-20220824133025715](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220824133025715.png)

![image-20220824133543777](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220824133543777.png)

### Inverted Index

![image-20220824140423009](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220824140423009.png)

* `Inverted Index`: 어느 문서의 어느 위치에 나타나는지를 저장해 놓는 것
  * 두개의 Inverted Index를 이용해서 두개의 단어 모두 들어있는 문서 추출 가능





### Matrix Multiplication

##### 1phase

![image-20220825140401502](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220825140401502.png)

* Mapping

![image-20220825140445449](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220825140445449.png)

![image-20220825140456201](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220825140456201.png)

##### 2phase

더해야 할 것들로 나눈뒤

더함

![image-20220825140536608](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220825140536608.png)

![image-20220825140551651](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220825140551651.png)

* Mapping

![image-20220825140632487](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220825140632487.png)



모든 처리를 한번에 할 수 있을만큼 메모리가 크다면 1phase 사용하는 것이 더 빠르다.

만약 메모리가 부족하다면 2phase 사용



실행

```bash
# hdfs에 inputdata를 넣을 폴더 생성
$ hdfs dfs -mkdir matmulti_test
# ubuntu local에서 hdfs로 inputdata 전송
$ hdfs dfs -put data/matmulti-data-2x2.txt matmulti_test
# ssafy.jar를 통해 실행
$ hadoop jar ssafy.jar matmulti A B 2 2 2 matmulti_test matmulti_test_out
```



#### 쎄타 조인(Theta Join)

* 조인-조건(join-predicate)에 비교 연산자인 (<. >, <=, >=, !=, ==)
  * 두 테이블 간의 모든 튜플 쌍에 대하여 조인 칼럼 값이 일치하면 해당 쌍을 출력



#### 모든 쌍 분할(All Pair Partition) 알고리즘

* 테이블 R과 S에 대해서 |R|*|S| 튜플 쌍을 다 고려함
  * R과 S를 각각 u개 파티션과 v개 파티션으로 분할함
  * |R|*|S|개의 튜플(레코드) 쌍을 u * v 개의 disjoint한 파티션으로 분할함
    * |R|은 R에 들어있는 튜플 개수
    * |S|는 S에 들어있는 튜플 개수
  * 각각의 파티션을 한 개의 reduce 함수로 처리함
* 장점
  * 어떤 조인 조건이라도 처리 가능함
  * 모든 reduce 함수에 들어가는 입력 사이즈가 다 비슷함
* 단점
  * 모든 튜플 쌍을 다 검사해야함(Brute Force)
  * 각각의 Reduce 함수의 출력 사이즈가 많이 다를 수 있음

![image-20220825154853401](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220825154853401.png)



![image-20220825154912569](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220825154912569.png)

```bash
// 입력 > table이름\t레코드ID\t컬럼2\t컬럼3

$ hdfs dfs -mkdir allpair_test
$ hdfs dfs -put data/equijoin-R-data.txt allpair_test
$ hdfs dfs -put data/equijoin-S-data.txt allpair_test
$ hadoop jar ssafy.jar allpair r s 2 allpair_test allpair_test_out
```



#### 셀프-조인을 위한 모든 쌍 분할 알고리즘

* Self-Join은 한개의 입력 테이블 D에 들어있는 레코드들 간의 조인을 말함
* 입력 케이스들에 있는 레코드들을 m개의 파티션으로 나눔

![image-20220825163701739](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220825163701739.png)

![image-20220825163717580](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220825163717580.png)

```bash
$ hdfs dfs -mkdir allpairself_test
$ hdfs dfs -put data/equijoin-S-data.txt allpairself_test
$ hadoop jar ssafy.jar allpair s 2 allpairself_test allpairself_test_out
```



#### Common Item Counting for Every Pair of Sets

![image-20220825170210512](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220825170210512.png)

![image-20220825170246874](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220825170246874.png)

```bash
$ hdfs dfs -mkdir itemcount_test
$ hdfs dfs -put data/setjoin-data.txt itemcount_test
$ hadoop jar ssafy.jar itemcount itemcount_test itemcount_test_out itemcount_test_out2
```

![image-20220825172914897](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220825172914897.png)![image-20220825172925943](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220825172925943.png)





#### Top-K Closest Point Search Algorithm

* 질의 포인트와 점들로 구성된 데이터 셋이 있을 때 질의 포인트로부터 가장 가까운 K개 포인트를 뽑는다.
  * ex)  현재 내 위치로부터 가장 가까운 식당 K개를 뽑아줌
* Max-Heap 자료구조를 이용

**Max Heap**

* Binary tree 형태인데 어떤 노드에서 보든지 부모는 그 노드보다 더 크거나 같은 수를 가지고 자식 노드는 그 노드보다 더 작거나 같은 수를 가진다.
* 루트 노드 (root node)에 Max-heap에 들어있는 데이터 중에서 가장 큰 수가 들어간다.

**How to Find Top-K Points**

* 데이터를 읽어가면서 Max-Heap에는 현재까지 본 수들 중에서 가장 작은 K개의 숫자를 유지한다.
* K = 8 일 때 먼저 8개의 수를 읽을 때 까지는 무조건 Max-Heap에 집어 넣어 Max-Heap을 만든다.
* 데이터를 하나씩 보면서 만약 peek에 있는 애보다 지금 들어오는 애가 작으면 heappop한번하고 heappush
* 만약 지금 들어오는애가 더 크면 무시하면 된다.



**Phase 1**

* 점별로 위치와 질의 포인트로부터의 거리를 반환
* 각 머신별로 K개씩만 뽑아서 Reduce한다.



![image-20220825184855785](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220825184855785.png)

**Phase2**

* 각 머신에서 모인 점들중 가장 거리가 짧은 K개 뽑는다.

![image-20220825184936292](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220825184936292.png)

```bash
$ hdfs dfs -mkdir topksearch_test
$ hdfs dfs -put data/topksearch-data.txt topksearch_test
$ hadoop jar ssafy.jar topksearch 3 "1:1" 3 topksearch_test topksearch_test_out1 topksearch_test_out2
```

