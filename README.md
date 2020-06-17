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
- Nettoyer les données : supprimer les noms d'artistes inconnus et les entrées d'écoute qui ne sont pas temporellement possibles, pour permettre une meilleure analyse.

<!-- # Your approach to testing and evaluation -->

# Résultats obtenus
Balancer toutes les stats
Montrer K-Means
Montrer une recommendation

# Améliorations futures possibles
- Optimiser les hyperparamètres pour le recommendeur : essayer de nombreuses combinaisons de paramètres d'entrée de l'algorithme d'apprentissage, pour trouver le meilleur modèle.
- Autre algorithme pour le recommendeur : essayer un autre algorithme que ALS -> mais lequel ? Existant ? Implémenté dans Spark ?
- Recommendations en temps réel : essayer un outil tel que [Oryx 2](http://oryx.io/) pour créer un système évolutif de recommendations de musiques, adaptant le modèle selon les nouvelles écoutes reçues en temps réel et personnalisé pour chaque utilisateur.
