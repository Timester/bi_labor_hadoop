# BI Labor - Hadoop

## Emlékeztető

### Hue - [Hadoop user Experience](http://gethue.com/)
A Hue egy webalkalmazás a Hadoop környezetekben leggyakrabban használt funkciók egyszerű, webes felületről történő kezeléséhez. A Hue a Cloudera nyílt forráskódú fejlesztése, minden Hadoop disztribúcióval kompatibilis, az egyes szolgáktatásokkal azok standard interfészein keresztül kommunikál. Az alkalmazás a következő funkcionalitást biztosítja:

* HDFS böngésző
* Hive / Impala lekérdezés szerkesztő
* Oozie ütemező, job-ok indítása, workflow szerkesztő
  * Spark
  * Hive
  * HDFS műveletek
  * shell
  * ...
* Apache Solr szerkesztő / felület
* Apache Sentry menedzsment
* Sqoop menedzsment

A Hue használatával megszabadulhatunk (az esetek nagyrészében) a parancssori interfészek kényelmetlenségeitől. 

### HDFS - [Hadoop Distributed File System](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html)
A Hadoop alapját két szolgáltatás képzi ezek a HDFS és a YARN. A YARN-al részletesen itt most nem foglalkozunk, röviden a Yet Another Resource Negotiator (YARN) egy erőforráskezelő szolgáltatás, ami a klaszterben található processzormagokat, memóriát és egyéb erőforrásokat szétosztja a futo job-ok közt. Mindemellett lebonyolítja és felügyeli az egyes folyamatok futását.

A HDFS azaz Hadoop Distributed File System egy elosztott redundáns blokk alapú fájlrendszer. Hadoop környezetekben 95%-ban HDFS fog az adattárolás eszközeként szolgálni, erre épülnek a különböző adatbázisok és eszközök.

A HDFS a Google File System-ről kiadott cikkek alapján készült azzal a céllal, hogy megbízható és hatékony adattárolást tegyen lehetővé akár olcsó, hétköznapi hardvereken is. Ezen kritériumok teljesítéséhez az adatokat replikálja, több példányban tárolja. A replikáció blokk szinten történik, egy-egy fájl blokkjai több gépen szétszórva, több példányban tárolódnak. Ez a módszer egyrészt garantálja az adatok biztonságát egy-egy gép kiesése esetén, valamint a párhuzamos olvasás miatt teljesítmény javulást is.

A HDFS klaszterben 2 típusú node létezik. Tipikusan 1 darab NameNode, ami a fájlok blokkjainak helyét és egyéb metainformációkat tárol, menedzsment feladatokat lát el. Valamint számos DataNode, amik az adatok tárolásárért és replikálásáért felelősek. A NameNode kisesése az összes adat olvashatatlanná válását is jelenti így ezt gyakran RAID-es valamint High Availability konfigurációkkal igyekeznek védeni.

A HDFS-el történő kommunikáció során (fájlok írása olvasása) kezdeti lépésben mindig a NameNode-hoz fordulunk, ami megmondja hol találjuk, vagy hova írhatjuk adott fájl darabjait. A konkrét adatmozgatás közvetlenül a DataNode-okhoz kapcsolódva történik, ezzel eloszlatva a terhelést.

A HDFS elérésére számos mód kínálkozik:
* Java API
* Parancssori kliens
* WebHDFS
* Hue

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

### Apache NiFi - [Flume](https://nifi.apache.org)
TODO

### Spark - [Spark](https://spark.apache.org/)
A Spark ma a legnépszerűb adatfeldolgozó eszköz Hadoop környezetben. A korábban igen elterjedt és nagy sikernek örvendő Map Reduce paradigmát szinte teljesen felváltotta. Térnyerése a kitűnő, Map Reduce programoknál akár százszor jobb teljesítményének valamint az egyszerű, jól használható funkcionális API-jának köszönheti. Fontos megjegyezni, hogy a Spark ezt a sebességet azzal éri el, hogy minden adatot memóriában tart így olyan adathalmazok feldolgozása, amik nem férnek be a memóriába bajos lehet.

