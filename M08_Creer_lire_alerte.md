---
title: "Atelier 8 — Créer, déclencher et lire une alerte sans écrire de code"
subtitle: "Document participant autonome — parcours non développeurs"
lang: fr-FR
---

# Créer, déclencher et lire une alerte sans écrire de code

## Situation professionnelle

Vous recevez régulièrement des alertes Datadog sans savoir pourquoi elles se déclenchent ni ce qu'il faut faire. Pour bien les lire, vous allez en créer une vous-même sur un service réel, la déclencher, puis régler son seuil pour qu'elle ne sonne qu'à bon escient.

## Objectif opérationnel

Créer avec le formulaire Datadog un monitor sur le débit de requêtes du traitement `console` du service réel `eu-interfaces`, le faire passer en `Alert` en réglant un seuil bas, lire l'alerte produite, puis calibrer le seuil pour revenir à `OK`.

## Livrable

À la fin de l'atelier, vous disposez :

- d'un monitor `[TRAINING] M08 APM - <participant>` sans aucun destinataire ;
- d'un historique montrant le passage `Alert` → `OK` ;
- d'un message d'alerte lisible par une personne qui ne connaît pas Datadog ;
- d'un fichier `reponses-A8.txt` contenant les seuils retenus et leur justification.

Le monitor sera réutilisé à l'atelier 9 : ne le supprimez pas avant la fin de la formation.

## Environnement

- Chrome connecté à l'organisation Datadog de la formation ;
- accès à APM et à **Monitors** ;
- droit de créer et modifier son propre monitor ;
- un éditeur de texte simple.

Aucun script, aucune clé API et aucun envoi de données ne sont nécessaires : le monitor lit les données réelles déjà présentes.

## Règles de sécurité

Les seules écritures autorisées sont :

1. la création de **votre** monitor préfixé `[TRAINING] M08` ;
2. la modification de son seuil, de son message et de ses tags ;
3. sa suppression en fin de formation, avec l'autorisation du formateur.

Vous ne devez pas :

- conserver un destinataire, une adresse, un canal ou une intégration, ni écrire le caractère `@` dans le message ;
- modifier, désactiver ou mettre en sourdine un monitor existant ;
- cliquer sur **Resolve**, **Assign to Me** ou **Declare Incident** ;
- utiliser un autre service que `eu-interfaces`.

Remplacez `<participant>` par l'identifiant fourni par le formateur, par exemple `loic`.

Si vous ne disposez pas du droit de créer un monitor, suivez l'atelier sur le monitor créé par le formateur et réalisez les thématiques 3 et 4 en lecture.

## Fichier de réponses

Créez dans votre éditeur un fichier nommé `reponses-A8.txt` et copiez-y la trame suivante.

```text
Atelier 8 — Mon alerte
Participant :

Plage observée dans l'aperçu (minimum - maximum, en hits/s) :
Seuil de déclenchement volontaire :
Seuils calibrés retenus (avertissement / alerte) et raison :
```

## Ce qu'est un monitor

Un monitor est une règle évaluée en continu par Datadog :

```text
SI   <mesure>  en moyenne sur  <fenêtre de temps>   dépasse   <seuil>
ALORS l'état passe en Alert et le message est envoyé aux destinataires
```

Par exemple : « **si** le débit de `console` en moyenne sur les 15 dernières minutes **dépasse** 0,9 requête par seconde, **alors** prévenir l'équipe d'exploitation ».

Un monitor a quatre états principaux :

| État | Signification | Exemple avec un avertissement à 0,6 et une alerte à 0,9 |
|---|---|---|
| `OK` | aucun seuil n'est dépassé | débit moyen de 0,3 requête par seconde |
| `Warn` | le seuil d'avertissement est dépassé | débit moyen de 0,7 |
| `Alert` | le seuil d'alerte est dépassé | débit moyen de 1,2 |
| `No Data` | aucune donnée n'arrive | le traitement ne s'exécute plus, ou le filtre ne correspond à rien |

