---
title: "Atelier 9 — Construire un dashboard et qualifier un incident"
subtitle: "Document participant autonome — parcours non développeurs"
lang: fr-FR
---

# Construire un dashboard et qualifier un incident

## Situation professionnelle

Les dashboards de votre organisation ont été construits par des profils techniques. Vous avez besoin d'une vue qui vous permette, lorsqu'un incident est signalé, de répondre en quelques minutes à quatre questions : les utilisateurs sont-ils touchés ? depuis quand ? sur quel périmètre ? faut-il escalader ?

## Objectif opérationnel

Auditer un dashboard existant, construire votre propre dashboard de qualification à partir des données réelles de l'application `peopulse` et du service `eu-interfaces` (traces et logs) et de monitors existants, l'exploiter pour lire une situation réelle, puis l'utiliser pour qualifier un incident signalé par une équipe métier.

## Livrable

À la fin de l'atelier, vous disposez :

- d'un dashboard `[TRAINING] M09 - <participant>` de treize widgets, organisé de l'impact vers le diagnostic ;
- d'une note de lecture et de quatre groupes de widgets ;
- d'une variable de dashboard `env` utilisée par tous les widgets de données ;

## Environnement

- Chrome connecté à l'organisation Datadog de la formation ;
- accès aux dashboards, à APM, aux logs, au RUM et à la liste des monitors ;
- droit de créer un dashboard ;
- un éditeur de texte simple.

## Règles d'écriture

La seule écriture autorisée est la création et la modification de **votre** dashboard `[TRAINING] M09 - <participant>`.

Remplacez `<participant>` par l'identifiant fourni par le formateur, par exemple `loic`.

## Sources de données utilisées

Le dashboard réunit trois sources réelles et deux monitors existants :

| Source | Ce qu'elle mesure | Rôle dans le dashboard |
|---|---|---|
| RUM `peopulse` | ce que vivent les utilisateurs dans le navigateur | impact utilisateur |
| APM `eu-interfaces` / `console` | l'exécution des traitements planifiés côté serveur | comportement des traitements |
| Logs `eu-interfaces` | les messages écrits par les traitements, dont le nom de la commande exécutée | logs applicatifs |
| Monitors existants « Facture V2 » et « app-bamboo » | l'état des alertes déjà en place dans l'organisation | alerte |

Les traces et les logs de `eu-interfaces` décrivent le même service, mais aucun identifiant ne relie une trace à ses logs. Aucun lien technique entre les sessions RUM de `peopulse` et les traitements de `eu-interfaces` existe. Elles sont réunies ici pour apprendre à organiser un dashboard selon l'ordre de lecture d'un incident. Sur un dashboard de production, ces liens doivent être vérifiés avant d'en tirer une conclusion.

## Plan du dashboard

```text
+----------------------------------------------------------------+
| Note de lecture                                                |
+----------------------------------------------------------------+
| Impact utilisateur          | erreurs | vues | chargement p75  |
+----------------------------------------------------------------+
| Comportement des traitements| requêtes | vs veille | erreurs | p95 |
+----------------------------------------------------------------+
| Logs applicatifs            | volume par statut | erreurs      |
|                             | commandes en erreur | dernières  |
+----------------------------------------------------------------+
| Alerte                      | monitor Facture V2 | Alert Graph |
+----------------------------------------------------------------+
```

Chaque widget répond à une question et conduit à une action.

# 1. Auditer un dashboard existant

## 1.1. Ouvrir `Monitoring EU-Interfaces` et lire sa période et ses variables

**Pourquoi :** observer une vue existante montre ce qui facilite ou ralentit une décision, avant d'en construire une.

- **A9-1.1.1** Ouvrez **Dashboards > Dashboard List**.
- **A9-1.1.2** Dans le champ de recherche, saisissez `Monitoring EU-Interfaces` et ouvrez le dashboard portant ce titre.
- **A9-1.1.3** Ne cliquez sur aucun widget pour l'éditer.
- **A9-1.1.4** Observez la période active et les variables affichées dans **Filter by**.

**Résultat attendu :** vous savez sur quelle période et quel périmètre portent les widgets. Une variable permet de changer le périmètre de tout le dashboard en un clic. Par exemple, une variable `version` permet de comparer deux versions déployées sans dupliquer les widgets.

## 1.2. Associer une question à chaque widget et repérer l'ordre de lecture impact → chronologie → périmètre → diagnostic

**Pourquoi :** pendant un incident, on lit un dashboard de haut en bas. L'ordre des widgets doit suivre l'ordre des questions.

- **A9-1.2.1** Parcourez les widgets de haut en bas.
- **A9-1.2.2** Pour chaque widget, déterminez la question à laquelle il répond, en vous aidant de son titre et de son contenu.
- **A9-1.2.3** Repérez le premier widget qui indique l'impact, par exemple **Nombre de cron en erreur**, affiché en orange lorsqu'il dépasse zéro.
- **A9-1.2.4** Cherchez un widget dédié à l'évolution dans le temps ; observez que seules de petites courbes apparaissent en fond des chiffres du haut.
- **A9-1.2.5** Repérez un widget qui permet de réduire le périmètre, par exemple le camembert **Exécutions par Interfaces**.
- **A9-1.2.6** Repérez un widget qui oriente vers le diagnostic, par exemple la liste **Exécutions en erreurs**.

**Résultat attendu :** en comparant vos repérages avec le tableau ci-dessous, vous constatez que l'impact, le périmètre et le diagnostic sont couverts, mais qu'aucun widget n'est consacré à la chronologie. L'ordre de référence est le suivant :

| Niveau | Question | Exemple de widget |
|---|---|---|
| Impact | Les utilisateurs sont-ils touchés ? | nombre d'erreurs côté navigateur |
| Chronologie | Depuis quand ? | courbe des requêtes ou des logs en erreur |
| Périmètre | Où exactement ? | classement des commandes en erreur |
| Diagnostic | Quelle piste ouvrir ? | liste des derniers messages d'erreur |

