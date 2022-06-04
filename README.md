# Altcord

Altcord est une framework permettant le création aisée de bot cross-platform.
L'idée est de fournir une API qui sera compatible avec certaines plateformes comme discord, mais aussi minecraft ou Telegram.

Ce fonctionnement permettra de créer des bot pour tout les publics sans avoir à recoder entièrement les fonctionnalités.

La couche d'abstraction des différentes APIs sera fournie par l'API entièrement customisée de Altcord.

Pour créer un bot, il faudra ajouter des plugins au cœur. Les plugins pourront spécifier une API en fonction de la plateforme (par exemple pour modifier un salon) et pourrons utiliser des traductions, avoir un accès facilité à la base de données, utiliser des paramètres en fonction du contexte (utilisateur, salon, serveur...).

## Structure du projet

Le projet sera constitué de deux parties, le cœur (core), et indépendamment les plugins.

Le cœur sera la partie principale du projet. Ce sera lui qui fera office d'interface entre les plugins et les plateformes comme Discord, minecraft, mais aussi les composants internes du bot comme la base de données, les traductions… Altcord est le cœur, et des plugins pourront être greffés sur ce cœur.

Les plugins seront des dossiers contenant les fonctionnalités qui seront visibles par l'utilisateur, comme par exemple une commande de ban, un plugin d'économie ou même la gestion de erreurs.

### Ce qui fera partie du cœur :
- le support des plateformes (discord, minecraft)
- la gestion de la base de données
- la gestion des plugins (ajouter, retirer, désactiver)
- un système de traductions

### Ce qui fera **peut-être** partie du cœur
- une api pour proposer des dépendances faciles entre les plugins
- une api pour générer une interface web de contrôle (peut poser des problèmes avec le système multi-plateformes)
- etc...

### Ce qui fera partie des plugins (et définitivement pas du cœur)
- les fonctionnalités, exemples :
  - un système de niveau
  - une commande de ban
  - un gestionnaire de rôle
  - un trou de vers (wormhole dans le jargon)
  - les commandes d'aide
  - des rappels
  - et pleins de fonctionnalités du genre

Il n'y aura pas de plugins par défaut dans Altcord, et le bot brut ne fera rien. Il faudra ajouter des plugins pour avoir des fonctionnalités.

Le cœur sera soigneusement documenté, pour pouvoir facilement le maintenir et pour permettre l'utilisation d'IAs comme Copilot.

### La structure des fichiers
```
📁 /
├─ 📄 start.py
├─ 📁 api
├─ 📁 config
├─ 📁 data # peut-être à supprimer en faveur d'uniquement l'utilisation de la base de données
├─ 📁 platforms
│  ├─ 📁 discord
│  ├─ 📁 telegram
│  ├─ 📁 minecraft
│  └─ 📁 ...
├─ 📁 docs
├─ 📁 langs
├─ 📁 logs
├─ 📁 plugins
│  └─ 📁 <plugin name>
│     ├─ 📄 main.py # ou plugin.py, entrypoint.py ? pas vraiment le fichier principal du projet
│     ├─ 📁 api
│     ├─ 📁 data # toujours nécessaire ? (si utilisation de sqlalchemy)
│     ├─ 📁 platforms
│     │  ├─ 📁 discord
│     │  ├─ 📁 telegram
│     │  ├─ 📁 minecraft
│     │  └─ 📁 ...
│     ├─ 📁 docs
│     ├─ 📁 langs
│     ├─ 📁 utils
│     └─ 📁 web # peut-être le mettre dans platforms ? ou un support avec un système de configuration ?
├─ 📁 utils
└─ 📁 web # peut-être le mettre dans platforms ?
```

### Le plugin

Chaque plugin aura un ID unique (plus ou moins) qui permettra d'éviter les collisions entre les différents plugins.

Chaque plugin pourra indiquer quels sont les plateformes qu'il supporte, par exemple un plugin de gestion de rôle n'aurait pas de sens sur minecraft.

### Les utilisateurs

Au vue du système multi-plateforme, il faudra fournir un moyen de relier plusieurs comptes à un seul utilisateurs.

Cet utilisateur aura un identifiant interne au bot et ce sera cet identifiant qui sera utilisé, pas un identifiant d'une tierce partie comme discord ou l'UUID de minecraft.
Ce comportement permettra d'utiliser un compte et dans le cas d'un changement de compte de spécifier le nouveau compte ou encore d'utiliser un compte discord avec les même données que un compte Telegram.