Le service étudié est `eu-interfaces`. Son opération `console` correspond à des traitements planifiés, qui s'exécutent plusieurs fois par minute en production. Leur débit est faible, inférieur à une requête par seconde : les seuils de cet atelier sont donc des nombres décimaux.

# 1. Préparer la question de l'alerte

## 1.1. Observer la régularité de l'activité de `console` en production dans la page du service

**Pourquoi :** un seuil ne se choisit pas au hasard. Il se fixe à partir du comportement normal du service : régulier, en pics ou avec des creux.

- **A8-1.1.1** Ouvrez **APM**, puis la page du service `eu-interfaces`.
- **A8-1.1.2** En haut de la page, vérifiez que le filtre **operation** vaut `console` et sélectionnez `prod` dans le filtre **env**.
- **A8-1.1.3** Sélectionnez la période **Past 1 Hour**.
- **A8-1.1.4** Observez la courbe des requêtes : est-elle régulière, présente-t-elle des pics, s'interrompt-elle ?

**Résultat attendu :** l'activité de `console` est continue, avec quelques pics.

**Interprétation :** une activité continue convient à un monitor de débit : si elle s'arrêtait, le monitor le verrait. Une activité qui s'interrompt la nuit, au contraire, déclencherait des alertes chaque soir avec un seuil bas. En production, on observerait le comportement sur plusieurs jours avant de choisir un seuil.

## 1.2. Formuler la question à laquelle le monitor doit répondre

**Pourquoi :** une alerte utile répond à une question précise. Sans question, on ne sait ni quel seuil choisir ni quoi faire quand elle sonne.

Le monitor répond à la question :

> Le débit des traitements `console` de `eu-interfaces` en production dépasse-t-il le niveau prévu ?

Un débit anormalement élevé peut signaler par exemple des traitements relancés en boucle, qui surchargent les systèmes appelés.

- **A8-1.2.1** Relisez la question et repérez ses quatre éléments : le service, l'opération, l'environnement et la mesure.

**Résultat attendu :** vous savez quoi sélectionner dans l'éditeur : `eu-interfaces`, `console`, `env:prod` et **Requests/s**.

# 2. Créer le monitor avec le formulaire

## 2.1. Ouvrir l'éditeur de monitor APM et sélectionner Requests/s, `eu-interfaces`, `console` et le filtre `env:prod`

**Pourquoi :** le monitor APM lit directement les mesures du service. Le formulaire guide chaque choix : aucune requête n'est à écrire.

- **A8-2.1.1** Ouvrez **Monitoring**, cliquez sur **+ New Monitor** en haut à droite de la liste, puis sur **APM** dans la liste des types ; ignorez le panneau **Bits Chat** qui peut s'ouvrir à droite.
- **A8-2.1.2** Vérifiez que l'onglet **APM Metrics** est sélectionné.
- **A8-2.1.3** Dans la première liste, qui affiche **Error Rate** par défaut, faites défiler et choisissez **Requests/s**.
- **A8-2.1.4** Dans **Service**, saisissez `eu-interfaces` et sélectionnez-le.
- **A8-2.1.5** Dans **Operation**, sélectionnez `console`, signalé comme **PRIMARY**.
- **A8-2.1.6** Laissez **Resource** sur `*`.
- **A8-2.1.7** Dans **Filter by**, saisissez `env:prod` et cliquez sur la suggestion `env:prod`.

**Résultat attendu :** l'aperçu à droite affiche une courbe en requêtes par seconde.

**Vérification :** la courbe n'est pas vide et ses valeurs restent inférieures à 1.

**Interprétation :** dès que le service est choisi, Datadog **pré-remplit le nom et le message du monitor** plus bas dans la page, et ce message contient votre adresse e-mail précédée de `@`. Vous les remplacerez à l'action 2.3.

