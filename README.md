# Altcord

Gipsy V2 (de son nouveau nom Altcord) est un utilitaire pour les développeurs permettant de créer facilement des bots discord au fonctionnalités poussées.
Un framework quoi.

Il sera basé sur nextcord.

## La structure générale du projet

Le projet sera constitué de deux parties, indépendantes l'une de l'autre : le cœur (core) et les plugins.

Le cœur sera la partie principale du projet. Ce sera lui qui fera office d'interface entre les plugins et le reste des interfaces, comme discord, la base de données, les traductions… Altcord est le cœur, et des plugins pourront être greffés sur ce cœur.

Les plugins seront des dossiers contenant les fonctionnalités qui seront visibles par l'utilisateur, comme par exemple une commande de ban, un plugin d'économie ou même la gestion de erreurs.

### Ce qui fera partie du cœur :
- la gestion de la base de données
- la gestion des plugins (ajouter, retirer, désactiver)
- la gestion interne des messages d'erreur, des logs
- une interface plus aisée avec les diverses fonctionnalités de discord comme les commandes slash et les différents messages, webhooks...

### Ce qui fera **peut-être** partie du cœur
- une api pour proposer des dépendances faciles entre les plugins
- une api pour générer une interface web de contrôle
- etc...

### Ce qui fera partie des plugins (et définitivement pas du cœur)
- les fonctionnalités, exemples :
  - un système de niveau
  - une commande de ban
  - un gestionnaire de rôle
  - un trou de vers (wormhole dans le jargon)
  - l'affichage au niveau de discord des messages d'erreur
  - les commandes d'aide

Dans un univers parfait, il n'y a pas de plugins par défaut dans Altcord, ce qui fera que le bot sera inutilisable tel quel.
Les plugins seront le plus indépendants possible pour permettre des installations customisées.

La modularité est l'objectif du projet.

## Dans la pratique

Dans la pratique, ces idées devront s'implémenter de la manière la plus propre et maintenable possible.

Il me paraît évident que tout devra soigneusement être documenté dans le cœur, non seulement pour permettre une meilleurs compréhension du projet mais aussi pour permettre l'utilisation d'IA comme Copilot et de les entraîner avec le commentaires.

### La structure des fichiers
```
Racine
├── 📄 start.py
├── 📁 bot (ou peut-être gunibot, fera changer le namespace)
│   ├── 📄 __init__.py (permettra de faire `from bot import translate ou qqch du genre`)
│   ├── 📄 bot.py
│   ├── 📄 database.py
│   ├── 📄 translate.py
│   └── ...
├── 📂configuration (ou 📄 configuration.json mais probablement pas une bonne idée)
│   ├── 📄 bot.json
│   ├── 📄 levels.json
│   ├── 📄 twitter.json
│   └── ...
├── 📁 plugins
│   ├── 📂 ban
│   │   ├── 📄 plugin.py
│   │   ├── 📁 translations
│   │   │   ├── 📄 fr.yaml
│   │   │   └── 📄 en.yaml
│   │   └── ...
│   └── ...
└── ...
```

### Base de données

La base de données sera gérée avec le module python sqlalchemy pour l'utilisation d'un ORM.


Chaque plugin aura un préfix unique et le nom de chaque table utilisée par ce plugin commencera par ce préfix. Cela permettra d'éviter les conflits entre les plugins si ils utilisent différentes tables avec le même nom.

Des aides pourront être implémentées pour permettre au développeur de stocker facilement des objets discord (un salon, un utilisateur) via l'ID directement dans la base de données. Dans le principe, quand le développeur récupère un objet dans la base de données, il pourra utiliser une fonction asynchrone nommée `ensure_objects` qui récupèrera les différents objets depuis Discord.

### Traductions

Le module de traduction sera disponible en l'important : `from bot import translator`.

Il y aura différentes fonctionnalités disponibles pour le développeur :
* `translator.translate` qui prendra en argument la clé de la traduction et permettra de formatter avec des arguments directement. Dans l'idéal, cette fonction n'est pas asynchrone, et peut prendre en argument un objet de contexte ou une interaction, et prendra en compte la langue selon les paramètres du bot (support de la localisation de l'utilisateur, des paramètres du serveur...).
* `translator.command_translate` qui aidera les développeurs avec le nouveau système de traduction des commandes de discord. L'idée est qu'il y a juste la clé de la traduction à donner et il retourne le dictionnaire des traductions avec en clé l'identifiant de la langue et en valeur la traduction. Le support du formatage ne sera probablement pas nécessaire ici, mais il faudra l'implémenter au cas où (par exemple si besoin du nom du bot ou quelque chose comme ça).
* `translator.random_translate` qui pourra être utile pour le développeur dans le cas d'un message aléatoire (par exemple le message de ban aléatoire de Gipsy Beta sur Gunivers), en choisissant une des traductions disponibles dans la langue cible. Éventuellement, cette fonctionnalité peut-être implémentée directement dans `translator.translate`, ce qui permettrais de modifier le bot pour n'avoir que des traductions avec plusieurs possibilités... ça pourrait être marrant. 

Peut-être :
* `translator.message_translate` qui permettra de stocker les arguments de création d'un message directement dans un fichier yaml. L'idée est de pouvoir faire quelque chose dans le genre de `await inter.send(**translator.message_translate("economy.failed", user=inter.user))`. Cela permettrais de faire une interface et un look très dynamique, mais ceci est au coût de ma clarté dans les fichiers de traductions. Dans l'idéal, le système s'adapterait au type de traduction proposé, en interprétant une chaîne de caractère simple en tant que contenu de message, mais en pouvant comprendre le contenu riche d'un embed. Éventuellement, on pourrait avoir des messages à formatter par défaut permettant une mise en forme consistante sur tout le bot et un thémage plus facile.

Une API permettra aux plugins de déclarer des fonctions pour savoir quelle langue afficher en fonction du contexte (si par exemple un plugin veut développer un système de langue avancé), ou ajouter des étapes au formatage pour par exemple changer la couleur des embeds, retirer la mise en forme... Dans cet ordre d'idée, on pourrait avoir la possibilité de créer de plugins de "thème", surtout si la fonctionnalité de traduction de message est disponible.

Ces fonctions seront aussi disponibles directement depuis le module bot grâce au `__init__.py` (par exemple `from bot import translate, random_translate` ou `bot.translate(...)`).

Pour éviter les conflits, les traductions de chaque plugins seront contenues dans un namespace, de la même manière que les tables dans la base de données.
