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

### Flume - [Flume](https://flume.apache.org)

A Flume egy elosztott, nagy rendelkezésreállású szolgáltatás nagy mennyiségű adatok aggregálására, mozgatására és gyűjtésére. 
Eredeti célja, hogy szerver logokat gyűjtsön és mentsen HDFS-re, de jellemzően ennél sokkal több területen használják. Számos előnnyel rendelkezik, melyek közé a jó testreszabhatóság mellett a rendkívül kis erőforrásigénye is tartozik. Utóbbival kapcsolatban fontos megemlíteni, hogy Flume-ot nem csak Big Data környezetben használhatunk, standalone alkalmazásként is futtatható, kis terhelés esetén ~100 MB memóriára van szüksége.
A Flume architekturája három alap elemből áll, ezek a source-ok, channelök és sinkek. Ezen architektúra minden eleme tetszés szerint testreszabható, bővíthető.

![Flume architektúra](https://flume.apache.org/_images/UserGuide_image00.png "Flume architektúra")

#### Source

Egy-egy source felel a különböző adatforrásból érkező adatok fogadásáért, esetleges feldolgozásáért (pl.: aggregálás, anonimizálás, formátum átalakítása). A Flume számos beépített source-al érkezik, melyek segítségével fogadhatunk adatot HTTP protokollon, üzenetsorokon, vagy akár a fájlrendszeren keresztül is. A Flume bármikor kiegészíthető egyedi source-okkal is, ezeknek az `org.apache.flume.Source` interfészt kell implementálniuk. Az elkészült plugint egy jar fájlba csomagolva kell a Flume rendelkezésére bocsátani.
A source-ok kiegészíthetők még úgynevezett interceptorokkal is, melyekkel a fentebb említett feldolgozásokat valósíthatjuk meg. A custom interceptorok az `org.apache.flume.interceptor.Interceptor` interfészt kötelesek megvalósítani, melynek az `Event intercept(Event event)` metódusában történik a valódi eseményfeldolgozás. A Flume-ba érkező adatokból a source-ok eventeket generálnak, amelyek header és body résszel rendelkeznek. Az interceptorok ezen eventeket módosíthatják, vagy cserélhetik le a fent említett metódusukban.

#### Channel

A source-ok az eseményeket egy vagy több channelbe helyezik, amely továbbítja azokat a sinkekhez. A channel feladata, hogy a betöltött Eventeket tárolja mindaddig, amíg azokat egy sink ki nem veszi belőlük. A channelök a legkevésbé gyakran customizált elemei az architektúrának, az esetek nagy részében a gyári Memory Channelt vagy File Channelt használjuk. A Memory Channel, ahogyan a neve is mutatja, egy in-memory queueban tárolja az eventeket, melyek maximális mérete konfigurálható. Ezt akkor használjuk, ha nagy áteresztőképességű rendszert fejlesztünk, és nem kritikus követelmény, hogy szélsőséges esetekben is minden esemény továbbításra kerüljön. A File Channel ennél jóval kisebb áteresztőképességgel rendelkezik, azonban itt még a Flume agent leaállása során sem vesznek el események.

#### Sink

Az eventek a channelt elhagyva úgynevezett sinkekbe érkeznek. Ezek feladata, hogy az eseményeket továbbítsák a megfelelő adatnyelő helyre. Rengeteg sink érkezik alapértelmezetten Flume-al együtt, ezek közül az egyik legfontosabb az HDFS Sink, amely HDFS fájlrendszerre tudja menteni az eseményeket, de továbbíthatjuk az eseményeket egy Kafka üzenetsorba, vagy JDBC-n keresztül rengeteg típusú adatbázisba is. Gyakori, hogy egy-egy probléma megoldása során a fejlesztők saját sinkeket használnak, melyeket viszonylag egyszerű implementálni is, de rengeteg jól használható open source plugin érhető el, így például MongoDB-hez, vagy RabbitMQ-hoz is illeszthetjük az adatbetöltő szolgáltatásunkat.

#### Flume konfiguráció

A Flume konfigurációja nem kódból, hanem hagyományos Java Properties fájlon keresztül történik, melyre az alábbiakban látható egy példa:

```
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```
A konfigurálás első lépéseként a Flume agent komponenseit deklaráljuk. Ezek után a source konfigurációját láthatjuk. Meghatározzuk, hogy az `r1` source típusa `netcat`, amely egy TCP porton keresztül érkező szöveg sorait csomagolja eventekbe. Ezek után meghatározzuk, hogy a localhost 44444-es portján hallgatózzon a source.

A sink típusa `logger`, amely az egyik legegyszerűbb sink, feladata, hogy az eseményeket INFO levellel logolja. Természetesen ezt leginkább csak tesztelési és debuggolási célokra használjuk.

A channel konfigurálása is az előzőekhez hasonlóan történik. A konfiguráció utolsó blokkjában azt határozzuk meg, hogy az `r1` source az eseményeit a `c1` channelbe továbbítsa, ahonnan a `k1` sink fogja kivenni őket.

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
CREATE EXTERNAL TABLE neptunkod_movies(id INT, title STRING, genre ARRAY<STRING>)
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY '\001'
COLLECTION ITEMS TERMINATED BY '|'
STORED AS TEXTFILE;

LOAD DATA INPATH '/user/.../bilabor/movies.dat' INTO TABLE neptunkod_movies;
```

```
CREATE EXTERNAL TABLE neptunkod_users(id INT, gender STRING, age STRING, occupation STRING, zip STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\001'
STORED AS TEXTFILE;

LOAD DATA INPATH '/user/.../bilabor/users.dat' INTO TABLE neptunkod_users;
```

```
CREATE EXTERNAL TABLE neptunkod_ratings(userid INT, movieid INT, rating INT, timestamp INT)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\001'
STORED AS TEXTFILE;

LOAD DATA INPATH '/user/.../bilabor/ratings.dat' INTO TABLE neptunkod_ratings;
```

Néhány egyszerű lekérdezés:

Akciófilmek listája:
```
SELECT * FROM neptunkod_movies WHERE array_contains(genre, "Action");
```

Értékelések eloszlása:
```
SELECT rating, count(*) FROM neptunkod_ratings GROUP BY rating;
```

### 3. Feladat - Spark analitika - imre

## Önálló feladatok

### 1. Feladat - Flume módosítása HTTP src-ra - adam

### 2. Feladat - Bonyolultabb Hive lekérdezés - imre

### 3. Feladat - Spark program - imre
