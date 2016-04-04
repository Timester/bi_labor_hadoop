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


### Hive - [Hive](https://hive.apache.org)

A Hive egy Hadoophoz készült adattárház megoldás, mely segítségével nagyméretű adathalmazokat menedzselhetünk és kérdezhetünk le. A segítségével HDFS-en tárolt fájlokon fogalmazhatunk meg lekérdezéseket, melyek végrehajtásához a MapReduce programozási modellt fogja használni. A Hive-ot open source projektként az Apache Software Foundation gondozza.

A Hive-al kezelt adatokat legtöbbször valamilyen CSV szerű formátumban adjuk meg, azonban a delimiter karakterre tett ajánlás itt a `\001` (`^A`) ASCII vezérlőkarakter. Ez az ajánlás nem kötelező jellegű, Hive segítségével feldolgozhatunk hagyományos `,` karakterrel elválasztott CSV fájlokat is, sőt akár teljesen más, például JSON vagy ByteStream formátumban érkező adatokat is.

Fontos tisztában lenni azzal, hogy a Hive nem OLTP (Online Transaction Processing) típusú használatra lett optimalizálva (ellentétben például az Oracle DBMS-el), hanem arra, hogy nagyméretű adatokat megbízhatóan, elosztott környezetben kezelhessünk. A Hive lekérdezések a MapReduce modellt kihasználva olyan méretű adathalmazokon is képesek lefutni, amelyeket egy hagyományos DBMS-el már nem tudunk kezelni. Az előzőek miatt a Hive-ot elsősorban analitikai feladatokra használjuk, ahol nem a lefutási idő, hanem a feldolgozandó adat mennyisége a kritikus szempont.

#### HiveQL