## 2.2. Régler la moyenne sur 15 minutes et un seuil d'alerte volontairement inférieur à l'activité normale

**Pourquoi :** pour observer une alerte sans attendre un vrai incident, vous allez fixer un seuil que l'activité normale dépasse déjà. Le monitor passera donc en `Alert` sur des données réelles.

- **A8-2.2.1** Dans **Evaluation Details**, conservez **average** dans **Evaluate the**.
- **A8-2.2.2** Ouvrez la liste **Of the query over the**, survolez **Rolling windows**, puis cliquez sur **last 15 minutes** dans le sous-menu.
- **A8-2.2.3** Vérifiez que la liste affiche maintenant **last 15 minutes** ; sinon, recommencez la tâche précédente.
- **A8-2.2.4** Survolez la courbe de l'aperçu et repérez sa valeur la plus basse et sa valeur la plus haute.
- **A8-2.2.5** Notez ces deux valeurs dans la rubrique « Plage observée » de `reponses-A8.txt`.
- **A8-2.2.6** Calculez un seuil égal à la moitié de la valeur la plus basse, arrondi au dixième inférieur, sans descendre sous `0.1`. Par exemple, pour une plage de 0,25 à 0,6, le seuil vaut `0.1`.
- **A8-2.2.7** Dans **Set alert conditions**, conservez **Threshold Alert** et **above**.
- **A8-2.2.8** Saisissez ce seuil dans **Alert threshold**, avec un point décimal, par exemple `0.1`.
- **A8-2.2.9** Laissez **Warning threshold** vide et ne modifiez pas **Advanced options**.
- **A8-2.2.10** Notez le seuil dans la rubrique « Seuil de déclenchement volontaire ».

**Résultat attendu :** une zone rouge apparaît sur l'aperçu au-dessus de la ligne `y > 0.1`.

**Vérification :** toute la courbe se trouve dans la zone rouge.

**Interprétation :** la fenêtre de 15 minutes lisse les variations : le monitor compare la **moyenne** du débit sur 15 minutes au seuil. Avec une fenêtre d'une minute, un pic isolé suffirait à déclencher l'alerte.

## 2.3. Remplacer le nom et le message pré-remplis par un nom identifiable et un message lisible sans destinataire

**Pourquoi :** la personne qui reçoit une alerte la lit souvent sur son téléphone, sans contexte. Le message doit dire ce qui se passe et quoi faire en premier. Le message pré-rempli, lui, enverrait un e-mail à chaque déclenchement.

- **A8-2.3.1** Dans **Recipient summary**, observez la mention **1 Email** : c'est le destinataire ajouté automatiquement.
- **A8-2.3.2** Dans **Monitor name**, effacez le nom proposé et saisissez :

  ```text
  [TRAINING] M08 APM - <participant>
  ```

- **A8-2.3.3** Cliquez dans la zone du message, sélectionnez tout le texte avec `Ctrl+A` et supprimez-le.
- **A8-2.3.4** Saisissez le message suivant :

  ```markdown
  {{#is_alert}}
  Débit inhabituel des traitements console sur eu-interfaces (prod).
  Valeur mesurée : {{value}} requête(s) par seconde, seuil : {{threshold}}.
  Première action : ouvrir la page APM du service et vérifier la latence et les erreurs.
  {{/is_alert}}

  {{#is_recovery}}
  Retour à la normale du débit des traitements console sur eu-interfaces (prod).
  {{/is_recovery}}

  Monitor pédagogique ORSYS sans destinataire. À supprimer en fin de formation.
  ```

- **A8-2.3.5** Vérifiez que **Recipient summary** affiche maintenant **No recipients in monitor message and no matching rules**.

**Résultat attendu :** le nom contient votre identifiant et le message ne contient aucun `@`.

