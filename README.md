# Présentation PFA — Structure des slides (question, v1–v4, conclusion)
## Question du projet (slide titre)
**Impact du réchauffement climatique sur le marché boursier**
Progression méthodologique : du **marché large** (S&P 500) au **marché agricole ciblé** (blé), en passant par une **échelle pays** (France) puis un **actif physique** (maïs).
---
## Version 1 — Température « mondiale » × S&P 500
**Objectif :** Tester si la **température à l’échelle mondiale** (NASA GISS agrégée) est liée au **marché actions mondial** proxy par le **S&P 500** (rendements mensuels).
**Méthode de réflexion :**
- Si l’anomalie de température **globale** (et ventilations nord/sud possibles) **explique ou prédit** une partie des **rendements / signe** du mois sur le S&P.
- Protocole **mensuel** : fusion données boursières + séries thermiques, **lags**, baseline **marché** vs **marché + climat** (voir `PFA_pipeline.ipynb`).
**Jeux de données :**
- `dataSets/sp500_monthly_returns.csv`
- `dataSets/gistemp250_GHCNv4.csv` (GISS, agrégation mensuelle globale / par hémisphères selon le script)
**Forme finale des données :** tableau **mensuel** fusionné ; dans le notebook pipeline : **≈ 1178 lignes**, **1928-01 → 2026-02** (calendrier commun S&P × température après alignement).
**Résultat du test :**
- **Pas de « signal climat » net et exploitable** au sens où, malgré un protocole affiné (baseline marché sérieuse, lissage climat, modèle plus riche), le **climat ne permet pas** de conclure à une **prédiction fiable** du marché : précision **proche** des baselines / **sous** des stratégies triviales comme « toujours hausier » sur certaines fenêtres ; le **delta** utile reste **faible** par rapport au bruit du marché.
**Donc :** Le **périmètre est peut‑être trop large** (indices globaux, échelle climat très agrégée, drivers macro dominants) → **affiner l’hypothèse** (pays, secteur, actif lié au climat).
---
## Version 2 — Météo France × AXA & CAC 40
**Objectif :** Tester si la **météo en France** est associée au cours **AXA** (`CS.PA`), **en contrôlant le marché français** (**CAC 40**).
**Méthode de réflexion :**
- La température / précipitations **nationales** (agrégat à partir de données régionales journalières) peuvent‑elles expliquer les **rendements AXA** une fois le **CAC** pris en compte ?
- Tester aussi si le **secteur / titre** change la plausibilité du lien (AXA **mondiale** vs indicateur **France**).
**Jeux de données :**
- `dataSets/v2/axa_monthly_returns.csv`
- `dataSets/v2/cac40_monthly_returns.csv`
- Données météo France (fichiers station / agrégats dans `dataSets/v2/` selon `PFA_v2_axa_france.ipynb`)
**Forme finale :** **mensuelle** ; sorties notebook : **AXA ≈ 428 mois (1990-09 → 2026-04)**, **CAC ≈ 433 mois (1990-04 → 2026-04)** ; tableau de modèle après **lags** et alignement (fenêtre effective = **intersection** des séries).
**Résultat du test :**
- **OLS / logit :** **gain d’ajustement très faible** quand on ajoute la météo (ex. **Δ R² ajusté ~ +0,007** ; pseudo R² logit **+0,006** — ordre de grandeur **faible**).
- **Gradient Boosting (optionnel) :** sur un **petit échantillon de test** (ex. **n_test = 27**), le modèle **+ météo** peut dépasser le **marché seul**, mais reste **loin** d’une baseline **classe majoritaire** (~**59 %**) → **pas de preuve robuste** pour dire que la météo France « fait » le cours AXA.
**Donc :** Même en **resserrant** géographiquement, le lien climat → **action** reste **ténue / fragile** ; **contrepartie** : AXA n’est pas une **pure** exposition « météo France ».
---
## Version 3 — Température + SPEI × marché agricole (maïs)
**Objectif :** Tester si **température** (GISS, **ceinture du maïs**) et **SPEI USA** expliquent les **rendements mensuels** d’un actif **directement lié** à l’agro : **maïs CBOT `ZC=F`**.
*(Dans `PFA_v3_corn_gistemp.ipynb`, c’est **le maïs** ; pas de série **soja** dans ce fichier — slide possible : « focus maïs ZC=F » ; extension soja seulement si tu l’as traitée.)*
**Méthode de réflexion :**
- Si le climat « **local agricole** » + stress hydrique **national** (SPEI) touchent un marché **commodité** plus que le S&P.
- Même logique **mensuelle**, **lags**, régression + classification **hausse / baisse**.
**Jeux de données :**
- `dataSets/v3/corn_monthly_returns.csv`
- `dataSets/v3/gistemp250_GHCNv4.csv`
- `dataSets/v3/spei_usa.csv`
**Forme finale :** **mensuel** ; panel fusionné **≈ 304 mois**, **2000-08 → 2025-11** (d’après les sorties du notebook).
**Résultat du test :**
- **Corrélations** globales **faibles** ; en **classification**, **accuracy test** proche d’une **baseline** (ex. **~0,57** ou **égalité modèle / baseline majoritaire ~0,40** selon la spec de la cellule) → **pas d’avantage clair et stable** du climat pour battre une règle simple sur le **signe** du mois.
**Donc :** Même sur un **actif agricole**, le **prix à terme** intègre **anticipation, monde, stocks, macro** — le **seul** couple température + SPEI (tel qu’agrégé) ne suffit pas pour une relation **forte** ; **poursuivre** sur une culture / série **blé** et une chaîne **climat → production → marché** (v4).
---
## Version 4 — Synthèse finale (blé : climat → production → marché)
**Objectif (fil conducteur) :**
1. **Climat** (temp. Plaines + **SPEI USA**) × **marché blé** (`ZW=F`) — **mensuel**.
2. Même climat × **production blé USA** (annuel).
3. **Production USA** × **marché** (rendement annuel composé à partir des mois).
**Méthode de réflexion :**
- **Étape 1 :** Le problème de v3 (« pas de lien net prix–climat mensuel ») peut‑il venir du **bruit du marché** ? Alors tester si le climat touche d’abord une variable **physique** (**production**).
- **Étape 2 :** La **production** « médiatisée » par le climat est‑elle cohérente avec les données ? (corrélations vs **prévision** hors échantillon.)
- **Étape 3 :** La **production** seule **explique‑t‑elle** le **marché** ? (Marché **mondial**, pas seulement offre US.)
**Jeux de données :**
| Partie | Fichiers (v4) | Forme / période (indicatif, selon notebook) |
|--------|----------------|---------------------------------------------|
| **Partie 1** marché mensuel | `dataSets/v4/wheat_monthly_returns.csv`, `gistemp250_GHCNv4.csv`, `spei_usa.csv` | **Mensuel** ; blé **~2000-08 → ~2026** (**n ≈ 309** mois après alignement intérieur) |
| **Partie 2** climat → production | Idem GISS + SPEI + `dataSets/v4/american_wheat_production.csv` | **Annuel** après lag météo ; **~1961 → ~2025**, **≈ 65** années en panel (convention lag **Y−1 → Y**) |
| **Partie 2.2** production → marché | `american_wheat_production.csv` + `wheat_monthly_returns.csv` | **Annuel** rendement ZW=F composé ; **≈ 26** années **2000–2025** |
**Résultats (3 points pour slide « résultats ») :**
1. **Partie 1 :** Climat en **lags** **n’améliore pas** de façon **claire** la prédiction du **signe** du rendement mensuel vs **baseline** (accuracy / F1 **proches** — comme en synthèse du notebook `PFA_v4_wheat_speitemp.ipynb`).
2. **Partie 2 :** Sur **long historique**, **corrélations** non nulles (temp / SPEI vs production) mais **modèle linéaire** : **R² test** **très mauvais** ; **MAE** un peu **mieux** que la moyenne → **association** possible mais **pas** un modèle **prédictif** simple / **tendances** et spec à nuancer.
3. **Partie 2.2 :** **Corrélations faibles** (~**0,24** en niveau et en croissance) ; **petit** gain **MAE** sur test, **R²** négatifs → **pas de lien direct fort** production US ↔ rendement annuel **ZW=F** (marché **mondial** + **anticipation**).
**Donc (slide avant-conclusion) :**
- La chaîne **climat → bourse** **ne se voit pas** de façon **robuste** avec ces **agrégats** et ces **échelles**.
- Le climat peut **coexister** avec la **production** sur données annuelles, mais **passer au marché** ajoute **tout le reste** : **demande mondiale, stocks, autres origines, taux, dollar, géopolitique, chocs d’offre ailleurs**, etc.
---
## Conclusion finale (1 slide)
**En résumé :**
Nous ne montrons pas qu’il **n’existe aucun** lien climat–finance ; nous montrons qu’**avec des données publiques, des agrégats spatiaux/temporels et des actifs très liquides**, le signal climatique est **noyé** par la **complexité des marchés**.
**Périmètre trop large (v1)** → **affiné** (v2 pays, v3–v4 **commodités**) ; même ainsi, **le cours ne réagit pas comme une simple fonction** de la température ou du SPEI.
**Message clé pour le jury :** l’**hypothèse** est raisonnable, le **protocole** est progressif ; le **résultat négatif** est **informatif** : il **justifie** l’ajout de variables **non climatiques** (demande, politique monétaire, crises) pour expliquer la bourse — ce qui **cadre** la question initiale sur le **réchauffement** et le **marché**.
---