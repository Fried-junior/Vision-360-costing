# Vision360 — Maîtriser les coûts par caméra d'une entreprise de vison par ordianteur
*Prototype Shiny + modèle élastique (volumes) + dataset simulé multi-verticales*

> **Objectif.** Aider à **chiffrer**, **suivre** et **tarifer** des projets de vision par ordinateur, en séparant coûts variables et overhead, et en tenant compte des **spécificités par verticale** (Aéroport, Autoroute, Usine, Retail/Boutique, Centre commercial, Stade).

## 🔎 Contenu du dépôt
- `app.R` — Application **Shiny** (simulateur prix–marge, coûts estimés par caméra et au total).
- `dataVision360.csv` — Jeu de données **simulé** réaliste (multi-verticales).
- `models/model_coefs.json` — Coefficients du **modèle poolé** (volumes uniquement).
- `notebooks/01_modele_et_validation.Rmd` — Reproductibilité du modèle (estimations + diagnostics).
- `docs/` — Captures d’écran, schémas, éventuelle doc additionnelle.

## 📦 Données : strictement **simulées**. Voir le *dataVision360* ci-dessus.
> La base regroupe des **séries mensuelles par projet** pour une entreprise fictive de vision par ordinateur. Elle couvre plusieurs **verticales** aux profils d’usage distincts : **Aéroport**, **Autoroute**, **Usine**, **Centre commercial**, **Retail/Boutique**, **Stade**. Chaque ligne correspond à un **projet donné sur un mois** (`projet × période`).

L’objectif de ce jeu de données est double :

1. **Modéliser** le **coût variable unitaire par caméra** à partir de **volumes d’usage** (et non de prix), via un modèle log-linéaire.
2. **Outiller** le **chiffrage/pricing** dans l’app Shiny en fournissant des ordres de grandeur réalistes et des cas d’usage variés.


### Grain, périodes et périmètre

* **Grain** : mensuel, par projet.
* **Période** : champ `periode` au format `YYYY-MM`.
* **Périmètre** : lignes **projets** (à utiliser dans le modèle) + lignes **“Charges\_entreprise”** et **“Investissement”** (servant à l’overhead/capex et à exclure des régressions sur coûts variables).

### Variables clés (volumes)

Le modèle n’utilise que des **volumes** ; ils sont fournis **au total projet par mois** et normalisés **par caméra** côté modèle/app.

* `nb_cameras` : nombre moyen de caméras actives (mois).
* `gpu_hours` : heures GPU consommées (mois).
* `bw_go` : bande passante sortante en Go (mois).
* `storage_go` : stockage facturé en Go-mois.
* `logs_go` : volume de logs en Go (mois).
* `dev_hours` : **temps de travail mensuel** dédié au projet (heures/mois). C’est la **mesure de développement variable** utilisée par le modèle (remplace les ETP).
* *(Optionnel)* `sms_push` : notifications envoyées (mois), utilisées seulement si l’on souhaite tester un driver “messaging”.
Ces volumes sont transformés en **ratios par caméra** (ex. `gpu_pc = gpu_hours / nb_cameras`) puis passés en **log** pour estimer les élasticités.

### Spécificités par verticale (rappels)

Les profils de consommation diffèrent :
* **Aéroport** : calcul et logs **intensifs**.
* **Autoroute** : logs et egress élevés.
* **Usine** : **stockage** massif.
* **Centre commercial** : mix équilibré entre calcul/BW/stockage.
* **Retail/Boutique** : charges légères et régulières.
* **Stade** : période événementielle, **alertes denses**.

Ces différences sont captées soit par des **dummies** (en estimation), soit par un **ajustement d’intercept** selon la verticale sélectionnée dans l’app.


## 🧠 Modèle (résumé)
- Forme **log-linéaire** sur **volumes** (par caméra) : `ln(cu) ~ ln(nb_cam) + ln(GPU/cam) + ln(BW/cam) + ln(Storage/cam) + ln(Logs/cam) + ln(Dev h/cam)`.
- **Effet verticale** via un **décalage d’intercept** (optionnel) issu d’une estimation avec dummies `Type` (Aéroport, Autoroute, etc.).
- Lecture business : élasticités positives sur GPU/BW/Storage/Logs/Dev ; effet de taille (nb_cam) capturé.