Un dashboard sans widget de chronologie ne permet pas de répondre à « depuis quand ? » : c'est le niveau que votre propre dashboard ajoutera avec des courbes dans le temps. Un dashboard qui commence par des détails techniques oblige, lui, le lecteur à reconstituer lui-même l'impact. Un titre comme « Erreurs » est insuffisant : « Logs - commandes en erreur » dit à la fois la source et la question.

## 1.3. Choisir le bon type de widget pour chaque question

**Pourquoi :** un même chiffre peut être présenté de plusieurs façons ; seule l'une d'elles répond vite à la question posée.

Lisez le tableau suivant, que vous appliquerez à la thématique 3 :

| Type de widget | Question typique | Exemple dans votre dashboard |
|---|---|---|
| **Query Value** | Combien maintenant ? | nombre d'erreurs navigateur |
| **Timeseries** | Quand la valeur change-t-elle ? | requêtes des traitements dans le temps |
| **Change** | Est-ce plus ou moins que d'habitude ? | requêtes par rapport à la veille |
| **Top List** | Qui contribue le plus ? | commandes qui produisent le plus d'erreurs |
| **List** | Que disent les derniers événements ? | derniers messages d'erreur |
| **Monitor Summary** | L'alerte est-elle active ? | état du monitor Facture V2 |
| **Alert Graph** | Pourquoi l'alerte sonne-t-elle, ou pas ? | erreurs comparées au seuil d'un monitor |

- **A9-1.3.1** Revenez à `Monitoring EU-Interfaces` et identifiez le type de deux de ses widgets.
- **A9-1.3.2** Pour chacun, déterminez sa question, retrouvez dans le tableau le type qui y répond le mieux, et comparez avec le type utilisé.

**Résultat attendu :** pour chacun des deux widgets, vous savez si son type est adapté à sa question ou lequel l'aurait mieux été. Un **Query Value** masque la chronologie ; une **Timeseries** montre mal qui contribue le plus. C'est pourquoi un bon dashboard combine plusieurs types sur une même source.

# 2. Créer le dashboard et son cadre de lecture

## 2.1. Créer un Blank Dashboard et le renommer `[TRAINING] M09 - <participant>`

**Pourquoi :** un dashboard vierge vous oblige à choisir chaque widget selon une question, au lieu d'accumuler des graphiques proposés automatiquement.

- **A9-2.1.1** Dans **Dashboards > Dashboard List**, cliquez sur **New Dashboard**.
- **A9-2.1.2** Sur la page **Build a Dashboard with Bits**, n'utilisez ni le champ de saisie ni les suggestions.
- **A9-2.1.3** En bas de la page, sous **Or start from scratch**, cliquez sur **Blank Dashboard**.
- **A9-2.1.4** Le dashboard est créé immédiatement avec un nom par défaut et le panneau **Widgets** s'ouvre à droite : fermez ce panneau avec sa croix.
- **A9-2.1.5** Cliquez sur le titre du dashboard, puis sur l'icône crayon qui apparaît à côté.
- **A9-2.1.6** Sélectionnez tout le titre avec `Ctrl+A`, saisissez le titre suivant et validez avec Entrée :

  ```text
  [TRAINING] M09 - <participant>
  ```

- **A9-2.1.7** Vérifiez l'apparition de la mention **Title Saved**, puis fermez la fenêtre du titre avec Échap.
- **A9-2.1.8** N'utilisez pas le bouton **Share**.

**Résultat attendu :** un dashboard vide portant votre identifiant est ouvert. L'onglet du navigateur affiche `[TRAINING] M09 - <participant>`. Datadog enregistre automatiquement le dashboard dès sa création. Un dashboard abandonné reste donc dans la liste avec son nom par défaut : renommez-le toujours immédiatement.

## 2.2. Ajouter la variable de dashboard `env` avec la valeur par défaut `prod`

**Pourquoi :** une variable change le périmètre de tous les widgets qui l'utilisent. Elle évite de dupliquer le dashboard pour chaque environnement.

- **A9-2.2.1** Sous le titre, cliquez sur **Add Variable**.
- **A9-2.2.2** Saisissez `env` dans la recherche, puis cliquez sur `env` dans **Matching Tags**.
- **A9-2.2.3** Dans le panneau **Edit $env**, conservez le type **Filter**.
- **A9-2.2.4** Dans **Default value**, saisissez `prod`, cochez la case `prod` proposée, puis cliquez dans une zone vide du panneau pour refermer la liste.
- **A9-2.2.5** Cliquez sur **Save** en haut du panneau.

**Résultat attendu :** le bandeau affiche **Filter by** avec la variable `env` à `prod`. La variable n'agit que sur les widgets dont la requête contient `$env`. Vous l'ajouterez dans chaque widget de données à la thématique 3.

## 2.3. Ajouter une note de lecture indiquant la question du dashboard, ses sources et ses limites

**Pourquoi :** une personne qui ouvre le dashboard pour la première fois doit comprendre en dix secondes à quoi il sert et ce qu'il ne permet pas de conclure.

- **A9-2.3.1** Cliquez sur **Add Widgets**, saisissez `note` dans la recherche du panneau et choisissez **Notes & Links**.
- **A9-2.3.2** Dans la fenêtre **Notes & Links Editor**, cliquez dans le champ **Markdown** et collez le contenu suivant, copié depuis cette page. Ne le tapez pas : l'éditeur ajoute lui-même un tiret à chaque retour à la ligne dans une liste.

  ```markdown
  ## Qualifier un incident sur eu-interfaces / console

  Lire de haut en bas : impact utilisateur, traitements, logs, alerte.

  - Impact utilisateur : application RUM peopulse (agrégats)
  - Traitements : APM eu-interfaces, opération console
  - Logs applicatifs : logs eu-interfaces, par commande
  - Alerte : monitors existants Facture V2 et app-bamboo

  Aucun lien technique entre peopulse et eu-interfaces n'a été vérifié,
  et les logs ne sont pas reliés aux traces :
  des variations simultanées ne prouvent aucune cause commune.
  ```

