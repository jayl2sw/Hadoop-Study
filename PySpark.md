# PySpark

```python
from pyspark import SparkContext
sc = SparkContext(master="spark://cluster.ssafy.io:7177")
rdd = sc.textFile("hdfs://cluster.ssafy.io:9000")

# YARN
improt os
os.environ["YARN_CONF_DIR"] = "/kikang/spark3/conf2"
conf = SparkConf()
conf.setMaster("yarn").setAppName("My Test App")

from pyspark.context import SparkContext
sc = SparkContext(conf=conf)

# YARN 2
imoprt os
os.environ["YARN_CONF_DIR"] = "/kikang/spark3/conf2"
spark = SparkSession.builder.master("yarn").getOrCreate()

df = spark.read.csv("")

```



spark-submit

```bash
spark-submit --jars /eco/spark3/jars/spark-sql-kafka-0-10_2.13-3.2.2.jar ~/kafkatest.py
```

# AirFlow

```bash
pip install flast-wtf==0.15
pip install WTForms==2.3.3
pip install Jinja2=3.1.2
pip install apache-airflow
```

