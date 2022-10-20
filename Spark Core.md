# Spark Core

## Spark RDD 개요 (Spark 실행 환경)

### Spark Application

* Spark Application 구성
  * Driver Program (Local)
  * Driver Program + Executors (Cluster)
  * 각각 독립된 JVM 프로세스

* Spark Application 배포
  * spark-submit, launcher
  * Local, Standalone, Mesos, YARN, Kubernetes
* Spark Application 개발
  * RDD 생성 > RDD 변환 > RDD 연산 / 저장
  * Spark Shell, Web Notebook, IDE

![image-20220911204917510](C:\Users\SSAFY\AppData\Roaming\Typora\typora-user-images\image-20220911204917510.png)

#### Driver Program

* main 함수가 실행되는 프로세스

* 사용자 프로그램을 Task로 변환하여 클러스터로 전송

  1. 연산들의 관계를 DAG(Directed Acyclic Graph)로 생성
  2. DAG를 물리적인 실행 계획으로 변환
     * 최적화를 거쳐 여러개의 Stage로 변환
     * 각 Stage는 여러개의 Task로 구성 (여러개의 스레드)
  3. 단위 작업들을 묶어서 클러스터로 전송

* Executor에서의 개별 Task들을 위한 스케줄링 조정

  * Executor들은 시작시 드라이버에 등록됨
    * Task와 가장 가까운 파티션을 묶어서 스케줄링
  * 드라이버는 항상 실행중인 Executor를 감시 
    * Task를 데이터 위치에 기반해 적절한 위치에서 실행되도록함

  * 웹 UI(default 4040 port)를 통해 실행 정보 제공



#### Executor

* 개별 Task를 Worker Node에서 실행하는 프로세스
* Task 실행 후 결과를 드라이버로 전송
* Cache하는 RDD를 저장하기 위한 메모리 공간 제공



#### Cluster Manager

* Driver Program은 Executor를 할당 받기 위해 Cluster Manager에 의존
* Standalone, Mesos, YARN, Kubernetes



### Spark 실행단계

1. spark-submit을 이용해 Application 제출
2. spark-submit은 Driver Program을 실행하여 main함수 호출
3. Driver Program이 Cluster Manager에게 Executor 실행을 위한 리소스 요청
4. Cluster Manager가 Executor 실행
5. Driver Program이 작업을 Task 단위로 나누어 Executor에 전송
6. Executor가 Task 실행
7. Application이 종료되면 Cluster Manager에게 리소스 반납





## Spark RDD 개요 (Spark Glossary)

* `Application`
  * User program built on Spark. Consists of a driver program and executors on the cluster
* `Application jar`
  * A jar containing the user's Spark application.
  * 자바 or 스칼라로 어플리케이션 작성시 
  * 컴파일 한 후에 jar 파일로 아카이브 
* `Driver program`
  * The process running the main() function of the application and creating the Spark Context
* `Cluster manager`
  * An external service for acquiring resources on the cluster
  * ex) Standalone, Mesos, YARN, Kubernetes

* `Deploy mode`
  * 클러스터에 배포할 때, 드라이버 프로세스가 어디서 동작하는지를 결정하는 것
  * Distinguishes where the driver process runs
  * cluster mode: framework launches the driver inside of the cluster
    * 클러스터 내의 한 서버에서 driver process가 작동
  * client mode: the submitter launches the driver outside of the cluster
    * 사용자가 spark application을 작성한 곳에서 driver process 작동

* `Worker node`
  * Any node that can run application code in the cluster
  * Excutor 프로세스가 돌아가는 곳
* `Executor`
  * A process launched for an application on a worker node, that runs tasks and keeps data in memory or disk storage across them. Each application has its own executors.

* `Task`
  * A unit of work that will be sent to one executor
* `Job`
  * **A parallel computation consisting of multiple tasks** that gets spawned in response to a Spark action (save, collect)
* Stage
  * Each job gets divided into smaller sets of tasks called stages that depend on each other
  * 하나의 job 안에 여러 슽
  * ex) map and reduce 