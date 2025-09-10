# Vision360 — Maîtriser les coûts par caméra en 90 jours
*Prototype Shiny + modèle élastique (volumes) + dataset simulé multi-verticales*

> **Objectif.** Aider à **chiffrer**, **suivre** et **tarifer** des projets de vision par ordinateur, en séparant coûts variables et overhead, et en tenant compte des **spécificités par verticale** (Aéroport, Autoroute, Usine, Retail/Boutique, Centre commercial, Stade).

## 🔎 Contenu du dépôt
- `app.R` — Application **Shiny** (simulateur prix–marge, coûts estimés par caméra et au total).
- `data/vision360_synthetique.csv` — Jeu de données **simulé** réaliste (multi-verticales).
- `models/model_coefs.json` — Coefficients du **modèle poolé** (volumes uniquement).
- `notebooks/01_modele_et_validation.Rmd` — Reproductibilité du modèle (estimations + diagnostics).
- `docs/` — Captures d’écran, schémas, éventuelle doc additionnelle.

> **Données** : strictement **simulées**. Voir le *data dictionary* ci-dessous.

## 🧠 Modèle (résumé)
- Forme **log-linéaire** sur **volumes** (par caméra) : `ln(cu) ~ ln(nb_cam) + ln(GPU/cam) + ln(BW/cam) + ln(Storage/cam) + ln(Logs/cam) + ln(Dev h/cam)`.
- **Effet verticale** via un **décalage d’intercept** (optionnel) issu d’une estimation avec dummies `Type` (Aéroport, Autoroute, etc.).
- Lecture business : élasticités positives sur GPU/BW/Storage/Logs/Dev ; effet de taille (nb_cam) capturé.

