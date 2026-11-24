**Q1.**

Si on le lance 2 fois avec `CREATE` on a 2 refus exxactement identiques
Les trois `CREATE CONSTRAINT` sont des contraintes d'unicité (un seul Joueur par pseudo, un seul Jeu par titre, une seule Guilde par nom).
Elles empêchent les doublons même si on se trompe, et elles créent un index, donc le MERGE va plus vite.

**Q2.**

Le résultat pour la première : "Pixel" et "Nova"
Elle est correcte/incorrecte

Le résultat pour la deuxième : "Pixel"
Elle est correcte/incorrecte

**Q3.**

`count(DISTINCT ami)` compte le nombre d'amis différents qui possèdent le jeu.
`count(*)` compte le nombre de lignes (chemins) : un même ami atteint par plusieurs chemins (à 1 saut et à 2 sauts) est compté plusieurs fois, donc le chiffre est gonflé.
Pour passer de *1..2 à *1..4, je change un seul endroit (les deux chiffres dans la même ligne).
En SQL il aurait fallu ajouter un niveau (une jointure ou une CTE) par saut en plus, donc environ 2 de plus, ou une CTE récursive.

**Q4.**

Parce que Nova le possède déjà : la requête contient NOT (moi)-[:POSSEDE]->(j), qui exclut tous les jeux que Nova a déjà.
(Ou alors aucun de ses amis ne l'a, mais je pense que c'est la première raison.)

**Q5.**

Parce que ce sont deux protocoles différents, sur deux ports différents du même serveur.
Le 17474 (7474 dans le conteneur) est le port HTTP pour le Browser web.
Le 17687 (7687 dans le conteneur) est le port Bolt, le protocole utilisé par le driver de l'application C#.
Le décalage de 10000 vient du docker-compose (pour éviter les conflits avec une installation locale).

**Q6.**

Cypher n'accepte pas de paramètre ($x) pour la longueur d'un chemin (\*1..2) : un paramètre ne peut être qu'une valeur, pas la structure de la requête.
On est donc obligé d'interpoler le nombre dans la chaîne. La précaution : le code vérifie que profondeur est entre 1 et 4 et lève une ArgumentOutOfRangeException sinon.
Sans ça, une valeur donnée dans l'URL (?depth=...) pourrait faire une requête énorme (explosion du nombre de chemins, Neo4j qui plante ou rame) et, comme c'est du texte collé dans la requête, ça ouvre la porte à une injection Cypher.

**Q7.**

Le IDriver est coûteux à créer (il gère un pool de connexions) et il est thread-safe : on le crée une fois pour toute l'appli (singleton).
Une session est légère, pas thread-safe, et ne sert que pour une opération : on en ouvre une par méthode et on la libère avec await using.
C'est la même règle que pour MongoClient et ConnectionMultiplexer : un seul client partagé pour toute l'application, créé au démarrage, jamais recréé à chaque requête.