- **A9-2.3.3** Vérifiez le rendu dans la zone **Preview** : chaque source apparaît sur une seule puce, sans puce imbriquée.
- **A9-2.3.4** Cliquez sur **Save**.
- **A9-2.3.5** La note s'affiche tronquée avec un bouton **More** : tirez son coin inférieur droit pour l'agrandir jusqu'à ce que tout le texte soit visible.

**Résultat attendu :** la note est visible sur le dashboard.

# 3. Construire les widgets de l'impact vers le diagnostic

Les widgets sont rangés dans quatre groupes. Pour chaque groupe, vous créez d'abord le groupe, puis vous ajoutez ses widgets **à l'intérieur**, avec le cadre **Add Widgets or Powerpacks** qui apparaît sous son titre. Déplacer un widget existant vers un groupe est plus difficile : respectez cet ordre.

Chaque widget s'ajoute de la même façon : choix du type dans le panneau **Widgets** (son champ de recherche permet de le retrouver par son nom), réglage de la source dans **Graph your data**, titre dans **Give your graph a title**, puis **Save** en bas à droite. Après chaque **Save**, vérifiez le titre affiché sur le widget : s'il a gardé le titre proposé par Datadog, survolez le widget, cliquez sur l'icône crayon et corrigez le titre.

Deux précautions évitent la plupart des blocages :

- avant de chercher un nouveau type de widget, effacez la recherche précédente du panneau **Widgets** avec sa croix : le champ conserve l'ancienne saisie ;
- après avoir saisi un titre, appuyez sur Tab avant de cliquer sur **Save** ;
- n'appuyez pas sur Échap dans l'éditeur d'un widget : cela le ferme sans l'enregistrer ;
- tapez toujours dans un champ sur lequel vous venez de cliquer : une lettre tapée ailleurs déclenche un raccourci de Datadog, par exemple `G` ouvre la fenêtre **Quick Graph** et `E` la modification d'un groupe ou d'un widget ; dans ce cas, cliquez sur **Cancel**.

Les widgets sont numérotés de 1 à 13 ; la note de lecture et les groupes ne sont pas comptés.

## 3.1. Créer le groupe `Impact utilisateur` et y ajouter trois widgets RUM `peopulse` : erreurs, vues et temps de chargement Pc75

**Pourquoi :** l'impact utilisateur est la première question d'un incident. Il détermine l'urgence.

Création du groupe :

- **A9-3.1.1** Cliquez sur **Add Widgets**, saisissez `group` dans la recherche du panneau et cliquez sur **Empty Group** : un groupe **New group** apparaît sur le dashboard.
- **A9-3.1.2** Survolez l'en-tête **New group**, cliquez sur l'icône crayon, saisissez `Impact utilisateur` dans **Title** et cliquez sur **Save**.

Les trois widgets utilisent la même recherche RUM :

```text
@application.name:peopulse $env
```

Widget 1 - erreurs navigateur. Question : combien d'erreurs les utilisateurs rencontrent-ils ? Action : ouvrir la page RUM **Summary** de `peopulse` si le chiffre augmente.

- **A9-3.1.3** Dans le groupe `Impact utilisateur`, cliquez sur **Add Widgets or Powerpacks**, puis choisissez **Query Value**.
- **A9-3.1.4** Dans **Graph your data**, ouvrez la liste de la source, qui affiche **Logs** par défaut, saisissez `RUM` et choisissez **RUM**.
- **A9-3.1.5** Dans la liste placée après **In**, qui affiche **Sessions**, choisissez **Errors**.
- **A9-3.1.6** Cliquez dans la barre de recherche du widget et saisissez la recherche RUM ci-dessus.
- **A9-3.1.7** Conservez **Count of all errors** et ne regroupez par aucun attribut.
- **A9-3.1.8** Donnez le titre `Impact - erreurs navigateur peopulse` et cliquez sur **Save**.

Widget 2 - vues. Question : les utilisateurs utilisent-ils l'application normalement ? Action : comparer avec la même heure d'un autre jour si la courbe chute.

- **A9-3.1.9** Dans le groupe, ajoutez un widget **Timeseries** avec la source **RUM**, le type **Views** et la même recherche.
- **A9-3.1.10** Conservez **Count of all views**, donnez le titre `Impact - vues peopulse` et cliquez sur **Save**.

Widget 3 - temps de chargement. Question : les utilisateurs attendent-ils plus que d'habitude ? Action : ouvrir la page RUM **Summary** de `peopulse` pour comparer avec la semaine précédente.

- **A9-3.1.11** Dans le groupe, ajoutez un widget **Query Value** avec la source **RUM**, le type **Views** et la même recherche.
- **A9-3.1.12** Cliquez sur la pastille jaune **all views**, à droite de **Count of**, puis choisissez **Loading Time (@view.loading_time)** dans la liste : l'affichage devient **Avg of Loading Time**.
- **A9-3.1.13** Cliquez sur **Avg of** et choisissez **Pc75**.
- **A9-3.1.14** Donnez le titre `Impact - temps de chargement p75 peopulse` et cliquez sur **Save**.

**Résultat attendu :** le groupe contient un nombre d'erreurs, l'évolution des vues et un temps de chargement d'environ une seconde. Si le nombre d'erreurs vaut zéro, sélectionnez **Past 1 Day** dans le sélecteur de période du dashboard : l'activité de `peopulse` dépend des heures de travail des utilisateurs. **Pc75** est le 75e centile : 75 % des pages se chargent plus vite que cette valeur. C'est l'indicateur retenu par les navigateurs pour juger l'expérience. Une chute brutale du nombre de vues en journée peut signaler que les utilisateurs n'arrivent plus à charger l'application, même sans hausse des erreurs.

## 3.2. Créer le groupe `Comportement des traitements` et y ajouter quatre widgets APM `eu-interfaces` / `console` : requêtes, comparaison avec la veille, erreurs et latence p95

**Pourquoi :** ces mesures disent si les traitements s'exécutent plus ou moins souvent que d'habitude, s'ils échouent et s'ils ralentissent.

