---
layout: post
title: Start Using Zeppelin with Spark and Cassandra on Centos 7
categories: posts
comments: true
en: true
description: Start Using Zeppelin with Spark and Cassandra on Centos 7
keywords: "zeppelin, spark, cassandra, centos, sql, nosql"
---


# Introduction

in this post we wanna install Zeppelin and configuring spark to connect with our cassandra cluster for starting analytic jobs.

__why we wanna do this ?__

* For view, analyze our big data records , making charts and graphs without writing single line of code
* Spark give use ability to make SQL queries and processing our data very fast and at all our process cost is not in our cassandra clusters
* its not enough ?

__what is [Apache Zeppelin](http://zeppelin.apache.org/) ?__

> A web-based notebook that enables interactive data analytics.
> You can make beautiful data-driven, interactive and collaborative documents with SQL, Scala and more.

in my opinion its very easy to use , many interpreters , configurable and cool :) .

__what is [Apache Spark](http://spark.apache.org/) ?__

>Apache Spark™ is a fast and general engine for large-scale data processing.

Well it's very fast , as they say its up to 100x faster than Hadoop MapReduce in memory .

__what about [Apache Cassandra](http://cassandra.apache.org/) ?__

>Apache Cassandra, a top level Apache project born at Facebook and built on Amazon’s Dynamo and Google’s BigTable, is a distributed database for managing large amounts of structured data across many commodity servers, while providing highly available service and no single point of failure.  Cassandra offers capabilities that relational databases and other NoSQL databases simply cannot match such as: continuous availability, linear scale performance, operational simplicity and easy data distribution across multiple data centers and cloud availability zones.

at all cassandra is really fast in INSERT and fast enough for many Big data models .

__Our Steps__

* Installing Apache Zeppelin
* configuring spark interpreters
* Create new note and write our first spark job

# Installing Apache Zeppelin

__installing requirements__

* Oracle JDK 1.7 (set JAVA_HOME)
* Git
* npm
* Maven 3.1.x or higher

``` bash

# installing jdk 7
sudo yum install java-1.7.0-openjdk java-1.7.0-openjdk-devel -y

# setting JAVA_HOME
export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.101-2.6.6.1.el7_2.x86_64/jre/

# installing npm
sudo yum install npm -y

# installing git
sudo yum install git -y

#installing Maven
wget http://www.eu.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
tar -zxf apache-maven-3.3.9-bin.tar.gz -C /usr/local/
ln -s /usr/local/apache-maven-3.3.9/bin/mvn /usr/local/bin/mvn
export MAVEN_OPTS="-Xmx2g -XX:MaxPermSize=1024m"

```

after installing ensure they installed correctly

``` bash
mvn -version
node --version
```


__Installing Zeppelin__

``` bash
git clone https://github.com/apache/zeppelin.git
cd zeppelin/
# compiling with cassandra and spark interpreters , it's take while
mvn clean package -Pcassandra-spark-1.5 -Dhadoop.version=2.6.0 -Phadoop-2.6 -DskipTests
```


__Starting Zeppelin__

``` bash
bin/zeppelin-daemon.sh start
```

After successful start, visit http://localhost:8080 with your web browser.

for stoping zeppelin

``` bash
bin/zeppelin-daemon.sh stop
```


# configuring spark interpreters

Add this properties to the Spark connector interpreter

* spark.cassandra.connection.host with value 127.0.0.1 (or IP of one of your Cassandra cluster node)
* spark.cassandra.auth.username and spark.cassandra.auth.password if you using authentication for connecting to your cassandra cluster

<img src="{{ site.url }}/assets/img/spark-connector-interpreter.png" width="400" height="250" />


# Create new note and write our first spark job

just create a new note and write your job

this example load cassandra records in rdd and then print sum of amount field

```
%spark

import com.datastax.spark.connector._

val rdd = sc.cassandraTable("my_keyspace", "transactions")
println(rdd.map(_.getInt("amount")).sum)
sc.stop
```

for understanding how write spark jobs for cassandra using datastax spark connector visit [this](https://github.com/datastax/spark-cassandra-connector/blob/master/doc/0_quick_start.md) link