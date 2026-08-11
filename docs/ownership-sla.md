# Propriété, accès & SLA

Qui répond de quoi, qui accède à quoi, et à quelle fraîcheur la donnée est promise.

## Propriété (data ownership)

| Domaine | Périmètre | Owner | Responsabilité |
|---|---|---|---|
| Ingestion & entrepôt | `raw`, pipeline ELT, orchestration | **Data Engineer** | disponibilité, fraîcheur, schéma |
| Modèle analytique | `staging`, `marts`, tests | **Data Engineer** | exactitude des transformations |
| Restitution & mesures | dashboard Power BI, mesures DAX | **Data Analyst** | pertinence métier, définitions |
| Définitions métier | dictionnaire, KPI | **Data Analyst** | sens des indicateurs |

## Politique d'accès

| Rôle | Accès |
|---|---|
| Data Engineer | lecture/écriture entrepôt (raw → marts) |
| Data Analyst | lecture `marts` + Power BI |
| Responsable pays | dashboard **restreint à son pays** (RLS `Responsable_FR`, voir projet 09) |
| Direction | dashboard agrégé (lecture) |

- **Secrets** : aucun identifiant en clair dans le code ; connexions via variables
  d'environnement (`DATABASE_URL`, `DBT_PROFILES_DIR`). La base de démo utilise des
  identifiants Docker locaux, non sensibles.
- **Données personnelles** : `email`, `full_name` (dim_customer) → accès restreint,
  jamais exposés dans les visuels publics du dashboard.

## SLA de fraîcheur

| Donnée | Fréquence de rafraîchissement | Seuil d'alerte |
|---|---|---|
| `raw` (ingestion) | quotidienne (planifiée) | > 26 h sans chargement → alerte |
| `marts` (dbt) | à chaque exécution du pipeline | échec dbt → alerte |
| Dashboard Power BI | après chaque run marts | données > 24 h → à signaler |

- Le pipeline échoue **proprement** (code de sortie non nul, logs Prefect) → toute
  rupture de SLA est détectable par l'ordonnanceur.
- Automatisation possible côté dbt : `dbt source freshness` (seuils `warn_after` /
  `error_after`) une fois un horodatage de chargement ajouté à la couche `raw`.
