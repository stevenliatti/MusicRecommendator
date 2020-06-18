# Music Recommendator

Ce projet Spark démarre avec le chapitre 3 de [Advanced Analytics with Spark, 2nd Edition](https://www.oreilly.com/library/view/advanced-analytics-with/9781491972946/) au sujet d'un "recommendeur" d'artistes musicaux. Nous avons en partie suivi les idées de ce chapitre, mais avons également exploré d'autres pistes, en cherchant également à augmenter notre jeu de données. Le notebook Zeppelin contenant toutes les données et le code se trouve [ici](notebooks/2FA8UEGZS/note.json).

# Description de l'ensemble des données

## Données initiales provenant du livre

Les données initiales, fournies par le livre, comportent trois fichiers : `user_artist_data.txt`, `artist_data.txt` et `artist_alias.txt`. Le premier fichier contient environ 24.3 millions de lignes et trois colonnes, la 1ère contenant un id d'utilisateur (qui écoute de la musique), la 2ème un id d'artiste et la 3ème le nombre de fois que cet utilisateur a écouté cet artiste.

```bash
$ head user_artist_data.txt
1000002 1 55
1000002 1000006 33
1000002 1000007 8
1000002 1000009 144
1000002 1000010 314
1000002 1000013 8
1000002 1000014 42
1000002 1000017 69
1000002 1000024 329
1000002 1000025 1
```

Le deuxième fichier contient un peu plus que 1.8 millions de lignes et deux colonnes, la 1ère contenant un id d'artiste et la 2ème le nom d'artiste associé à l'id.

```bash
$ head artist_data.txt
1134999	06Crazy Life
6821360	Pang Nakarin
10113088	Terfel, Bartoli- Mozart: Don
10151459	The Flaming Sidebur
6826647	Bodenstandig 3000
10186265	Jota Quest e Ivete Sangalo
6828986	Toto_XX (1977
10236364	U.S Bombs -
1135000	artist formaly know as Mat
10299728	Kassierer - Musik für beide Ohren
```

Le troisième fichier contient 193'027 lignes et deux colonnes, la 1ère contenant un id d'artiste (au nom mal orthographié) et la 2ème l'id "canonique" pour cet artiste (au nom correctement orthographié).

```bash
$ head artist_alias.txt
1092764	1000311
1095122	1000557
6708070	1007267
10088054	1042317
1195917	1042317
1112006	1000557
1187350	1294511
1116694	1327092
6793225	1042317
1079959	1000557

```

## Données provenant de MusicBrainz

Nous voulions réaliser de nombreuses statistiques sur ces données en tirant profit de la puissance de Spark et SparkSQL, et même si la quantité des données est plutôt conséquente, le nombre d'informations différentes n'est pas énorme. Pour rester dans le thème, nous avons décidé de travailler sur les données provenant de [MusicBrainz](https://musicbrainz.org/), une source de données libre sur la musique et artistes. À partir de [cette description](https://musicbrainz.org/doc/MusicBrainz_Database/) des données, nous avons sélectionné les tables `artist`, `artist_type`, `gender`, `artist_tag`, `tag`, `area` et `area_type` que nous avons exporté en CSV pour les manipuler plus facilement dans Zeppelin. Ces différentes tables donnent des infos sur les artistes, comme leur nom, type (groupe, artiste seul, etc.), sexe (si applicable), indication (booléen) et dates de début et fin d'activité, zone géographique de provenance et des tags (genres) associés.

# Description des *features* utilisées
user id, artist id, count pour ALS
begin_date_year pour algos de clustering

# Questions pour lesquelles vous espérez obtenir une réponse à partir de l'analyse

# Algorithmes appliqués
K-means ?
- Clustering sur les gens qui écoutent les mêmes morceaux
- Tentative de graphe : les artistes et les users sont des noeuds et les écoutes sont des arcs


# Optimisations effectuées
Nous avons nettoyé les données "originales" en supprimant les noms d'artistes inconnus et les entrées d'écoute qui ne sont pas temporellement possibles : si on prend l'utilisateur avec l'id 2064012, nous aperçevons qu'il a écouté System Of A Down 439'771 fois. Un morceau de musique dure environ 4 minutes en moyenne, ce qui correspond dans notre cas à une écoute ininterrompue de 33 ans. Non seulement c'est une durée incroyable de manière absolue pour un temps d'écoute mais en plus le groupe System Of A Down a été créé dans les années 1990, ce qui confirme l'infaisabilité de ce cas.

<!-- # Your approach to testing and evaluation -->

# Résultats obtenus

## Statistiques sur les données originales

Nous avons réalisé de nombreuses statistiques sur les données, nous allons les présenter sous forme de tableaux et de graphiques.

### Fusion des données
Nous avons commencé par réaliser un Dataframe Spark, nommé `userArtistNamesCountDF`, avec les utilisateurs (ids), artistes (id et noms) et nombre d'écoutes que nous allons utiliser abondamment.

```scala
val userArtistNamesCountDF = 
    trainData
    .join(artistByID, artistByID("id") === trainData("artist"))
    .select("user", "artist", "name", "count")
userArtistNamesCountDF.cache
userArtistNamesCountDF.show
```
|   user|artist|      name|count|
|--------|-----|----------|----------|
|1000019|   463|The Smiths|    1|
|1000020|   463|The Smiths|  199|
|1000022|   463|The Smiths|   20|
|1000033|   463|The Smiths|  466|
|1000056|   463|The Smiths|   10|
|1000067|   463|The Smiths|   18|

### À propos des utilisateurs
Nous avons sommé le total des écoutes par utilisateur et retenu les 20 premiers par ordre décroissant d'écoutes :

|   user| total|
|-------|------|
|2069337|393515|
|2023977|285978|
|1059637|241350|
|1046559|183972|
|1052461|175822|
|1070932|168977|
|1031009|167028|
|2020513|165642|
|2062243|151482|
|2069889|143092|
|1001440|136266|
|2014936|135235|
|2017397|134032|
|1024631|122303|
|1007308|111274|
|2064012|108640|
|2023742|102030|
|1058890| 98472|
|1021940| 97318|
|1059245| 96037|

Nous pouvons voir que le premier a réalisé presque 400'000 écoutes cumulées ! Quelle attention !

Nous pouvons constaté qu'un utilisateur (1059637) a écouté "My Chemical Romance" presque 156'000 fois. Un vrai fan.

|   user|                name| total|
|-------|--------------------|------|
|1059637| My Chemical Romance|155895|
|2069889| Something Corporate|101076|
|2020513|      Mates of State| 89592|
|1073421|    Guided by Voices| 67548|
|2023977|            Maroon 5| 62815|
|2023977|   Alanis Morissette| 51039|
|2014936|          Pink Floyd| 36083|
|2069337|        Sage Francis| 34800|
|2013784|     Cannibal Corpse| 32768|
|2069337|          Jawbreaker| 31321|
|1073435|ASIAN KUNG-FU GEN...| 30043|
|2023977|          Guano Apes| 29983|
|1052225|               浜崎あゆみ| 29933|
|2017397|     New Found Glory| 26394|
|1045479|         The Beatles| 26135|
|2062243|           Music 205| 26107|
|1039100|              R.E.M.| 25858|
|1053554|    Boards of Canada| 25303|
|2017397|              Thrice| 24876|
|2216281|           Rammstein| 24067|

Voyons les moyennes d'écoutes sur les 148'077 utilisateurs différents : 

|summary|             count|
|-------|------------------|
|  count|            148077|
|   mean|162.66439757693632|
| stddev| 215.3813187758636|
|    min|                 1|
|    max|              6734|

Nous voyons qu'en moyenne, un utilisateur a écouté 162 artistes différents.

### À propos des utilisateurs
Nous avons dressé le top 20 des artistes les plus écoutés sur les 1'554'191 artistes au total.

|                name|  total|
|--------------------|-------|
|           Radiohead|2502596|
|         The Beatles|2259825|
|           Green Day|1931143|
|           Metallica|1543430|
|          Pink Floyd|1399665|
|     Nine Inch Nails|1361977|
|        Modest Mouse|1328969|
|         Bright Eyes|1234773|
|             Nirvana|1203348|
|                Muse|1148684|
| Death Cab for Cutie|1117277|
|Red Hot Chili Pep...|1088701|
|       Elliott Smith|1080542|
|           Rammstein|1047119|
|         Linkin Park|1028921|
|                  U2|1015494|
|           Nightwish|1010869|
|            Coldplay|1001417|
|    System of a Down| 986483|
|            Interpol| 979770|

Nous voyons, sans surprises, que des groupes mythiques s'y trouvent, comme les Beatles ou Metallica.
A l'inverse, nous avons également sélectionné les 20 moins bons :

|                name|total|
|--------------------|-----|
|Joe Budden, A Tea...|    1|
|Rae & Chritian - ...|    1|
|Alex Melcher, Peh...|    1|
|Dolly Parton - Mi...|    1|
|        Oneiropagida|    1|
|          Bravissimo|    1|
|The Middle Spunk ...|    1|
|The Last Emperor ...|    1|
|      12 Urban Tribe|    1|
|3EO35A5QUKMQXL3WV...|    1|
|         Jon Eberson|    1|
|gwar - Preskool P...|    1|
|Ayla-Liebe-Atb Remix|    1|
|macy gray (bugz i...|    1|
|Boys to Men & Cha...|    1|
|Apani B. Fly Emce...|    1|
|  Makaveli ft. Storm|    1|
|       Opération raï|    1|
|Parle ft jadakiss...|    1|
|Lil Kim, Mya, Pin...|    1|

On constate que ce tableau ne donne pas beaucoup d'informations, de nombreux artistes ont une seule écoute.
Voyons ce qu'il en est plus en détails. Regardons combien d'artistes ont été écoutés moins de 10 fois : 

```scala
val lessThanTenTimesListenedArtistsDF = listenedArtistsDF.filter("total <= 10")
val count = lessThanTenTimesListenedArtistsDF.count
val ratio = count.toDouble / distinctArtistsNumber.toDouble
```

```
count: Long = 1161120
ratio: Double = 0.7470896434222049
```

Plus qu'un million d'artistes ont été écoutés moins que 10 fois, cela représente quand même 75% des artistes.
Voyons voir la distribution entre 1 et 10 écoutes :

|total| count|
|-----|------|
|    1|494512|
|    2|220020|
|    3|127612|
|    4| 86240|
|    5| 62579|
|    6| 48704|
|    7| 38467|
|    8| 32332|
|    9| 27004|
|   10| 23650|

![lessThanTenTimesListenedArtists](doc/images/lessThanTenTimesListenedArtists.png)

On voit que la plupart des artistes ont été écoutés qu'un seule fois. Voyons si la médianne confirme nos dires :

```scala
val mean = listenedArtistsDF.agg(avg("total")).head.getDouble(0)
val median = listenedArtistsDF.stat.approxQuantile("total", Array(0.5), 0.01)(0)
```

```
mean: Double = 238.32715067690853
median: Double = 3.0
```

La moyenne est à 238 tandis que la médianne est 3, ce qui confirme le graphique précédent, ce qui veut dire que la moitié des artistes ont été écoutés moins de trois fois, ce qu'on pourrait assimiler à des découvertes ou même à des erreurs d'écoutes.

### À propos des noms d'artistes mal orthographiés
Voyons quels artistes sont le moins bien orthographiés. On commence par réaliser une `Map[Int,Array[Int]]` faisant correspondre un id d'artiste "correct" à la liste des ids pointant vers des noms écorchés :

```scala
val aliasToListWrongNames = rawArtistAlias.flatMap { line =>
    val Array(artist, alias) = line.split('\t')
    if (artist.isEmpty) None
    else Some((alias.toInt, artist.toInt))
}
.collect
.groupBy { case (k, v) => k }
.map { case (k, v) => (k, v.map { case (_, v2) => v2 } ) }

val artistsWithMostWrongAlias = aliasToListWrongNames.map { case (k,v) => (k, v.length) }.toList.toDF("artist", "count")
```

|summary|             count|
|-------|------------------|
|  count|             22478|
|   mean| 8.492392561615802|
| stddev|28.895991802786074|
|    min|                 1|
|    max|              1204|


| artist|count|
|-------|-----|
|1000024| 1204|
|1034635|  955|
|     82|  671|
|1003694|  656|
|   1854|  656|
|1000113|  629|
|    930|  628|
|    979|  620|
|1000107|  537|
|    976|  536|
|   1182|  527|
|   1205|  451|
|   1274|  450|
|   4061|  424|
|1001646|  424|
|1000323|  413|
|2003588|  390|
|1256375|  385|
|1247272|  373|
|1008093|  365|

On constate qu'il y 22'478 noms d'artistes (~ 1.5% des artistes) mal orthographiés et que le "pire" nom d'artiste est mal orthographié 1024 fois. Voyons de qui il s'agit :

```scala
artistByID.select("name").filter("id == 1000024").show
```

|     name|
|---------|
|Metallica|

Oh non ! C'est Metallica ! Comment est-ce possible ? Voyons quelques échantillons des 1024 noms alternatifs :

```scala
val aliasIdsToMetallica = aliasToListWrongNames.getOrElse(1000024, Array()).toList.toDF("id")
val aliasNamesToMetallica = aliasIdsToMetallica.join(artistByID, artistByID("id") === aliasIdsToMetallica("id"))
aliasNamesToMetallica.show(false)
```

|id      |id      |name                                           |
|--------|--------|-----------------------------------------------|
|1240322 |1240322 |Metallica & The SF Symphony Orchestra          |
|10113173|10113173|Metallica - B&P - 14                           |
|10024375|10024375|Metallica - B&P - 21                           |
|10024376|10024376|Metallica - ...And Justice For All -           |
|10024488|10024488|Metallica - B&P - 24                           |
|1136616 |1136616 |30B10001806F00287800.Metallica and The SF      |
|7004497 |7004497 |Metallica & San-Francisco-Orchester            |
|10025794|10025794|Metallica - Live in Belgrade 26.6              |
|10186355|10186355|Metallica Some Kind of Monster                 |
|6945944 |6945944 |03. Metallica                                  |
|1246630 |1246630 |Metallica & Michael Kamen                      |
|1138243 |1138243 |Metallica -[Disc 2(10)                         |
|6946149 |6946149 |Metallica -1987- Garage Days                   |
|1247944 |1247944 |Metallica 06                                   |
|6672703 |6672703 |Metallica -1998- Garage Inc,disc I             |
|9934619 |9934619 |Metallica & San Fran                           |
|7031937 |7031937 || METALLICA                                    |
|6991351 |6991351 |Metallica (w                                   |
|2038418 |2038418 |METALLICA@Apoptygma Berzerk                   |
|9918437 |9918437 |Metallica - Live 2004-05-28 - Helsinki, Finland|

On réalise que ce sont plus des chaines de caractères contenant le nom "Metallica" dedans avec souvent plus de détails que réellement des erreurs d'orthographe.

## Statistiques sur les données de MusicBrainz

Nous avons commencé par lire les données en CSV dans des Dataframes et grâce à des joins successifs, nous obtenons ce Dataframe : 

```scala
val mbArtistTypeGenderAreaDF = 
    mbArtistCleanDF
    .join(mbArtistTypeDF.select($"id", $"name".alias("type")), mbArtistCleanDF("type_id") === mbArtistTypeDF("id"))
    .join(mbGenderDF.select($"id", $"name".alias("gender")), mbArtistCleanDF("gender_id") === mbGenderDF("id"))
    .join(mbAreaFullDF, mbArtistCleanDF("area_id") === mbAreaFullDF("id"))
    .select(mbArtistCleanDF("id"), $"name", $"type", $"gender", $"area", $"area_type", $"ended", $"begin_date_year", $"begin_date_month", $"begin_date_day", $"end_date_year", $"end_date_month", $"end_date_day").cache
```

|                name|  type|gender|         area|  area_type|ended|begin_date_year|begin_date_month|begin_date_day|end_date_year|end_date_month|end_date_day|
|--------------------|------|------|-------------|-----------|-----|---------------|----------------|--------------|-------------|--------------|------------|
|    Margaret Hendrie|Person|Female|        Nauru|    Country|    t|           1924|            null|          null|         1990|          null|        null|
|       Robin Freeman|Person|  Male|Noord-Brabant|Subdivision|    f|           1952|            null|          null|         null|          null|        null|
|       Lord of Sp33d|Person|  Male|Noord-Brabant|Subdivision|    f|           null|            null|          null|         null|          null|        null|
|  Michèle van der Aa|Person|Female|Noord-Brabant|Subdivision|    f|           1982|               9|             9|         null|          null|        null|
|      Thomas Pieters|Person|  Male|Noord-Brabant|Subdivision|    f|           null|            null|          null|         null|          null|        null|
|         Lya de Haas|Person|Female|Noord-Brabant|Subdivision|    f|           1958|              11|             6|         null|          null|        null|
|      The Lapin King|Person|  Male|   Norrbotten|Subdivision|    f|           null|            null|          null|         null|          null|        null|
|Maria Casandra Hauși|Person|Female|    Maramureș|Subdivision|    f|           null|               3|            11|         null|          null|        null|
|       Prince Pronto|Person|  Male| San Fernando|       City|    f|           1997|              12|             7|         null|          null|        null|
|    Friederike Brion|Person|Female|       Alsace|Subdivision|    t|           1752|               4|            19|         1813|             4|           3|
|       Ernst Stadler|Person|  Male|       Alsace|Subdivision|    t|           1883|               8|            11|         1914|            10|          30|
|Christine le Ross...|Person|Female|       Alsace|Subdivision|    f|           null|            null|          null|         null|          null|        null|
|  Guillaume Schleret|Person|  Male|       Alsace|Subdivision|    f|           null|            null|          null|         null|          null|        null|
|      Lionel Bascole|Person|  Male|       Alsace|Subdivision|    f|           null|            null|          null|         null|          null|        null|
|    Guillaume Fleith|Person|  Male|       Alsace|Subdivision|    f|           null|            null|          null|         null|          null|        null|
|          Fred Drill|Person|  Male|       Alsace|Subdivision|    f|           null|            null|          null|         null|          null|        null|
|       Eugène Maegey|Person|  Male|       Alsace|Subdivision|    f|           null|            null|          null|         null|          null|        null|
|       Léopoldine HH|Person|Female|       Alsace|Subdivision|    f|           null|            null|          null|         null|          null|        null|
|                  Go|Person|  Male|       Alsace|Subdivision|    f|           null|            null|          null|         null|          null|        null|
|        James Taplin|Person|  Male|    Doncaster|Subdivision|    f|           1988|               9|            12|         null|          null|        null|

Voyons les types d'artistes : 


|     type| count|
|---------|------|
|    Group|392098|
|   Person|889647|
|Character|  7077|
|    Other|  2291|
|    Choir|  5932|
|Orchestra|  6612|

Sur les 1.6 millions présents, 1.3 ont un attribut de type, qui indique si l'artiste est seul ou en groupe. On peut voir que les types principaux sont "Person" et "Group", ce qui est raisonnable.

Voyons combien d'artistes sont encore présents sur la scène musicale (actifs) ou non (inactifs) :

|                name|ended|begin_date_year|
|--------------------|-----|---------------|
| Thirteen Over Eight|    f|           null|
|         Dr. I-Bolit|    f|           null|
|             Astolat|    f|           null|
|         Pete Moutso|    f|           null|
|   The Insignificant|    f|           null|
|        Aric Leavitt|    f|           null|
|       The Wanderers|    f|           null|
|           Al Street|    f|           null|
|     Andrew Greville|    f|           null|
|          Sintellect|    f|           null|
|      Project/Object|    f|           null|
|  Jean-Pierre Martin|    f|           null|
|          Imagimusic|    f|           null|
|wecamewithbrokent...|    f|           null|
|Disappointment In...|    f|           null|
|          Giant Tomo|    f|           null|
|Elvin Jones & Jim...|    f|           null|
|          Stereobate|    f|           null|
|          Diskobitch|    f|           null|
|  Sailing Conductors|    f|           null|

|                name|ended|begin_date_year|end_date_year|
|--------------------|-----|---------------|-------------|
|        The Vaqueros|    t|           null|         null|
|Biljarten na half...|    t|           1988|         null|
|    The Acid Gallery|    t|           null|         null|
|             Warhate|    t|           1996|         null|
|    Sonny Clark Trio|    t|           null|         null|
|       Out of Hatred|    t|           null|         null|
|     The Third Bardo|    t|           1967|         null|
|               Karna|    t|           1997|         null|
|               Reign|    t|           1996|         null|
|   Vijay Deverakonda|    t|           1989|         null|
|De Zingende Zwervers|    t|           1951|         null|
|       The Pinafores|    t|           1945|         null|
|     Swarm of Angels|    t|           null|         null|
|      Extreme Hatred|    t|           1992|         null|
|Queen Sarah Saturday|    t|           1990|         null|
|    Day of the Sword|    t|           null|         null|
|       Six Feet Deep|    t|           1991|         null|
|               PanAm|    t|           2000|         null|
|            On Wings|    t|           1994|         null|
|        Taiconderoga|    t|           null|         null|


La majorité des artistes est active, avec 1'559'972 qui sont actifs et 99'488 qui ne le sont pas.

Voyons maintenant le sexe des artistes :

|        gender| count|
|--------------|------|
|        Female|147705|
|         Other|   813|
|Not applicable|   435|
|          Male|529926|

On voit que la majorité des artistes de cette base de données sont des hommes.


Maintenant, tirons profit des jointures pour répondre à la question suivante : Combien d'artistes féminines anglaies sont ou ont été actives depuis 1986 ?

```scala
val femaleArtistEnglish = mbArtistTypeGenderAreaDF
    .filter($"type" === "Person")
    .filter($"gender" === "Female")
    .filter($"area" === "England")
    .filter("begin_date_year >= 1986")
    .select("name", "ended", "begin_date_year", "end_date_year")
    .sort("begin_date_year")

println(femaleArtistEnglish.count)
femaleArtistEnglish.show(30)
```

|                name|ended|begin_date_year|end_date_year|
|--------------------|-----|---------------|-------------|
|      Ellie Goulding|    f|           1986|         null|
|       Jenna Coleman|    f|           1986|         null|
|         Lowri Dixon|    f|           1987|         null|
|      Betty Kendrick|    f|           1987|         null|
|     Jess McAllister|    f|           1987|         null|
|   Anna Clare Wilson|    f|           1988|         null|
|          Lily James|    f|           1989|         null|
|         Emma McGann|    f|           1990|         null|
|       Jasmine Chloe|    f|           1991|         null|
|      Ruth Patterson|    f|           1992|         null|
|          Tek Notice|    f|           1993|         null|
|                Zyra|    f|           1994|         null|
|Catherine Ward-Th...|    f|           1994|         null|
|        Sophie Heard|    f|           1994|         null|
|          HM Silvers|    f|           1994|         null|
|Alexandra Catheri...|    f|           1994|         null|
|   Lizzy Ward Thomas|    f|           1994|         null|
|        Orla O'Neill|    f|           1996|         null|
|             Låpsley|    f|           1996|         null|
|      Megan Lara Mae|    f|           1996|         null|
|                Maya|    f|           1996|         null|
|             emellia|    f|           1997|         null|
|              Ivy HB|    f|           1997|         null|
|          Florescent|    f|           1998|         null|
|      Noella Usborne|    f|           1999|         null|
|          Sarah Kate|    f|           1999|         null|
|        Erin Bloomer|    f|           2002|         null|
|     Stephanie Gaunt|    f|           2002|         null|

Nous en avons "seulement" exactement 28, toutes actives encore aujourd'hui.

Voyons maintenant la provenance des artistes :

|          area| count|
|--------------|------|
| United States|141078|
|United Kingdom| 58886|
|         Japan| 52228|
|       Germany| 51264|
|        France| 29410|
|       Belgium| 20292|
|         Italy| 19744|
|       Finland| 16634|
|        Sweden| 16521|
|        Canada| 15582|
|         Spain| 14111|
|     Australia| 12850|
|   Netherlands| 12542|
|        Russia|  9410|
|       Estonia|  7769|
|        Greece|  7591|
|       Denmark|  7560|
|        Poland|  7131|
|        Brazil|  7009|
|       Austria|  6993|

On peut voir que les USA sont largement en tête.

Regardons quels artistes aujourd'hui inactifs ont duré le plus de temps, en années :

```scala
val longers = 
    mbArtistCleanDF
    .filter($"ended" === "t")
    .filter($"begin_date_year" !== "null")
    .filter($"end_date_year" !== "null")
    .select("name", "begin_date_year", "end_date_year")
    .withColumn("duration", col("end_date_year") - col("begin_date_year"))
    .filter($"end_date_year" <= 2020)
    .filter($"duration" < 100)
    .orderBy(desc("duration"))
```

|                name|begin_date_year|end_date_year|duration|
|--------------------|---------------|-------------|--------|
|         Gaby Basset|           1902|         2001|    99.0|
|          Franz Thon|           1910|         2009|    99.0|
|     Jester Hairston|           1901|         2000|    99.0|
|     Bernice Petkere|           1901|         2000|    99.0|
|        Sir Lancelot|           1902|         2001|    99.0|
|Eduardo Hernández...|           1911|         2010|    99.0|
|    Manuel Rosenthal|           1904|         2003|    99.0|
|  Marie-Louise Girod|           1915|         2014|    99.0|
|      Nimrod Workman|           1895|         1994|    99.0|
| Mary Elizabeth Frye|           1905|         2004|    99.0|
|"Carlos Alberto F...|           1907|         2006|    99.0|
|  Jonathan Sternberg|           1919|         2018|    99.0|
|Leopoldo Benedett...|           1815|         1914|    99.0|
|           Aadu Regi|           1912|         2011|    99.0|
|   Irena Kwiatkowska|           1912|         2011|    99.0|
|George Coles Steb...|           1846|         1945|    99.0|
|      Ola M. Vanberg|           1869|         1968|    99.0|
|        Eddie Albert|           1906|         2005|    99.0|
|       Chet Williams|           1918|         2017|    99.0|
|     Yvonne Verbeeck|           1913|         2012|    99.0|

Nous avons filtré les durées plus grandes que 100 (pas très réalistess). Malheureusement nous voyons que les données de MusicBrainz ne sont pas réellement correctes, les dates de début et de fin sont parfois confondues avec les dates de naissance et mort des artistes. Nous ne pouvons pas en tirer de conclusions.

Voyons plutôt combien de tags (= genres) nous avons et groupons-les par artistes :

```scala
val tags = mbTagFullDF.groupBy("tag").count
println(tags.count)
```

On peut voir que nous avons 40'436 tags, ce qui en fait beaucoup, ça signifie sûrement que certains tags n'ont pas vraiment de sens. Voyons si la médiane peut à nouveau nous aider : 

```scala
val median = tags.stat.approxQuantile("count", Array(0.5), 0.01)(0)
```

La médiane est à 1. On peut alors filtrer beaucoup de tags : 

```scala
val tagsFilter = tags.filter("count > 1").orderBy(desc("count"))
```

|             tag|count|
|----------------|-----|
|            jazz|14901|
|            rock|10413|
|            punk| 8532|
|production music| 5901|
|         hip hop| 5711|
|       classical| 4605|
|    likedis auto| 4250|
|              uk| 4171|
|        musician| 4128|
|             usa| 3981|
|        composer| 3890|
|             pop| 3873|
|        american| 3562|
|      electronic| 3433|
|            folk| 3181|
|           metal| 2936|
|         british| 2753|
|         latvian| 2366|
|            soul| 1968|
|alternative rock| 1809|

Comme attendu, nous avons un mix de genres musicaux communs, "jazz" en tête.

Voyons quel artiste a le plus de tags :

|                name|count|
|--------------------|-----|
|         SSHäuptling|  360|
|              Yabamm|  360|
|       Sadgda Jamama|  360|
|                Ekho|  269|
|     Various Artists|  266|
|         Suellen Luz|  249|
|       Gabriele Tosi|  215|
|            Virginia|  171|
|      Michael Samson|  136|
|        SoUnD WaVeS-|  123|
|          Simon Daum|  117|
|          Jay Random|  114|
|               Roiel|  102|
|Michael Ash Sharb...|   95|
|               milan|   94|
|         Picaporters|   93|
|Jamie and the Fir...|   93|
|      digitalTRAFFIC|   91|
|     The Indelicates|   88|
|                SB19|   83|

Nous pensons que les tags n'ont en fin de compte pas beaucoup de sens, même après un premier filtrer, les artistes les plus "taggés" ont 360 tags.

Essayons pour finir de faire une jointure entre les données "originelles" et les données de MusicBrainz : 

```scala
val originArtistsDF = userArtistNamesCountDF.select("name").distinct
val joinOriginMbDF = originArtistsDF.join(mbArtistCleanDF, originArtistsDF("name") === mbArtistCleanDF("name"))
joinOriginMbDF.count
```

```
res27: Long = 317114
```

Nous obtenons "seulement" 317'114 artistes communs aux deux *data sets*, moins que ce nous espérions.
Pour cette raison, nous n'allons pas tenter de fusionner ces deux *data sets*.

## Tentative avec les graphes

Nous avons essayé d'utiliser la librairie [GraphX](https://spark.apache.org/docs/latest/graphx-programming-guide.html) de Spark, avec comme noeuds utilisateurs et artistes et arête le nombre d'écoutes entre les premiers et les second, le tout pour appliquer des algorithmes de graphes sur ces données. Voici le code permettant de créer les noeuds (vu comme des ids) et les arêtes. Comme certains artistes et users possèdent le même id numérique (cas courant lors de bases de données avec id auto incrémenté et généré), nous avons du créer nos propres ids numériques uniques pour chaque noeud :


```scala
import org.apache.spark.graphx._
import org.apache.spark.rdd._

val subDF = userArtistNamesCountDF

val users = subDF
    .select("user") // select only users
    .distinct // remove duplicates
    .rdd // convert to rdd (for graph creation needs)
    .map(_.getInt(0)) // convert it to a list of INTs
    .zipWithIndex // add the "auto increment" unique id, or VertexId
    .map(_.swap) // swap the tuple, because the graph need a RDD[(VertexId, T)]
    
val lastUserId = users.count

val artists = subDF
    .select("artist") // select only artists
    .distinct // remove duplicates
    .rdd // convert to rdd (for graph creation needs)
    .map(_.getInt(0)) // convert it to a list of INTs
    .zipWithIndex // add the "auto increment" unique id, or VertexId
    .map(_.swap) // swap the tuple, because the graph need a RDD[(VertexId, T)]
    .map { case (vertexId, id) => (vertexId + lastUserId, id) } // to maintain unique VertexId, add users.count to each artist VertexId

users.cache
artists.cache

val nodes: RDD[(VertexId, Int)] = users.union(artists)

val edges: RDD[Edge[Int]] = subDF.map { line =>
    val userId = line.getInt(0)
    val artistId = line.getInt(1)
    val userVertexId = users.filter{ case (vertexId, id) => id == userId }.first._1
    val artistVertexId = artists.filter{ case (vertexId, id) => id == artistId }.first._1
    val count = line.getInt(2)
    Edge(userVertexId, artistVertexId, count)
}.rdd

val graph = Graph(nodes, edges)
graph.cache
```

Nous avons dit essayé car bien que le code de création des noeuds et des arêtes ci-dessus fonctionne, impossible d'exécuter une quelconque fonctions du graphe. Par exemple, le code suivant ne compile pas :

```scala
graph.vertices.count
```

Et retourne cette même erreur : 

```
org.apache.spark.SparkException: This RDD lacks a SparkContext. It could happen in the following cases:
(1) RDD transformations and actions are NOT invoked by the driver, but inside of other transformations; for example, rdd1.map(x => rdd2.values.count() * x) is invalid because the values transformation and count action cannot be performed inside of the rdd1.map transformation. For more information, see SPARK-5063.
(2) When a Spark Streaming job recovers from checkpoint, this exception will be hit if a reference to an RDD not defined by the streaming job is used in DStream operations. For more information, See SPARK-13758.
```

Après quelques recherches infructueuses sur le web, il semblerait que ce soit le premier cas (SPARK-5063), il manque un contexte Spark au graphe pour s'exécuter. Après de nombreuses et vaines tentatives, nous n'avons pas approfondi la chose.

## K-Means


## Recommendations



# Améliorations futures possibles
- Optimiser les hyperparamètres pour le recommendeur : essayer de nombreuses combinaisons de paramètres d'entrée de l'algorithme d'apprentissage, pour trouver le meilleur modèle.
- Autre algorithme pour le recommendeur : essayer un autre algorithme que ALS -> mais lequel ? Existant ? Implémenté dans Spark ?
- Recommendations en temps réel : essayer un outil tel que [Oryx 2](http://oryx.io/) pour créer un système évolutif de recommendations de musiques, adaptant le modèle selon les nouvelles écoutes reçues en temps réel et personnalisé pour chaque utilisateur.
