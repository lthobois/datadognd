---
title: "Atelier 4 — Filtrer et cadrer une analyse avec les tags"
subtitle: "Document participant autonome — parcours non développeurs"
lang: fr-FR
---

# Filtrer et cadrer une analyse avec les tags

## Situation professionnelle

Votre responsable vous demande : « Est-ce que `eu-interfaces` a eu des erreurs en production aujourd'hui ? ». Pour répondre juste, vous devez regarder le bon service, le bon environnement et la bonne période, sans mélanger les tests et la production.

## Objectif opérationnel

Lire les tags d'un service réel, s'en servir pour filtrer les traces et les logs, vérifier qu'un même tag permet de passer d'un signal à l'autre, puis enregistrer ce périmètre dans une vue réutilisable.

## Livrable

À la fin de l'atelier, vous disposez d'une vue enregistrée `[TRAINING] M04 - <participant>` dans le Trace Explorer, qui restitue en un clic le périmètre service, environnement et opération de `eu-interfaces`.

## Environnement

- Chrome connecté à l'organisation Datadog de la formation ;
- accès en lecture à APM, aux logs et au Software Catalog ;
- droit d'enregistrer une vue dans le Trace Explorer ;

## Règle d'écriture

La seule écriture autorisée est l'enregistrement de **votre** vue `[TRAINING] M04 - <participant>`. Ne modifiez aucun service, aucun tag existant et aucune vue enregistrée par une autre personne.

Remplacez `<participant>` par l'identifiant fourni par le formateur, par exemple `loic-thobois`.

## Ce qu'est un tag

Un tag est une étiquette `clé:valeur` attachée à chaque donnée envoyée à Datadog. Par exemple, `env:prod` indique l'environnement de production et `service:eu-interfaces` le service qui a produit la donnée.

Trois tags jouent un rôle particulier, car Datadog s'en sert pour relier les signaux entre eux :

| Tag | Question à laquelle il répond | Exemple |
|---|---|---|
| `service` | Quelle application a produit la donnée ? | `service:eu-interfaces` |
| `env` | Dans quel environnement ? | `env:prod`, `env:staging` |
| `version` | Avec quelle version déployée ? | `version:2.14.0` |

Lorsque ces trois tags ont exactement les mêmes valeurs dans les traces, les logs et les métriques, vous passez d'un signal à l'autre en un clic. Lorsqu'ils diffèrent, par exemple `env:prod` d'un côté et `env:production` de l'autre, la liaison est rompue.

# 1. Lire les tags d'un service réel

## 1.1. Relever les valeurs `env` et `version` de `eu-interfaces` dans sa page APM

**Pourquoi :** avant de filtrer, il faut connaître les valeurs réellement utilisées par le service. Les deviner conduit à des vues vides.

- **A4-1.1.1** Ouvrez **APM**, puis la page du service `eu-interfaces`.
- **A4-1.1.2** Ouvrez le sélecteur **Env** en haut de la page et observez la liste des environnements proposés.
- **A4-1.1.3** Refermez le sélecteur sans changer la valeur.
- **A4-1.1.4** Ouvrez **Deployments** et observez les valeurs de version affichées.

**Résultat attendu :** le sélecteur propose trois environnements : `prod`, `pre` et `recette`. Une seule version est active.

**Interprétation :** chaque analyse doit préciser l'environnement lu. Par exemple, un pic d'erreurs en `recette` pendant une campagne de tests n'a pas la même gravité qu'en `prod`.

## 1.2. Lire la fiche Catalog de `eu-interfaces` en lecture seule pour repérer l'équipe et les tags déclarés

**Pourquoi :** le Catalog décrit le service : l'équipe qui en est responsable, ses liens utiles et ses tags déclarés. Pendant un incident, c'est là que vous trouvez à qui transmettre l'information.

