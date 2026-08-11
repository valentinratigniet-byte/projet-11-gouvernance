# Conventions de nommage

Règles appliquées sur toute la plateforme (base 07 → pipeline 10 → BI 09).
Objectif : un nom doit se lire sans documentation.

## Schémas (couches medallion)

| Schéma | Rôle | Écrit par |
|---|---|---|
| `public` | base opérationnelle (OLTP) | application source |
| `raw` | copie brute, jamais transformée | `extract_load.py` |
| `staging` | nettoyée, typée, renommée | dbt (`stg_*`) |
| `marts` | modèle en étoile analytique | dbt (`fct_*` / `dim_*`) |

## Tables

- **staging** : préfixe `stg_` + entité au pluriel → `stg_orders`, `stg_customers`.
- **marts** : `fct_` pour les faits, `dim_` pour les dimensions → `fct_sales`, `dim_product`.
- Un modèle staging = **une** source (règle 1:1).

## Colonnes

- **snake_case** partout, en minuscules.
- Clés : suffixe `_key` → `customer_key`, `product_key`, `date_key`.
- Clé de date au format entier `AAAAMMJJ` (`date_key`).
- Booléens : préfixe `is_`/`est_` → `is_active`, `est_weekend`.
- Montants : type décimal, nom explicite → `line_amount`, `unit_price`.
- Dates/horodatages : `_date` (date métier) / `_at` (horodatage technique) → `order_date`, `paid_at`.

## Mesures (Power BI)

- Nom métier lisible, pas technique → `CA`, `Panier moyen`, `Croissance YoY %`.
- Rangées en dossiers d'affichage : `1 Base`, `2 Temps`, `3 Clients`, `4 Ratios`.

## Objets dbt

- Tests nommés automatiquement par dbt (`unique_`, `not_null_`, `relationships_`, `accepted_values_`).
- Sources déclarées dans `_sources.yml`, modèles documentés dans `_staging.yml` / `_marts.yml`.
