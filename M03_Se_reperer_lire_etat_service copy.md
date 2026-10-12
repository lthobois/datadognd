---
title: "Atelier 3 — Se repérer dans Datadog et lire l'état d'un service"
subtitle: "Document participant autonome — parcours non développeurs"
lang: fr-FR
---

# Se repérer dans Datadog et lire l'état d'un service

## Situation professionnelle

Un utilisateur signale que « l'application est lente ». Vous n'avez pas écrit le code et vous ne le modifierez pas. Votre rôle est de savoir où regarder, de lire ce que Datadog montre et de distinguer ce que vous constatez de ce que vous supposez.

## Objectif opérationnel

Parcourir les vues d'exploitation de Datadog en partant d'un dashboard, rejoindre le service réel `eu-interfaces`, lire son état, puis consulter les logs, l'expérience utilisateur et les monitors sans rien configurer.

## Livrable

À la fin de l'atelier, vous disposez :

- d'un parcours de lecture « dashboard → service → détail » que vous savez refaire seul ;
- d'une conclusion qui sépare fait et hypothèse, et dont vous avez vérifié l'hypothèse dans Datadog.

## Environnement

- Chrome connecté à l'organisation Datadog de la formation ;
- accès en lecture aux dashboards, à APM, aux logs, au RUM et aux monitors.

Cet atelier ne crée, ne modifie et ne supprime aucune ressource Datadog.

# 1. Se repérer dans l'interface

## 1.1. Repérer le menu, la recherche globale et les grands domaines sans rien configurer

**Pourquoi :** le menu regroupe des domaines d'usage ; il ne décrit pas une procédure. Savoir ce que contient chaque domaine évite de cliquer au hasard pendant un incident.

- **A3-1.1.1** Observez le menu de gauche sans ouvrir aucune commande de configuration.
- **A3-1.1.2** Repérez **Search Datadog** en haut du menu.
- **A3-1.1.3** Repérez les domaines utilisés dans cet atelier :
  - **Dashboards** : les tableaux de bord partagés ;
  - **APM** : le comportement des services applicatifs ;
  - **Logs** : les messages produits par les applications ;
  - **Digital Experience** : l'expérience des utilisateurs dans le navigateur (RUM) ;
  - **Monitors** ou **Monitoring** : les alertes.
- **A3-1.1.4** Repérez **Help** en bas du menu.

**Résultat attendu :** vous situez les cinq domaines dans le menu. Un domaine visible dans le menu ne prouve pas qu'il est alimenté en données. Par exemple, **Synthetic Monitoring** peut apparaître alors qu'aucun test n'a été créé.

## 1.2. Retrouver le dashboard `Monitoring API` avec le champ Search Datadog

**Pourquoi :** la recherche globale est le chemin le plus court vers un objet dont vous connaissez le nom.

- **A3-1.2.1** Cliquez sur le champ **Search Datadog**, en haut au centre de la page, ou appuyez sur `Ctrl+K` sous Windows et Linux, `Cmd+K` sous macOS.
- **A3-1.2.2** Cliquez dans le champ **Search for products and entities…** qui s'ouvre et saisissez `Monitoring API`.
- **A3-1.2.3** Vérifiez que la première ligne indique `Monitoring API` avec le type **Dashboard**, puis appuyez sur Entrée.
- **A3-1.2.4** S'il n'apparaît pas, fermez la recherche, ouvrez **Dashboards > Dashboard List** et saisissez `Monitoring API` dans le champ de recherche de la liste.

**Résultat attendu :** le dashboard `Monitoring API` est ouvert. Son titre est affiché en haut de la page. La recherche globale retrouve un objet connu par son nom. Pour retrouver des événements selon des critères, par exemple « toutes les erreurs de la dernière heure », il faut un explorateur.

**Attention :** une lettre tapée alors qu'aucun champ n'est sélectionné déclenche un raccourci de Datadog. Par exemple, `G` ouvre la fenêtre **Quick Graph** ; fermez-la avec **Cancel** et cliquez dans le champ avant de taper.

