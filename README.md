# Banc d'essai — Courbes d'apprentissage & ETP productif

Outil web autonome (un seul fichier HTML) pour **ajuster plusieurs modèles de courbes d'apprentissage à des données observées** et les **départager par les critères d'information** (AIC, BIC), plutôt que d'imposer une forme de courbe *a priori*.

L'outil produit, à partir de couples `(mois, productivité)` :

- l'ajustement de sept modèles concurrents par moindres carrés ;
- un tableau comparatif (RMSE, MAPE, R², AICc, ΔAICc, BIC) classé du meilleur au moins bon ;
- une interprétation automatique signalant les modèles statistiquement équivalents ;
- une projection du meilleur modèle au-delà des données, avec un ETP productif normalisé lorsque le modèle possède une asymptote finie.

> **Nature de l'outil.** Il s'agit d'un prototype d'exploration et de comparaison, pas d'un logiciel de calcul certifié. Les résultats doivent être recoupés avec une implémentation de référence avant toute publication (voir *Limites*).

---

## Démarrage

Aucune installation. Ouvrez `banc-essai-courbes_-_v3.html` dans un navigateur récent (Firefox, Chrome, Edge, Safari).

Deux dépendances sont chargées depuis un CDN et **nécessitent une connexion Internet** au premier affichage :