- **A9-3.2.1** Créez un groupe **Empty Group** et nommez-le `Comportement des traitements`, comme à l'action 3.1.

Les quatre widgets utilisent la source **APM Metrics**, le service `eu-interfaces`, l'opération `console` et le filtre `$env`. Pour régler un widget sur cette source :

- **A9-3.2.2** Ouvrez la liste de la source, saisissez `APM Metrics` et choisissez **APM Metrics**.
- **A9-3.2.3** Dans **Service**, saisissez `eu-interfaces` et validez : **Operation** se remplit automatiquement avec `console`.
- **A9-3.2.4** Dans **Filter by**, saisissez `$env` et cliquez sur la suggestion `$env`.

Widget 4 - requêtes. Question : les traitements s'exécutent-ils au rythme habituel ? Action : ouvrir la page APM du service si la courbe s'arrête ou s'emballe.

- **A9-3.2.5** Dans le groupe, ajoutez un widget **Timeseries** et réglez la source comme ci-dessus.
- **A9-3.2.6** Conservez la mesure **Requests**.
- **A9-3.2.7** Donnez le titre `Traitements - requêtes eu-interfaces / console` et cliquez sur **Save**.

Widget 5 - comparaison avec la veille. Question : y a-t-il plus ou moins de traitements qu'hier à la même heure ? Action : vérifier un changement de planification ou de version si l'écart est important.

- **A9-3.2.8** Dans le groupe, ajoutez un widget **Change** : il s'ouvre sur des logs groupés par service, comparés à **an hour before**.
- **A9-3.2.9** Réglez la source comme ci-dessus, avec la mesure **Requests**.
- **A9-3.2.10** Dans **Compare base value**, ouvrez la liste **an hour before** et choisissez **a day before**.
- **A9-3.2.11** Dans **Show the**, choisissez **relative change** pour afficher l'écart en pourcentage.
- **A9-3.2.12** Donnez le titre `Traitements - requêtes par rapport à la veille` et cliquez sur **Save**.

Widget 6 - erreurs. Question : des traitements échouent-ils ? Action : ouvrir les traces en erreur depuis la page APM si le chiffre est supérieur à zéro.

- **A9-3.2.13** Dans le groupe, ajoutez un widget **Query Value**, réglez la source comme ci-dessus, puis choisissez la mesure **Errors** dans la liste qui affiche **Requests**.
- **A9-3.2.14** Donnez le titre `Traitements - erreurs APM eu-interfaces / console` et cliquez sur **Save**.

Widget 7 - latence. Question : les traitements durent-ils plus longtemps que d'habitude ? Action : ouvrir les traces les plus longues avant d'attribuer une cause.

- **A9-3.2.15** Dans le groupe, ajoutez un widget **Timeseries**, réglez la source comme ci-dessus, puis choisissez la mesure **p95 Latency**.
- **A9-3.2.16** Donnez le titre `Traitements - latence p95 eu-interfaces / console` et cliquez sur **Save**.

**Résultat attendu :** le groupe affiche le volume, l'écart avec la veille en pourcentage, les erreurs et la latence des traitements. Avant que vous le remplaciez, le titre suggéré par Datadog contient `for {$env}` ou `for {env:prod}` : la variable est bien appliquée. La latence p95 de `console` se compte en minutes, car ce sont des traitements planifiés et non des pages web. Placez toujours le volume avant la latence : avec peu d'exécutions, un seul traitement long suffit à faire bondir la p95. Le widget **Change** évite de mémoriser les valeurs habituelles : il affiche directement l'écart, par exemple `+2,3 %`.

## 3.3. Créer le groupe `Logs applicatifs` et y ajouter quatre widgets de logs `eu-interfaces` : volume par statut, erreurs, commandes en erreur et derniers messages

**Pourquoi :** les traces de `console` ne montrent ni le nom de la commande exécutée ni le message d'erreur. Les logs, eux, portent l'attribut `@cmd.name`, qui indique quelle interface a été traitée : ils localisent le problème.

- **A9-3.3.1** Créez un groupe **Empty Group** et nommez-le `Logs applicatifs`.

Les quatre widgets utilisent la source **Logs**, sélectionnée par défaut, et la recherche de base :

```text
service:eu-interfaces $env
```

Widget 8 - volume par statut. Question : quand les erreurs apparaissent-elles, et en quelle proportion ? Action : survoler le pic pour en lire l'heure.

- **A9-3.3.2** Dans le groupe, ajoutez un widget **Timeseries**.
- **A9-3.3.3** Cliquez dans la barre de recherche du widget et saisissez la recherche de base.
- **A9-3.3.4** Cliquez sur **(Everything)** après **by**, saisissez `status` et choisissez **Status (status)**, la première proposition ; d'autres attributs nommés « Status » existent, ne les choisissez pas.
- **A9-3.3.5** Cliquez sur le titre **Graph your data** pour refermer la liste.
- **A9-3.3.6** Dans **Display**, remplacez **Lines** par **Bars**.
- **A9-3.3.7** Donnez le titre `Logs - volume par statut` et cliquez sur **Save**.

Widget 9 - erreurs. Question : combien de logs en erreur sur la période ? Action : ouvrir les logs en erreur si le chiffre augmente.

- **A9-3.3.8** Dans le groupe, ajoutez un widget **Query Value** et saisissez la recherche :

  ```text
  service:eu-interfaces $env status:error
  ```

- **A9-3.3.9** Donnez le titre `Logs - erreurs eu-interfaces` et cliquez sur **Save**.

Widget 10 - commandes en erreur. Question : quelle interface produit les erreurs ? Action : transmettre le nom de la commande en tête à l'équipe propriétaire.