# 2. Lire un dashboard existant

## 2.1. Lire la période, le fuseau et les variables de `Monitoring API` avant les courbes

**Pourquoi :** la période et les variables définissent les données affichées. Une même courbe se lit différemment sur 15 minutes et sur 7 jours.

- **A3-2.1.1** Repérez le sélecteur de période en haut à droite et observez la valeur active.
- **A3-2.1.2** Repérez le fuseau horaire affiché à côté de la période ou dans le sélecteur.
- **A3-2.1.3** Repérez la zone des variables, souvent intitulée **Filter by**, et observez le nom et la valeur de chaque variable.
- **A3-2.1.4** Ne modifiez aucune variable pour l'instant.

**Résultat attendu :** vous savez sur quelle période et sur quel périmètre portent les widgets. Une variable à `*` signifie « toutes les valeurs ». Par exemple, une variable `version` à `*` mélange toutes les versions déployées du service.

## 2.2. Associer une question à trois widgets de `Monitoring API` : barres, répartition et classement

**Pourquoi :** chaque widget devrait répondre à une question. Un widget dont on ne sait pas dire la question est difficile à exploiter pendant un incident.

- **A3-2.2.1** Parcourez le dashboard de haut en bas sans entrer en mode édition.
- **A3-2.2.2** Repérez le widget **Appels /api/token**, une série de barres dans le temps, et survolez une barre : l'infobulle affiche l'heure et la valeur.
- **A3-2.2.3** Repérez le camembert **Appels par status http** et observez le chiffre au centre et la part de chaque code de réponse.
- **A3-2.2.4** Repérez le widget **Appels par resource**, où chaque rectangle est d'autant plus grand que la ressource est appelée.
- **A3-2.2.5** Pour chacun des trois widgets, déterminez la question à laquelle il répond, puis comparez avec la colonne **Exemple** du tableau ci-dessous.

**Résultat attendu :** la question que vous avez déterminée pour chaque widget correspond à celle du tableau, par exemple « quelle part des appels se termine par un code `200` ? » pour le camembert des codes de réponse. Le type de widget suit la question :

| Type de widget | Question typique | Exemple dans `Monitoring API` |
|---|---|---|
| Barres ou courbe dans le temps (**Timeseries**) | Quand la valeur change-t-elle ? | « À quelle heure les appels à `/api/token` augmentent-ils ? » |
| Camembert (**Pie Chart**) | Quelle part représente chaque catégorie ? | « Quelle part des appels se termine par un code `200` ? » |
| Rectangles proportionnels (**Treemap**) | Qui contribue le plus ? | « Quelle ressource est la plus appelée ? » |
| Chiffre (**Query Value**) | Combien maintenant ? | « Combien d'erreurs sur la période ? » |

Un camembert donne une répartition, mais ne dit pas quand elle a changé : pour dater un incident, il faut une courbe ou des barres dans le temps.

## 2.3. Élargir la période d'un widget vide pour distinguer « vide » et « absent »

**Pourquoi :** une vue vide ne prouve pas qu'il ne se passe rien. La période, un filtre ou les droits peuvent masquer les données.

- **A3-2.3.1** Sélectionnez **Past 15 Minutes** dans le sélecteur de période.
- **A3-2.3.2** Repérez un widget vide ou peu rempli.
- **A3-2.3.3** Sélectionnez **Past 1 Day**.
- **A3-2.3.4** Observez si le widget se remplit.
- **A3-2.3.5** Revenez à la période initiale.

**Résultat attendu :** certains widgets se remplissent sur une période plus longue. Un service peu sollicité peut ne rien montrer sur 15 minutes et beaucoup sur une journée. Avant de conclure « aucune donnée », élargissez la période, puis réduisez à nouveau le périmètre une fois les données retrouvées.

# 3. Lire l'état d'un service dans APM

## 3.1. Ouvrir APM Home et retrouver `eu-interfaces` dans la liste des services avec son environnement et sa période

