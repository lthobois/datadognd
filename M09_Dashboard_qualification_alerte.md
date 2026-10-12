---
title: "Atelier 9 — Construire un dashboard et qualifier une alerte"
subtitle: "Document participant autonome — parcours non développeurs"
lang: fr-FR
---

# Construire un dashboard et qualifier une alerte

## Situation professionnelle

Les dashboards de votre organisation ont été construits par des profils techniques. Vous avez besoin d'une vue qui vous permette, lorsqu'une alerte arrive, de répondre en quelques minutes à quatre questions : les utilisateurs sont-ils touchés ? depuis quand ? sur quel périmètre ? faut-il escalader ?

## Objectif opérationnel

Auditer un dashboard existant, construire votre propre dashboard de qualification à partir des données réelles de l'application `peopulse` et du service `eu-interfaces` (traces, logs et monitor), l'exploiter pour lire une situation réelle, le relier au monitor créé à l'atelier 8, puis l'utiliser pour qualifier une alerte de bout en bout.

## Livrable

À la fin de l'atelier, vous disposez :

- d'un dashboard `[TRAINING] M09 - <participant>` de treize widgets, organisé de l'impact vers le diagnostic ;
- d'une note de lecture et de quatre groupes de widgets ;
- d'une variable de dashboard `env` utilisée par tous les widgets de données ;
- d'un lien dans le message de votre monitor qui ouvre ce dashboard ;
- d'une lecture de la situation réelle et d'une qualification d'alerte rédigées dans `reponses-A9.txt`.

## Environnement

- Chrome connecté à l'organisation Datadog de la formation ;
- accès aux dashboards, à APM, aux logs, au RUM et aux monitors ;
- droit de créer un dashboard et de modifier votre propre monitor ;
- le monitor `[TRAINING] M08 APM - <participant>` créé à l'atelier 8 ;
- le fichier `reponses-A8.txt` ;
- un éditeur de texte simple.

## Règles d'écriture

Les seules écritures autorisées sont :

1. la création et la modification de **votre** dashboard `[TRAINING] M09 - <participant>` ;
2. la modification du seuil et du message de **votre** monitor `[TRAINING] M08 APM - <participant>`.

N'activez aucun partage public, ne modifiez aucun dashboard existant et n'ajoutez aucun destinataire au monitor. Ne regroupez jamais un widget par utilisateur, session, adresse IP ou URL complète.

Remplacez `<participant>` par l'identifiant fourni par le formateur, par exemple `loic`.

Si vous ne disposez pas du droit de créer un dashboard, réalisez la thématique 1, puis suivez les thématiques 2 à 6 sur le dashboard du formateur.

## Fichier de réponses

Créez dans votre éditeur un fichier nommé `reponses-A9.txt` et copiez-y la trame suivante. Vous compléterez la première partie à la thématique 4 et la seconde à la thématique 6.

```text
Atelier 9 — Dashboard de qualification
Participant :

Lecture de la situation réelle
  Anomalie la plus marquée (widget, heure) :
  Périmètre (commande ou opération concernée) :
  Signal non touché :

Qualification d'alerte
  Alerte reçue (monitor, heure) :
  Impact utilisateur :
  Depuis quand :
  Périmètre touché :
  Décision (clore / surveiller / escalader) et raison :
```

## Sources de données utilisées

Le dashboard réunit trois sources réelles et votre monitor :

| Source | Ce qu'elle mesure | Rôle dans le dashboard |
|---|---|---|
| RUM `peopulse` | ce que vivent les utilisateurs dans le navigateur | impact utilisateur |
| APM `eu-interfaces` / `console` | l'exécution des traitements planifiés côté serveur | comportement des traitements |
| Logs `eu-interfaces` | les messages écrits par les traitements, dont le nom de la commande exécutée | logs applicatifs |
| Monitor `[TRAINING] M08 APM - <participant>` | l'état de votre règle d'alerte | alerte |