- **A9-3.3.10** Dans le groupe, ajoutez un widget **Top List** avec la recherche `service:eu-interfaces $env status:error` : il est groupé par défaut **by Service**.
- **A9-3.3.11** Cliquez sur **Service** après **by**, saisissez `cmd.name` et choisissez **@cmd.name**.
- **A9-3.3.12** Cliquez sur le titre **Graph your data**, puis retirez le regroupement **Service** avec sa croix : seul **@cmd.name** doit rester.
- **A9-3.3.13** Conservez la limite **top 10**.
- **A9-3.3.14** Donnez le titre `Logs - commandes en erreur` et cliquez sur **Save**.

Widget 11 - derniers messages. Question : que disent les dernières erreurs ? Action : lire deux ou trois messages pour formuler une hypothèse à transmettre.

- **A9-3.3.15** Dans le groupe, ajoutez un widget **List** : il s'ouvre sur la source **Issues**.
- **A9-3.3.16** Ouvrez la liste **Issues** et choisissez **Logs**.
- **A9-3.3.17** Saisissez la recherche `service:eu-interfaces $env status:error`.
- **A9-3.3.18** Conservez les colonnes **Date**, **Host**, **Service** et **Content** ; n'ajoutez aucune colonne d'identifiant.
- **A9-3.3.19** Donnez le titre `Logs - dernières erreurs` et cliquez sur **Save**.

**Résultat attendu :** le groupe affiche le volume de logs par statut en barres, le nombre d'erreurs, un classement des commandes et une liste de messages. Sur **Past 1 Day**, le widget 10 affiche des noms de commandes de la forme `mission:export:…` ou `facture:export:…`. Sur une heure sans erreur, les widgets 9 et 10 affichent « (No data) » ou « No Data » : ce n'est pas une panne, mais l'absence de logs en erreur sur la période. Une application peut écrire en `Error` des situations attendues, par exemple « aucun fichier à exporter ». Le volume d'erreurs seul ne suffit donc pas : c'est la concentration sur une commande, ou l'apparition d'un nouveau message, qui signale un problème. Si un message affiché contient un identifiant de personne, supprimez le widget **List** et conservez le classement par commande.

## 3.4. Créer le groupe `Alerte` et y ajouter l'état du monitor existant « Facture V2 » et le graphique d'un monitor de traitements

**Pourquoi :** un dashboard de qualification montre aussi ce que surveillent déjà les alertes de l'organisation. Vous n'en créez aucune : vous affichez deux monitors existants pour savoir, pendant un incident, s'ils ont sonné et ce qu'ils couvrent.

Les deux monitors utilisés existent déjà sur la plateforme :

| Monitor existant | Ce qu'il surveille | Type |
|---|---|---|
| `[Facture V2] Erreurs 500 sur l'export multi-client des factures` | les logs d'erreur 500 du service `app-sigman-calc` lors de l'export des factures | monitor de logs |
| `🔴 [PROD] app-bamboo - Spike in application errors` | le nombre d'erreurs des traitements `console` du service `app-bamboo` sur 5 minutes, seuil 5 | monitor de métrique |

- **A9-3.4.1** Créez un groupe **Empty Group** et nommez-le `Alerte`.

Widget 12 - état du monitor des factures. Question : l'alerte sur les exports de factures est-elle active ? Action : ouvrir le monitor et son historique.

- **A9-3.4.2** Dans le groupe, cliquez sur **Add Widgets or Powerpacks**, saisissez `monitor summary` dans la recherche du panneau et choisissez **Monitor Summary** : la fenêtre **Monitor Summary Editor** liste d'abord tous les monitors de l'organisation.
- **A9-3.4.3** Conservez le type **Monitor** dans **Select summary type**.
- **A9-3.4.4** Dans **Search for monitors to summarize**, saisissez `Facture V2` et vérifiez que l'aperçu indique **Showing 1-1 of 1 monitor**.
- **A9-3.4.5** Conservez l'affichage **Both**, saisissez `Alerte - état du monitor Facture V2` dans **Widget title**, appuyez sur Tab, puis cliquez sur **Save**.

Widget 13 - erreurs d'un traitement et seuil d'alerte. Question : à quoi ressemble une alerte bien réglée sur des traitements `console` ? Action : comparer la courbe au seuil avant de conclure à un incident.

- **A9-3.4.6** Dans le groupe, ajoutez un widget **Alert Graph** : saisissez `alert graph` dans la recherche du panneau.
- **A9-3.4.7** Cliquez dans **Choose an alert**, saisissez `bamboo` et choisissez `🔴 [PROD] app-bamboo - Spike in application errors`.
- **A9-3.4.8** Conservez la visualisation **Timeseries**.
- **A9-3.4.9** Donnez le titre `Alerte - erreurs console app-bamboo et seuil`, appuyez sur Tab, puis cliquez sur **Save**.

**Résultat attendu :** le widget 12 affiche le monitor « Facture V2 » et son état, en général `OK` ; le widget 13 affiche une courbe d'erreurs, la zone rouge au-dessus du seuil `y > 5` et l'état du monitor. L'**Alert Graph** n'accepte que les monitors de métriques : un monitor de logs comme « Facture V2 » n'apparaît pas dans sa liste, d'où l'usage du **Monitor Summary** pour ce dernier. Aucun de ces deux monitors ne surveille `eu-interfaces` : vous constaterez à la thématique 5 ce que cette absence implique.

## 3.5. Vérifier l'ordre de lecture et enregistrer la période de référence

**Pourquoi :** le dashboard doit se lire de haut en bas sans effort.

- **A9-3.5.1** Vérifiez que la note de lecture est en haut, suivie des groupes `Impact utilisateur`, `Comportement des traitements`, `Logs applicatifs` et `Alerte`.
- **A9-3.5.2** Si un widget se trouve hors de son groupe, faites-le glisser par sa barre de titre jusque dans le groupe.
- **A9-3.5.3** Sélectionnez **Past 1 Day** comme période.

**Résultat attendu :** le dashboard se lit de haut en bas, conformément au plan du dashboard. Chaque widget porte un titre qui commence par son groupe, par exemple `Impact -`, `Traitements -`, `Logs -` ou `Alerte -`.

# 4. Exploiter le dashboard sur la situation réelle

## 4.1. Repérer l'anomalie la plus marquée de la journée et la localiser avec le classement des commandes