- **A4-1.2.1** Ouvrez **Developer Portal**, puis l'onglet **Catalog**.
- **A4-1.2.2** Dans la colonne de gauche, sous **Software**, cliquez sur **Services**.
- **A4-1.2.3** Vérifiez que le filtre **Env** en haut de la liste vaut `*`.
- **A4-1.2.4** Repérez la ligne `eu-interfaces` et observez l'étiquette d'environnement affichée à côté de son nom ainsi que sa colonne **ERROR RATE**.
- **A4-1.2.5** Cliquez sur `eu-interfaces` : un panneau s'ouvre à droite, sur l'onglet **Performance**.
- **A4-1.2.6** Ouvrez l'onglet **Ownership** et repérez l'équipe propriétaire (**Team**) et les contacts, s'ils sont renseignés. Ne cliquez pas sur **Add Metadata**.
- **A4-1.2.7** Faites défiler jusqu'à **Setup Guidance** et observez les éléments marqués **DETECTED** ou **NOT DETECTED**, par exemple **Monitors** et **SLOs**.
- **A4-1.2.8** Fermez le panneau avec sa croix, sans cliquer sur **Service Page**, **Edit** ni **Run Workflow**.

**Résultat attendu :** la ligne `eu-interfaces` affiche un environnement suivi d'un compteur, par exemple `env:dev +5`, et un taux d'erreur. L'onglet **Ownership** propose **Add metadata to this entity** : aucune équipe n'est déclarée. **Setup Guidance** indique par exemple **SLOs NOT DETECTED**.

**Interprétation :** avec le filtre **Env** à `*`, le taux d'erreur mélange tous les environnements. Il peut afficher plus de 10 % alors que les traitements `console` en `prod` n'ont aucune erreur. C'est exactement le piège que les tags permettent d'éviter. Une fiche sans propriétaire est, elle, un constat de gouvernance : pendant un incident, elle oblige à chercher le bon interlocuteur, ce qui retarde l'escalade.

# 2. Filtrer un explorateur avec les tags

## 2.1. Filtrer le Trace Explorer sur `service:eu-interfaces` et un environnement avec la facette Env

**Pourquoi :** la barre de recherche et les facettes produisent le même filtre. Les facettes évitent les fautes de frappe, car elles ne proposent que des valeurs qui existent.

- **A4-2.1.1** Ouvrez **APM > Traces** (Trace Explorer).
- **A4-2.1.2** Sélectionnez **Past 1 Hour**.
- **A4-2.1.3** Dans la barre de recherche, saisissez `service:eu-interfaces`, puis validez.
- **A4-2.1.4** Observez le nombre de résultats affiché au-dessus de la liste.
- **A4-2.1.5** Dans la colonne de gauche, ouvrez la facette **Env** et cochez l'environnement relevé à l'action 1.1.
- **A4-2.1.6** Observez que la barre de recherche contient maintenant `env:prod`.
- **A4-2.1.7** Observez le nouveau nombre de résultats.

**Résultat attendu :** la barre contient deux filtres et le nombre de résultats a diminué ou est resté identique.

**Interprétation :** deux filtres séparés par un espace se cumulent, comme un « ET ». `service:eu-interfaces env:prod` signifie « les traces de `eu-interfaces` **et** de production ».

## 2.2. Isoler les erreurs de l'opération `console` avec la facette Status

**Pourquoi :** la question de départ porte sur les erreurs. Il faut réduire le périmètre jusqu'à la population qui répond à la question.

- **A4-2.2.1** Dans la colonne de gauche, ouvrez la facette **Operation Name** ou **Resource**, puis sélectionnez `console` si cette valeur est proposée.
- **A4-2.2.2** Ouvrez la facette **Status** et cochez **Error**.
- **A4-2.2.3** Observez la barre de recherche et le nombre de résultats.
- **A4-2.2.4** Sélectionnez **Past 1 Day** si aucun résultat n'apparaît.

**Résultat attendu :** la liste ne contient plus que les traces en erreur de `console` sur la période, ou aucune trace si aucune erreur n'a eu lieu.

**Interprétation :** « aucune erreur sur `console` en production depuis 24 heures » est une réponse complète et utile, à condition de citer les filtres et la période.

## 2.3. Exclure une valeur avec le signe moins et observer l'effet sur le résultat

**Pourquoi :** exclure est parfois plus simple que sélectionner. Par exemple, pour tout voir sauf un environnement de test bruyant.