**Pourquoi :** APM présente les services applicatifs et trois indicateurs essentiels : le volume de requêtes, le taux d'erreur et la latence.

- **A3-3.1.1** Dans le menu de gauche, cliquez sur **APM**.
- **A3-3.1.2** Repérez le filtre **Env** et observez sa valeur sans la modifier.
- **A3-3.1.3** Repérez le sélecteur de période et sélectionnez **Past 1 Hour**.
- **A3-3.1.4** Dans la liste des services, recherchez `eu-interfaces`.
- **A3-3.1.5** Observez les colonnes **Requests**, **Error Rate** et **p95 Latency** lorsqu'elles sont affichées.

**Résultat attendu :** la ligne `eu-interfaces` affiche des valeurs pour l'environnement et la période sélectionnés. Si la ligne affiche des tirets, élargissez la période à **Past 1 Day**. La **p95 Latency** est la durée en dessous de laquelle se terminent 95 % des requêtes. Par exemple, une p95 de 800 ms signifie que 5 % des requêtes durent plus de 800 ms. Cette mesure décrit mieux la gêne ressentie qu'une moyenne, qui masque les requêtes les plus lentes.

**Question :** un taux d'erreur nul prouve-t-il que le service fonctionne bien ?

**Réponse :** non. Un service peut répondre sans erreur tout en étant trop lent. Il faut lire ensemble le volume, les erreurs et la latence.

## 3.2. Lire les courbes requêtes, erreurs et latence de `eu-interfaces` dans la page du service

**Pourquoi :** la page du service répond à deux premières questions d'incident : « le service se dégrade-t-il ? » et « depuis quand ? ».

- **A3-3.2.1** Cliquez sur `eu-interfaces`.
- **A3-3.2.2** En haut de la page, repérez les filtres **operation**, **env** et **version**, puis sélectionnez `prod` dans **env**.
- **A3-3.2.3** Dans **Service Summary**, repérez les encadrés **Incidents**, **Service Health Monitors** et **SLOs**, puis les courbes de requêtes, d'erreurs et de latence.
- **A3-3.2.4** Repérez un éventuel pic ou une rupture sur l'une des courbes et survolez-le pour lire l'heure.
- **A3-3.2.5** Si aucune rupture n'est visible, observez que la courbe est stable.

**Résultat attendu :** vous savez dire si le service est stable sur la période et, sinon, à quelle heure le comportement change. Une rupture dans la courbe donne une heure de départ ; elle ne donne pas la cause. Les encadrés vides, par exemple « There are no service health monitors configured », sont aussi des constats : aucune alerte ni aucun objectif de fiabilité n'est défini pour ce service.

## 3.3. Lire la ligne `console` dans la vue Resources : requêtes, temps total et latence p95

**Pourquoi :** un service rend une ou plusieurs opérations. Lire leurs requêtes, leur temps total et leur latence permet de savoir où se concentre l'activité avant d'aller plus loin.

- **A3-3.3.1** Dans la page du service, ouvrez **Resources** dans la colonne de gauche.
- **A3-3.3.2** Observez les trois graphiques **Requests**, **p95 Latency** et **Errors**.
- **A3-3.3.3** Dans le tableau, repérez la ligne `console` et lisez les colonnes **REQUESTS**, **TOTAL TIME** et **P95 LATENCY**.
- **A3-3.3.4** En haut de la page, ouvrez le filtre **operation** et observez les opérations proposées pour l'environnement `prod`, puis refermez-le sans rien changer.

**Résultat attendu :** en `prod`, une seule opération est proposée, `console`, avec quelques centaines de requêtes par heure, un temps total de plusieurs heures et une p95 de plusieurs minutes. **TOTAL TIME** additionne la durée de toutes les requêtes : quelques centaines de traitements de plusieurs minutes représentent des heures de travail cumulé par heure, donc plusieurs traitements s'exécutent en même temps. Pour un service web, la même lecture servirait à comparer les opérations : une opération lente mais rarement appelée pèse moins qu'une opération moyennement lente appelée en permanence. Lisez toujours la latence avec le volume.