**Pourquoi :** un dashboard se juge à sa capacité à faire apparaître une anomalie réelle et à en réduire le périmètre en quelques secondes.

- **A9-4.1.1** Avec la période **Past 1 Day**, parcourez les groupes de haut en bas.
- **A9-4.1.2** Dans `Logs - volume par statut`, repérez le pic de logs en erreur le plus marqué et survolez-le pour lire son heure.
- **A9-4.1.3** Dans `Logs - commandes en erreur`, repérez la commande en tête et la part qu'elle représente.
- **A9-4.1.4** Dans le groupe `Comportement des traitements`, regardez si les requêtes, les erreurs APM ou la latence changent à la même heure.
- **A9-4.1.5** Dans le groupe `Impact utilisateur`, regardez si les erreurs ou les vues changent à la même heure.
- **A9-4.1.6** Complétez la partie « Lecture de la situation réelle » de `reponses-A9.txt`.

Exemple de réponse attendue :

```text
Anomalie la plus marquée (widget, heure) : Logs - volume par statut, pic d'erreurs dans la nuit
Périmètre (commande ou opération concernée) : cmad:export:efs, près de 90 % des logs en erreur
Signal non touché : erreurs APM toujours à 0 ; pas d'impact visible côté navigateur
```

**Résultat attendu :** l'anomalie est datée et attribuée à une commande précise. Le dashboard révèle ici une situation fréquente : les traitements se terminent sans erreur côté APM, alors que leurs logs signalent des erreurs massives sur une seule commande. Sans le widget de classement, on aurait seulement vu « beaucoup d'erreurs ». C'est l'information à transmettre à l'équipe propriétaire, accompagnée du nom de la commande et de l'heure.

## 4.2. Changer la variable `env` puis la période pour vérifier ce que chaque widget affiche

**Pourquoi :** un dashboard doit rester compréhensible quand on change son périmètre, et un widget vide doit pouvoir s'expliquer.

- **A9-4.2.1** Dans le bandeau, ouvrez la variable `env`, décochez `prod`, cochez `recette`, puis cliquez dans une zone vide du dashboard.
- **A9-4.2.2** Observez quels widgets se mettent à jour et lesquels deviennent vides.
- **A9-4.2.3** Revenez à `prod` de la même façon : décochez `recette` et cochez `prod`.
- **A9-4.2.4** Sélectionnez **Past 1 Hour**, observez les widgets qui se vident, puis revenez à **Past 1 Day**.

**Résultat attendu :** les widgets APM se mettent à jour en `recette` ; les widgets RUM deviennent vides, car `peopulse` n'envoie des données qu'en `prod` ; les widgets de logs affichent les logs de `recette` s'il y en a sur la période. Un widget vide n'indique pas une panne. Il peut venir de l'environnement, de la période ou de l'absence d'activité. Les widgets du groupe `Alerte` ne changent pas : ils affichent les monitors tels qu'ils ont été définis.

## 4.3. Ouvrir le détail d'un widget sans quitter le dashboard

**Pourquoi :** le dashboard qualifie la situation ; le diagnostic se poursuit dans les logs ou les traces, avec les mêmes filtres.

- **A9-4.3.1** Dans `Logs - commandes en erreur`, cliquez sur la commande en tête de liste : un menu s'ouvre.
- **A9-4.3.2** Choisissez **View logs** : un panneau s'ouvre à droite du dashboard avec les logs de cette commande.
- **A9-4.3.3** Vérifiez que la recherche du panneau contient `service:eu-interfaces`, `env:prod`, `status:error` et le nom de la commande, puis lisez deux messages.
- **A9-4.3.4** Repérez l'icône d'ouverture dans un nouvel onglet en haut à droite de la liste : elle ouvre la même recherche dans le Log Explorer.
- **A9-4.3.5** Fermez le panneau avec sa croix.
- **A9-4.3.6** Dans `Traitements - requêtes eu-interfaces / console`, cliquez sur un point de la courbe et observez que **View traces** et **View logs** sont grisés ; ne cliquez pas sur **Add annotation**, puis cliquez à côté du menu pour le fermer.
- **A9-4.3.7** Dans `Alerte - état du monitor Facture V2`, cliquez sur le nom du monitor, lisez sa requête et son historique sans rien modifier, puis revenez au dashboard.

**Résultat attendu :** le détail des logs s'ouvre en un clic depuis le classement des commandes, et le monitor depuis son résumé. Les widgets **APM Metrics** ne proposent pas de lien vers les traces : pour les traces, ouvrez la page APM du service. C'est une limite à connaître lorsqu'on conçoit un dashboard : un widget de logs ou de monitor mène directement au détail, un widget de métrique APM non.

## 4.4. Afficher un widget en plein écran puis relire sa requête en mode édition sans la modifier

**Pourquoi :** savoir lire la requête d'un widget permet de vérifier ce qu'il compte réellement, y compris sur un dashboard construit par quelqu'un d'autre.

- **A9-4.4.1** Survolez `Logs - commandes en erreur` et ouvrez-le en plein écran avec l'icône d'agrandissement.
- **A9-4.4.2** Fermez le plein écran.
- **A9-4.4.3** Ouvrez le même widget en édition.
- **A9-4.4.4** Identifiez la source, la recherche, la mesure et le regroupement.
- **A9-4.4.5** Cliquez sur **Cancel** sans rien modifier.

**Résultat attendu :** vous savez dire en une phrase ce que compte le widget : « le nombre de logs en erreur de `eu-interfaces`, dans l'environnement choisi, par commande ». Un titre peut mentir, la requête non. Avant de vous fier à un widget hérité, relisez sa requête.

# 5. Qualifier un incident signalé avec le dashboard

L'équipe comptabilité signale : « depuis ce matin, certaines factures ne sont pas exportées vers nos fournisseurs ». Votre rôle est de qualifier ce signalement avec le dashboard, puis de décider de la suite.

## 5.1. Cadrer la période sur les dernières heures et lire les trois groupes dans l'ordre

