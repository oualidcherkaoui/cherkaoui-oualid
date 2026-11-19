# Fiche de Synthèse du Projet : Machine Learning appliqué au Commerce International

* **Étudiant :** Oualid Cherkaoui
* **Code Apogée :** 23009078
* **Établissement :** ENCG Settat
* **Sujet attribué :** CI16 — Catégories de partenaires
* **Dépôt GitHub :** `23009078-cherkaoui-ci`

---

## 1. Contexte et Problématique
Dans un environnement économique international marqué par une forte volatilité conjoncturelle, l'identification précoce des partenaires commerciaux les plus dynamiques constitue un levier stratégique pour orienter les politiques d'exportation et d'allocation des ressources.

* **Problématique :** *Dans quelle mesure l'historique récent des flux commerciaux permet-il de classifier automatiquement la dynamique de croissance future des partenaires commerciaux en trois catégories (faible, modérée, forte) sans fuite d'information temporelle ?*
* **Objectif opérationnel :** Construire un pipeline de classification supervisée reproductible surpassant significativement une référence naïve sur des périodes futures non vues.

---

## 2. Données et Préparation sans Fuite (*No Data Leakage*)
* **Source officielle :** Données réelles extraites de la plateforme institutionnelle de la Banque mondiale (WITS / Indicateurs du développement dans le monde) / UN Comtrade.
* **Ingénierie des variables (exclusivement issues du passé) :**
  * `lag_1` : valeur/croissance observée à la période $t-1$.
  * `lag_2` : valeur/croissance observée à la période $t-2$.
  * `diff_1` : accélération récente (`lag_1` - `lag_2`).
* **Variable cible (`cible`) :** Discrétisation de la croissance en 3 classes équilibrées ($0$ : Faible, $1$ : Modérée, $2$ : Forte) à l'aide des quantiles calculés **uniquement sur le jeu d'entraînement**.
* **Découpage chronologique strict :** 80 % des périodes les plus anciennes pour l'entraînement/validation et 20 % des années les plus récentes réservées pour le test final.
* **Encapsulation :** Toutes les transformations (`SimpleImputer` médian et `StandardScaler`) sont intégrées dans un `ColumnTransformer` / `Pipeline` ajusté (`fit`) exclusivement sur `X_train`.

---

## 3. Méthodologie et Algorithme Principal
* **Algorithme retenu :** **Forêts Aléatoires (*Random Forest Classifier*)** — Famille Classification (algorithme du cours).
* **Justification :** Capacité à modéliser des relations non linéaires et des effets de seuil propres aux cycles commerciaux tout en réduisant la variance par agrégation bootstrap (*bagging*), contrairement à une régression logistique (trop rigide linéairement) ou au KNN (sensible aux valeurs extrêmes).
* **Formulation mathématique :**
  $$\hat{y} = \text{mode}\left\{ h_1(\mathbf{x}), h_2(\mathbf{x}), \dots, h_B(\mathbf{x}) \right\}$$
  où $\mathbf{x}$ représente le vecteur des variables retardées, $B$ le nombre d'arbres (`n_estimators`) et $h_b(\mathbf{x})$ la prédiction du $b$-ième arbre construit sur un échantillon bootstrap.

---

## 4. Protocole Expérimental et Évaluation
* **Référence naïve (*Baseline*) :** `DummyClassifier(strategy="most_frequent")` prédisant systématiquement la classe majoritaire du jeu d'entraînement.
* **Validation croisée temporelle :** Optimisation des hyperparamètres (`n_estimators` et `max_depth`) via `GridSearchCV` couplé à `TimeSeriesSplit(n_splits=3)` sur `X_train` uniquement.
* **Métriques :** **F1-macro** (métrique principale pour traiter équitablement les 3 classes), complétée par l'**Accuracy** et l'analyse de la **matrice de confusion**.

---

## 5. Synthèse des Résultats et Analyse Critique
* **Apport du modèle :** Le modèle *Random Forest* améliore nettement le score F1-macro par rapport à la référence naïve. L'analyse de l'importance des variables confirme le poids prépondérant de la dynamique immédiate (`lag_1`) et de l'accélération (`diff_1`). Les erreurs résiduelles se concentrent principalement entre classes adjacentes lors des retournements conjoncturels.
* **Limites identifiées :**
  1. L'agrégation macroéconomique annuelle masque les disparités sectorielles par produit (codes SH) et les variations infra-annuelles.
  2. La dépendance aux seules variables endogènes passées limite l'anticipation des chocs exogènes brutaux (crises sanitaires, ruptures tarifaires ou géopolitiques).
* **Perspectives d'amélioration :**
  1. Intégrer des variables exogènes bilatérales (droits de douane appliqués, taux de change réel effectif, croissance du PIB des partenaires).
  2. Désagréger les données au niveau sectoriel (chapitres SH2/SH4) et mensuel, et tester des modèles de gradient boosting (*HistGradientBoostingClassifier*).

---

## 6. Structure du Dépôt GitHub
```text
23009078-cherkaoui-ci/
├── data/
│   └── raw/
│       └── donnees_CI16.csv
├── figures/
│   ├── figure_1.png
│   ├── figure_2.png
│   ├── figure_3.png
│   ├── schema_algorithme.png
│   └── figure_4_resultats.png
├── notebook_CI16_Cherkaoui.ipynb
├── RAPPORT_CI16.md
├── DECLARATION_IA.md
├── requirements.txt
└── README.md
```