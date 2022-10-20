# Spark Practice

### 1. 서버 환경 구성

1. User & Sudo 설정

   ```bash
   # OS User 계정 설정 (Spark)
   $ sudo useradd dokcho -m -s /bin/bash
   
   # 계정 암호 설정
   $ sudo passwd spark
   
   # Sudo 설정 파일 편집
   $ sudo visudo
   
   dokcho ALL=(ALL) NOPASSWD: ALL
   
   # 실습 기본 디렉토리 생성 (/kikang)
   $ sudo mkdir /hadoopeco
   
   # 소유자 변경 (spark)
   $ sudo chown dokcho:dokcho /hadoopeco
   
   # 필요한 툴 다운로드 
   $ sudo apt install -y wget unzip bzip2 net-tools
   ```

   

2. Host Name, etchosts 설정

   ```bash
   # hostname 변경
   $ sudo hostnamectl set-hostname ssafy-e201
   
   # hosts 파일 변경
   $ sudo vi /etc/hosts
   
   # private IP와 hostname 연결 
   10.0.9.106 spark-master-01
   10.0.4.101 spark-worker-01
   10.0.10.77 spark-worker-02
   10.0.10.103 spark-worker-03
   ```



3. SSH key, authorized_keys 설정

   * SSH Connection Test(Permission denied)

   * Server간 Password 입력 없이 SSH 연결 가능한지 테스트
     * SSH_key 생성 후 authorized_keys 설정

   ```bash
   $ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
   
   # authorized_keys 만들기
   $ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
   ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
   cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
   
   # 공개키 열어보기
   $ vi ~/.ssh/id_rsa.pub
   
   # 권한 바꿈
   $ chmod 600 ~/.ssh/authorized_keys
   ```



### 2. Spark 설치	

1. apache spark 설치

   ```bash
   # spark 설치
   $ wget https://archive.apache.org/dist/spark/spark-3.2.1/spark-3.2.1-bin-hadoop3.2.tgz
   
   # tarball 압축 해제
   $ tar xvfz spark-3.2.1-bin-hadoop3.2.tgz
   ```

   

2. Java 설치

   ```bash
   # jdk 1.8 설치
   $ wget https://github.com/ojdkbuild/contrib_jdk8u-ci/releases/download/jdk8u322-b06/jdk-8u322-ojdkbuild-linux-x64.zip
   
   $ unzip jdk-8u322-ojdkbuild-linux-x64.zip
   ```



3. Spark Java Home 설정

   spark3 > conf > spark-env.sh.template을 복사하여 spark-env.sh 작성

   ```
   # JAVA 경로 설정
   JAVA_HOME=/kikang/jdk8
   ```



4. Spark 실행

   ```bash
   # spark/bin/spark-shell 실행해보기
   $ ./bin/spark-shell
   ```

   ![image-20220903184449984](C:\Users\SSAFY\Desktop\assets\image-20220903184449984.png)

   ```scala
   sc
   spark
   sc.master
   sc.uiWebUrl
   
   // Spark Driver UI URL
   http://spark-master-01:4040/
   ```

   ```bash
   # Java path 설정
   
   # .profile 열어서
   $ vi ~/.profile
   # 추가 후
   export PATH=/kikang/jdk8/bin:$PATH
   # 적용
   $ source ~/.profile
   ```

   

### 3. Spark History Server 구성

1. Log Directory 설정

   ```bash
   $ cd /kikang/spark3/conf
   $ cp spark-defaults.conf.template spark-defaults.conf
   $ vi spark-defaults.conf
   
   # > spark-defulats.conf
   spark.history.fs.logDirectory file:///kikang/spark3/history
   
   spark.eventLog.enabled true
   spark.eventLog.dir file:///kikang/spark3/history
   
   # log directory 생성
   $ mkdir -p /hadoopeco/spark3/history
   
   # history server 시작
   $ /hadoopeco/spark3/sbin/start-history-server.sh
   
   # history Server UI URL
   http://dokcho-master-01:18080/
   ```

   ![image-20220903191035319](C:\Users\SSAFY\Desktop\assets\image-20220903191035319.png)

   * 방화벽을 열어야 history server web UI 사용 가능

   Log Check Test(Word Count)

   ```scala
   val rdd = sc.textFile("./spark3/README.md")
   val rdd_word = rdd.flatMap(line => line.split(" "))
   val rdd_tuple = rdd_word.map(word => (word, 1))
   val rdd_wordcount = rdd_tuple.reduceByKey((v1, v2) => v1 + v2)
   // 분산되어 있는 word count들을 driver에 모음
   // array가 된다. driver 프로그램의 메모리에 할당
   val arr_wordcount = rdd_wordcount.collect()
   arr_wordcount.take(10).foreach(x => println(x))
   ```

   PySpark

   ```python
   rdd = sc.textFile("README.md")
   rdd_word = rdd.flatMap(lambda line: line.split(" "))
   rdd_tuple = rdd_word.map(lambda word: (word, 1))
   rdd_wordcount = rdd_tuple.reduceByKey(lambda v1, v2: v1 + v2)
   arr_wordcount = rdd_wordcount.collect()
   for wc in arr_wordcount[:10]:
       print(wc)
   ```

   spark-sql

   ```SPARQL
   > select word, count(*) from (select explode(split(*, ' ')) as word from text.`README.md`) group by word limit 10;
   > exit;
   ```

   