**Pourquoi :** qualifier un incident, c'est établir des faits sur la bonne période avant de chercher une cause.

- **A9-5.1.1** Sélectionnez **Past 4 Hours** dans le sélecteur de période du dashboard et vérifiez que la variable `env` vaut `prod`.
- **A9-5.1.2** Lisez le groupe `Impact utilisateur` : erreurs, vues et temps de chargement de `peopulse` ont-ils changé sur la période ?
- **A9-5.1.3** Lisez le groupe `Comportement des traitements` : le nombre de traitements est-il proche de la veille ? échouent-ils ou ralentissent-ils côté APM ?
- **A9-5.1.4** Lisez le groupe `Logs applicatifs` : dans `Logs - commandes en erreur`, repérez les commandes qui commencent par `facture:export`, par exemple `facture:export:vplog` ou `facture:export:lohr`.
- **A9-5.1.5** Lisez le groupe `Alerte` : le monitor « Facture V2 » est-il en alerte ?

**Résultat attendu :** au moins une commande `facture:export:…` apparaît dans le classement des commandes en erreur, alors que les groupes `Impact utilisateur` et `Comportement des traitements` ne montrent pas de rupture et que le monitor « Facture V2 » reste `OK`. Le signalement est confirmé par les logs et localisé sur des commandes précises, mais il ne se voit ni dans l'APM ni dans le RUM. C'est un incident métier : les traitements s'exécutent, mais une partie de leur travail échoue. Le monitor « Facture V2 » reste `OK` parce qu'il surveille un autre service, `app-sigman-calc` : aucune alerte ne couvre les exports de factures de `eu-interfaces`. C'est un trou de détection, à signaler en même temps que l'incident.

## 5.2. Lire la cause probable dans les logs de la commande `facture:export` la plus touchée

**Pourquoi :** la décision dépend de la nature de l'erreur : panne technique, donnée manquante ou situation attendue.

- **A9-5.2.1** Dans `Logs - commandes en erreur`, cliquez sur la commande `facture:export:…` qui compte le plus d'erreurs et choisissez **View logs**.
- **A9-5.2.2** Dans le panneau, lisez trois messages d'erreur et repérez ce qu'ils ont en commun.
- **A9-5.2.3** Fermez le panneau avec sa croix.
- **A9-5.2.4** Survolez les barres d'erreur de `Logs - volume par statut` pour repérer à partir de quelle heure les erreurs apparaissent.

**Résultat attendu :** les messages se répètent, par exemple « Pas de code analytique pour la facture BC4/00017/24 » ou « Facture id … invalide : CODE_FOURNISSEUR est manquant », et les erreurs reviennent par salves à chaque heure pleine. Un message qui se répète à l'identique pour de nombreuses factures désigne une cause commune. Ici, des données manquent dans les factures, par exemple le code analytique, et les mêmes factures échouent à chaque exécution horaire du traitement : la correction relève de l'équipe qui gère ces données, pas d'une intervention technique sur le serveur.

## 5.3. Décider de la suite et rédiger la qualification dans `reponses-A9.txt`

**Pourquoi :** une qualification utile se termine par une décision justifiée, transmissible à l'équipe concernée sans autre explication.

- **A9-5.3.1** Choisissez une décision :
  - **clore** : le signalement ne correspond à aucune anomalie visible ;
  - **surveiller** : une anomalie existe, mais reste limitée et sans conséquence immédiate ;
  - **escalader** : une anomalie est confirmée et une équipe doit agir.
- **A9-5.3.2** Complétez la partie « Qualification de l'incident signalé » de `reponses-A9.txt`.
- **A9-5.3.3** Comparez votre qualification avec l'exemple ci-dessous et vérifiez que chaque rubrique cite une valeur, une commande ou un widget du dashboard.

Exemple de qualification de référence :

```text
Impact utilisateur : aucun impact visible sur peopulse (erreurs et temps de chargement stables) ;
  aucun lien vérifié entre peopulse et les exports de factures.
Depuis quand : erreurs facture:export par salves à chaque heure pleine sur les 4 dernières heures.
Périmètre touché : commande facture:export:vplog, 45 erreurs sur 4 heures, toujours les mêmes factures ;
  traitements console sans rupture côté APM, nombre proche de la veille.
Cause probable lue dans les logs : « Pas de code analytique pour la facture BC4/00017/24 ».
Alerte déclenchée (oui / non) et monitor concerné : non — le monitor Facture V2 surveille
  app-sigman-calc ; aucun monitor ne couvre les exports de factures de eu-interfaces.
Décision et raison : escalader à l'équipe propriétaire de l'interface vplog -
  anomalie confirmée, données analytiques manquantes ; signaler aussi l'absence d'alerte.
```

**Résultat attendu :** votre qualification contient les mêmes types d'éléments que l'exemple, avec les valeurs visibles le jour de l'atelier. Les valeurs et la commande concernée peuvent changer d'un jour à l'autre. Si aucune commande `facture:export` n'apparaît en erreur, la bonne décision est « clore » : le dashboard ne confirme pas le signalement sur la période.

# 6. Recette

## 6.1. Contrôler le dashboard avec la grille de recette

**Pourquoi :** un dashboard partagé doit être exact, lisible et sans donnée sensible.

- **A9-6.1.1** Vérifiez chaque point de la grille ci-dessous dans le dashboard.
- **A9-6.1.2** Corrigez uniquement votre dashboard si un point n'est pas conforme.

| Contrôle | Résultat attendu |
|---|---|
| Titre du dashboard | `[TRAINING] M09 - <participant>`, unique |
| Note de lecture | en haut, avec les limites de corrélation |
| Groupes | Impact utilisateur, Comportement des traitements, Logs applicatifs, Alerte |
| Widgets 1 à 3 (RUM) | agrégés, filtrés sur `peopulse` et `$env`, sans utilisateur, session ni URL |
| Widgets 4 à 7 (APM) | `eu-interfaces` / `console`, avec `$env` |
| Widgets 8 à 11 (logs) | `service:eu-interfaces $env`, classement par `@cmd.name`, aucun identifiant de personne |
| Widgets 12 et 13 (alerte) | monitors existants Facture V2 et app-bamboo, affichés sans modification |
| Titres | préfixés par leur groupe, avec la source et la question |
| Partage | aucun partage public |