A Spark megjelenésével a Hadoop elkezdett a batch alapú szemléletből nyitni a real-time foldolgozás irányába is. Ennek egyik vezér eleme a Spark Streaming, ami microbatching-el megvalósított stream feldolgozásra képes. A Spark-hoz sok további kiegészítő csomag is készült melyek gráf feldolgozási, SQL vagy gépi tanulási könyvtárakat, algoritmusokat adnak a fejleszetők kezébe. Ki és bemeneti adatforrásokat tekintve sem szenvedünk hiányt, a HDFS, HBase, Hive mind támogatott. Spark programokat Scala, Java, Python és R nyelven is lehet írni, a Spark maga Scala-ban készült. Mivel az API-ja funkcionális jellegű ezért a legszebb kódot ezt támogató nyelvekben, azaz Scala-ban vagy Python-ban lehet készíteni, teljesítmény szempontjából egyértelműen a Scala preferált.

#### Egy Spark program alapjai
Egy egyszerű Spark programban tipikusan betöltünk valamilyen adatokat egy forrásból, ezeken műveleteket hajtunk végre majd a kívánt eredményeket eltároljuk valahova. A Spark egy nagyon fontos központi figalma az RDD (Resilient Distributed Dataset), egy olyan adatszerkezet mely a kleszteren elosztottan kerül tárolásra. Ezeken az RDD-ken végezhetünk műveleteket melyeknek két típusa létezik transzformáció és akció. A transzformációk új RDD-t fognak eredményezni ilyen a mappelés, szűrés stb. Az akciók az RDD-n valamilyen aggregációt hajtanak végre, eredményük tipikusan egy szám, egy pár, egy objektum, de nem RDD. A Spark a transzformációkat lazy módon hajtja végre. Mindaddíg semmit nem csinál, amíg egy akció nem következik. Így lehetősége van a parancsok sorozatát kielemezni és optimalizálni.

Az alábbi kódrészlet egy szövegben számolja meg az egyes szavak előfordulásainak számát, bemutatva az imént említett főbb lépéseket.

```
val textFile = sc.textFile("hdfs://...")
val counts = textFile
                 .flatMap(line => line.split(" "))
                 .map(word => (word, 1))
                 .reduceByKey(_ + _)
counts.saveAsTextFile("hdfs://...")
```

Első lépésként betöltjük a forrás adatokat a HDFS-ről, ezt követően jön a feldolgozás. A bemeneti szöveget soronként feldolgozva szóközök mentén szavakra tördeljük, majd minden szót egy (szó, 1) párra mappelünk ahol a szó a kulcs és a hozzá tartozó érték minden esetben egy. A következő lépésben a reduceByKey metódussal kulcsonként csoportosítva összeadjuk az értékeket, így kapjuk meg, hogy 1 szó pontosan hányszor szerepelt a szövegben. A szó - szószám párokból álló listát végül HDFS-re mentjük.  

## Vezetett rész

### 0. Feladat - környezet elérése