A lekérdezések megfogalmazásához a Hive definiál egy lekérdező nyelvet, a HiveQL-t, melynek szintaktikája nagyon hasonlatos az SQL nyelvéhez. A HiveQL segítségével azonban a lekérdezéseinkbe beépíthetünk custom MapReduce algoritmusokat is, hogy még szélesebb legyen az adatfeldolgozási lehetőségek skálája. A nyelv dokumentációját a Hive wiki oldalán [érhetjük el](https://cwiki.apache.org/confluence/display/Hive/LanguageManual), de a laborfeladatok elvégzése során nagy mértékben támaszkodhatunk a korábban megszerzett SQL tudásunkra is.

#### Hive tábla létrehozása
A Hive hagyományos adatfájlokon képes működni, azokra a lekérdezések futási idejében rákényszerít egy sémát, melyet a táblák létrehozásakor adunk meg. A tábla metaadatait az úgynevezett Metastore table tartalmazza, amely egy hagyományos relációs adatbázis, azonban az adatok fizikailag továbbra is a HDFS-en egy szöveges fájl formájában léteznek.

Hive táblák létrehozására az alábbiakban láthatunk egy példát:
```
CREATE EXTERNAL TABLE movie_data (
  userid INT, movieid INT,
  rating INT, unixtime STRING)
  ROW FORMAT DELIMITED
  FIELDS TERMINATED BY '\t'
  PARTITIONED BY (year INT)
  LOCATION '/user/movies'
```
A parancs a HiveQL ismerete nélkül is könnyen értelmezhető, azonban észrevehetünk benne olyan kulcsszavakat, amelyekkel korábban az SQL nyelv használata során nem találkoztunk.

Vegyük észre például az `EXTERNAL` kulcsszót. Ennek használata opcionális, a default működést módosíthatjuk a használatával. Alapvetően a Hive önmaga menedzseli a fizikai adatfájlokat, így például egy tábla eldobása után a Hive a hozzá tartozó adatfájlokat fizikailag is törli. External táblák esetén ennek a működésnek az ellentettje valósul meg, így már meglévő adatfájlokat használhatunk anélkül, hogy a Hive azokat elmozgatná, törölné, stb.

Az SQL nyelv használata során a `ROW FORMAT` vezérlőszavakkal is ritkán találkozunk. A Hive esetén így adhatjuk meg azt, hogy az adatforrás sorait milyen formában tudja deszerializálni a végrehajtó motor. Itt megadható bármilyen osztály, amely megvalósítja a `org.apache.hadoop.hive.serde2.SerDe` interfészt, amely az uniója a `org.apache.hadoop.hive.serde2.Deserializer` és `org.apache.hadoop.hive.serde2.Serializer` interfészeknek. CSV fájlok esetén azonban a `DELIMITED` értéket használjuk, amely azt fejezi ki, hogy a fájl valamilyen vezérlőkarakterekkel elválasztott értékeket tartalmaz. Ezen vezérlőkaraktert a `FIELDS TERMINATED BY` kulcsszavak után adhatjuk meg, jelen esetben ez a tabulátor karakter.

A Hive segítségével kezelhetünk partícionált táblákat is, amely azt jelenti, hogy az adatfájlunk több partícióra bontva, külön mappákban van elhelyezve a HDFS-en. A `PARTITIONED BY (year INT)` azt jelenti, hogy a mappák nevei egy `INT` értéket vesznek fel, melyek a `year` oszlopot reprezentálják. A Hive lekérdezésekben ezek után a `year` attribútumot ugyanúgy kezelhetjük, mint az összes többi, partícionálásra nem használt attribútumot.

External táblák esetén meg kell adni, hogy a tábla alapját képező adatfájlok milyen elérési út alatt találhatók meg, ezért szerepel a parancsban a `LOCATION '/user/movies'` sor is.

#### Hive a Facebooknál (kitekintés)
A Hive-ot a Facebook kezdte el fejleszteni, majd 2008-ban tette azt nyílt forráskódúvá. Motivációja az volt, hogy a cég megalapítása óta kereskedelmi forgalomban lévő RDBMS-eket használt, melyek egy idő után nem voltak képesek kezelni a felhasználók által generált óriási adatmennyiséget, és annak nagy ütemű gyarapodását (2007-ben 15 TB adattal gazdálkodtak, amely 2009-re 2 PB-ra nőtt). Ilyen körülmények között voltak olyan naponta futtatandó jobok, melyek futási ideje tovább tartott, mint 24 óra, ami nyilvánvalóan sürgető szükségét hozta egy új adattárház rendszer bevezetésének.

Úgy döntöttek, hogy az új rendszer a Hadoop alapjaira fog épülni, azonban ez a kezdetekben sok plusz terhet rótt a fejlesztőkre, hiszen egy egyszerű lekérdezéshez is MapReduce programokat kellett írniuk. Így született meg a Hive ötlete, amely segítségével sokkal egyszerűbben tudták az adatokat kezelni és azokon lekérdezéseket megfogalmazni. A Hive már a kezdetek óta nagy népszerűségre tett szert a cégen belül, 2009-re a naponta betöltendő 15 TB adatmennyiséget több ezer job dolgozta fel.

Forrás: [Hive - A Petabyte Scale Data Warehouse using Hadoop](https://www.facebook.com/notes/facebook-engineering/hive-a-petabyte-scale-data-warehouse-using-hadoop/89508453919/)

### Flume - [Flume](https://flume.apache.org)

A Flume egy elosztott, nagy rendelkezésreállású szolgáltatás nagy mennyiségű adatok aggregálására, mozgatására és gyűjtésére. 
Eredeti célja, hogy szerver logokat gyűjtsön és mentsen HDFS-re, de jellemzően ennél sokkal több területen használják. Számos előnnyel rendelkezik, melyek közé a jó testreszabhatóság mellett a rendkívül kis erőforrásigénye is tartozik. Utóbbival kapcsolatban fontos megemlíteni, hogy Flume-ot nem csak Big Data környezetben használhatunk, standalone alkalmazásként is futtatható, kis terhelés esetén ~100 MB memóriára van szüksége.
A Flume architekturája három alap elemből áll, ezek a source-ok, channelök és sinkek. Ezen architektúra minden eleme tetszés szerint testreszabható, bővíthető.

![Flume architektúra](https://flume.apache.org/_images/UserGuide_image00.png "Flume architektúra")

#### Source

Egy-egy source felel a különböző adatforrásból érkező adatok fogadásáért, esetleges feldolgozásáért (pl.: aggregálás, anonimizálás, formátum átalakítása). A Flume számos beépített source-al érkezik, melyek segítségével fogadhatunk adatot HTTP protokollon, üzenetsorokon, vagy akár a fájlrendszeren keresztül is. A Flume bármikor kiegészíthető egyedi source-okkal is, ezeknek az `org.apache.flume.Source` interfészt kell implementálniuk. Az elkészült plugint egy jar fájlba csomagolva kell a Flume rendelkezésére bocsátani.
A source-ok kiegészíthetők még úgynevezett interceptorokkal is, melyekkel a fentebb említett feldolgozásokat valósíthatjuk meg. A custom interceptorok az `org.apache.flume.interceptor.Interceptor` interfészt kötelesek megvalósítani, melynek az `Event intercept(Event event)` metódusában történik a valódi eseményfeldolgozás. A Flume-ba érkező adatokból a source-ok eventeket generálnak, amelyek header és body résszel rendelkeznek. Az interceptorok ezen eventeket módosíthatják, vagy cserélhetik le a fent említett metódusukban.

#### Channel

A source-ok az eseményeket egy vagy több channelbe helyezhetik, amelyek továbbítják azokat a sinkekhez. A channel feladata, hogy a betöltött eventeket tárolja mindaddig, amíg azokat egy sink ki nem veszi belőlük. A channelök a legkevésbé gyakran customizált elemei az architektúrának, az esetek nagy részében a gyári Memory Channelt vagy File Channelt használjuk. A Memory Channel, ahogyan a neve is mutatja, egy in-memory queue-ban tárolja az eventeket, melynek maximális mérete konfigurálható. Ezt akkor használjuk, ha nagy áteresztőképességű rendszert fejlesztünk, és nem kritikus követelmény, hogy szélsőséges esetekben is minden esemény továbbításra kerüljön. A File Channel ennél jóval kisebb áteresztőképességgel rendelkezik, azonban itt még a Flume agent leaállása során sem vesznek el események.

#### Sink

Az eventek a channelt elhagyva úgynevezett sinkekbe érkeznek. Ezek feladata, hogy az eseményeket továbbítsák a megfelelő adatnyelő helyre. Rengeteg sink érkezik alapértelmezetten Flume-al együtt, ezek közül az egyik legfontosabb a HDFS Sink, amely HDFS fájlrendszerre tudja menteni az eseményeket, de továbbíthatjuk az eseményeket egy Kafka üzenetsorba, vagy JDBC-n keresztül rengeteg típusú adatbázisba is. Gyakori, hogy egy-egy probléma megoldása során a fejlesztők saját sinkeket használnak, melyeket viszonylag egyszerű implementálni is, de rengeteg jól használható open source plugin érhető el, így például MongoDB-hez, vagy RabbitMQ-hoz is illeszthetjük az adatbetöltő szolgáltatásunkat.

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

### 1. Feladat - adatbetöltés Flume-al

A `/user/data/movielens` elérési út alatt megtalálhatunk három adathalmazt, amelyet a [http://movielens.org](http://movielens.org) oldalon található filmadatbázisból, és a hozzá tartozó értékelésekből nyertek ki. A labor során ezekkel az adathalmazokkal fogunk dolgozni, így célszerű betölteni a saját mappánkba ezeket.

#### 1.1 Feladat - Movies dataset betöltése

Az első betöltendő adathalmaz néhány népszerű film adatait tartalmazza. Flume használatával töltse be ezeket az adatokat a `/user/NEPTUN/movies` mappába.

Első lépésként deklaráljuk a `movieagent` komponenseit:
```
# Name the components on this agent
movieagent.sources = r1
movieagent.sinks = k1
movieagent.channels = c1
```

Konfiguráljuk az `r1` source-ot:
```
# Describe/configure the source
movieagent.sources.r1.type = spooldir
movieagent.sources.r1.spoolDir = /user/data/movielens/movies/NEPTUN
```

Konfiguráljuk a `k1` sinket:
```
# Describe the sink
movieagent.sinks.k1.type = file_roll
movieagent.sinks.k1.sink.directory = /user/NEPTUN/movies
movieagent.sinks.k1.batchSize = 1000
```

Konfiguráljuk a `c1` channelt:
```
# Use a channel which buffers events in memory
movieagent.channels.c1.type = memory
movieagent.channels.c1.capacity = 1000
movieagent.channels.c1.transactionCapacity = 100
```

Kössük össze a komponenseket:
```
# Bind the source and sink to the channel
movieagent.sources.r1.channels = c1
movieagent.sinks.k1.channel = c1
```

#### 1.2 Feladat - Ratings dataset betöltése

A filmek értékelését tartalmazó adathalmazt is be kell tölteni, azonban ha vetünk egy pillantást a `ratings.dat` fájlra, láthatjuk, hogy itt a `!?!?` karaktersorozat választja el a sorok egyes mezőit. Ez a későbbiekben problémákhoz vezethet, így a betöltés során cseréljük le ezt a karaktersorozatot a Hive által ajánlott `^A` karakterre.

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
