# TP2

Point de progression : ligne 406
Arrivée à la ligne
repasser par les étapes : aux lignes 26 et 62 pour les tests
et faire le
bash
`use pixelhub`
ça dépend
**Exercices**

- Le nombre de jeux dans la collection est de 10

**Q1.**

Le \_id n'est pas fabriqué par le serveur MongoDB mais par l'outil qui envoie les documents, donc mongoimport.
Il le crée avant l'envoi, donc on le connaît avant que le document soit dans la base.
Au TP 1, l'Id 4 de Pixel avait été choisi par PostgreSQL au moment de l'insertion (auto-incrément), et on ne le connaissait qu'après.

**Q2.**
En SQL avec un modèle normalisé il faudrait une table de liaison, par exemple jeux_plateformes(jeu_id, plateforme), et faire une jointure :

```sql
SELECT j.titre
FROM jeux j
JOIN jeux_plateformes jp ON jp.jeu_id = j.id
WHERE jp.plateforme = 'Switch';
```

Dans Mongo le tableau est directement dans le document, donc { plateformes: "Switch" } suffit.

**Q3.**

Le compte pixelhub a été créé dans la base admin (au démarrage du conteneur, avec les variables du compose).
--authenticationDatabase admin dit à MongoDB où chercher le compte.
Sans ça il le cherche dans la base utilisée (pixelhub),
ne le trouve pas, et l'authentification échoue.

**Q4.**

Je dirais que c'est un peu des deux.
C'est une bonne nouvelle parce que c'est pratique :
on peut ajouter un champ sans modifier toute la base, et les jeux n'ont pas tous les mêmes caractéristiques.
Mais c'est un problème dans un projet à quatre développeurs six mois plus tard :
chacun peut inventer ses champs, on aura note, Note, rating, des fautes de frappe, des types différents, et personne ne saura quelle est la structure réelle.
Le code qui lit la base risque de planter sur des documents inattendus.
Le schéma n'a pas disparu, il est juste dans la tête des développeurs ou dans le code au lieu d'être dans la base.

**Q5.**

Ça ne m'a pas trop surpris après coup : le résultat est vide.
db.Jeux n'est pas db.jeux, MongoDB est sensible à la casse, donc il cherche dans une collection Jeux qui n'existe pas.
Il n'y a aucune erreur parce que MongoDB ne vérifie pas qu'une collection existe : une collection inexistante est traitée comme une collection vide.

**Q6.**
Le message d'erreur est : MongoInvalidArgumentError: Update document requires atomic operators.
MongoDB refuse parce que updateOne attend des opérateurs comme $set, $inc, etc.
Sans opérateur, il ne sait pas si on veut modifier un champ ou remplacer tout le document, donc il bloque pour éviter d'écraser le document par erreur.
Pour vraiment remplacer le document en entier il faut replaceOne :

```javascript
db.jeux.replaceOne({ titre: "Valorant" }, { titre: "Valorant", note: 4.4 });
```

**Q7.**

En SQL il faudrait d'abord modifier la structure de la table :

```sql
ALTER TABLE jeux ADD COLUMN nb_votes INT;
```

La conséquence est que toutes les autres lignes auraient aussi cette colonne, avec la valeur NULL (ou une valeur par défaut).
Alors que dans Mongo seul le document modifié a le champ.

**Q8.**
Non, le mot « Mongo » n'apparaît nulle part dans l'interface IGameCatalog : elle ne parle que de jeux (GetAllAsync, GetByGenreAsync...).
C'est important parce que le reste de l'application (les endpoints) ne dépend pas de MongoDB, seulement de l'interface.
Si on veut changer de moteur ou ajouter un cache (Redis à la séance 3), on change l'implémentation sans toucher aux endpoints.

**Q9.**
J'obtiens une FormatException du genre : Element 'joueursParEquipe' does not match any field or property of class PixelHub.Api.Models.Game.
Le driver essaie de transformer chaque champ du document Mongo en propriété de la classe Game.
Quand un document contient un champ (joueursParEquipe) qui n'existe pas dans la classe, il ne sait pas où le mettre et il lève une erreur.

**Q10.**
[BsonIgnoreExtraElements] fait disparaître l'erreur mais au prix de la perte de ces champs côté application : le driver les ignore silencieusement.
Donc joueursParEquipe n'apparaît plus dans la réponse de /games. Il est toujours dans MongoDB, mais l'API ne le montre pas.

**Q11.**

- [BsonIgnoreExtraElements] : Avantage, c'est très simple et ça ne plante plus.
  Inconvénient : on perd les champs spécifiques, sans aucun message.

