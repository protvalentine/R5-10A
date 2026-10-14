**Q1.**

Je pense que c'est pour pas avoir à tout réinstaller à chaque séance.
Comme on va utiliser les autres bases plus tard (Mongo, Redis, Neo4j), vous avez mis les quatre d'un coup dans le fichier.
Aujourd'hui on se sert que de PostgreSQL.

**Q2.**

C'est Redis qui n'a pas de volumes.
Donc si on supprime le conteneur, les données sont perdues.
Les autres ont un volume donc eux gardent leurs données.

**Q3.**

15432:5432, c'est un port pour le PC et un port pour le conteneur.
Le premier (15432) c'est celui qu'on utilise depuis notre machine, et le deuxième (5432) c'est celui de PostgreSQL dans le conteneur.
Docker fait le lien entre les deux.
Je crois que c'est décalé pour pas avoir de conflit si on a déjà un PostgreSQL installé.

**Q4.**
Pour le TP c'est bon parce que c'est juste en local et que ce sont des données de test.
Mais en production non, parce que n'importe qui qui a accès au fichier ou au dépôt Git voit le mot de passe.
Il faudrait le mettre dans des variables d'environnement éventuellement.

**Q5.**

docker compose exec, ça lance une commande dans un conteneur qui tourne déjà.
Donc redis-cli s'exécute dans le conteneur Redis et pas sur mon ordi.
C'est pour ça qu'on n'a pas besoin d'installer Redis chez nous.

**Q6.**

Ça marche avec localhost parce que le port 15432 de notre machine est relié au port 5432 du conteneur (comme dans la Q3).
Notre application tourne sur notre PC, donc elle passe par localhost et Docker redirige vers PostgreSQL.

**port**

le port est 5199

**Q7.**

Non, les joueurs ne sont pas dupliqués.
D'abord parce que les données sont enregistrées dans PostgreSQL, donc elles restent après l'arrêt de l'appli.
Ensuite parce que dans le code il y a un if (!db.Players.Any()), donc il n'ajoute les joueurs que si la table est vide.

arrivée ligne 231