- **A4-2.3.1** Retirez le filtre `status:error` de la barre de recherche.
- **A4-2.3.2** Remplacez `env:prod` par `-env:prod`, en conservant le signe moins devant `env`.
- **A4-2.3.3** Observez le nombre de résultats et les environnements restants dans la facette **Env**.
- **A4-2.3.4** Rétablissez `env:prod` sans le signe moins.

**Résultat attendu :** avec le signe moins, la facette **Env** ne propose plus que `pre` et `recette`.

**Interprétation :** `-env:prod` se lit « tout sauf prod ». Par exemple, pour vérifier qu'une correction testée en `recette` et en `pre` ne produit plus d'erreur avant sa mise en production.

## 2.4. Saisir une valeur d'environnement inexistante pour reconnaître une vue vide due au filtre

**Pourquoi :** c'est l'erreur la plus fréquente : un filtre mal orthographié produit une vue vide qui ressemble à une absence d'activité.

- **A4-2.4.1** Remplacez `env:prod` par `env:production`, une valeur qui n'existe pas dans la facette.
- **A4-2.4.2** Observez le résultat.
- **A4-2.4.3** Ouvrez la facette **Env** et comparez avec les valeurs proposées.
- **A4-2.4.4** Rétablissez la valeur exacte.

**Résultat attendu :** la variante inexistante produit zéro résultat.

**Interprétation :** Datadog compare les valeurs caractère par caractère : `prod`, `Prod` et `production` sont trois valeurs différentes. Avant de conclure « aucune donnée », vérifiez la valeur dans la facette.

# 3. Passer d'un signal à l'autre grâce aux tags

## 3.1. Rechercher les logs de `eu-interfaces` avec les mêmes filtres que les traces

**Pourquoi :** si les logs portent les mêmes tags que les traces, la même requête fonctionne dans les deux explorateurs.

- **A4-3.1.1** Ouvrez **Logs > Log Explorer**.
- **A4-3.1.2** Sélectionnez la même période que dans le Trace Explorer.
- **A4-3.1.3** Saisissez `service:eu-interfaces env:prod`.
- **A4-3.1.4** Observez le nombre de logs retournés.
- **A4-3.1.5** Si aucun log n'est retourné, retirez `env:prod` et observez de nouveau.

**Résultat attendu :** des centaines de milliers de logs sont retrouvés sur une journée avec `service:eu-interfaces env:prod`.

**Interprétation :** chacun de ces trois résultats décrit l'état réel de l'étiquetage :

| Résultat | Ce qu'il indique |
|---|---|
| Logs trouvés avec `service` et `env` | les logs et les traces partagent les mêmes tags |
| Logs trouvés seulement sans `env` | les logs ne portent pas l'environnement ou utilisent une autre valeur |
| Aucun log trouvé | les logs ne portent pas ce nom de service, ou ne sont pas collectés dans Datadog |

## 3.2. Ouvrir une trace de `eu-interfaces` et vérifier si l'onglet Logs affiche les logs associés

**Pourquoi :** depuis une trace, Datadog peut afficher les logs écrits pendant cette même requête. C'est le raccourci le plus utile pendant un incident, s'il est en place.

- **A4-3.2.1** Revenez au Trace Explorer filtré sur `service:eu-interfaces env:prod`.
- **A4-3.2.2** Ouvrez une trace.
- **A4-3.2.3** Dans le panneau de détail, ouvrez l'onglet **Logs**.
- **A4-3.2.4** Observez si des logs sont affichés pour cette trace.
- **A4-3.2.5** Fermez la trace.

**Résultat attendu :** l'onglet s'intitule **Logs 0** : aucun log n'est associé à la trace, alors que le service produit des logs en grand nombre (action 3.1).

**Interprétation :** l'absence de logs liés est un constat technique à remonter, pas une erreur de votre part. Elle oblige à chercher les logs à la main, par heure, ce qui ralentit chaque investigation. Pour la corriger, l'équipe technique doit ajouter l'identifiant de la trace dans les logs de l'application.

# 4. Enregistrer et réutiliser un périmètre

## 4.1. Enregistrer la vue `[TRAINING] M04 - <participant>` avec les filtres service, env et opération