**Interprétation :** `{{value}}` et `{{threshold}}` sont remplacés par Datadog au moment de l'alerte : le lecteur voit « 0.344 requête(s) par seconde, seuil : 0.1 » plutôt qu'une formule. Les blocs `{{#is_alert}}` et `{{#is_recovery}}` affichent un texte différent selon que l'alerte commence ou se termine.

## 2.4. Ajouter les tags de formation dans Metadata puis créer le monitor

**Pourquoi :** les tags permettent de retrouver votre monitor parmi ceux de l'organisation et de le supprimer sans risque en fin de formation.

- **A8-2.4.1** Faites défiler jusqu'à la section **Metadata** et repérez le champ **Tags**, qui contient déjà `service:eu-interfaces` et `env:prod`.
- **A8-2.4.2** Cliquez dans le champ **Tags** après les tags existants et saisissez `training:true`.
- **A8-2.4.3** Cliquez sur la proposition **Add option: training:true** qui apparaît sous le champ.
- **A8-2.4.4** Ajoutez de la même façon, un par un, `training_module:08`, puis `participant:<participant>`, en cliquant chaque fois sur **Add option**.
- **A8-2.4.5** Vérifiez que le champ contient les cinq tags, sans étiquette vide ni tag inconnu ; supprimez avec sa croix tout tag ajouté par erreur.
- **A8-2.4.6** Ne modifiez ni **Team**, ni **Priority**, ni la section **Define permissions and audit notifications**.
- **A8-2.4.7** Contrôlez une dernière fois le service, l'opération, le filtre `env:prod`, la fenêtre **last 15 minutes**, le seuil et l'absence de destinataire.
- **A8-2.4.8** Cliquez sur **Create and Publish** en bas à droite.

**Résultat attendu :** la page de statut de votre monitor s'ouvre.

**Vérification :** en haut de la page, les tags `env:prod`, `training:true` et `participant:<participant>` sont affichés, et le panneau de droite indique **0 sent to recipients**.

**Interprétation :** l'état peut afficher `NO DATA` pendant quelques instants : le monitor n'a pas encore été évalué.

# 3. Observer et lire l'alerte

## 3.1. Observer le passage en `Alert` sur la page du monitor

**Pourquoi :** Datadog évalue la règle à intervalle régulier. L'état change à l'évaluation suivante, pas au moment exact de la création.

- **A8-3.1.1** Actualisez la page du monitor jusqu'à ce que l'état affiché en haut à gauche change.
- **A8-3.1.2** Repérez le graphique **Monitor behavior** et la ligne de seuil `y > 0.1`.
- **A8-3.1.3** Dans le panneau de droite, repérez la requête évaluée et la mention **AVG over the last 15m**.

**Résultat attendu :** le monitor passe en `ALERT`.

**Vérification :** le graphique montre la mesure au-dessus de la ligne de seuil.

**Interprétation :** le monitor ne « voit » pas un incident : il applique une règle. Ici, la règle est remplie parce que le seuil est trop bas, pas parce que le service va mal.

## 3.2. Ouvrir l'événement `ALERT` dans Event Timeline et lire le message produit

**Pourquoi :** c'est ce que reçoit une personne d'astreinte. Il faut savoir y trouver l'heure, la valeur et l'action à mener.

- **A8-3.2.1** Sous le graphique, dans **Event Timeline**, repérez la ligne marquée `ALERT`.
- **A8-3.2.2** Cliquez dessus : le panneau **Event Details** s'ouvre.
- **A8-3.2.3** Lisez le titre du message, qui commence par `[Triggered]`.
- **A8-3.2.4** Vérifiez que le message affiche une valeur et un seuil chiffrés, par exemple `0.344` et `0.1`, et non `{{value}}`.
- **A8-3.2.5** Repérez les tags de l'événement, dont `env:prod` et `participant:<participant>`.

**Résultat attendu :** le message est lisible sans connaître la configuration du monitor.

