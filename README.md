# BI Labor - Hadoop

## Emlékeztető

### Hue - [Hadoop user Experience](http://gethue.com/)
Webalkalmazás a Hadoop környezetek leggyakoribb funkcióinak kezeléséhez:
* HDFS böngésző
* Hive / Impala query editor
* Oozie ütemező, job-ok indítása, workflow szerkesztő
  * Spark
  * Hive
  * HDFS műveletek
  * shell
  * ...
* Apache Solr szerkezstő / felület
* Apache Sentry editor
* Sqoop 

### HDFS - [Hadoop Distributed File System](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html)
* Elosztott, redundáns, blokkalapú adattárolás
* A Hadoop alapja
* Elérés
  * REST (WebHDFS)
  * shell
  * Java API


* hive - adam
* flume - adam
* spark - imre

## Vezetett rész

### 0. Feladat - környezet elérése

Azure felhőben futó Cloudera Hadoop disztribúció. Elérhetőségek:
* [Hue](http://sensorhub.autsoft.hu)
  * Usernév: neptunkód
  * Jelszó: valami
* [Cloudera Manager](http://sensorhub.autsoft.hu)
  * Usernév: neptunkód
  * Jelszó: valami más

### 1. Feladat - adatbetöltés Flume-al - adam

### 2. Feladat - Hive lekérdezés az adatokon - imre

Táblák létrehozása: 

```
CREATE EXTERNAL TABLE movies(id INT, title STRING, genre STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\001'
STORED AS TEXTFILE
LOCATION '<hdfs_location>';
```

```
CREATE EXTERNAL TABLE users(id INT, gender STRING, age STRING, occupation STRING, zip STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\001'
STORED AS TEXTFILE
LOCATION '<hdfs_location>';
```

```
CREATE EXTERNAL TABLE ratings(userid INT, movieid INT, rating INT, timestamp INT)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\001'
STORED AS TEXTFILE
LOCATION '<hdfs_location>';
```

### 3. Feladat - Spark analitika - imre

## Önálló feladatok

### 1. Feladat - Flume módosítása HTTP src-ra - adam

### 2. Feladat - Bonyolultabb Hive lekérdezés - imre

### 3. Feladat - Spark program - imre