## 3.4. Ouvrir une trace de `eu-interfaces` et lire sa cascade sans interpréter le code

**Pourquoi :** une trace montre le parcours d'une seule requête et le temps passé dans chacune de ses étapes. Vous n'avez pas besoin de comprendre le code pour voir où le temps est consommé.

- **A3-3.4.1** Dans la page du service, ouvrez **Traces**.
- **A3-3.4.2** Vérifiez que la barre de recherche contient `service:eu-interfaces` et l'environnement sélectionné.
- **A3-3.4.3** Cliquez sur l'en-tête **DURATION** pour afficher les traces les plus longues en premier.
- **A3-3.4.4** Ouvrez la première trace.
- **A3-3.4.5** En haut du panneau de la trace, sélectionnez la vue **Waterfall**.
- **A3-3.4.6** Repérez la barre principale `eu-interfaces` et les barres plus courtes placées dessous, par exemple `rabbitmq` ou `amqp`.
- **A3-3.4.7** Survolez les barres pour lire leur nom et leur durée.
- **A3-3.4.8** Fermez la trace.

**Résultat attendu :** vous repérez où se concentre la durée de cette requête. La cascade se lit comme un diagramme de Gantt : chaque barre est une étape, sa longueur est sa durée. Pour les traitements `console`, une trace dure plusieurs minutes alors que les appels vers `rabbitmq` ou `amqp` ne durent que quelques millisecondes : le temps est passé dans le traitement lui-même, sans être détaillé. Si au contraire une étape nommée par exemple `pdo` occupait 90 % de la barre, la requête aurait surtout attendu la base de données. Dans les deux cas, c'est une piste à transmettre à l'équipe technique, pas encore une cause démontrée : une seule trace peut être exceptionnelle.

## 3.5. Repérer les versions déployées de `eu-interfaces` dans la vue Deployments

**Pourquoi :** un changement de version est souvent la première question posée pendant un incident.

- **A3-3.5.1** Revenez à la page du service et ouvrez **Deployments**.
- **A3-3.5.2** Dans le tableau, observez la colonne **VERSION**, la date d'apparition (**FIRST SEEN**) et le statut **ACTIVE**.
- **A3-3.5.3** Si aucune version n'est affichée, observez-le : le service ne transmet peut-être pas son numéro de version.

**Résultat attendu :** vous savez combien de versions sont actives et depuis quand la plus récente est apparue. Une dégradation qui commence juste après un déploiement rend ce déploiement suspect, pas coupable. Par exemple, une hausse de trafic au même moment peut produire le même effet.

# 4. Rejoindre les autres signaux

## 4.1. Rechercher les logs de `eu-interfaces` en production sur une journée dans le Log Explorer

**Pourquoi :** les logs sont les messages écrits par l'application. Ils précisent ce que les courbes résument, par exemple le texte exact d'une erreur.

- **A3-4.1.1** Ouvrez **Logs > Log Explorer**.
- **A3-4.1.2** Dans la barre de recherche, saisissez `service:eu-interfaces env:prod` et validez.
- **A3-4.1.3** Sélectionnez **Past 1 Day** dans le sélecteur de période.
- **A3-4.1.4** Observez le nombre de logs affiché au-dessus de la liste.
- **A3-4.1.5** Dans la colonne de gauche, repérez la facette **Status** et observez la répartition entre `Info`, `Warn`, `Error` et les autres statuts.
- **A3-4.1.6** Ouvrez un log en erreur et lisez son message, sans le recopier.

**Résultat attendu :** plusieurs centaines de milliers de logs sont visibles sur une journée, dont une part importante en `Error`. Un grand nombre de logs `Error` ne signifie pas forcément un incident : une application peut écrire en `Error` des situations attendues, par exemple « aucun fichier à exporter ». C'est ce qui rend la lecture des logs difficile sans connaître l'application, et c'est pourquoi on part d'une vue agrégée avant de lire un log isolé.

