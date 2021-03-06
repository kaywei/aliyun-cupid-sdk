* [1. 开发环境准备](#1)
    * [1.1 下载并解压spark包](#1.1)
    * [1.2 设置环境变量](#1.2)
    * [1.3 设置Spark-defaults.conf](#1.3)
* [2. 获取并编译样例模板](#2)
* [3. Maven依赖](#3)
* [4. 常用接口及运行应用](#4)
    * [4.1 Spark Shell](#4.1)
    * [4.2 Spark R](#4.2)
    * [4.3 Spark SQL](#4.3)
    * [4.4 Spark JDBC](#4.4)
* [5. 常用场景案例演示](#5)
    * [5.1 WordCount案例](#5.1)
    * [5.2 访问OSS案例](#5.2)
    * [5.3 MaxCompute Table ReadWrite案例](#5.3)
        * [5.3.1 读非分区表](#5.3.1)
        * [5.3.2 读单列分区表](#5.3.2)
        * [5.3.3 读多列分区表](#5.3.3)
        * [5.3.4 指定多个分区(每个分区多列)读表](#5.3.4)
        * [5.3.5 存储RDD至非分区表](#5.3.5)
        * [5.3.6 指定具体分区存储RDD至分区表](#5.3.6)
        * [5.3.7 动态指定分区存储RDD至分区表](#5.3.7)
    * [5.4 MaxCompute Table Spark-SQL案例](#5.4)
    * [5.5 MaxCompute自研Client模式案例](#5.5)
    * [5.6 MaxCompute Table PySpark案例](#5.6)
    * [5.7 Mllib案例](#5.7)
    * [5.8 pyspark交互式案例](#5.8)
    * [5.9 spark-shell交互式案例(读表)](#5.9)
    * [5.10 spark-shell交互式案例(mlib+OSS读写)](#5.10)
    * [5.11 sparkR交互式案例](#5.11)
    * [5.12 GraphX--PageRank案例](#5.12)
    * [5.13 Spark Streaming--NetworkWordCount案例](#5.13)
* [6. 特殊说明](#6)
    * [6.1 Spark Streaming任务特殊配置](#6.1)
    * [6.2 Tracking Url说明](#6.2)
* [7. 相关参考](#7)
* [8. License](#8)

<h1 id="1">1 开发环境准备</h1>

**!!!注意!!!**

master分支的文档以及代码仅支持MaxCompute AliSpark **专有云输出**

[MaxCompute AliSpark公共云入口](https://github.com/aliyun/aliyun-cupid-sdk/tree/1.1.0-public)


<h2 id="1.1">1.1 下载并解压spark包</h2>

1. 下载[Spark on MaxCompute](https://promotion.aliyun.com/ntms/act/dedicatedcloud/doc.html)安装包。
2. 对下载好的Spark on MaxCompute最新发布包进行解压，解压后的文件夹结构如下。
```
    .
    |-- R
    |-- RELEASE
    |-- __spark_libs__.zip
    |-- bin
    |-- conf
    |-- cupid
    |-- derby.log
    |-- examples
    |-- jars
    |-- logs
    |-- metastore_db
    |-- python
    |-- sbin
    |-- yarn
```

<h2 id="1.2">1.2 设置环境变量</h2>

JAVA_HOME设置
```
export JAVA_HOME=/path/to/jdk
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH
```
SPARK_HOME设置
```
export SPARK_HOME=/path/to/spark_extracted_package
export PATH=$SPARK_HOME/bin:$PATH
```
对于SparkR用户，需要安装R至/home/admin/R目录，并设置path：
```
export PATH=/home/admin/R/bin/:$PATH
```
对于pyspark用户，选装python2.7，并设置path：
```
export PATH=/path/to/python/bin/:$PATH
```

<h2 id="1.3">1.3 设置Spark-defaults.conf</h2>

在$SPARK_HOME/conf路径下存在spark-defaults.conf文件，需要在该文件中设置MaxCompute相关的账号信息后，才可以提交Spark任务到MaxCompute。默认配置内容如下，将空白部分根据实际的账号信息填上即可。
```
# OdpsAccount Info Setting
spark.hadoop.odps.project.name=
spark.hadoop.odps.access.id=
spark.hadoop.odps.access.key=
spark.hadoop.odps.end.point=
#spark.hadoop.odps.moye.trackurl.host=
#spark.hadoop.odps.cupid.webproxy.endpoint=
spark.sql.catalogImplementation=odps
# spark-shell Setting
spark.driver.extraJavaOptions -Dscala.repl.reader=com.aliyun.odps.spark_repl.OdpsIntera
ctiveReader -Dscala.usejavacp=true
# SparkR Setting
# odps.cupid.spark.r.archive=/path/to/R-PreCompile-Package.zip
# Cupid Longtime Job
# spark.hadoop.odps.cupid.engine.running.type=longtime
# spark.hadoop.odps.cupid.job.capability.duration.hours=8640
# spark.hadoop.odps.moye.trackurl.dutation=8640
# spark.r.command=/home/admin/R/bin/Rscript
# spark.hadoop.odps.cupid.disk.driver.enable=false
spark.hadoop.odps.cupid.bearer.token.enable=false
spark.hadoop.odps.exec.dynamic.partition.mode=nonstrict
```
<h1 id="2">2 获取并编译样例模板</h1>

执行如下命令，获取并编译样例模板spark-example：
```
git clone https://github.com/aliyun/aliyun-cupid-sdk.git
cd aliyun-cupid-sdk
mvn -T 1C clean install -DskipTests
```

<h1 id="3">3 Maven依赖</h1>

上述章节中的GitHub项目，即spark-example可以作为用户QuickStart开发的模版，如果用户需要自定义开发，请确认pom.xml文件如下所示。spark的模块的版本使用社区2.1.0的版本即可，并且保证scope是provided。
```
<dependency>
  <groupId>org.apache.spark</groupId>
  <artifactId>spark-core_2.11</artifactId>
  <version>2.1.0</version>
  <scope>provided</scope>
</dependency>
```
MaxCompute插件已经发布到Maven仓库，需添加以下依赖。
```
<dependency>
  <groupId>com.aliyun.odps</groupId>
  <artifactId>cupid-core_2.11</artifactId>
  <version>1.0.0</version>
  <scope>provided</scope>
</dependency>
<dependency>
  <groupId>com.aliyun.odps</groupId>
  <artifactId>cupid-datasource_2.11</artifactId>
  <version>1.0.0</version>
</dependency>
<dependency>
  <groupId>com.aliyun.odps</groupId>
  <artifactId>cupid-client_2.11</artifactId>
  <version>1.0.0</version>
</dependency>
```
maven库的依赖列表如下所示：
- Cupid平台core代码，包括cupid task提交接口封装，以及父子进程读写表相关接口。
```
<dependency>
  <groupId>com.aliyun.odps</groupId>
  <artifactId>cupid-core_2.11</artifactId>
  <version>1.0.0</version>
</dependency>
```
- datasource，封装了spark相关的MaxCompute Table读写的用户接口。
```
<dependency>
  <groupId>com.aliyun.odps</groupId>
  <artifactId>cupid-datasource_2.11</artifactId>
  <version>1.0.0</version>
</dependency>
```
- 封装了Cupid Client模式的用户SDK。
```
<dependency>
  <groupId>com.aliyun.odps</groupId>
  <artifactId>cupid-client_2.11</artifactId>
  <version>1.0.0</version>
</dependency>
```

<h1 id="4">4 常用接口及运行应用</h1>

Spark目前支持spark shell、spark r、spark sql及spark jdbc等四种接口向集群提交任务，具体的运行应用如下。
<h2 id="4.1">4.1 Spark Shell</h2>

执行如下命令，启动应用
```
$cd $SPARK_HOME
$bin/spark-shell --master yarn
```
简单应用示例如下：
```
sc.parallelize(0 to 100, 2).collect
sql("show tables").show
sql("select * from spark_user_data").show(200,100)
```
<h2 id="4.2">4.2 Spark R</h2>

执行如下命令，启动应用。
```
// 创建一个目录R并解压该目录下的R.zip
$mkdir -p /home/admin/R && unzip ./R/R.zip -d /home/admin/R/
// 设置环境变量
$export PATH=/home/admin/R/bin/:$PATH
// 选择运行模式并启动应用
$bin/sparkR --master yarn --archives ./R/R.zip
```
简单应用示例如下：
```
df <- as.DataFrame(faithful)
df
head(select(df, df$eruptions))
head(select(df, "eruptions"))
head(filter(df, df$waiting < 50))
results <- sql("FROM spark_user_data SELECT *")
head(results)
```
<h2 id="4.3">4.3 Spark SQL</h2>

执行如下命令，启动应用。
```
// 进入spark目录
$cd $SPARK_HOME
// 选择运行模式并启动应用
$bin/spark-sql --master yarn
```
应用示例如下：
```
show tables;
select * from spark_user_data limit 3;
quit;
```

<h2 id="4.4">4.4 Spark JDBC</h2>

执行如下命令，启动应用。
```
// 停止线程
$sbin/stop-thriftserver.sh
// 重启线程
$sbin/start-thriftserver.sh
// 启动应用
$bin/beeline
```
简单应用示例如下：
```
!connect jdbc:hive2://localhost:10000/odps_smoke_test
show tables;
select * from mr_input limit 3;
!quit
```

<h1 id="5">5 常用场景案例演示</h1>

本章节将给出一些常用场景的案例演示，帮助用户更快的了解Spark on MaxCompute的相关使用。
用户需要下载GitHub项目并对项目进行编译后，才能够运行相关Demo。
```
// 下载GitHub项目
git clone https://github.com/aliyun/aliyun-cupid-sdk.git
cd aliyun-cupid-sdk
// 编译GitHub项目
mvn -T 1C clean install -DskipTests
```
执行完上述步骤后，会产生相应的jar包，这些jar包会在下面具体的案例演示Demo中使用。
<h2 id="5.1">5.1 WordCount案例</h2>
Spark运行简单的WordCount。
案例代码如下。

```
package com.aliyun.odps.spark.examples
import org.apache.spark.sql.SparkSession
object WordCount {
  def main(args: Array[String]) {
    val spark = SparkSession
      .builder()
      .appName("WordCount")
      .getOrCreate()
    val sc = spark.sparkContext
    try {
      sc.parallelize(1 to 100, 10).map(word => (word, 1)).reduceByKey(_ + _, 10).take(100).
      foreach(println)
    } finally {
      sc.stop()
    }
  }
}
```
提交作业的运行命令如下。
```
bin/spark-submit \
--master yarn-cluster \
--class com.aliyun.odps.spark.examples.WordCount \
/path/to/aliyun-cupid-sdk/examples/spark-examples/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
```
<h2 id="5.2">5.2 访问OSS案例</h2>

Spark可以访问OSS数据，示例如下。
```
package com.aliyun.odps.spark.examples.oss
import org.apache.spark.sql.SparkSession
object SparkUnstructuredDataCompute {
  def main(args: Array[String]) {
    val spark = SparkSession
      .builder()
      .appName("SparkUnstructuredDataCompute")
      .config("spark.hadoop.fs.oss.accessKeyId", "***")
      .config("spark.hadoop.fs.oss.accessKeySecret", "***")
      .config("spark.hadoop.fs.oss.endpoint", "oss-cn-hangzhou-zmf.aliyuncs.com")
      .getOrCreate()
    val sc = spark.sparkContext
    try {
      val pathIn = "oss://bucket/inputdata/"
      val inputData = sc.textFile(pathIn, 5)
      val cnt = inputData.count
      println(s"count: $cnt")
    } finally {
      sc.stop()
    }
  }
}
```
提交作业的运行命令如下
```
./bin/spark-submit \
--jars cupid/hadoop-aliyun-package-3.0.0-alpha2-odps-jar-with-dependencies.jar \
--class com.aliyun.odps.spark.examples.oss.SparkUnstructuredDataCompute \
/path/to/aliyun-cupid-sdk/examples/spark-examples/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
```
<h2 id="5.3">5.3 MaxCompute Table ReadWrite案例</h2>
读写MaxCompute Table，转化为Spark RDD。

```
⚠️注意：
案例Demo中的Project/Table必须存在，或者更改为对应的Project/Table。
```
案例代码如下。

<h3 id="5.3.1">5.3.1 读非分区表</h3>

java API：
```
/**
 *  read from normal table via rdd api
 *  desc cupid_wordcount;
 *  +------------------------------------------------------------------------------------+
 *  | Field           | Type       | Label | Comment                                     |
 *  +------------------------------------------------------------------------------------+
 *  | id              | string     |       |                                             |
 *  | value           | string     |       |                                             |
 *  +------------------------------------------------------------------------------------+
 */
JavaSparkContext ctx = new JavaSparkContext(sparkConf);
JavaOdpsOps javaOdpsOps = new JavaOdpsOps(ctx);
JavaRDD<Tuple2<String, String>> rdd_0 = javaOdpsOps.readTable(
        projectName,
        "cupid_wordcount",
        new Function2<Record, TableSchema, Tuple2<String, String>>() {
            public Tuple2<String, String> call(Record v1, TableSchema v2) throws Exception {
                return new Tuple2<String, String>(v1.getString(0), v1.getString(1));
            }
        },
        0
);
```
scala API:
```
/**
  * read from normal table via rdd api
  * desc cupid_wordcount;
  * +------------------------------------------------------------------------------------+
  * | Field           | Type       | Label | Comment                                     |
  * +------------------------------------------------------------------------------------+
  * | id              | string     |       |                                             |
  * | value           | string     |       |                                             |
  * +------------------------------------------------------------------------------------+
  */
val sc = new SparkContext(conf)
val odpsOps = new OdpsOps(sc)
val rdd_0 = odpsOps.readTable(
  projectName,
  "cupid_wordcount",
  (r: Record, schema: TableSchema) => (r.getString(0), r.getString(1))
)
```

<h3 id="5.3.2">5.3.2 读单列分区表</h3>

java API:
```
/**
 *  read from single partition column table via rdd api
 *  desc dftest_single_parted;
 *  +------------------------------------------------------------------------------------+
 *  | Field           | Type       | Label | Comment                                     |
 *  +------------------------------------------------------------------------------------+
 *  | id              | string     |       |                                             |
 *  | value           | string     |       |                                             |
 *  +------------------------------------------------------------------------------------+
 *  | Partition Columns:                                                                 |
 *  +------------------------------------------------------------------------------------+
 *  | pt              | string     |                                                     |
 *  +------------------------------------------------------------------------------------+
 */
JavaSparkContext ctx = new JavaSparkContext(sparkConf);
JavaOdpsOps javaOdpsOps = new JavaOdpsOps(ctx);
JavaRDD<Tuple3<String, String, String>> rdd_1 = javaOdpsOps.readTable(
        projectName,
        "dftest_single_parted",
        "pt=20160101",
        new Function2<Record, TableSchema, Tuple3<String, String, String>>() {
            public Tuple3<String, String, String> call(Record v1, TableSchema v2) throws Exception {
                return new Tuple3<String, String, String>(v1.getString(0), v1.getString(1), v1.getString("pt"));
            }
        },
        0
);
```
scala API:
```
/**
  * read from single partition column table via rdd api
  * desc dftest_single_parted;
  * +------------------------------------------------------------------------------------+
  * | Field           | Type       | Label | Comment                                     |
  * +------------------------------------------------------------------------------------+
  * | id              | string     |       |                                             |
  * | value           | string     |       |                                             |
  * +------------------------------------------------------------------------------------+
  * | Partition Columns:                                                                 |
  * +------------------------------------------------------------------------------------+
  * | pt              | string     |                                                     |
  * +------------------------------------------------------------------------------------+
  */
val sc = new SparkContext(conf)
val odpsOps = new OdpsOps(sc)
val rdd_1 = odpsOps.readTable(
  projectName,
  "dftest_single_parted",
  Array("pt=20160101"),
  (r: Record, schema: TableSchema) => (r.getString(0), r.getString(1), r.getString("pt"))
)
```

<h3 id="5.3.3">5.3.3 读多列分区表</h3>

java API:
```
/**
 *  read from multi partition column table via rdd api
 *  desc dftest_parted;
 *  +------------------------------------------------------------------------------------+
 *  | Field           | Type       | Label | Comment                                     |
 *  +------------------------------------------------------------------------------------+
 *  | id              | string     |       |                                             |
 *  | value           | string     |       |                                             |
 *  +------------------------------------------------------------------------------------+
 *  | Partition Columns:                                                                 |
 *  +------------------------------------------------------------------------------------+
 *  | pt              | string     |                                                     |
 *  | hour            | string     |                                                     |
 *  +------------------------------------------------------------------------------------+
 */
JavaSparkContext ctx = new JavaSparkContext(sparkConf);
JavaOdpsOps javaOdpsOps = new JavaOdpsOps(ctx);
JavaRDD<Tuple4<String, String, String, String>> rdd_2 = javaOdpsOps.readTable(
        projectName,
        "dftest_parted",
        "pt=20160101,hour=12",
        new Function2<Record, TableSchema, Tuple4<String, String, String, String>>() {
            public Tuple4<String, String, String, String> call(Record v1, TableSchema v2) throws Exception {
                return new Tuple4<String, String, String, String>(v1.getString(0), v1.getString(1), v1.getString("pt"), v1.getString(3));
            }
        },
        0
);
```
scala API:
```
/**
  * read from multi partition column table via rdd api
  * desc dftest_parted;
  * +------------------------------------------------------------------------------------+
  * | Field           | Type       | Label | Comment                                     |
  * +------------------------------------------------------------------------------------+
  * | id              | string     |       |                                             |
  * | value           | string     |       |                                             |
  * +------------------------------------------------------------------------------------+
  * | Partition Columns:                                                                 |
  * +------------------------------------------------------------------------------------+
  * | pt              | string     |                                                     |
  * | hour            | string     |                                                     |
  * +------------------------------------------------------------------------------------+
  */
val sc = new SparkContext(conf)
val odpsOps = new OdpsOps(sc)
val rdd_2 = odpsOps.readTable(
  projectName,
  "dftest_parted",
  Array("pt=20160101,hour=12"),
  (r: Record, schema: TableSchema) => (r.getString(0), r.getString(1), r.getString("pt"), r.getString(3))
)
```

<h3 id="5.3.4">5.3.4 指定多个分区(每个分区多列)读表</h3>

java API:
```
/**
 *  read with multi partitionSpec definition via rdd api
 *  desc cupid_partition_table1;
 *  +------------------------------------------------------------------------------------+
 *  | Field           | Type       | Label | Comment                                     |
 *  +------------------------------------------------------------------------------------+
 *  | id              | string     |       |                                             |
 *  | value           | string     |       |                                             |
 *  +------------------------------------------------------------------------------------+
 *  | Partition Columns:                                                                 |
 *  +------------------------------------------------------------------------------------+
 *  | pt1             | string     |                                                     |
 *  | pt2             | string     |                                                     |
 *  +------------------------------------------------------------------------------------+
 */
JavaSparkContext ctx = new JavaSparkContext(sparkConf);
JavaOdpsOps javaOdpsOps = new JavaOdpsOps(ctx);
JavaRDD<Tuple4<String, String, String, String>> rdd_3 = javaOdpsOps.readTable(
        projectName,
        "cupid_partition_table1",
        new String[]{"pt1=part1,pt2=part1", "pt1=part1,pt2=part2", "pt1=part2,pt2=part3"},
        new Function2<Record, TableSchema, Tuple4<String, String, String, String>>() {
            public Tuple4<String, String, String, String> call(Record v1, TableSchema v2) throws Exception {
                return new Tuple4<String, String, String, String>(v1.getString(0), v1.getString(1), v1.getString("pt1"), v1.getString("pt2"));
            }
        },
        0
);
```
scala API:
```
/**
  * read with multi partitionSpec definition via rdd api
  * desc cupid_partition_table1;
  * +------------------------------------------------------------------------------------+
  * | Field           | Type       | Label | Comment                                     |
  * +------------------------------------------------------------------------------------+
  * | id              | string     |       |                                             |
  * | value           | string     |       |                                             |
  * +------------------------------------------------------------------------------------+
  * | Partition Columns:                                                                 |
  * +------------------------------------------------------------------------------------+
  * | pt1             | string     |                                                     |
  * | pt2             | string     |                                                     |
  * +------------------------------------------------------------------------------------+
  */
val sc = new SparkContext(conf)
val odpsOps = new OdpsOps(sc)
val rdd_3 = odpsOps.readTable(
  projectName,
  "cupid_partition_table1",
  Array("pt1=part1,pt2=part1", "pt1=part1,pt2=part2", "pt1=part2,pt2=part3"),
  (r: Record, schema: TableSchema) => (r.getString(0), r.getString(1), r.getString("pt1"), r.getString("pt2"))
)
```

<h3 id="5.3.5">5.3.5 存储RDD至非分区表</h3>

java API:
```
/**
 *  save rdd into normal table
 *  desc cupid_wordcount_empty;
 *  +------------------------------------------------------------------------------------+
 *  | Field           | Type       | Label | Comment                                     |
 *  +------------------------------------------------------------------------------------+
 *  | id              | string     |       |                                             |
 *  | value           | string     |       |                                             |
 *  +------------------------------------------------------------------------------------+
 */
JavaSparkContext ctx = new JavaSparkContext(sparkConf);
JavaOdpsOps javaOdpsOps = new JavaOdpsOps(ctx);
VoidFunction3<Tuple2<String, String>, Record, TableSchema> transfer_0 =
        new VoidFunction3<Tuple2<String, String>, Record, TableSchema>() {
            @Override
            public void call(Tuple2<String, String> v1, Record v2, TableSchema v3) throws Exception {
                v2.set("id", v1._1());
                v2.set(1, v1._2());
            }
        };
javaOdpsOps.saveToTable(projectName, "cupid_wordcount_empty", rdd_0.rdd(), transfer_0, true);
```
scala API:
```
/**
  * save rdd into normal table
  * desc cupid_wordcount_empty;
  * +------------------------------------------------------------------------------------+
  * | Field           | Type       | Label | Comment                                     |
  * +------------------------------------------------------------------------------------+
  * | id              | string     |       |                                             |
  * | value           | string     |       |                                             |
  * +------------------------------------------------------------------------------------+
  */
val sc = new SparkContext(conf)
val odpsOps = new OdpsOps(sc)
val transfer_0 = (v: Tuple2[String, String], record: Record, schema: TableSchema) => {
  record.set("id", v._1)
  record.set(1, v._2)
}
odpsOps.saveToTable(projectName, "cupid_wordcount_empty", rdd_0, transfer_0, true)
```

<h3 id="5.3.6">5.3.6 指定具体分区存储RDD至分区表</h3>

java API:
```
/**
 *  save rdd into partition table with single partition spec
 *  desc cupid_partition_table1;
 *  +------------------------------------------------------------------------------------+
 *  | Field           | Type       | Label | Comment                                     |
 *  +------------------------------------------------------------------------------------+
 *  | id              | string     |       |                                             |
 *  | value           | string     |       |                                             |
 *  +------------------------------------------------------------------------------------+
 *  | Partition Columns:                                                                 |
 *  +------------------------------------------------------------------------------------+
 *  | pt1             | string     |                                                     |
 *  | pt2             | string     |                                                     |
 *  +------------------------------------------------------------------------------------+
 */
JavaSparkContext ctx = new JavaSparkContext(sparkConf);
JavaOdpsOps javaOdpsOps = new JavaOdpsOps(ctx);
VoidFunction3<Tuple2<String, String>, Record, TableSchema> transfer_1 =
        new VoidFunction3<Tuple2<String, String>, Record, TableSchema>() {
            @Override
            public void call(Tuple2<String, String> v1, Record v2, TableSchema v3) throws Exception {
                v2.set("id", v1._1());
                v2.set("value", v1._2());
            }
        };
javaOdpsOps.saveToTable(projectName, "cupid_partition_table1", "pt1=test,pt2=dev", rdd_0.rdd(), transfer_1, true);
```
scala API:
```
/**
  * save rdd into partition table with single partition spec
  * desc cupid_partition_table1;
  * +------------------------------------------------------------------------------------+
  * | Field           | Type       | Label | Comment                                     |
  * +------------------------------------------------------------------------------------+
  * | id              | string     |       |                                             |
  * | value           | string     |       |                                             |
  * +------------------------------------------------------------------------------------+
  * | Partition Columns:                                                                 |
  * +------------------------------------------------------------------------------------+
  * | pt1             | string     |                                                     |
  * | pt2             | string     |                                                     |
  * +------------------------------------------------------------------------------------+
  */
val sc = new SparkContext(conf)
val odpsOps = new OdpsOps(sc)
val transfer_1 = (v: Tuple2[String, String], record: Record, schema: TableSchema) => {
  record.set("id", v._1)
  record.set("value", v._2)
}
odpsOps.saveToTable(projectName, "cupid_partition_table1", "pt1=test,pt2=dev", rdd_0, transfer_1, true)
```

<h3 id="5.3.7">5.3.7 动态指定分区存储RDD至分区表</h3>

java API:
```
/**
 *  dynamic save rdd into partition table with multiple partition spec
 *  desc cupid_partition_table1;
 *  +------------------------------------------------------------------------------------+
 *  | Field           | Type       | Label | Comment                                     |
 *  +------------------------------------------------------------------------------------+
 *  | id              | string     |       |                                             |
 *  | value           | string     |       |                                             |
 *  +------------------------------------------------------------------------------------+
 *  | Partition Columns:                                                                 |
 *  +------------------------------------------------------------------------------------+
 *  | pt1             | string     |                                                     |
 *  | pt2             | string     |                                                     |
 *  +------------------------------------------------------------------------------------+
 */
JavaSparkContext ctx = new JavaSparkContext(sparkConf);
JavaOdpsOps javaOdpsOps = new JavaOdpsOps(ctx);
VoidFunction4<Tuple2<String, String>, Record, PartitionSpec, TableSchema> transfer_2 =
        new VoidFunction4<Tuple2<String, String>, Record, PartitionSpec, TableSchema>() {
            @Override
            public void call(Tuple2<String, String> v1, Record v2, PartitionSpec v3, TableSchema v4) throws Exception {
                v2.set("id", v1._1());
                v2.set("value", v1._2());

                String pt1_value = new Random().nextInt(10) % 2 == 0 ? "even" : "odd";
                String pt2_value = new Random().nextInt(10) % 2 == 0 ? "even" : "odd";
                v3.set("pt1", pt1_value);
                v3.set("pt2", pt2_value);
            }
        };
javaOdpsOps.saveToTableForMultiPartition(projectName, "cupid_partition_table1", rdd_0.rdd(), transfer_2, true);
```
scala API:
```
/**
  * dynamic save rdd into partition table with multiple partition spec
  * desc cupid_partition_table1;
  * +------------------------------------------------------------------------------------+
  * | Field           | Type       | Label | Comment                                     |
  * +------------------------------------------------------------------------------------+
  * | id              | string     |       |                                             |
  * | value           | string     |       |                                             |
  * +------------------------------------------------------------------------------------+
  * | Partition Columns:                                                                 |
  * +------------------------------------------------------------------------------------+
  * | pt1             | string     |                                                     |
  * | pt2             | string     |                                                     |
  * +------------------------------------------------------------------------------------+
  */
val sc = new SparkContext(conf)
val odpsOps = new OdpsOps(sc)
val transfer_2 = (v: Tuple2[String, String], record: Record, part: PartitionSpec, schema: TableSchema) => {
  record.set("id", v._1)
  record.set("value", v._2)

  val pt1_value = if (new Random().nextInt(10) % 2 == 0) "even" else "odd"
  val pt2_value = if (new Random().nextInt(10) % 2 == 0) "even" else "odd"
  part.set("pt1", pt1_value)
  part.set("pt2", pt2_value)
}
odpsOps.saveToTableForMultiPartition(projectName, "cupid_partition_table1", rdd_0, transfer_2, true)
```
提交作业的运行命令如下。
```
bin/spark-submit \
--master yarn-cluster \
--class com.aliyun.odps.spark.examples.OdpsTableReadWrite \
/path/to/aliyun-cupid-sdk/examples/spark-examples/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
```
MaxCompute Table读取并发度可以通过如下两个方法进行调整。
- 调整spark.hadoop.odps.input.split.size，该值越大，map task越少，默认为256MB。
- 通过OdpsOps.readTable中的numPartition进行设置，该值可直接决定map task个数，模式按照spark.hadoop.odps.input.split.size进行计算。

<h2 id="5.4">5.4 MaxCompute Table Spark-SQL案例</h2>

使用sqlContext读写MaxCompute Table。
```
⚠️注意：
- 案例Demo中的Project/Table必须存在，或者更改为对应的Project/Table。
- Spark-defaults.conf里必须设置spark.sql.catalogImplementation = odps。
```
案例代码如下。
```
package com.aliyun.odps.spark.examples
import org.apache.spark.sql.SparkSession
object OdpsTableReadWriteViaSQL {
  def main(args: Array[String]) {
    // please make sure spark.sql.catalogImplementation=odps in spark-defaults.conf
    // to enable odps catalog
    val spark = SparkSession
      .builder()
      .appName("OdpsTableReadWriteViaSQL")
      .getOrCreate()
    val projectName = spark.sparkContext.getConf.get("odps.project.name")
    val tableName = "cupid_wordcount"
    // get a ODPS table as a DataFrame
    val df = spark.table(tableName)
    println(s"df.count: ${df.count()}")
    // Just do some query
    spark.sql(s"select * from $tableName limit 10").show(10, 200)
    spark.sql(s"select id, count(id) from $tableName group by id").show(10, 200)
    // any table exists under project could be use
    // productRevenue
    spark.sql(
      """
        |SELECT product,
        | category,
        | revenue
        |FROM
        | (SELECT product,
        | category,
        | revenue,
        | dense_rank() OVER (PARTITION BY category
        | ORDER BY revenue DESC) AS rank
        | FROM productRevenue) tmp
        |WHERE rank <= 2
      """.stripMargin).show(10, 200)
    spark.stop()
  }
}
```
提交作业的运行命令如下。
```
bin/spark-submit
--master yarn-cluster
--class com.aliyun.odps.spark.examples.OdpsTableReadWrite
/path/to/aliyun-cupid-sdk/examples/spark-examples/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
```

<h2 id="5.5">5.5 MaxCompute自研Client模式案例</h2>

由于安全考虑，MaxCompute里的机器不能进行直连。所以原生Spark中的yarn-client模式无法使用。为了解决交互式的需求，MaxCompute团队自主研发了Client模式。
案例代码如下。
```
package com.aliyun.odps.spark.examples
import com.aliyun.odps.cupid.client.spark.client.CupidSparkClientRunner
object SparkClientNormalFT {
  def main(args: Array[String]) {
    val cupidSparkClient = CupidSparkClientRunner.getReadyCupidSparkClient()
    val jarPath = args(0) //client-jobexamples jar path
    val sparkClientNormalApp = new SparkClientNormalApp(cupidSparkClient)
    sparkClientNormalApp.runNormalJob(jarPath)
    cupidSparkClient.stopRemoteDriver()
  }
}
```
完整Client模式含如下文件:
- Client Control
```
spark-examples/src/main/scala/com/aliyun/odps/spark/examples/SparkClientNormalFT.scala
spark-examples/src/main/scala/com/aliyun/odps/spark/examples/SparkClientNormalApp.scala
```
- SparkJob Unit
```
client-jobexamples/src/main/scala/com/aliyun/odps/cupid/client/spark/CacheRddJob.scala
client-jobexamples/src/main/scala/com/aliyun/odps/cupid/client/spark/ParallRddJob.scala
client-jobexamples/src/main/scala/com/aliyun/odps/cupid/client/spark/UseCacheRdd.scala
client-jobexamples/src/main/scala/com/aliyun/odps/cupid/client/spark/SparkJobKill.scala
```
提交作业的运行命令如下。
```
java -cp \
/path/to/aliyun-cupid-sdk/examples/spark-examples/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar:$SPARK_HOME/jars/* \
com.aliyun.odps.spark.examples.SparkClientNormalFT \
/path/to/aliyun-cupid-sdk/examples/client-jobexamples/target/client-jobexamples_2.11-1.0.0-SNAPSHOT.jar
```

<h2 id="5.6">5.6 MaxCompute Table PySpark案例</h2>

PySpark读写MaxCompute Table。
案例代码如下。
```
from odps.odps_sdk import OdpsOps
from pyspark import SparkContext, SparkConf
from pyspark.sql import SQLContext, DataFrame
if __name__ == '__main__':
conf = SparkConf().setAppName("odps_pyspark")
sc = SparkContext(conf=conf)
sql_context = SQLContext(sc)
project_name = "cupid_testa1"
in_table_name = "cupid_wordcount"
out_table_name = "cupid_wordcount_py"
normal_df = OdpsOps.read_odps_table(sql_context, project_name, in_table_name)
for i in normal_df.sample(False, 0.01).collect():
  print i
    print "Read normal odps table finished"
OdpsOps.write_odps_table(sql_context, normal_df.sample(False, 0.001), project_name,
  out_table_name)
print "Write normal odps table finished"
```
提交作业的运行命令如下。
```
spark-submit \
--master yarn-cluster \
--jars /path/to/aliyun-cupid-sdk/external/cupid-datasource/target/cupid-datasource_2.11-1.0.0-SNAPSHOT.jar \
--py-files /path/to/aliyun-cupid-sdk/examples/spark-examples/src/main/python/odps.zip \
/path/to/aliyun-cupid-sdk/examples/spark-examples/src/main/python/odps_table_rw.py
```

<h2 id="5.7">5.7 Mllib案例</h2>

Mllib的model建议使用OSS进行读写。
案例代码如下。
```
package com.aliyun.odps.spark.examples.mllib
import org.apache.spark.mllib.clustering.KMeans._
import org.apache.spark.mllib.clustering.{KMeans, KMeansModel}
import org.apache.spark.mllib.linalg.Vectors
import org.apache.spark.sql.SparkSession
object KmeansModelSaveToOss {
  val modelOssDir = "oss://bucket/kmeans-model"
  def main(args: Array[String]) {
    //1. train and save the model
    val spark = SparkSession
      .builder()
      .appName("KmeansModelSaveToOss")
      .config("spark.hadoop.fs.oss.accessKeyId", "***")
      .config("spark.hadoop.fs.oss.accessKeySecret", "***")
      .config("spark.hadoop.fs.oss.endpoint", "***")
      .getOrCreate()
    val sc = spark.sparkContext
    val points = Seq(
      Vectors.dense(0.0, 0.0),
      Vectors.dense(0.0, 0.1),
      Vectors.dense(0.1, 0.0),
      Vectors.dense(9.0, 0.0),
      Vectors.dense(9.0, 0.2),
      Vectors.dense(9.2, 0.0)
    )
    val rdd = sc.parallelize(points, 3)
    val initMode = K_MEANS_PARALLEL
    val model = KMeans.train(rdd, k = 2, maxIterations = 2, runs = 1, initMode)
    val predictResult1 = rdd.map(feature => "cluster id: " + model.predict(feature) + " feature:" +
      feature.toArray.mkString(",")).collect
    println("modelOssDir=" + modelOssDir)
    model.save(sc, modelOssDir)
    //2. predict from the oss model
    val modelLoadOss = KMeansModel.load(sc, modelOssDir)
    val predictResult2 = rdd.map(feature => "cluster id: " + modelLoadOss.predict(feature) + "
      feature:" + feature.toArray.mkString(",")).collect
    assert(predictResult1.size == predictResult2.size)
    predictResult2.foreach(result2 => assert(predictResult1.contains(result2)))
  }
}
```
提交作业的运行命令如下。
```
./bin/spark-submit \
--jars cupid/hadoop-aliyun-package-3.0.0-alpha2-odps-jar-with-dependencies.jar \
--class com.aliyun.odps.spark.examples.mllib.KmeansModelSaveToOss \
/path/to/aliyun-cupid-sdk/examples/spark-examples/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
```

<h2 id="5.8">5.8 pyspark交互式案例</h2>

只有能直连计算集群的机器上可执行pyspark。
选装python2.7，并设置path：
```
export PATH=/path/to/python/bin/:$PATH
```
启动命令：
```
bin/pyspark --master yarn-client
```
交互执行：
```
df=spark.sql("select * from spark_user_data")
df.show()
```
<h2 id="5.9">5.9 spark-shell交互式案例(读表)</h2>
只有能直连计算集群的机器上可执行spark-shell。
启动命令：
```
bin/spark-shell --master yarn
```
交互执行：
```
sc.parallelize(0 to 100, 2).collect
sql("show tables").show
sql("select * from spark_user_data").show(200,100)
```

<h2 id="5.10">5.10 spark-shell交互式案例(mlib+OSS读写)</h2>

只有能直连计算集群的机器上可执行spark-shell。
在conf/spark-defaults.conf中加入以下配置：
```
spark.hadoop.fs.oss.accessKeyId=***
spark.hadoop.fs.oss.accessKeySecret=***
spark.hadoop.fs.oss.endpoint=***
```
启动命令：
```
bin/spark-shell --master yarn --jars cupid/hadoop-aliyun-package-3.0.0-alpha2-odps-jar-withdependencies.jar
```
交互执行：
```
import org.apache.spark.mllib.clustering.KMeans._
import org.apache.spark.mllib.clustering.{KMeans, KMeansModel}
import org.apache.spark.mllib.linalg.Vectors
val modelOssDir = "oss://your_bucket/kmeans-model"
val points = Seq(
  Vectors.dense(0.0, 0.0),
  Vectors.dense(0.0, 0.1),
  Vectors.dense(0.1, 0.0),
  Vectors.dense(9.0, 0.0),
  Vectors.dense(9.0, 0.2),
  Vectors.dense(9.2, 0.0)
)
val rdd = sc.parallelize(points, 3)
val initMode = K_MEANS_PARALLEL
val model = KMeans.train(rdd, k = 2, maxIterations = 2, runs = 1, initMode)
val predictResult1 = rdd.map(feature => "cluster id: " + model.predict(feature) + " feature:" +
  feature.toArray.mkString(",")).collect
println("modelOssDir=" + modelOssDir)
model.save(sc, modelOssDir)
val modelLoadOss = KMeansModel.load(sc, modelOssDir)
val predictResult2 = rdd.map(feature => "cluster id: " + modelLoadOss.predict(feature) + "
  feature:" + feature.toArray.mkString(",")).collect
assert(predictResult1.size == predictResult2.size)
predictResult2.foreach(result2 => assert(predictResult1.contains(result2)))
```

<h2 id="5.11">5.11 sparkR交互式案例</h2>

只有能直连计算集群的机器上可执行sparkR，并且需要安装R至/home/admin/R目录，同时设置path：
```
export PATH=/home/admin/R/bin/:$PATH
```
启动命令：
```
bin/sparkR --master yarn -—archives ./R/R.zip
```
交互执行：
```
df <- as.DataFrame(faithful)
df
head(select(df, df$eruptions))
head(select(df, "eruptions"))
head(filter(df, df$waiting < 50))
results <- sql("FROM spark_user_data SELECT *")
head(results)
```

<h2 id="5.12">5.12 GraphX--PageRank案例</h2>

支持原生的graphx。
案例代码如下。
```
package com.aliyun.odps.spark.examples.graphx
import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.graphx._
import org.apache.spark.rdd.RDD
object PageRank {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName("pagerank")
    val sc = new SparkContext(conf)
    // construct vertices
    val users: RDD[(VertexId, Array[String])] = sc.parallelize(List(
      "1,BarackObama,Barack Obama",
      "2,ladygaga,Goddess of Love",
      "3,jeresig,John Resig",
      "4,justinbieber,Justin Bieber",
      "6,matei_zaharia,Matei Zaharia",
      "7,odersky,Martin Odersky",
      "8,anonsys"
    ).map(line => line.split(",")).map(parts => (parts.head.toLong, parts.tail)))
    // construct edges
    val followers: RDD[Edge[Double]] = sc.parallelize(Array(
      Edge(2L,1L,1.0),
      Edge(4L,1L,1.0),
      Edge(1L,2L,1.0),
      Edge(6L,3L,1.0),
      Edge(7L,3L,1.0),
      Edge(7L,6L,1.0),
      Edge(6L,7L,1.0),
      Edge(3L,7L,1.0)
    ))
    // construct graph
    val followerGraph: Graph[Array[String], Double] = Graph(users, followers)
    // restrict the graph to users with usernames and names
    val subgraph = followerGraph.subgraph(vpred = (vid, attr) => attr.size == 2)
    // compute PageRank
    val pageRankGraph = subgraph.pageRank(0.001)
    // get attributes of the top pagerank users
    val userInfoWithPageRank = subgraph.outerJoinVertices(pageRankGraph.vertices) {
      case (uid, attrList, Some(pr)) => (pr, attrList.toList)
      case (uid, attrList, None) => (0.0, attrList.toList)
    }
    println(userInfoWithPageRank.vertices.top(5)(Ordering.by(_._2._1)).mkString("\n"))
  }
}
```
提交作业的运行命令如下。
```
bin/spark-submit \
--master yarn-cluster \
--class com.aliyun.odps.spark.examples.graphx.PageRank \
/path/to/aliyun-cupid-sdk/examples/spark-examples/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar
```

<h2 id="5.13">5.13 Spark Streaming--NetworkWordCount案例</h2>

支持原生Spark Streaming，以NetworkWordCount为例，首先本地需安装Netcat，并执行以下命令：
```
$ nc -lk 9999
```
此时，控制台的输入将成为SparkStreaming的输入。
案例代码如下。
```
package com.aliyun.odps.spark.examples.streaming
import org.apache.spark.SparkConf
import org.apache.spark.examples.streaming.StreamingExamples
import org.apache.spark.storage.StorageLevel
import org.apache.spark.streaming.{Seconds, StreamingContext}
object NetworkWordCount {
  def main(args: Array[String]) {
    if (args.length < 2) {
      System.err.println("Usage: NetworkWordCount <hostname> <port>")
      System.exit(1)
    }
    StreamingExamples.setStreamingLogLevels()
    // Create the context with a 1 second batch size
    val sparkConf = new SparkConf().setAppName("NetworkWordCount")
    val ssc = new StreamingContext(sparkConf, Seconds(1))
    // Create a socket stream on target ip:port and count the
    // words in input stream of \n delimited text (eg. generated by 'nc')
    // Note that no duplication in storage level only for running locally.
    // Replication necessary in distributed scenario for fault tolerance.
    val lines = ssc.socketTextStream(args(0), args(1).toInt, StorageLevel.MEMORY_AND
      _DISK_SER)
    val words = lines.flatMap(_.split(" "))
    val wordCounts = words.map(x => (x, 1)).reduceByKey(_ + _)
    wordCounts.print()
    ssc.start()
    ssc.awaitTermination()
  }
}
```
提交作业的运行命令如下。
```
bin/spark-submit \
--master local[4] \
--class com.aliyun.odps.spark.examples.streaming.NetworkWordCount \
/path/to/aliyun-cupid-sdk/examples/spark-examples/target/spark-examples_2.11-1.0.0-SNAPSHOT-shaded.jar localhost 9999
```

<h1 id="6">6 特殊说明</h1>

本章节将对使用中的一些特殊情况进行相关说明。

<h2 id="6.1">6.1 Spark Streaming任务特殊配置</h2>

MaxCompute支持Spark Streaming，为了支持长任务运行，需要在spark-defaults.conf中添加如下的特殊配置。
不同于离线作业，Streaming作业有一些特殊配置，这些配置也是配置在spark-defaults.conf即可生效
```
// 配置成longtime才不会被回收。
spark.hadoop.odps.cupid.engine.running.type=longtime
// 配置时间长度。
spark.hadoop.odps.cupid.job.capability.duration.hours=25920
// 配置failOver最大重试次数。
spark.yarn.maxAppAttempts=10
// 是否启用writeAheadLog模式，能够保证数据不丢失，但是效率会降低。
spark.streaming.receiver.writeAheadLog.enable=true
```

<h2 id="6.2">6.2 Tracking Url说明</h2>

提交完成作业后，一般会有如下输出。
```
17/08/28 14:53:26 INFO Client:
client token: N/A
diagnostics: N/A
ApplicationMaster host: 11.137.199.2
ApplicationMaster RPC port: 57524
queue: queue
start time: 1503903179541
final status: SUCCEEDED
tracking URL: http://jobview.odps.aliyun-inc.com/proxyview/jobview/?h=http://service.odps.aliyun-inc.com/api&p=odps_public_dev&i=20170828065141675g5h4t6u1&t=spark&id=application_1503903039442_1185611255&metaname=20170828065141675g5h4t6u1&token=L0dSMHRkSlNXS2ZkeFE1UkVsckthTTZQWHV3PSxPRFBTX09CTzoxMDU5NTgyNzI0MzIyOTk5LDE1MDQxNjIzODMseyJTdGF0ZW1lbnQiOlt7IkFjdGlvbiI6WyJvZHBzOlJlYWQiXSwiRWZmZWN0IjoiQWxsb3ciLCJSZXNvdXJjZSI6WyJhY3M6b2RwczoqOnByb2plY3RzL29kcHNfcHVibGljX2Rldi9pbnN0YW5jZXMvMjAxNzA4MjgwNjUxNDE2NzVnNWg0dDZ1MSJdfV0sIlZlcnNpb24iOiIxIn0=
user: user
```
在输出示例中看到TrackingUrl，表示用户的作业已经提交到MaxCompute集群。上面输出示例中的TrackingUrl非常关键，它既是SparkWebUI，也是HistoryServer的Url。

<h1 id="7">7 相关参考</h1>

如果用户希望获取更多的Spark的相关内容，请参考：[Spark Configuration](https://spark.apache.org/docs/2.1.0/configuration.html)。

<h1 id="8">8 License</h1>

Licensed under the Apache License 2.0