### Base de données

La base de données sera gérée par le module python `sqlalchemy` qui permettra d'utiliser facilement n'importe quel type de base de données SQL (sqlite3, MariaDB, MySQL...) sans se prendre la tête.

Ça permettra notamment de fournir des utilitaires pour gérer plus facilement les utilisateurs et différentes fonctionnalités.

Chaque table de plugin devra avoir son id en début de table (par ex `ban_users`), pour éviter des problèmes si plusieurs plugins utilisent un table avec le même nom.

Au chargement d'un plugin, une fonction sera appelée où le bot pourra inscrire sa table.

Pour les mises à jour de format de tables, il y a deux possibilités :
1. le plugin met à jour tout seul la table au chargement du plugin
2. une commande permettra d'installer le plugin et de mettre à jour la table (par exemple `python altcord.py plugin update ban`)

### Commandes

Les commandes seront gérées par Altcord qui fera la traduction en commandes Discord, Telegram, Minecraft.

Il faudra permettre un support avancé de toutes les fonctionnalités des différentes plateformes tout en évitant de rendre une plateforme spécifique inutilisable.

Par exemple, les commandes de Discord supportent les arguments, ce qui n'est pas le cas des commandes Telegram. La librairie devra gérer la traduction entre les différentes possibilités de mise en forme.

### Traductions

Comme par défaut le moyen d'envoyer un message en réponse à une interaction (une commande, un appuis sur un bouton, un message reçu) sera géré par la librairie, on pourra intégrer un système de traduction très profond.

Par exemple, on pourra juste donner la clé d'une traduction à l'envois d'un message et l'objet contexte vérifiera lui même la langue avant d'envoyer le message.

Les traductions seront stockées dans des fichiers YAML dans le dossier de chaque plugin avec quelques traductions par défaut (le nom du bot, des messages communs comme `Valider`, `Annuler`...).

Les traductions pourront aussi être intégrées au système de commandes de manière profonde en fonction de la plateforme.

### Configuration

Le bot devra proposer une API facile d'utilisation pour gérer des configurations en fonction des serveurs, des salons, des utilisateurs...

L'idée est de ne pas implémenter une interface de configuration pour l'utilisateur final mais de proposer une API pour enregistrer des configuration sous forme d'un objet python avec un nom, une description, un type, une valeur par défaut, un validateur synchrone et un autre asynchrone (si besoin), permettant de retourner un message d'erreur.

Ces configurations seront modifiables par l'utilisateur final via l'utilisation d'un plugin qui implémentera les modifications de ces variables, comme par exemple une interface web.

Au niveau des plugins "clients" du système de configuration, une configuration pourrait se déclarer de cette manière :
```py
def user_on_gunivers():
  pass # on va dire que cette fonction vérifie que l'utilisateur est sur les serveurs de Gunivers

bot.register_config(
  Configuration(
    id="allow_ban",
    name="Allow ban",
    description="Allow other active user to \"ban\" you from the Gunivers server",
    type=bool,
    target=nextcord.Member,
    default=True,
    available=user_on_gunivers,
  )
)
```

Chaque configuration sera enregistrée avec la forme `<plugin_namespace>.<config_name>` (avec le nom du plugin pour éviter les problèmes de compatibilité entre les différents plugins).

Au niveau de la base de données, la table de configuration pourrait ressembler à quelque chose de ce genre :
```
PRIMARY KEY id -> l'identifiant de la configuration (par exemple `gunivers.allow_ban`)
PRIMARY KEY target -> l'identifiant de la cible (un utilisateur, un serveur, un salon textuel...)
value -> la valeur de la configuration (un nombre, un booléen, un ID, une chaîne de caractère, du JSON...)
```
Le deux clés primaires permettraient d'éviter des conflits.
Éventuellement, il sera peut-être plus pratique d'utiliser plusieurs colonnes en fonction du type de la configuration, en ajoutant une colonne type.

Puisque les objets donnés par les évènements seront normalisés, on pourra faire par exemple :
```py
if ctx.user.config['ban.allow_ban']:
    await ctx.user.discord_user.ban()
```

Il faudra créer des fonctions permettant de récupérer des listes de configuration en fonction des plugins, pour permettre la création d'un outil de configuration.