Les traces et les logs de `eu-interfaces` décrivent le même service, mais aucun identifiant ne relie une trace à ses logs. Aucun lien technique entre les sessions RUM de `peopulse` et les traitements de `eu-interfaces` n'a été vérifié. Elles sont réunies ici pour apprendre à organiser un dashboard selon l'ordre de lecture d'un incident. Sur un dashboard de production, ces liens doivent être vérifiés avant d'en tirer une conclusion.

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
| Alerte                      | Alert Graph | état du monitor   |
+----------------------------------------------------------------+
```

Chaque widget répond à une question et conduit à une action. Les questions sont rappelées sous chaque action de la thématique 3.

# 1. Auditer un dashboard existant

## 1.1. Ouvrir `Monitoring EU-Interfaces` et lire sa période et ses variables

**Pourquoi :** observer une vue existante montre ce qui facilite ou ralentit une décision, avant d'en construire une.

- **A9-1.1.1** Ouvrez **Dashboards > Dashboard List**.
- **A9-1.1.2** Dans le champ de recherche, saisissez `Monitoring EU-Interfaces` et ouvrez le dashboard portant ce titre.
- **A9-1.1.3** Ne cliquez sur aucun widget pour l'éditer.
- **A9-1.1.4** Observez la période active et les variables affichées dans **Filter by**.

**Résultat attendu :** vous savez sur quelle période et quel périmètre portent les widgets.

**Interprétation :** une variable permet de changer le périmètre de tout le dashboard en un clic. Par exemple, une variable `version` permet de comparer deux versions déployées sans dupliquer les widgets.

## 1.2. Associer une question à chaque widget et repérer l'ordre de lecture impact → chronologie → périmètre → diagnostic

**Pourquoi :** pendant un incident, on lit un dashboard de haut en bas. L'ordre des widgets doit suivre l'ordre des questions.

- **A9-1.2.1** Parcourez les widgets de haut en bas.
- **A9-1.2.2** Pour chaque widget, déterminez la question à laquelle il répond, en vous aidant de son titre et de son contenu.
- **A9-1.2.3** Repérez le premier widget qui indique l'impact, par exemple un nombre de traitements en erreur.
- **A9-1.2.4** Repérez le premier widget qui montre une évolution dans le temps.
- **A9-1.2.5** Repérez un widget qui permet de réduire le périmètre, par exemple un classement par interface.
- **A9-1.2.6** Repérez un widget dont la question n'est pas claire à la seule lecture de son titre.

**Résultat attendu :** en comparant vos repérages avec le tableau ci-dessous, vous savez si l'ordre de lecture suit les questions d'un incident et quel titre devrait être amélioré.

**Interprétation :** l'ordre de référence est le suivant :

| Niveau | Question | Exemple de widget |
|---|---|---|
| Impact | Les utilisateurs sont-ils touchés ? | nombre d'erreurs côté navigateur |
| Chronologie | Depuis quand ? | courbe des requêtes ou des logs en erreur |
| Périmètre | Où exactement ? | classement des commandes en erreur |
| Diagnostic | Quelle piste ouvrir ? | liste des derniers messages d'erreur |

Un dashboard qui commence par des détails techniques oblige le lecteur à reconstituer lui-même l'impact. Un titre comme « Erreurs » est insuffisant : « Logs - commandes en erreur » dit à la fois la source et la question.

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
| **Alert Graph** | Pourquoi l'alerte sonne-t-elle ? | débit comparé aux seuils du monitor |
| **Monitor Summary** | L'alerte est-elle active ? | état de votre monitor |

- **A9-1.3.1** Revenez à `Monitoring EU-Interfaces` et identifiez le type de deux de ses widgets.
- **A9-1.3.2** Pour chacun, déterminez sa question, retrouvez dans le tableau le type qui y répond le mieux, et comparez avec le type utilisé.

**Résultat attendu :** pour chacun des deux widgets, vous savez si son type est adapté à sa question ou lequel l'aurait mieux été.

**Interprétation :** un **Query Value** masque la chronologie ; une **Timeseries** montre mal qui contribue le plus. C'est pourquoi un bon dashboard combine plusieurs types sur une même source.

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

**Résultat attendu :** un dashboard vide portant votre identifiant est ouvert.

**Vérification :** l'onglet du navigateur affiche `[TRAINING] M09 - <participant>`.

**Interprétation :** Datadog enregistre automatiquement le dashboard dès sa création. Un dashboard abandonné reste donc dans la liste avec son nom par défaut : renommez-le toujours immédiatement.

## 2.2. Ajouter la variable de dashboard `env` avec la valeur par défaut `prod`

**Pourquoi :** une variable change le périmètre de tous les widgets qui l'utilisent. Elle évite de dupliquer le dashboard pour chaque environnement.

- **A9-2.2.1** Sous le titre, cliquez sur **Add Variable**.
- **A9-2.2.2** Saisissez `env` dans la recherche, puis cliquez sur `env` dans **Matching Tags**.
- **A9-2.2.3** Dans le panneau **Edit $env**, conservez le type **Filter**.
- **A9-2.2.4** Dans **Default value**, saisissez `prod` et cochez la valeur `prod` proposée.
- **A9-2.2.5** Cliquez sur **Save** en haut du panneau.

**Résultat attendu :** le bandeau affiche **Filter by** avec la variable `env` à `prod`.

**Interprétation :** la variable n'agit que sur les widgets dont la requête contient `$env`. Vous l'ajouterez dans chaque widget de données à la thématique 3.

## 2.3. Ajouter une note de lecture indiquant la question du dashboard, ses sources et ses limites

**Pourquoi :** une personne qui ouvre le dashboard pour la première fois doit comprendre en dix secondes à quoi il sert et ce qu'il ne permet pas de conclure.

- **A9-2.3.1** Cliquez sur **Add Widgets**, saisissez `note` dans la recherche du panneau et choisissez **Notes & Links**.
- **A9-2.3.2** Dans la fenêtre **Notes & Links Editor**, cliquez dans le champ **Markdown** et saisissez le contenu suivant :

  ```markdown
  ## Qualifier une alerte sur eu-interfaces / console

  Lire de haut en bas : impact utilisateur, traitements, logs, alerte.

  - Impact utilisateur : application RUM peopulse (agrégats)
  - Traitements : APM eu-interfaces, opération console
  - Logs applicatifs : logs eu-interfaces, par commande
  - Alerte : monitor [TRAINING] M08 APM

  Aucun lien technique entre peopulse et eu-interfaces n'a été vérifié,
  et les logs ne sont pas reliés aux traces :
  des variations simultanées ne prouvent aucune cause commune.
  ```

- **A9-2.3.3** Vérifiez le rendu dans la zone **Preview**, puis cliquez sur **Save**.

**Résultat attendu :** la note est visible sur le dashboard.

# 3. Construire les widgets de l'impact vers le diagnostic

Les widgets sont rangés dans quatre groupes. Pour chaque groupe, vous créez d'abord le groupe, puis vous ajoutez ses widgets **à l'intérieur**, avec le cadre **Add Widgets or Powerpacks** qui apparaît sous son titre. Déplacer un widget existant vers un groupe est plus difficile : respectez cet ordre.

Chaque widget s'ajoute de la même façon : choix du type dans le panneau **Widgets** (son champ de recherche permet de le retrouver par son nom), réglage de la source dans **Graph your data**, titre dans **Give your graph a title**, puis **Save** en bas à droite. Après chaque **Save**, vérifiez le titre affiché sur le widget : s'il a gardé le titre proposé par Datadog, survolez le widget, cliquez sur l'icône crayon et corrigez le titre.

Deux précautions évitent la plupart des blocages :

- n'appuyez pas sur Échap dans l'éditeur d'un widget : cela le ferme sans l'enregistrer ;
- tapez toujours dans un champ sur lequel vous venez de cliquer : une lettre tapée ailleurs déclenche un raccourci de Datadog, par exemple `G` ouvre la fenêtre **Quick Graph** ; dans ce cas, cliquez sur **Cancel**.

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
- **A9-3.1.12** Cliquez sur **Count of**, saisissez `loading` dans la recherche de la liste et choisissez **Loading Time (@view.loading_time)** : l'affichage devient **Avg of Loading Time**.
- **A9-3.1.13** Cliquez sur **Avg of** et choisissez **Pc75**.
- **A9-3.1.14** Donnez le titre `Impact - temps de chargement p75 peopulse` et cliquez sur **Save**.

**Résultat attendu :** le groupe contient un nombre d'erreurs, l'évolution des vues et un temps de chargement d'environ une seconde.

**Vérification :** si le nombre d'erreurs vaut zéro, sélectionnez **Past 1 Day** dans le sélecteur de période du dashboard : l'activité de `peopulse` dépend des heures de travail des utilisateurs.

**Interprétation :** **Pc75** est le 75e centile : 75 % des pages se chargent plus vite que cette valeur. C'est l'indicateur retenu par les navigateurs pour juger l'expérience. Une chute brutale du nombre de vues en journée peut signaler que les utilisateurs n'arrivent plus à charger l'application, même sans hausse des erreurs.

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

**Résultat attendu :** le groupe affiche le volume, l'écart avec la veille en pourcentage, les erreurs et la latence des traitements.

**Vérification :** avant que vous le remplaciez, le titre suggéré par Datadog contient `for {$env}` ou `for {env:prod}` : la variable est bien appliquée.

**Interprétation :** la latence p95 de `console` se compte en minutes, car ce sont des traitements planifiés et non des pages web. Placez toujours le volume avant la latence : avec peu d'exécutions, un seul traitement long suffit à faire bondir la p95. Le widget **Change** évite de mémoriser les valeurs habituelles : il affiche directement l'écart, par exemple `+2,3 %`.

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

**Résultat attendu :** le groupe affiche le volume de logs par statut en barres, le nombre d'erreurs, un classement des commandes et une liste de messages.

**Vérification :** le widget 10 affiche des noms de commandes de la forme `mission:export:…` ou `facture:export:…`.

**Interprétation :** une application peut écrire en `Error` des situations attendues, par exemple « aucun fichier à exporter ». Le volume d'erreurs seul ne suffit donc pas : c'est la concentration sur une commande, ou l'apparition d'un nouveau message, qui signale un problème. Si un message affiché contient un identifiant de personne, supprimez le widget **List** et conservez le classement par commande.

## 3.4. Créer le groupe `Alerte` et y ajouter l'Alert Graph et le Monitor Summary de votre monitor

**Pourquoi :** voir la mesure du monitor avec ses seuils explique pourquoi l'alerte sonne ou ne sonne pas ; le résumé donne son état en un mot.

- **A9-3.4.1** Créez un groupe **Empty Group** et nommez-le `Alerte`.

Widget 12 - débit et seuils. Question : pourquoi l'alerte sonne-t-elle, ou pourquoi ne sonne-t-elle pas ? Action : comparer la courbe au seuil avant de conclure à un incident.

- **A9-3.4.2** Dans le groupe, cliquez sur **Add Widgets or Powerpacks**, saisissez `alert` dans la recherche du panneau et choisissez **Alert Graph**.
- **A9-3.4.3** Dans **Choose an alert**, saisissez `M08 APM - <participant>` et cliquez sur votre monitor dans la liste.
- **A9-3.4.4** Conservez la visualisation **Timeseries**.
- **A9-3.4.5** Donnez le titre `Alerte - débit console et seuils` et cliquez sur **Save**.

Widget 13 - état de l'alerte. Question : l'alerte est-elle active ? Action : ouvrir le monitor et son historique.

- **A9-3.4.6** Dans le groupe, ajoutez un widget **Monitor Summary** : la fenêtre **Monitor Summary Editor** liste d'abord tous les monitors de l'organisation.
- **A9-3.4.7** Conservez le type **Monitor** dans **Select summary type**.
- **A9-3.4.8** Dans **Search for monitors to summarize**, saisissez le nom exact de votre monitor :

  ```text
  [TRAINING] M08 APM - <participant>
  ```

- **A9-3.4.9** Vérifiez que l'aperçu indique **Showing 1-1 of 1 monitor** et porte votre identifiant.
- **A9-3.4.10** Conservez l'affichage **Both**, saisissez `Alerte - état de mon monitor` dans **Widget title**, puis cliquez sur **Save**.

**Résultat attendu :** le widget 12 affiche la courbe en hits/s avec une zone colorée au-dessus des seuils ; le widget 13 affiche votre monitor en `OK` ou `Warn`.

**Interprétation :** certains pics de la courbe approchent le seuil d'alerte sans la déclencher : le monitor compare la **moyenne sur 15 minutes** au seuil, alors que la courbe montre les valeurs brutes. L'état `OK` signifie que la règle n'est pas remplie, pas que tout va bien.

## 3.5. Vérifier l'ordre de lecture et enregistrer la période de référence

**Pourquoi :** le dashboard doit se lire de haut en bas sans effort.

- **A9-3.5.1** Vérifiez que la note de lecture est en haut, suivie des groupes `Impact utilisateur`, `Comportement des traitements`, `Logs applicatifs` et `Alerte`.
- **A9-3.5.2** Si un widget se trouve hors de son groupe, faites-le glisser par sa barre de titre jusque dans le groupe.
- **A9-3.5.3** Sélectionnez **Past 1 Day** comme période.

**Résultat attendu :** le dashboard se lit de haut en bas, conformément au plan du dashboard.

**Vérification :** chaque widget porte un titre qui commence par son groupe, par exemple `Impact -`, `Traitements -`, `Logs -` ou `Alerte -`.

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

**Résultat attendu :** l'anomalie est datée et attribuée à une commande précise.

**Interprétation :** le dashboard révèle ici une situation fréquente : les traitements se terminent sans erreur côté APM, alors que leurs logs signalent des erreurs massives sur une seule commande. Sans le widget de classement, on aurait seulement vu « beaucoup d'erreurs ». C'est l'information à transmettre à l'équipe propriétaire, accompagnée du nom de la commande et de l'heure.

## 4.2. Changer la variable `env` puis la période pour vérifier ce que chaque widget affiche

**Pourquoi :** un dashboard doit rester compréhensible quand on change son périmètre, et un widget vide doit pouvoir s'expliquer.

- **A9-4.2.1** Dans le bandeau, passez la variable `env` à `recette`.
- **A9-4.2.2** Observez quels widgets se mettent à jour et lesquels deviennent vides.
- **A9-4.2.3** Revenez à `prod`.
- **A9-4.2.4** Sélectionnez **Past 1 Hour**, observez les widgets qui se vident, puis revenez à **Past 1 Day**.

**Résultat attendu :** les widgets APM se mettent à jour en `recette` ; les widgets RUM deviennent vides, car `peopulse` n'envoie des données qu'en `prod` ; les widgets de logs affichent les logs de `recette` s'il y en a sur la période.

**Interprétation :** un widget vide n'indique pas une panne. Il peut venir de l'environnement, de la période ou de l'absence d'activité. Le widget **Monitor Summary** et l'**Alert Graph** ne changent pas : ils affichent le monitor tel qu'il a été défini, en `env:prod`.

## 4.3. Ouvrir le détail d'un widget sans quitter le dashboard

**Pourquoi :** le dashboard qualifie la situation ; le diagnostic se poursuit dans les logs, les traces ou le monitor, avec les mêmes filtres.

- **A9-4.3.1** Dans `Logs - commandes en erreur`, cliquez sur la commande en tête de liste : un menu s'ouvre.
- **A9-4.3.2** Choisissez **View logs** : un panneau s'ouvre à droite du dashboard avec les logs de cette commande.
- **A9-4.3.3** Vérifiez que la recherche du panneau contient `service:eu-interfaces`, `env:prod`, `status:error` et le nom de la commande, puis lisez deux messages.
- **A9-4.3.4** Repérez l'icône d'ouverture dans un nouvel onglet en haut à droite de la liste : elle ouvre la même recherche dans le Log Explorer.
- **A9-4.3.5** Fermez le panneau avec sa croix.
- **A9-4.3.6** Dans `Traitements - requêtes eu-interfaces / console`, cliquez sur un point de la courbe et observez que **View traces** et **View logs** sont grisés ; ne cliquez pas sur **Add annotation**, puis cliquez à côté du menu pour le fermer.
- **A9-4.3.7** Dans `Alerte - état de mon monitor`, cliquez sur le nom du monitor, vérifiez que sa page s'ouvre, puis revenez au dashboard.

**Résultat attendu :** le détail des logs s'ouvre en un clic depuis le classement des commandes, et le monitor depuis son résumé.

**Interprétation :** les widgets **APM Metrics** ne proposent pas de lien vers les traces : pour les traces, ouvrez la page APM du service. C'est une limite à connaître lorsqu'on conçoit un dashboard : un widget de logs ou de monitor mène directement au détail, un widget de métrique APM non.

## 4.4. Afficher un widget en plein écran puis relire sa requête en mode édition sans la modifier

**Pourquoi :** savoir lire la requête d'un widget permet de vérifier ce qu'il compte réellement, y compris sur un dashboard construit par quelqu'un d'autre.

- **A9-4.4.1** Survolez `Logs - commandes en erreur` et ouvrez-le en plein écran avec l'icône d'agrandissement.
- **A9-4.4.2** Fermez le plein écran.
- **A9-4.4.3** Ouvrez le même widget en édition.
- **A9-4.4.4** Identifiez la source, la recherche, la mesure et le regroupement.
- **A9-4.4.5** Cliquez sur **Cancel** sans rien modifier.

**Résultat attendu :** vous savez dire en une phrase ce que compte le widget : « le nombre de logs en erreur de `eu-interfaces`, dans l'environnement choisi, par commande ».

**Interprétation :** un titre peut mentir, la requête non. Avant de vous fier à un widget hérité, relisez sa requête.

# 5. Relier l'alerte au dashboard

## 5.1. Copier l'adresse du dashboard et l'ajouter au message du monitor

**Pourquoi :** la personne qui reçoit l'alerte doit atteindre la bonne vue en un clic, sans chercher le dashboard.

- **A9-5.1.1** Copiez l'adresse du dashboard depuis la barre d'adresse du navigateur.
- **A9-5.1.2** Ouvrez votre monitor `[TRAINING] M08 APM - <participant>` et cliquez sur **Edit**.
- **A9-5.1.3** Dans le bloc `{{#is_alert}}` du message, remplacez la ligne « Première action » par :

  ```markdown
  Première action : qualifier l'alerte avec le dashboard <adresse-copiée>
  ```

- **A9-5.1.4** Collez l'adresse du dashboard à la place de `<adresse-copiée>`.
- **A9-5.1.5** Vérifiez que **Recipient summary** indique toujours **No recipients** et cliquez sur **Save and Publish**.

**Résultat attendu :** le message du monitor contient le lien vers votre dashboard.

**Interprétation :** c'est le principe d'une alerte actionnable : elle dit quoi faire en premier et donne l'outil pour le faire.

# 6. Qualifier une alerte de bout en bout

## 6.1. Déclencher l'alerte en vidant l'avertissement puis en abaissant le seuil d'alerte

**Pourquoi :** pour vous entraîner dans des conditions réalistes, vous allez provoquer une alerte sur des données réelles, comme à l'atelier 8.

- **A9-6.1.1** Dans votre monitor, cliquez sur **Edit**.
- **A9-6.1.2** Videz d'abord **Warning threshold** : l'avertissement doit toujours rester inférieur à l'alerte.
- **A9-6.1.3** Saisissez ensuite dans **Alert threshold** le seuil de déclenchement volontaire noté dans `reponses-A8.txt`, par exemple `0.1`.
- **A9-6.1.4** Cliquez sur **Save and Publish** et actualisez la page jusqu'au passage en `ALERT`.

**Résultat attendu :** le monitor est en `ALERT`.

## 6.2. Ouvrir l'événement `ALERT` et rejoindre le dashboard par le lien du message

**Pourquoi :** c'est le geste réel d'une personne d'astreinte : l'alerte arrive, elle ouvre la vue de qualification.

- **A9-6.2.1** Dans **Event Timeline**, cliquez sur le dernier événement `ALERT`.
- **A9-6.2.2** Notez son heure dans la rubrique « Alerte reçue » de `reponses-A9.txt`.
- **A9-6.2.3** Dans **Event Details**, cliquez sur le lien du dashboard.
- **A9-6.2.4** Vérifiez que le widget `Alerte - état de mon monitor` affiche `Alert` et que l'Alert Graph montre la courbe dans la zone rouge.

**Résultat attendu :** le dashboard s'ouvre et confirme l'état de l'alerte.

## 6.3. Répondre aux questions impact, chronologie, périmètre et décider de la suite

**Pourquoi :** qualifier une alerte, c'est décider si elle mérite une intervention. Cette décision s'appuie sur des faits lus dans le dashboard.

- **A9-6.3.1** Sélectionnez **Past 4 Hours** pour centrer la lecture sur l'heure de l'alerte.
- **A9-6.3.2** Lisez le groupe `Impact utilisateur` : erreurs, vues et temps de chargement ont-ils changé autour de l'heure de l'alerte ?
- **A9-6.3.3** Lisez le groupe `Comportement des traitements` : le débit a-t-il changé par rapport à la veille ? les traitements échouent-ils ou ralentissent-ils ?
- **A9-6.3.4** Lisez le groupe `Logs applicatifs` : une commande produit-elle des erreurs inhabituelles sur cette période ?
- **A9-6.3.5** Lisez l'Alert Graph : la courbe a-t-elle bougé, ou est-ce le seuil qui a changé ?
- **A9-6.3.6** Choisissez une décision :
  - **clore** : l'alerte ne correspond à aucune dégradation ;
  - **surveiller** : un signal évolue, mais sans impact constaté ;
  - **escalader** : un impact est constaté ou la dégradation s'aggrave.
- **A9-6.3.7** Complétez la partie « Qualification d'alerte » de `reponses-A9.txt` et enregistrez.

Exemple de réponse attendue :

```text
Alerte reçue (monitor, heure) : [TRAINING] M08 APM, heure de l'événement ALERT
Impact utilisateur : non déterminé — aucun lien vérifié entre peopulse et eu-interfaces ;
  erreurs navigateur et temps de chargement stables.
Depuis quand : la courbe de l'Alert Graph est au-dessus du seuil sur toute la période.
Périmètre touché : aucun ; débit proche de la veille, erreurs APM à 0, latence stable,
  pas de nouvelle commande en erreur sur la période.
Décision et raison : clore — le débit est normal, seul le seuil a été abaissé.
  Action : rétablir le seuil calibré.
  À signaler séparément : erreurs de la nuit sur cmad:export:efs (thématique 4).
```

**Résultat attendu :** la qualification repose uniquement sur des valeurs visibles dans le dashboard.

**Interprétation :** la bonne décision ici est de clore et de corriger le seuil : l'activité est normale et seule la règle est fausse. C'est le cas le plus fréquent d'alerte inutile en production. Le dashboard permet en même temps de ne pas perdre l'anomalie réelle repérée dans les logs, qui n'a rien à voir avec cette alerte et doit être transmise par un autre canal.

## 6.4. Rétablir l'alerte puis l'avertissement calibrés et vérifier le retour à `OK` sur le dashboard

**Pourquoi :** une alerte close doit disparaître ; sinon, elle masque la prochaine.

- **A9-6.4.1** Dans votre monitor, cliquez sur **Edit**.
- **A9-6.4.2** Saisissez d'abord le seuil d'alerte calibré noté dans `reponses-A8.txt`, par exemple `0.9`.
- **A9-6.4.3** Saisissez ensuite l'avertissement calibré, par exemple `0.6`.
- **A9-6.4.4** Cliquez sur **Save and Publish**.
- **A9-6.4.5** Revenez au dashboard et actualisez-le jusqu'à ce que le widget `Alerte - état de mon monitor` affiche `OK` ou `Warn`.

**Résultat attendu :** le dashboard affiche de nouveau un état normal.

**Vérification :** **Event Timeline** du monitor montre la séquence `ALERT` puis le retour à la normale.

# 7. Recette

## 7.1. Contrôler le dashboard et le monitor avec la grille de recette

**Pourquoi :** un dashboard partagé doit être exact, lisible et sans donnée sensible.

- **A9-7.1.1** Vérifiez chaque point de la grille ci-dessous dans le dashboard et dans le monitor.
- **A9-7.1.2** Corrigez uniquement vos propres ressources si un point n'est pas conforme.

| Contrôle | Résultat attendu |
|---|---|
| Titre du dashboard | `[TRAINING] M09 - <participant>`, unique |
| Note de lecture | en haut, avec les limites de corrélation |
| Groupes | Impact utilisateur, Comportement des traitements, Logs applicatifs, Alerte |
| Widgets 1 à 3 (RUM) | agrégés, filtrés sur `peopulse` et `$env`, sans utilisateur, session ni URL |
| Widgets 4 à 7 (APM) | `eu-interfaces` / `console`, avec `$env` |
| Widgets 8 à 11 (logs) | `service:eu-interfaces $env`, classement par `@cmd.name`, aucun identifiant de personne |
| Widgets 12 et 13 (alerte) | votre seul monitor, seuils visibles |
| Titres | préfixés par leur groupe, avec la source et la question |
| Message du monitor | lien vers le dashboard, aucun `@`, **No recipients** |
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

## Nettoyage en fin de formation

Uniquement avec l'autorisation du formateur :

1. recherchez le dashboard `[TRAINING] M09 - <participant>` et le monitor `[TRAINING] M08 APM - <participant>` par leur nom exact ;
2. vérifiez le préfixe et votre identifiant ;
3. supprimez ces deux ressources et aucune autre.

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
- cliquez sur **Count of** puis saisissez `loading` dans la recherche de la liste ; signalez la référence `A9-3.1.12` au formateur si la mesure reste absente.

### L'attribut `@cmd.name` n'est pas proposé dans le Top List

- saisissez `cmd.name` en entier dans la liste **by** et choisissez **@cmd.name**, et non « Add group by cmd.name » ;
- signalez la référence `A9-3.3.11` au formateur et regroupez provisoirement par **Host**.

### Le Top List affiche des barres de couleur empilées

- le regroupement **Service** est encore présent à côté de **@cmd.name** ; retirez-le avec sa croix.

### Une fenêtre « Quick Graph » s'est ouverte

- une lettre a été tapée hors d'un champ ; cliquez sur **Cancel**, puis cliquez dans le champ voulu avant de taper.

### Un widget a gardé le titre proposé par Datadog

- survolez le widget, cliquez sur l'icône crayon, corrigez **Give your graph a title** et cliquez sur **Save**.

### L'éditeur d'un widget s'est fermé sans enregistrer

- la touche Échap ferme l'éditeur ; recommencez l'ajout du widget et terminez toujours par **Save**.

### Le lien du message n'ouvre pas le dashboard

- vérifiez que l'adresse collée commence par `https://` ;
- vérifiez que le lien se trouve à l'intérieur du bloc `{{#is_alert}}`.

## Références officielles

- [Datadog — Dashboards](https://docs.datadoghq.com/dashboards/)
- [Datadog — Template Variables](https://docs.datadoghq.com/dashboards/template_variables/)
- [Datadog — Change Widget](https://docs.datadoghq.com/dashboards/widgets/change/)
- [Datadog — Top List Widget](https://docs.datadoghq.com/dashboards/widgets/top_list/)
- [Datadog — List Widget](https://docs.datadoghq.com/dashboards/widgets/list/)
- [Datadog — Alert Graph Widget](https://docs.datadoghq.com/dashboards/widgets/alert_graph/)
- [Datadog — Monitor Summary Widget](https://docs.datadoghq.com/dashboards/widgets/monitor_summary/)
- [Datadog — Group Widget](https://docs.datadoghq.com/dashboards/widgets/group/)