**Pourquoi :** une vue enregistrée mémorise la requête, les facettes et la mise en forme. Elle évite de refaire les mêmes filtres à chaque incident.

- **A4-4.1.1** Dans le Trace Explorer, saisissez la requête :

  ```text
  service:eu-interfaces env:prod operation_name:console
  ```

  Si `operation_name:console` ne retourne rien, utilisez le filtre sélectionné à l'action 2.2 au moyen de la facette.

- **A4-4.1.2** Cliquez sur l'icône **⋮** placée à droite des boutons **Spans** et **Traces**, en haut de l'explorateur.
- **A4-4.1.3** Choisissez **Save view as** : un formulaire **New view** s'ouvre à gauche.
- **A4-4.1.4** Dans **Name**, saisissez le nom :

  ```text
  [TRAINING] M04 - <participant>
  ```

- **A4-4.1.5** Ne choisissez aucune équipe dans **Teams** et cliquez sur **Save**.

**Résultat attendu :** la vue apparaît en tête de la liste des vues enregistrées, sous **My view**.

**Vérification :** son nom contient votre identifiant.

## 4.2. Rouvrir la vue enregistrée depuis un explorateur vide et vérifier les filtres restaurés

**Pourquoi :** une vue n'a de valeur que si elle restitue exactement le périmètre prévu.

- **A4-4.2.1** Effacez la barre de recherche du Trace Explorer.
- **A4-4.2.2** Cliquez sur **Views**, en haut à gauche, et cliquez sur `[TRAINING] M04 - <participant>` dans la liste. Ne cliquez pas sur **Update your default view** sous **My view**.
- **A4-4.2.3** Vérifiez que les trois filtres sont restaurés.

**Résultat attendu :** la requête complète réapparaît.

**Interprétation :** la vue mémorise les filtres, pas la période absolue. Vérifiez toujours la période après avoir ouvert une vue.

## Questions de synthèse

### Pourquoi une faute dans la valeur d'un tag est-elle dangereuse ?

**Réponse :** parce qu'elle produit une vue vide, sans message d'erreur. Le lecteur peut croire qu'il ne se passe rien alors que le filtre exclut toutes les données.

### Que faire si un service n'a pas de tag `env` ?

**Réponse :** le signaler à l'équipe technique, car toutes les données se mélangent. En attendant, filtrer sur un autre attribut fiable, par exemple le nom d'hôte, et le préciser dans chaque conclusion.

### Pourquoi la liaison traces → logs compte-t-elle pour un non-développeur ?

**Réponse :** c'est elle qui permet de passer d'une requête lente au message d'erreur correspondant en un clic. Sans elle, l'investigation repose sur une recherche manuelle par heure, longue et peu fiable.

## Nettoyage en fin de formation

Uniquement avec l'autorisation du formateur :

1. ouvrez le menu des vues du Trace Explorer ;
2. vérifiez que le nom commence par `[TRAINING] M04` et contient votre identifiant ;
3. supprimez cette vue et aucune autre.

## Aide au diagnostic

### La facette Env n'apparaît pas

- utilisez le champ de recherche des facettes en haut de la colonne de gauche ;
- saisissez `env:` directement dans la barre de recherche : Datadog propose les valeurs existantes.

### L'option Save view as n'est pas disponible

- vos droits ne permettent peut-être pas d'enregistrer une vue ;
- signalez la référence `A4-4.1.3` au formateur et notez la requête pour la saisir de nouveau lors d'un prochain incident.

## Références officielles

- [Datadog — Getting Started with Tags](https://docs.datadoghq.com/getting_started/tagging/)
- [Datadog — Unified Service Tagging](https://docs.datadoghq.com/getting_started/tagging/unified_service_tagging/)
- [Datadog — Trace Explorer Search Syntax](https://docs.datadoghq.com/tracing/trace_explorer/query_syntax/)
- [Datadog — Saved Views](https://docs.datadoghq.com/logs/explorer/saved_views/)
- [Datadog — Correlate Logs and Traces](https://docs.datadoghq.com/tracing/other_telemetry/connect_logs_and_traces/)