**Résultat attendu :** tous les contrôles sont conformes.

## Questions de synthèse

### Pourquoi placer l'impact utilisateur avant le comportement du service ?

**Réponse :** parce que l'impact détermine l'urgence. Un traitement qui ralentit la nuit sans utilisateur connecté n'appelle pas la même réaction qu'un ralentissement pendant les heures d'activité.

### Pourquoi ajouter des widgets de logs alors que l'APM montre déjà les erreurs ?

**Réponse :** parce que les deux ne voient pas la même chose. Ici, l'APM compte zéro erreur alors que les logs signalent des milliers d'erreurs sur une commande : le traitement se termine « normalement » du point de vue technique, mais son travail échoue. Seuls les logs le révèlent, et seul le classement par commande dit laquelle.

### Pourquoi limiter le dashboard à treize widgets ?

**Réponse :** chaque widget répond à une question et conduit à une action. Au-delà, le dashboard devient une collection de graphiques que plus personne ne lit en situation d'incident ; le détail reste dans les explorateurs, que les pivots ouvrent en un clic.

### Pourquoi un dashboard ne remplace-t-il pas l'investigation ?

**Réponse :** il qualifie la situation et oriente vers la bonne vue. Par exemple, le classement des commandes indique où se concentrent les erreurs ; seule la lecture des logs de cette commande dans le Log Explorer permet de comprendre pourquoi.

### Que faut-il pour qu'un dashboard de qualification soit fiable en production ?

**Réponse :** les sources affichées doivent appartenir à la même chaîne applicative et porter les mêmes tags `service`, `env` et `version`, et les logs doivent être reliés aux traces. C'est ce qui permet de relier un impact utilisateur à un traitement précis.

### Pourquoi afficher des monitors qui ne surveillent pas `eu-interfaces` ?

**Réponse :** parce que le dashboard doit montrer ce qui est réellement surveillé. Voir le monitor « Facture V2 » rester `OK` pendant que les exports de factures de `eu-interfaces` échouent révèle un trou de détection : sans le dashboard, l'incident n'aurait été connu que par le signalement de l'équipe métier.

## Nettoyage en fin de formation

Uniquement avec l'autorisation du formateur :

1. recherchez le dashboard `[TRAINING] M09 - <participant>` par son nom exact ;
2. vérifiez le préfixe et votre identifiant ;
3. supprimez ce dashboard et aucun autre.

## Aide au diagnostic

### Un dashboard au nom par défaut apparaît dans la liste

- vous avez cliqué plusieurs fois sur **Blank Dashboard** ; signalez-le au formateur sans le supprimer vous-même.

### Un widget APM reste vide

- vérifiez la valeur de la variable `env` dans le bandeau ;
- ouvrez le widget et vérifiez que **Filter by** contient `$env` ;
- élargissez la période à **Past 1 Day**.

### La variable `env` ne modifie pas un widget

- ouvrez le widget et vérifiez que sa requête contient `$env` ;
- ouvrez la variable dans le bandeau et cliquez sur **Select All** dans **Apply to Widgets**.

### Le widget RUM affiche une erreur de requête

- vérifiez la recherche exacte `@application.name:peopulse $env` ;
- vérifiez que le type d'événement après **In** est **Errors** ou **Views**.

### La mesure Loading Time n'est pas proposée

- vérifiez que le type d'événement est **Views** et non **Sessions** ;
- cliquez sur la pastille jaune **all views** et non sur **Count of** ; signalez la référence `A9-3.1.12` au formateur si la mesure reste absente.

### L'attribut `@cmd.name` n'est pas proposé dans le Top List

- saisissez `cmd.name` en entier dans la liste **by** et choisissez **@cmd.name**, et non « Add group by cmd.name » ;
- signalez la référence `A9-3.3.11` au formateur et regroupez provisoirement par **Host**.

### Le Top List affiche des barres de couleur empilées

- le regroupement **Service** est encore présent à côté de **@cmd.name** ; retirez-le avec sa croix.

### Le monitor « Facture V2 » n'apparaît pas dans l'Alert Graph

- c'est normal : l'**Alert Graph** n'accepte que les monitors de métriques ; affichez « Facture V2 » avec le **Monitor Summary** (widget 12).

### La variable `env` affiche deux valeurs

- la variable accepte plusieurs valeurs cochées ; décochez celle qui ne doit plus s'appliquer.

### Une fenêtre « Quick Graph » s'est ouverte

- une lettre a été tapée hors d'un champ ; cliquez sur **Cancel**, puis cliquez dans le champ voulu avant de taper.

### Un widget a gardé le titre proposé par Datadog

- survolez le widget, cliquez sur l'icône crayon, corrigez **Give your graph a title** et cliquez sur **Save**.

### L'éditeur d'un widget s'est fermé sans enregistrer

- la touche Échap ferme l'éditeur ; recommencez l'ajout du widget et terminez toujours par **Save**.

## Références officielles

- [Datadog — Dashboards](https://docs.datadoghq.com/dashboards/)
- [Datadog — Template Variables](https://docs.datadoghq.com/dashboards/template_variables/)
- [Datadog — Change Widget](https://docs.datadoghq.com/dashboards/widgets/change/)
- [Datadog — Top List Widget](https://docs.datadoghq.com/dashboards/widgets/top_list/)
- [Datadog — List Widget](https://docs.datadoghq.com/dashboards/widgets/list/)
- [Datadog — Monitor Summary Widget](https://docs.datadoghq.com/dashboards/widgets/monitor_summary/)
- [Datadog — Alert Graph Widget](https://docs.datadoghq.com/dashboards/widgets/alert_graph/)
- [Datadog — Group Widget](https://docs.datadoghq.com/dashboards/widgets/group/)