## 4.2. Ouvrir le résumé RUM de `peopulse` avec Search Datadog et lire les vues et le taux d'erreur sans ouvrir de session

**Pourquoi :** le RUM mesure ce que vivent les utilisateurs dans leur navigateur. Il répond à la question « qui subit le problème ? ».

- **A3-4.2.1** Cliquez sur **Search Datadog**, cliquez dans le champ de recherche et saisissez `Real User Monitoring`.
- **A3-4.2.2** Appuyez sur Entrée sur la ligne **Real User Monitoring — Digital Experience**.
- **A3-4.2.3** En haut de la page, vérifiez que le sélecteur d'application affiche `peopulse` ; sinon, ouvrez-le et choisissez `peopulse`.
- **A3-4.2.4** Vérifiez que l'onglet **Summary** est sélectionné.
- **A3-4.2.5** Observez le nombre total de vues (**total views**), le temps de chargement (**Loading Time**) et le taux d'erreur (**Error Rate**), ainsi que leur évolution par rapport à la période précédente.
- **A3-4.2.6** N'ouvrez aucune session individuelle ni aucun Session Replay.

**Résultat attendu :** sur la période par défaut d'une semaine, vous lisez plus de cent mille vues, un temps de chargement d'environ une seconde et un taux d'erreur d'environ 1 %. Un clic sur **Digital Experience** dans le menu ouvre une autre page, **Journey Monitoring** : la recherche globale est ici plus directe. Les sessions individuelles peuvent contenir des données personnelles ; les chiffres agrégés suffisent pour mesurer l'impact sur les utilisateurs. `peopulse` et `eu-interfaces` sont deux applications distinctes : leurs chiffres ne se lisent pas ensemble.

## 4.3. Lire la liste des monitors, filtrer par état et ouvrir un monitor existant

**Pourquoi :** un monitor transforme une donnée en alerte. Savoir lire son état et sa règle permet de comprendre pourquoi une alerte a été reçue.

- **A3-4.3.1** Ouvrez **Monitoring** dans le menu de gauche : la liste des monitors s'affiche sur l'onglet **List**.
- **A3-4.3.2** Dans la colonne de gauche, section **Status**, repérez les états **Alert**, **Warn**, **No Data** et **OK** et le nombre de monitors associé à chacun.
- **A3-4.3.3** Cliquez sur l'état `Alert` pour filtrer la liste.
- **A3-4.3.4** Ouvrez un monitor de la liste. S'il n'y en a aucun, retirez le filtre et ouvrez un monitor `OK`.
- **A3-4.3.5** Repérez sa requête, son seuil et l'historique de ses états.
- **A3-4.3.6** Revenez à la liste sans rien modifier.

**Résultat attendu :** vous savez trouver les monitors en alerte et lire la règle qui les déclenche. `No Data` signifie que le monitor ne reçoit plus de données. Cela peut venir d'un service arrêté, mais aussi d'une collecte interrompue ou d'un filtre mal écrit. Ce n'est pas la même chose qu'une alerte.

# 5. Formuler une conclusion prudente

## 5.1. Classer des affirmations sur `eu-interfaces` en faits, hypothèses ou conclusions prématurées

**Pourquoi :** pendant un incident, une conclusion trop rapide oriente toute l'équipe sur une fausse piste.

Pour chaque affirmation, décidez de son classement en masquant les deux dernières colonnes, puis vérifiez votre choix :

| Affirmation | Classement | Raison |
|---|---|---|
| « La p95 de `eu-interfaces` vaut la valeur affichée sur la dernière heure. » | Fait | valeur observée, avec son périmètre et sa période |
| « La base de données ralentit peut-être les requêtes. » | Hypothèse | « peut-être » annonce une piste à vérifier |
| « Le dernier déploiement est la cause. » | Conclusion prématurée | la proximité dans le temps ne prouve pas la cause |
| « Le service va bien car le taux d'erreur est nul. » | Conclusion prématurée | la latence et l'impact utilisateur restent inconnus |
| « Le monitor est en `No Data`, donc le service est arrêté. » | Conclusion prématurée | la collecte et le filtre doivent d'abord être vérifiés |