- [BsonExtraElements] : Avantage, on garde tous les champs spécifiques dans un dictionnaire / BsonDocument, sans les perdre.
  Inconvénient : ils ne sont pas typés (on manipule des object), c'est moins propre à utiliser, et il faut gérer la sérialisation JSON (d'où les deux propriétés).

- Hiérarchie de classes (FpsGame : Game...) : Avantage, tout est bien typé, avec de vraies propriétés pour chaque genre.
  Inconvénient : c'est plus lourd à écrire, il faut une classe par genre et gérer la désérialisation polymorphe, et à chaque nouveau champ ou genre il faut modifier le code.

**Q12.**

La requête SQL équivalente :

```sql
SELECT genre, AVG(note) AS note_moyenne, COUNT(*) AS nombre
FROM jeux
GROUP BY genre
ORDER BY note_moyenne DESC;
```

Les deux font la même chose et le SQL est plus court.
Donc non, ce n'est pas ici que MongoDB apporte quelque chose.
L'apport se situe plutôt dans la souplesse du schéma (champs spécifiques selon le genre, tableaux plateformes et tags, avis imbriqués) : tout est dans un seul document sans jointure.

**Q13.**
taille 273 octets avant d'y toucher

C'est la requête e ($elemMatch) qui est la bonne.
La requête d vérifie les deux conditions séparément sur le tableau : il existe un avis dont l'auteur est Krayz, et il existe un avis (pas forcément le même) avec une note de 5.
Pour Stardew Valley, Krayz a bien un avis (note 4) et Nova en a un avec 5, donc le document est renvoyé alors que Krayz a mis 4.
Avec $elemMatch, les deux conditions doivent être vraies sur le même avis.

taille après avoir fait les modifications : 516 octets

**Q14.**

Un avis (auteur, note, texte) pèse environ 100 octets (d'après la différence entre mes deux mesures de Stardew Valley divisée par 3).
La limite est de 16 777 216 octets, donc environ 16 777 216 / 100 ≈ 160 000 avis avant d'être bloqué.
Pour un jeu très populaire, avec des centaines de milliers d'avis, c'est un risque réel.
Et avant la limite, le document devient lourd à lire à chaque fois.
Ça rejoint les critères du cours : on n'imbrique que si c'est borné.
Les avis ne sont pas bornés, donc il vaut mieux les référencer dans une collection séparée.

**Q15.**

$lookup renvoie toujours un tableau parce qu'il ne sait pas à l'avance combien de documents vont correspondre :
ça peut en être zéro, un ou plusieurs (comme une jointure un-à-plusieurs). 
Avec $unwind: "$studio", le tableau est « déplié » : le champ studio devient un objet au lieu d'un tableau d'un élément (et un jeu dont le tableau est vide disparaît du résultat).

**Q16.**

- Expérience 1 : studio référencé, il suffit de modifier 1 document (dans studios). S'il était imbriqué dans chaque jeu, il faudrait en modifier 40.
- Expériences 2 et 3 : MongoDB n'a rien signalé du tout. Avec la faute de frappe ou le studio supprimé, le champ studio est juste un tableau vide.
- PostgreSQL avec studio_id REFERENCES studios(id) aurait refusé de supprimer le studio tant que des jeux le référencent (erreur de clé étrangère), ou aurait appliqué une règle (cascade, set null).
- Dans PixelHub, c'est nous, le code de l'application (le développeur), qui sommes responsables de l'intégrité des références. La base ne le fait pas.


**Q17.** 

Le prix actuel du jeu (1 200 pièces) vit dans le document du jeu, dans MongoDB (catalogue). 
Le prix payé par Nova (900 pièces) doit vivre côté PostgreSQL, avec les joueurs et leurs pièces, dans l'achat lui-même (par exemple une table achats avec le prix payé et la date). 
Dupliquer le prix est juste ici parce que ce n'est pas une copie qui doit rester synchronisée : c'est un fait historique. 
Le prix payé à l'achat doit rester à 900 même si le prix du jeu change ensuite.


**Q18.**

$slice: -2 dans le $push garde seulement les deux derniers éléments du tableau après l'ajout :
quand on ajoute un 3e avis, le plus ancien est retiré de derniersAvis. 
La collection avis référence le jeu par son _id et pas par son titre parce que le titre peut changer (renommage) et il faudrait alors modifier tous les avis, alors que l'_id ne change jamais.

**Q19.**
La base est dans un état incohérent :
l'avis est dans la collection avis, mais derniersAvis dans le jeu n'est pas à jour (il manque le dernier avis). 
C'est la collection avis qui fait foi (c'est la source de vérité), derniersAvis n'est qu'une copie. 
Pour réparer, on recalcule derniersAvis à partir de avis (les deux avis les plus récents du jeu) et on réécrit le champ dans le jeu. 
Le vrai problème (deux écritures non atomiques) sera vu en séance 5.