**Interprétation :** un bon message d'alerte répond à trois questions : quoi, combien et que faire. Comparez avec le message pré-rempli par Datadog, « throughput deviated too much from its usual value » : il ne dit ni la valeur ni l'action attendue.

## 3.3. Rejoindre la page APM de `eu-interfaces` depuis le monitor

**Pourquoi :** une alerte n'est qu'un point de départ. Il faut savoir rejoindre rapidement les données détaillées.

- **A8-3.3.1** En haut de la page du monitor, cliquez sur l'étiquette `eu-interfaces`.
- **A8-3.3.2** Vérifiez que la page ouverte concerne `eu-interfaces`.
- **A8-3.3.3** Revenez à la page du monitor avec le bouton précédent du navigateur.

**Résultat attendu :** vous passez de l'alerte à la page du service en un clic.

**Interprétation :** à l'atelier 9, vous ajouterez dans le message un lien vers un dashboard de qualification construit pour cette alerte.

# 4. Calibrer le seuil

## 4.1. Saisir une alerte à 1,5 fois le maximum observé puis un avertissement égal à ce maximum

**Pourquoi :** un seuil bien réglé ne se déclenche pas sur un fonctionnement normal.

- **A8-4.1.1** En haut à droite de la page du monitor, cliquez sur **Edit**.
- **A8-4.1.2** Observez que l'éditeur affiche une présentation différente, avec **Define the metric** : c'est normal, Datadog ouvre l'éditeur détaillé de la même règle.
- **A8-4.1.3** Calculez le seuil d'alerte : une fois et demie la valeur la plus haute notée dans `reponses-A8.txt`, arrondie au dixième. Par exemple, pour un maximum de 0,6, le seuil d'alerte vaut `0.9`.
- **A8-4.1.4** Cliquez dans **Alert threshold**, sélectionnez la valeur avec `Ctrl+A` et saisissez le nouveau seuil d'alerte.
- **A8-4.1.5** Seulement ensuite, saisissez dans **Warning threshold** la valeur la plus haute observée, par exemple `0.6`.
- **A8-4.1.6** Vérifiez qu'aucun message rouge n'apparaît sous les seuils.
- **A8-4.1.7** Cliquez sur **Save and Publish**.
- **A8-4.1.8** Actualisez la page du monitor jusqu'à la prochaine évaluation.

**Résultat attendu :** le monitor revient à `OK`, ou passe en `Warn` si le débit moyen atteint le niveau le plus haut observé.

**Vérification :** **Event Timeline** affiche un nouvel événement après l'événement `ALERT`.

**Interprétation :** l'avertissement doit toujours rester inférieur à l'alerte. Si vous le saisissez avant d'avoir relevé l'alerte, Datadog affiche **The warning threshold must be less than the alert threshold** et refuse l'enregistrement. Ce réglage à deux niveaux réduit le bruit : l'avertissement signale un débit inhabituel sans réveiller personne, l'alerte ne sonne qu'au-delà, avec une marge.

## 4.2. Analyser le risque de bruit et consigner les seuils retenus

**Pourquoi :** une alerte qui sonne trop souvent finit par être ignorée, y compris le jour où elle est justifiée.

Lisez les questions suivantes et leurs réponses.

**Question :** pourquoi le premier seuil aurait-il été inacceptable en production ?

**Réponse :** il se déclenchait en permanence sur une activité normale. L'équipe aurait vécu une alerte continue et aurait appris à l'ignorer.

**Question :** pourquoi ne pas fixer le seuil très haut, par exemple 100 fois la valeur normale ?

**Réponse :** le monitor ne se déclencherait jamais, même pendant un vrai emballement des traitements. Un seuil trop haut est aussi inutile qu'un seuil trop bas.

**Question :** pourquoi une heure d'observation ne suffit-elle pas en production ?

**Réponse :** l'activité varie selon l'heure, le jour et les périodes de l'année. Par exemple, des traitements de fin de mois peuvent doubler le débit habituel et déclencher à tort un seuil calé sur une matinée ordinaire.

