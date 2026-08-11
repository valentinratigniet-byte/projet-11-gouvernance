# Règles & suivi de la qualité des données

La qualité n'est pas un audit ponctuel : elle est **testée automatiquement** à
chaque exécution du pipeline (dbt test). Un test rouge = le pipeline refuse de
livrer des données incohérentes.

## Catégories de règles

| Règle | Question | Implémentation dbt |
|---|---|---|
| **Unicité** | Une clé n'apparaît-elle qu'une fois ? | `unique` sur les clés primaires |
| **Complétude** | Un champ obligatoire est-il toujours rempli ? | `not_null` |
| **Intégrité référentielle** | Chaque clé étrangère pointe-t-elle vers une vraie ligne ? | `relationships` |
| **Validité de domaine** | La valeur est-elle dans l'ensemble autorisé ? | `accepted_values` (ex. `status`) |

## Suivi qualité (dernier run)

Exécuté via `dbt test` sur le périmètre 07/10 :

```
Done. PASS=28  WARN=0  ERROR=0  SKIP=0  TOTAL=28
```

| Contrôle | Nb tests | Résultat |
|---|---|---|
| Unicité des clés (dims + staging) | 6 | ✅ PASS |
| Complétude (`not_null`) | 10 | ✅ PASS |
| Intégrité référentielle | 7 | ✅ PASS |
| Validité de domaine (`accepted_values` status) | 1 | ✅ PASS |
| Autres (relations staging) | 4 | ✅ PASS |
| **Total** | **28** | **✅ 100 %** |

## Rejouer les contrôles

```bash
cd projet-10-pipeline-elt/dbt_ecommerce
export DBT_PROFILES_DIR=$PWD        # Windows : set DBT_PROFILES_DIR=%CD%
python -m dbt.cli.main test
```

Un échec affiche la ligne fautive et la règle violée → correction avant livraison.

## Extensions possibles

- **Plages de valeurs** (`quantity` > 0, `line_amount` ≥ 0) via un package
  (`dbt_utils.accepted_range`) ou un test *singular* SQL.
- **Fraîcheur** automatisée : `dbt source freshness` (voir [ownership-sla.md](ownership-sla.md)).
- **Great Expectations** pour des attentes plus riches sur la couche `raw`.