- **A3-5.1.1** Classez chacune des cinq affirmations, puis comparez avec les colonnes **Classement** et **Raison**.
- **A3-5.1.2** Pour chaque écart, relisez la raison donnée et identifiez le mot de l'affirmation qui la justifie, par exemple « peut-être » ou « donc ».

**Résultat attendu :** vos classements correspondent au tableau ; un fait contient toujours une valeur, un périmètre et une période.

## 5.2. Construire une conclusion fait / hypothèse / prochaine vue et vérifier l'hypothèse dans le Trace Explorer

**Pourquoi :** une conclusion courte et structurée est ce qu'une équipe technique attend d'un premier niveau d'analyse. Une hypothèse ne vaut que si l'on sait quelle vue la confirme ou l'infirme.

Exemple de conclusion de référence :

```text
Fait constaté : sur la dernière heure, en prod, les traitements console de eu-interfaces
ont une p95 d'environ 8 minutes et aucune erreur APM.
Hypothèse à vérifier : le temps est passé dans le traitement lui-même, pas dans les appels externes.
Prochaine vue à ouvrir : Trace Explorer filtré sur console, plusieurs traces longues à comparer.
```

- **A3-5.2.1** Ouvrez la page APM de `eu-interfaces` en `prod` sur **Past 1 Hour** et vérifiez que la p95 et le nombre d'erreurs correspondent au fait de référence ; sinon, retenez les valeurs affichées.
- **A3-5.2.2** Ouvrez **Traces**, vérifiez le filtre `service:eu-interfaces`, puis triez par **DURATION** pour afficher les traces les plus longues.
- **A3-5.2.3** Ouvrez successivement les trois premières traces en vue **Waterfall**.
- **A3-5.2.4** Pour chacune, comparez la longueur de la barre principale `eu-interfaces` avec celle des appels placés dessous.

**Résultat attendu :** dans les trois traces, la barre principale occupe presque toute la durée et les appels externes ne durent que quelques millisecondes. L'hypothèse de référence est confirmée sur trois traces et non sur une seule. L'hypothèse devient un fait limité à l'échantillon observé : « sur trois traces longues, le temps est passé dans le traitement lui-même ». La suite de l'analyse, savoir ce que fait le traitement pendant ces minutes, demande une instrumentation plus fine que seule l'équipe technique peut ajouter.

## Synthèse

Le parcours de lecture de référence est le suivant :

```text
Dashboard  ->  période et périmètre  ->  service APM  ->  opération lente
           ->  trace  ->  logs  ->  impact utilisateur (RUM)  ->  monitors
```

Commencez toujours par une vue agrégée : elle confirme qu'un problème existe, depuis quand et sur quel périmètre. Ouvrir directement une trace ou un log isolé conduit souvent à analyser un cas non représentatif.

## Aide au diagnostic

### La liste APM ne contient pas `eu-interfaces`

- vérifiez le filtre **Env** ;
- élargissez la période à **Past 1 Day** ;
- utilisez le champ de recherche de la liste plutôt que le défilement.

### La page RUM ne propose pas `peopulse`

- vérifiez le sélecteur d'application en haut de la page ;
- élargissez la période ;
- signalez la référence `A3-4.2.3` au formateur : vos droits peuvent limiter l'accès au RUM.

## Références officielles

- [Datadog — Dashboards](https://docs.datadoghq.com/dashboards/)
- [Datadog — Service Page](https://docs.datadoghq.com/tracing/services/service_page/)
- [Datadog — Trace View](https://docs.datadoghq.com/tracing/trace_explorer/trace_view/)
- [Datadog — RUM Explorer](https://docs.datadoghq.com/real_user_monitoring/explorer/)
- [Datadog — Manage Monitors](https://docs.datadoghq.com/monitors/manage/)