- **A8-4.2.1** Ouvrez `reponses-A8.txt`.
- **A8-4.2.2** Complétez la rubrique « Seuils calibrés retenus » en une phrase.
- **A8-4.2.3** Enregistrez le fichier.

Exemple de réponse attendue :

```text
Seuils calibrés retenus et raison : avertissement à 0.6, alerte à 0.9 ;
0.6 est le maximum observé sur une heure, 0.9 laisse une marge de 50 %.
À confirmer sur plusieurs semaines d'activité avant une mise en production.
```

# 5. Préparer l'atelier 9

## 5.1. Retrouver le monitor dans Manage Monitors et vérifier son nom, ses tags et son état

**Pourquoi :** le dashboard de l'atelier 9 retrouvera votre monitor par son nom. Il doit être unique.

- **A8-5.1.1** Ouvrez **Monitoring** : la liste des monitors s'affiche sur l'onglet **List**.
- **A8-5.1.2** Dans le champ **Filter monitors**, saisissez `[TRAINING] M08 APM - <participant>`.
- **A8-5.1.3** Vérifiez qu'une seule ligne correspond.
- **A8-5.1.4** Vérifiez que l'état est `OK` ou `Warn`.

**Résultat attendu :** votre monitor est unique, étiqueté et calibré.

**Interprétation :** laissez le monitor actif : l'atelier 9 le fera de nouveau passer en `Alert` pour s'entraîner à qualifier une alerte depuis un dashboard.

## Aide au diagnostic

### L'aperçu de l'éditeur est vide

- vérifiez que le filtre `env:prod` a été validé en cliquant sur la suggestion ;
- vérifiez l'opération `console` ;
- retirez le filtre d'environnement pour vérifier que des données existent.

### La fenêtre reste sur « last 5 minutes »

- ouvrez la liste, survolez **Rolling windows** et attendez l'ouverture du sous-menu avant de cliquer sur **last 15 minutes**.

### Recipient summary affiche toujours « 1 Email »

- le message contient encore une mention `@` ; effacez tout le message avec `Ctrl+A`, puis saisissez de nouveau le texte de l'action 2.3.

### Un tag vide ou inconnu apparaît

- dans **Edit**, section **Metadata**, supprimez-le avec sa croix ;
- ajoutez chaque tag en cliquant sur **Add option** plutôt qu'en appuyant sur Entrée ;
- signalez la référence `A8-2.4.5` au formateur si le problème persiste.

### Le monitor reste `OK` malgré le seuil bas

- attendez la prochaine évaluation et actualisez la page ;
- vérifiez que la condition est **above** et non **below** ;
- vérifiez que le seuil est inférieur à la courbe de l'aperçu.

### Le monitor affiche `No Data` durablement

- les traitements `console` ne se sont peut-être pas exécutés sur la fenêtre ;
- vérifiez le filtre `env:prod` et l'opération dans **Edit** ;
- signalez la référence `A8-2.1.7` au formateur.

### Datadog refuse d'enregistrer les nouveaux seuils

- l'avertissement doit être inférieur à l'alerte : saisissez d'abord l'alerte, puis l'avertissement.

## Nettoyage en fin de formation

Conservez le monitor jusqu'à la fin de l'atelier 9. Ensuite, uniquement avec l'autorisation du formateur :

1. recherchez le monitor par son nom exact ;
2. vérifiez le préfixe `[TRAINING] M08` et votre identifiant ;
3. supprimez ce monitor et aucun autre.

## Références officielles

- [Datadog — APM Monitor](https://docs.datadoghq.com/monitors/types/apm/)
- [Datadog — Configure Monitors](https://docs.datadoghq.com/monitors/configuration/)
- [Datadog — Notifications et variables de message](https://docs.datadoghq.com/monitors/notify/variables/)
- [Datadog — Monitor Status](https://docs.datadoghq.com/monitors/status/)
