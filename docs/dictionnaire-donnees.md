# Dictionnaire de données — plateforme e-commerce

Catalogue de référence des 3 couches de l'entrepôt. Chaque champ a un **sens**,
un **propriétaire** et une **règle de qualité** (contrôlée automatiquement, voir
[qualite.md](qualite.md)). Source de vérité technique : le projet dbt (07/10).

Légende owner : **DE** = Data Engineer · **DA** = Data Analyst.

## Couche `raw` (copie brute)

Copie fidèle des tables opérationnelles, chargée par `extract_load.py`. Aucune
transformation. Propriétaire : **DE**. Règle : conformité 1:1 avec la source.

`raw.customer` · `raw.category` · `raw.product` · `raw.orders` · `raw.order_item`
— mêmes colonnes que la source (voir projet 07).

## Couche `staging` (nettoyée)

| Modèle | Colonne | Type | Description | Owner | Qualité |
|---|---|---|---|---|---|
| `stg_customers` | customer_key | int | Clé client | DE | unique, not_null |
| | email | text | Email (mis en minuscules) | DE | — |
| | full_name | text | Nom complet (prénom + nom) | DA | — |
| | country | text | Pays ISO alpha-2 | DA | — |
| `stg_products` | product_key | int | Clé produit | DE | unique, not_null |
| | product_name | text | Libellé | DA | — |
| | category | text | Catégorie (dénormalisée) | DA | — |
| | price | decimal | Prix catalogue | DA | — |
| `stg_orders` | order_id | int | Identifiant commande | DE | unique, not_null |
| | customer_key | int | → dim_customer | DE | not_null, FK |
| | order_date | timestamp | Date de commande | DA | — |
| | status | text | Statut de la commande | DA | — |
| `stg_order_items` | order_id | int | → stg_orders | DE | not_null, FK |
| | product_key | int | → stg_products | DE | not_null, FK |
| | quantity | int | Quantité | DA | not_null |
| | line_amount | decimal | quantité × prix unitaire | DA | — |

## Couche `marts` (étoile, consommée par Power BI)

| Modèle | Colonne | Type | Description | Owner | Qualité |
|---|---|---|---|---|---|
| `fct_sales` | order_id | int | Commande (grain : ligne) | DE | not_null |
| | customer_key | int | → dim_customer | DE | FK |
| | product_key | int | → dim_product | DE | not_null, FK |
| | date_key | int | → dim_date (AAAAMMJJ) | DE | FK |
| | status | text | Statut | DA | not_null, **accepted_values** |
| | quantity | int | Quantité vendue | DA | not_null |
| | unit_price | decimal | Prix à l'achat (figé) | DA | — |
| | line_amount | decimal | Montant de la ligne (base du CA) | DA | not_null |
| `dim_customer` | customer_key | int | Clé primaire | DE | unique, not_null |
| | country | text | Pays — axe géo + **filtre RLS** | DA | — |
| `dim_product` | product_key | int | Clé primaire | DE | unique, not_null |
| | category | text | Catégorie | DA | — |
| `dim_date` | date_key | int | Clé primaire (AAAAMMJJ) | DE | unique, not_null |
| | annee / trimestre / mois_num | int | Niveaux hiérarchie | DA | — |

> **Catalogue interactif** : le lignage et le catalogue complet (types réels,
> dépendances) sont générés par dbt → voir [`dbt-docs/`](../dbt-docs/) et
> [lignage.md](lignage.md).