A labor során a Cloudera Hadoop disztribúcióját fogjuk használni, amely egyetlen virtuális gépen, a [Cloudera Quickstart VM](http://www.cloudera.com/downloads/quickstart_vms/5-5.html)-en fog futni.
Ebben a környezetben a legfontosabb komponensek mind elérhetők, ezzel jár, hogy a virtuális gép viszonylag sok erőforrást igényel.
A virtuális gép indítása előtt ellenőrizzük, hogy legalább 6 GB memória, illetve 2 CPU mag allokálásra került-e a gép számára.

Ha a gép elindult a Hue a következő címen érhető el: `10.0.2.15:8888`.
A virtuális gépre általánosságban igaz, hogy ahol felhasználónév/jelszó párost kér, ott a `cloudera`/`cloudera` értékek használhatók.

A VM indítása után külön el kell indítanunk az Apache NiFi servicet is.
Annak érdekében, hogy az Apache NiFi megfelelően működjön a virtuális gépen elérhető viszonylag szűkös erőforrások mellett is, módosítsuk a `conf/bootstrap.conf` konfigurációs fájlt olyan módon, hogy a 48-53 sor elejéről távolítsuk el a komment jeleket.
Nyissunk meg egy terminált, és adjuk ki a következő parancsot: `/home/cloudera/Desktop/nifi-0.7.2/bin/nifi.sh start`!
Ezzel elindítottuk az Apache Nifit, amely a `localhost:8080/nifi` címen elérhető webes felületen keresztül konfigurálható.
A laborvezető segítségével ismerkedjünk meg a felülettel.

### 1. Feladat - adatbetöltés Apache NiFivel

A repository `data` mappájában megtalálhatunk három adathalmazt, amelyet a [http://movielens.org](http://movielens.org) oldalon található filmadatbázisból, és a hozzá tartozó értékelésekből nyertek ki.
A labor során ezekkel az adathalmazokkal fogunk dolgozni.
A könnyebb munka érdekében töltsük le a teljes repository tartalmát, és másoljuk át a `data` könyvtár tartalmát az asztalra.

#### 1.1 Feladat - Movies dataset betöltése

Az első betöltendő adathalmaz néhány népszerű film adatait tartalmazza.
Apache Nifi használatával töltsük be a fájl tartalmát HDFS-re, a `/user/cloudera/movies` mappába.

Ehhez egy egyszerű workflowt fogunk létrehozni, amely mindössze két Processorból áll: az adatok beolvasásárért a `GetFile`, míg a HDFS-re helyezésért a `PutHDFS` Processor a felelős.
Konfiguráljuk be ezeket úgy, hogy a GetFile a következő mappát figyelje: `/home/cloudera/Desktop/raw/movies`.
Ez a mappa egyelőre nem létezik, ezért hozzuk is létre, majd helyezzük el benne a `movies.dat` fájl egy *másolatát*.
(A `GetFile` Processor törli a beolvasott fájlokat, ezért fontos az, hogy a másolatot helyezzük el az adott helyen.)

A PutHDFS Processor konfigurálása során meg kell adnunk a HDFS eléréséhez szükséges konfigurációs fájlokat, ezért a Hadoop Configuration Resources mezőbe írjuk be a következő értéket: `/etc/hadoop/conf/core-site.xml,/etc/hadoop/conf/hdfs-site.xml`.
A cél mappát is állítsuk be megfelelően: `/user/cloudera/movies`.

A flow elindításával a fájlok rögtön áthelyezésre is kerülnek, ellenőrizzük ezt le a Hue segítségével.

*Ellenőrzés:* A jegyzőkönyvben helyezzen el egy képet a létrejött flowról, illetve arról, hogy a Hueban látszódik az újonnan létrehozott fájl.

#### 1.2 Feladat - Ratings dataset betöltése

A filmek értékelését tartalmazó adathalmazt is be kell tölteni, azonban ha vetünk egy pillantást a `ratings.dat` fájlra, láthatjuk, hogy itt a `!` karaktersorozat választja el a sorok egyes mezőit. Ez a későbbiekben problémákhoz vezethet, így a betöltés során cseréljük le ezt a karaktersorozatot a `,` karakterre.

Annak érdekében, hogy átláthatóbb legyen a NiFi Flow konfigurációnk, hozzunk létre egy új Process Groupot, ahova bemásoljuk az eddigi Processorokat.
Ezen kívül hozzunk létre egy másik Process Groupot is, az aktuális feladat számára.

Itt is hasonló megoldást fogunk követni, mint az előzőekben, azonban ki fogjuk azt egészíteni egy új elemmel, amely a bejövő adatokban lévő `!` karaktereket lecseréli `,` karakterekre.
Ezt a cserét nagyon egyszerűen megoldhatjuk a `ReplaceText` Processor segítségével.

A `ReplaceText` Processor konfigurációja során figyeljünk arra, hogy az `Evaluation Mode` értéke soronkénti, míg a `Replacement Strategy` értéke `Literal Replace` legyen!

A laborvezető segítségével állítsuk össze ezt a Flowt is, majd ellenőrizzük le a kapott eredményt!

*Ellenőrzés:* A jegyzőkönyvben helyezzen el egy képet a létrejött flowról, illetve arról, hogy a Hueban látszódik az újonnan létrehozott fájl.

### 2. Feladat - Hive lekérdezés az adatokon

#### Táblák létrehozása

A táblákat externalként hozzuk létre, a sémát az adatfájloknak megfelelően adjuk meg. Mivel a `movies.dat` adatfájl a genre mezőben több értéket is tárol, ez remek alkalom a Hive összetett adattípusainak kipróbálására.
Jelen esetben `ARRAY<STRING>` típusként vesszük fel ezt a mezőt. A tömb elemeit elválasztó karaktert a `COLLECTION ITEMS TERMINATED BY '|'` kulcsszavakkal definiáljuk.
Az adatok helyét nem a korábban ismertetett módon adjuk meg, hanem egy külön paranccsal töltjük be.
Erre azért van szükség mert a `LOCAION` paraméteréül csak mappa adható meg.

```
CREATE EXTERNAL TABLE movies(id INT, title STRING, genre ARRAY<STRING>)
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY '\001'
COLLECTION ITEMS TERMINATED BY '|'
STORED AS TEXTFILE;

LOAD DATA INPATH '/user/cloudera/movies' INTO TABLE movies;
```

A ratings táblánál nincs szükség összetett adattípus használatára így létrehozása egyszerűbb, de az előzőhöz teljesen hasonló.

```
CREATE EXTERNAL TABLE ratings(userid INT, movieid INT, rating INT, timestamp INT)
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY '!'
STORED AS TEXTFILE;

LOAD DATA INPATH '/user/cloudera/ratings' INTO TABLE ratings;
```

#### Néhány egyszerű lekérdezés

Akciófilmek listája:
```
SELECT * FROM movies WHERE array_contains(genre, "Action");
```

Értékelések eloszlása:
```
SELECT rating, count(*) FROM ratings GROUP BY rating;
```

### 3. Feladat - Spark analitika

A Spark segítségével tetszőleges kódot írhatunk és futtathatunk az adatainkon, így jóval rugalmasabb mint a Hive, de egyszerű példáknál sok átfedés van a két eszköz tudása közt. Ezt szemléltetendő elkészítjük az előző feladat Hive-os példáit Spark segítségével is. 

Akciófilmek listája (Java 8):
```
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;

public class SparkActionMovieCount {

    public static void main(String[] args) throws Exception {
        if (args.length < 2) {
            System.err.println("Usage: SparkActionMovieCount <input-file> <output-folder>");
            System.exit(1);
        }

        final String outputPath = args[1];
        SparkConf sparkConf = new SparkConf().setAppName("SparkActionMovieCount").setMaster("local");
        JavaSparkContext ctx = new JavaSparkContext(sparkConf);

        // line example: 10GoldenEye (1995)Action|Adventure|Thriller
        JavaRDD<String> lines = ctx.textFile(args[0], 1);

        JavaRDD<String> actionMovies = lines.filter(x -> x.substring(x.lastIndexOf("\u0001")).contains("Action"));

        actionMovies.saveAsTextFile(outputPath);

        ctx.stop();
    }
}
```


(Java 7):
```
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.Function;

public class SparkActionMovieCount {

	public static void main(String[] args) {
		if (args.length < 2) {
			System.err.println("Usage: SparkActionMovieCount <input-file> <output-folder>");
			System.exit(1);
		}

		final String outputPath = args[1];
		SparkConf sparkConf = new SparkConf().setAppName("SparkActionMovieCount").setMaster("local");
		JavaSparkContext ctx = new JavaSparkContext(sparkConf);

		// line example: 10�GoldenEye (1995)�Action|Adventure|Thriller
		JavaRDD<String> lines = ctx.textFile(args[0], 1);

		JavaRDD<String> actionMovies = lines.filter(new Function<String, Boolean>() {
			public Boolean call(String s) { return s.substring(s.lastIndexOf("\u0001")).contains("Action"); }
		});

		actionMovies.saveAsTextFile(outputPath);

		ctx.stop();

	}

}

```

A megoldás alapgondolata, hogy a forrás adat beolvasását követően egy szűrést alkalmazunk, amivel eldobjuk azokat a sorokat amikben nem szerepel az Action mint kategória. Az eredményül kapott RDD-t csak el kell mentenünk és kész is vagyunk.

Értékelések eloszlása (Java8):
```
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import scala.Tuple2;

public class SparkRatingsCount {

    public static void main(String[] args) throws Exception {
        if (args.length < 2) {
            System.err.println("Usage: SparkRatingsCount <input-file> <output-folder>");
            System.exit(1);
        }

        final String outputPath = args[1];
        SparkConf sparkConf = new SparkConf().setAppName("SparkRatingsCount").setMaster("local");
        JavaSparkContext ctx = new JavaSparkContext(sparkConf);

        // line example: 1!2804!5!978300719
        JavaRDD<String> lines = ctx.textFile(args[0], 1);

        JavaPairRDD<String, Integer> ratingOnePairs = lines.mapToPair(s -> new Tuple2<>(s.split("!")[2], 1));

        JavaPairRDD<String, Integer> results = ratingOnePairs.reduceByKey((i1, i2) -> i1 + i2);

        results.saveAsTextFile(outputPath);

        ctx.stop();
    }
}
```

Java 7:

```
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFunction;

import scala.Tuple2;

public class SparkRatingsCount {

    public static void main(String[] args) throws Exception {
        if (args.length < 2) {
            System.err.println("Usage: SparkRatingsCount <input-file> <output-folder>");
            System.exit(1);
        }

        final String outputPath = args[1];
        SparkConf sparkConf = new SparkConf().setAppName("SparkRatingsCount").setMaster("local");
        JavaSparkContext ctx = new JavaSparkContext(sparkConf);

        // line example: 1!2804!5!978300719
        JavaRDD<String> lines = ctx.textFile(args[0], 1);

        JavaPairRDD<String, Integer> ratingOnePairs = lines.mapToPair(new PairFunction<String, String, Integer>() {
			public Tuple2 call(String s) { return new Tuple2<>(s.split("!")[2], 1); }
		});

        JavaPairRDD<String, Integer> results = ratingOnePairs.reduceByKey(new Function2<Integer, Integer, Integer>() {
        	public Integer call(Integer i, Integer j) { return i + j; }  
        });

        results.saveAsTextFile(outputPath);

        ctx.stop();
    }
}
```

A feladat megoldása csak egy hangyányit bonyolultabb mint az előző esetben. Beolvassuk a forrást, majd a bemenet sorait kulcs - érték párokká mappeljük. Ezekben a párokban a kulcs maga az értékelés, az érték pedig egy darab 1-es. Innentől a korábban bemutatott wordcount-hoz hasonlóan kulcs alapján összeadjuk az értékeket és így megkapjuk, hogy melyikből mennyi van.


## Önálló feladatok

### 1. Feladat - Users dataset betöltése Apache NiFi segítségével
Töltsük be a `users` adatállományt is a HDFS `/user/cloudera/users` mappába!
A betöltés során szűrjük ki a 18 év alatti felhasználókat.
Az adatszerkezet leírása jelen repository `data/README` fájljában található.

Tippek:
1. A bemenő fájlt soronként érdemes feldolgozni, ehhez hasznos lehet a `SplitText` Processor.
2. A sorokra bontott fájlt szűrés után érdemes újra összefűzni, hiszen a HDFS nagyméretű fájlokra van optimalizálva.
3. A `GetFile` Processor a FlowFileok `filename` attribútumában eltárolja a bemenő fájl nevét. A `PutHDFS` Processor ezt az attribútumot használja a fájl mentéséhez, azonban ha ütközés lép fel, azt nem tudja megfelelően kezelni. Érdemes ezért a `filename` attribútum értékét megváltoztatni olyan módon, hogy az minden FlowFile esetén egyedi legyen. (Használjuk ehhez az Apachi NiFi expression language ${nextInt()} kifejezését.)

*Ellenőrzés:* A jegyzőkönyvben helyezzen el egy képet a létrejött flowról, illetve arról, hogy a Hueban látszódik az újonnan létrehozott fájl. A jegyzőkönyvben jelenjenek meg az egyes Processorok konfigurációi is.

### 2. Feladat - Bonyolultabb Hive lekérdezések
Készítsen external adattáblát az előző feladatban betöltött felhasználói adatokhoz.

Írjon egy lekérdezést, amely kiírja a 10 legtöbbet értékelt film címét, azonosítóját és a rá érkezett értékelések számát!

Írjon egy lekérdezést, amely kiírja a 10 legtöbb 1-es osztályzattal értékelt film címét, azonosítóját és a rá érkezett 1-es értékelések számát!

Elfogadható, de kisebb értékű megoldás, ha a filmek címét nem, csak az azonosítóját jeleníti meg.

### 3. Feladat - Spark programozás

Írjon Spark programot, ami egy fájlba kiírja az egyedi userek számát a ratings.dat adatok alapján.

(opcionális) Írjon Spark programot, amely a ratings.dat adatok alapján megadja, hogy egyes felhasználók átlagosan milyen értékeléseket adtak.

Segítség: [Spark programming guide](http://spark.apache.org/docs/latest/programming-guide.html)