- [Chart.js](https://www.chartjs.org/) (v4) — tracé des graphes ;
- [MathJax](https://www.mathjax.org/) (v3) — rendu des formules dans la fenêtre « Modèles & sources ».

Si vous travaillez hors ligne, il faut héberger ces deux bibliothèques localement et adapter les balises `<script>` correspondantes.

---

## Mode d'emploi

### 1. Saisir les données

Trois façons d'alimenter la grille :

- **Saisie directe** dans les deux colonnes *Mois (x)* et *Productivité (y)*. La touche `Entrée` crée une nouvelle ligne ; les flèches `↑`/`↓` déplacent le curseur d'une ligne.
- **Coller depuis Excel** via le bouton dédié : collez deux colonnes, puis « Importer ». Les séparateurs tabulation, point-virgule, virgule et espace sont reconnus, et une première ligne d'en-tête textuelle est ignorée automatiquement.
- **Données de démo** : un jeu synthétique bruité (forme exponentielle connue), utile pour prendre l'outil en main.

La **virgule décimale** (`0,34`) est acceptée et convertie. Les cellules non numériques sont **surlignées en rose** et exclues du calcul ; un compteur indique en permanence le nombre de points valides.

> **Ambiguïté connue.** Si une source utilise la virgule à la fois comme séparateur décimal *et* comme séparateur de colonnes, le découpage est indécidable de façon fiable. Le collage depuis Excel utilise normalement la tabulation comme séparateur de colonnes, ce qui lève l'ambiguïté ; en cas de doute, vérifiez la grille après import.

La variable `x` représente le **temps cumulé d'expérience** (ici exprimé en mois). La variable `y` est une mesure de productivité libre : taux, nombre de dossiers traités par jour, pourcentage de cible, etc.

### 2. Choisir les modèles

Cochez les modèles à confronter (les sept sont actifs par défaut). Le nombre de paramètres de chaque modèle est rappelé entre parenthèses.

### 3. Ajuster et comparer

Le bouton « Ajuster & comparer » lance l'optimisation. Un minimum de **4 points valides** est requis. Le tableau de résultats est classé par AICc croissant ; la ligne du meilleur modèle est surlignée et marquée d'une étoile.

### 4. Lire la projection

Le meilleur modèle est extrapolé jusqu'à l'horizon choisi (curseur). Lorsque le modèle a une asymptote finie, une courbe d'**ETP productif normalisé** (productivité ÷ asymptote théorique) est tracée, et le mois d'atteinte de 95 % de l'asymptote est indiqué. Pour les modèles à croissance non bornée, l'ETP n'est pas tracé (voir *Note sur l'ETP*).

---

## Les sept modèles

`x` désigne le temps d'expérience, `y(x)` la productivité prédite. Les paramètres sont estimés par minimisation de la somme des carrés des écarts (SSE).

| Modèle | Paramètres | Forme | Asymptote finie |
|---|---|---|---|
| Wright (loi de puissance) | 2 | `y = a·(x+1)^b` | non |
| Exponentielle | 3 | `y = A·(1 − e^(−x/τ)) + c₀` | oui : `A + c₀` |
| Stanford-B | 3 | `y = a·(x + B + 1)^b` | non |
| DeJong | 3 | `y = a·[M + (1−M)·(x+1)^b]` | non |
| Logistique | 4 | `y = L / (1 + e^(−k(x−x₀))) + c₀` | oui : `L + c₀` |
| Gompertz | 4 | `y = L·e^(−e^(−k(x−x₀))) + c₀` | oui : `L + c₀` |
| Hyperbolique à 3 paramètres | 3 | `y = c₀ + (L − c₀)·x / (x + h)` | oui : `L` |

Quelques repères d'interprétation :

- **Wright / loi de puissance** est le modèle historique des courbes d'apprentissage en production (Wright, 1936). Le `+1` évite la singularité en `x = 0`.
- **Stanford-B** intègre l'expérience antérieure du travailleur via un décalage `B` : c'est une formalisation publiée de la notion d'« ancienneté déjà acquise ».
- **DeJong** introduit un facteur d'incompressibilité `M ∈ [0,1]` représentant la part de la tâche non améliorable : c'est une formalisation publiée de la notion de « plancher de compétence initiale ».
- **Logistique** et **Gompertz** produisent un véritable « S » (démarrage lent, accélération, plateau). La littérature suggère que la forme en S apparaît surtout lorsque le taux d'apprentissage est faible ; elle n'est pas universelle.
- **Hyperbolique à 3 paramètres** a été mis en avant comme le mieux ajusté dans certaines études empiriques d'order picking et d'assemblage.

> **Sur la fidélité des équations.** Les formes ci-dessus sont reprises de revues de littérature, et non vérifiées une à une dans les articles d'origine. Avant toute citation formelle, recoupez-les avec les sources primaires (notamment Wright 1936 et Anzanello & Fogliatto 2011).

---

## Méthodologie de comparaison

Pour chaque modèle ajusté, l'outil calcule :

- **SSE / RMSE** — erreur résiduelle brute ;
- **MAPE** — erreur absolue moyenne en pourcentage, lisible côté métier ;
- **R²** — part de variance expliquée ;
- **AIC / AICc / BIC** — critères pénalisant le nombre de paramètres pour limiter le surajustement. **Le plus petit l'emporte.**

Les critères AIC et BIC sont calculés sous l'hypothèse d'erreurs gaussiennes homoscédastiques, via la forme `n·ln(SSE/n) + pénalité`. L'**AICc** (AIC corrigé pour les petits échantillons) est utilisé pour le classement, car il est plus prudent lorsque le nombre de points est faible au regard du nombre de paramètres.

Règle d'interprétation usuelle retenue dans l'outil : **ΔAICc < 2** entre deux modèles signifie qu'ils sont pratiquement équivalents au regard des données. Dans ce cas, l'outil recommande le modèle le plus parcimonieux (le moins de paramètres).

> **« Meilleur ajustement » ≠ « meilleure prédiction ».** Les critères mesurent la qualité d'ajustement aux données observées, pas la capacité de généralisation. Une validation croisée ou une évaluation sur données hors échantillon reste nécessaire pour juger du pouvoir prédictif.

---

## Note sur l'ETP productif

L'« ETP productif » normalise la productivité par la valeur maximale atteignable par l'agent, de sorte que `1.0` corresponde à un agent au plein de ses moyens.

Cette normalisation n'a de sens que si le modèle possède une **asymptote finie**. L'outil utilise donc l'asymptote *théorique* (forme analytique, calculée à partir des paramètres ajustés) et non le maximum numérique observé sur l'horizon. Pour les modèles à croissance non bornée (Wright, Stanford-B, DeJong), aucune asymptote finie n'existe : la courbe d'ETP n'est alors pas tracée, et l'outil l'indique explicitement. Un modèle à plateau est requis pour obtenir un ETP interprétable.

---

## Implémentation

- **Un seul fichier HTML**, sans build ni serveur. Logique en JavaScript, sans framework.
- **Ajustement** : optimiseur Nelder–Mead écrit en JavaScript, avec redémarrages multiples depuis des points initiaux perturbés, pour réduire le risque de minimum local.
- **Aucune donnée n'est transmise** : tous les calculs sont locaux au navigateur.

> **Qualité de l'optimiseur.** Le Nelder–Mead maison convient à l'exploration. Sur des données générées par une forme connue, il retrouve correctement le modèle et ses paramètres lors des tests. Il ne remplace toutefois pas un optimiseur de référence : pour un travail destiné à publication, recoupez les résultats avec `scipy.optimize.curve_fit` (ajustement avec intervalles de confiance sur les paramètres).

---

## Limites

1. **Domaine d'application.** Aucune étude portant spécifiquement sur le travail administratif à complexité constante n'a été identifiée. La transposition s'appuie sur la littérature des courbes d'apprentissage en production répétitive et sur la psychologie de l'acquisition de compétences. À ce titre, l'application à un contexte administratif relève de l'hypothèse à valider.
2. **Agrégation.** Modéliser des agents individuels n'équivaut pas à modéliser une moyenne d'équipe : une forme lisse observée au niveau agrégé peut être un artefact de moyennage entre trajectoires individuelles hétérogènes.
3. **Optimiseur.** Voir ci-dessus : implémentation exploratoire, à confronter à une référence.
4. **Équations.** Reprises de revues secondaires, à vérifier dans les sources primaires.
5. **Échantillons réduits.** Avec peu de points, les critères AIC/BIC sont peu fiables et les modèles à quatre paramètres surajustent facilement. L'outil affiche un avertissement en dessous de 8 points.

---

## Références

Les références ci-dessous ont été identifiées par recherche documentaire et **doivent être vérifiées dans leur source primaire** avant toute citation formelle.

- Wright, T. P. (1936). *Factors affecting the cost of airplanes*. Journal of the Aeronautical Sciences.
- Anzanello, M. J. & Fogliatto, F. S. (2011). *Learning curve models and applications: literature review and research directions*. International Journal of Industrial Ergonomics, 41, 573–583.
- Heathcote, A., Brown, S. & Mewhort, D. J. K. (2000). *The power law repealed: the case for an exponential law of practice*. Psychonomic Bulletin & Review, 7(2), 185–207.
- Newell, A. & Rosenbloom, P. S. (1981). *Mechanisms of skill acquisition and the law of practice*.

---

## Licence

GPLv3 — https://mdph.over.blog/
