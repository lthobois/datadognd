# Ateliers — parcours non développeurs

## Public

Participants qui exploitent Datadog sans écrire de code : exploitation, support, produit ou pilotage. Ils lisent les signaux, reçoivent des alertes, construisent des dashboards et décident d'une escalade. Ils n'instrumentent pas d'application et n'exécutent aucun script.

## Principes

- Toutes les manipulations se font dans l'interface Datadog, sans script, sans clé API et sans envoi de données.
- Les ateliers utilisent uniquement les données réelles déjà présentes : APM `eu-interfaces` avec l'opération `console`, RUM `peopulse` en agrégats, dashboards `Monitoring API` et `Monitoring EU-Interfaces`.
- Les écritures sont limitées à des ressources préfixées `[TRAINING]` et portant l'identifiant du participant.
- Chaque participant travaille individuellement ; les ateliers 8 et 9 utilisent un fichier de réponses `.txt` local.

## Parcours

| Atelier | Fichier | Dérivé de | Écriture |
|---:|---|---|---|
| 3 | [Se repérer dans Datadog et lire l'état d'un service](M03_Se_reperer_lire_etat_service.md) | M03, allégé | aucune |
| 9 | [Construire un dashboard et qualifier une alerte](M09_Dashboard_qualification_alerte.md) | M09 et raisonnement du M10 | dashboard `[TRAINING] M09 - <participant>` |

Continuité :

```text
Atelier 3 lire  ->  Atelier 4 filtrer avec les tags (env:prod)
      ->  Atelier 8 monitor APM sur données réelles, seuils notés dans reponses-A8.txt
      ->  Atelier 9 dashboard relié au monitor, qualification d'une alerte réelle
```

Les ateliers 3 et 4 sont indépendants des ateliers 8 et 9. L'atelier 9 nécessite le monitor de l'atelier 8.

## Différences avec le parcours développeurs

| Parcours existant | Parcours non développeurs |
|---|---|
| M04 : création d'une définition de service dans le Catalog | lecture des tags réels et filtrage des explorateurs |
| M05 et M10 : envoi de logs et de métriques par PowerShell ou cURL | supprimés |
| M08 : monitors déclenchés par un log ou une métrique envoyés par script | monitor APM sur données réelles, déclenché par réglage du seuil |
| M09 : onze widgets dont six dépendent des données envoyées par script | treize widgets sur données réelles (RUM, APM, logs par commande, monitor), exploitation d'une anomalie réelle et lien depuis l'alerte |

## Préparation formateur

Droits à vérifier pour chaque participant :

- lecture des dashboards, d'APM, des logs, du RUM et du Software Catalog ;
- enregistrement d'une vue dans le Trace Explorer ;
- création et modification de son propre monitor ;
- création de son propre dashboard.

Repères de la plateforme utilisés par les ateliers :

- `eu-interfaces` envoie des traces dans trois environnements : `prod`, `pre` et `recette` ; les ateliers utilisent `env:prod` ;
- l'opération `console` correspond à des traitements planifiés : activité continue en `prod` (débit moyen d'environ 0,3 hits/s sur 15 minutes dans l'éditeur de monitor), latence p95 de plusieurs minutes, aucune erreur APM ;
- les logs de `eu-interfaces` en `prod` portent l'attribut `@cmd.name` (une quinzaine de commandes en erreur sur une journée, de la forme `mission:export:…` ou `facture:export:…`) ; lors de la préparation, `cmad:export:efs` concentrait près de 90 % des logs en erreur, avec un pic nocturne, alors que l'APM comptait zéro erreur : vérifier le jour même quelle anomalie l'atelier 9 fera apparaître ;
- les logs de `eu-interfaces` portent `service` et `env`, mais l'onglet **Logs** d'une trace affiche **Logs 0** : la liaison traces → logs est absente ;
- l'application RUM `peopulse` est en `env:prod` ; son nombre d'erreurs est très faible tôt le matin, d'où la consigne d'élargir la période ;
- dans le Catalog, `eu-interfaces` affiche `env:dev +5` et un taux d'erreur tous environnements confondus supérieur à 10 %, utilisé comme exemple à l'atelier 4.

Ressources à supprimer en fin de formation, après vérification du préfixe et de l'identifiant : vue `[TRAINING] M04`, monitor `[TRAINING] M08 APM`, dashboard `[TRAINING] M09`.
