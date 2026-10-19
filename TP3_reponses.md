# TP 3

**Q1.**
Atomique : la commande se fait en une fois, personne ne peut s'intercaler.
Donc aucun +1 n'est perdu, même si 1000 joueurs finissent une partie en même temps.
En SQL, UPDATE n = n + 1 marche aussi, mais c'est plus lourd à cause des verrous et du disque, alors que Redis travaille en mémoire.

**Q2.**
Avec un JSON dans une String, il faut lire le JSON, modifier xp, réécrire le tout.
Si deux requêtes le font en même temps, elles lisent la même valeur et la dernière écrase l'autre, donc on perd 250 XP.
HINCRBY règle ça car il fait tout en une seule commande.

**Q3.**
ZRANGE va du plus petit score au plus grand, ZREVRANGE fait l'inverse.
Pour un top 3, je prends ZREVRANGE parce qu'on veut les meilleurs en premier.

arrivée à la ligne : 152 il faut commencer la 2.3

**Q4.**
Le rang commence à 0, donc le meilleur joueur verrait « 0 ».
Il faut donc afficher rang + 1 pour que ça commence à 1.

**Q5.**
Nova sort en premier (premier entré, premier sorti).
C'est parce que LPUSH ajoute à gauche et RPOP prend à droite.
Avec LPOP, c'est Ombre qui sortirait, ce serait une pile et pas une file.

**Q6.**
Après 30 s : TTL = -2 et EXISTS = 0, la clé a disparu toute seule.
Sans expiration, comme compteur:parties, TTL = -1. En résumé : -2 = n'existe pas, -1 = n'expire jamais, sinon c'est le nombre de secondes restantes.

**Q7.**
Il faudrait une colonne expire_at et une tâche qui supprime les lignes périmées.
Le risque, c'est que la tâche plante et que les données s'accumulent, ou qu'on oublie le filtre dans une requête.

**Q8.**
Le ConnectionMultiplexer est la connexion partagée vers Redis, il doit être créé une seule fois.
Si on en crée un par requête HTTP, on ouvre une nouvelle connexion à chaque fois et ça finit par saturer.
En TP on ne le voit pas car on est seul à utiliser l'API, en local.

**Q9.**
Sans Order.Descending, ça ne se voit pas : la liste a l'air normale mais le pire joueur est premier.
C'est dangereux car il n'y a aucune erreur, juste un 200 OK, et avec 3 ou 4 joueurs de test on peut ne pas le remarquer.

**Q10.**
Il ne sort jamais du Set, même s'il ferme son navigateur.
Redis ne permet pas de faire expirer un seul membre d'un Set, le TTL ne marche que sur la clé entière.

**Q11.**
Je prends un sorted set presence:enligne, avec le pseudo comme membre et l'heure du dernier signe de vie comme score.
ZADD met à jour le signe de vie, ZREMRANGEBYSCORE supprime ceux de plus de 60 s, et ZCARD compte les joueurs restants.

**Q12.**
J'ai modifié 0 ligne dans les endpoints.
Ils utilisent l'interface IGameCatalog, donc il suffit de changer l'enregistrement.
C'est possible grâce au pattern Repository de la séance 2.

**Q13.**
Mes mesures : miss = **_ ms, hits = _** ms.
Le gain est petit en local avec peu de jeux, donc la mesure n'est pas très parlante.
Un cache devient vraiment utile quand la requête est lourde, que la base est loin ou qu'il y a beaucoup de lecteurs en même temps.

**Q14.**
Les joueurs voient l'ancien titre jusqu'à 60 s, car Redis n'est pas prévenu que Mongo a changé.
On peut réduire le TTL, mais le cache protège alors moins la base.
On peut aussi faire un DEL à chaque modification, mais ça ne marche pas si on modifie directement dans Mongo.
