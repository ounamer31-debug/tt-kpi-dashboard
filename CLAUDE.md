# CLAUDE.md — Projet PFE : KPI de vente Tunisie Télécom

> Fichier de contexte pour Claude Code. Il décrit le projet, son état
> d'avancement et la marche à suivre pour le continuer.

## 1. Contexte du projet

Projet de Fin d'Études (Licence Appliquée) réalisé chez **Tunisie Télécom**,
durée du stage : **2 mois**. L'objectif est une application qui **calcule,
suit et prévoit** les indicateurs de performance (KPI) de vente.

L'étudiant est **débutant** (quelques bases en Python). → **Explique
clairement chaque fichier et chaque concept que tu introduis**, car il doit
comprendre son code pour la soutenance. Avance **étape par étape**, ne
génère pas tout le projet d'un coup.

## 2. Objectif fonctionnel

À partir des ventes journalières par sous-catégorie et des objectifs mensuels
par catégorie :

1. Calculer le **cumul mensuel** des ventes par catégorie.
2. Comparer au **objectif mensuel** → **taux de réalisation (%)**.
3. Suivre le **cumul annuel** vs objectif cumulé.
4. Visualiser en **histogrammes** (dashboard interactif).
5. **Prévoir** les ventes futures (Machine Learning, modèle Prophet).
6. Estimer la **probabilité d'atteinte** des objectifs annuels.
7. **Détecter les anomalies** de vente.
8. Analyser par **région / agence**.

## 3. Stack technique

- **Langage** : Python 3.14 (environnement virtuel `venv`)
- **Données** : Pandas / NumPy
- **Visualisation** : Plotly
- **Interface / dashboard** : Streamlit
- **Machine Learning** : Prophet (prévision de séries temporelles),
  scikit-learn (détection d'anomalies)
- **Source de données** : fichiers CSV dans `data/` (extensible vers SQL)

> ⚠️ Python 3.14 est très récent : si `pip install prophet` échoue (pas de
> wheel compatible), proposer un repli avec `statsmodels` (SARIMA /
> Holt-Winters) ou une régression scikit-learn sur features temporelles
> (tendance + saisonnalité). Ne pas bloquer l'étudiant sur l'installation.

## 4. Modèle de données

**`data/ventes.csv`** — une ligne = ventes d'une sous-catégorie un jour donné :

| colonne | type | exemple |
|---|---|---|
| date | date (AAAA-MM-JJ) | 2024-01-01 |
| categorie | texte | Internet Fixe |
| sous_categorie | texte | Rapido |
| quantite | entier | 6 |

**`data/objectifs.csv`** — un objectif mensuel par catégorie :

| colonne | type | exemple |
|---|---|---|
| categorie | texte | Internet Fixe |
| annee | entier | 2026 |
| mois | entier (1–12) | 1 |
| objectif_mensuel | entier | 1309 |

Catégories et sous-catégories :

- **Internet Fixe** : Rapido, ADSL, VDSL, FO, WAFI, Box
- **Mobile** : Prepaye, Postpaye, Data

Les données actuelles sont **simulées** (démo) et couvrent 2024, 2025 et
janvier→juin 2026. Les mois de juillet à décembre 2026 sont donc à 0 dans le
réalisé — c'est normal, et c'est ce que le module ML devra prédire.
L'objectif annuel = somme des 12 objectifs mensuels de l'année.

## 5. Structure du projet

```
tt_kpi/
├── data/
│   ├── ventes.csv          # ventes journalières (simulées)
│   └── objectifs.csv       # objectifs mensuels par catégorie
├── decouverte.py           # étape 2 : aperçu des données
├── calcul_kpi.py           # étape 3 : cumuls mensuels + taux de réalisation
├── kpi.py                  # module partagé : calcul KPI (importé par calcul_kpi.py + app.py)
├── app.py                  # étape 4-5 : dashboard Streamlit (5 onglets, design perso, logo)
├── forecast.py             # étape 6 : prévision Prophet
├── prediction_atteinte.py  # étape 7 : probabilité d'atteinte de l'objectif annuel
├── anomalies.py            # étape 7 : détection des jours de vente anormaux (z-score)
├── assets/logo_tt.png      # logo Tunisie Télécom (en-tête du dashboard)
├── .streamlit/config.toml  # thème du dashboard
├── requirements.txt        # librairies à installer (pip install -r)
├── README.md               # fiche du projet (installation, lancement, structure)
├── .gitignore              # exclut venv/ et __pycache__/ du dépôt Git
├── venv/                   # environnement virtuel
└── CLAUDE.md               # ce fichier
```

## 6. État d'avancement

- [x] **Étape 1 — Environnement** : Python 3.14.6, venv, librairies de base.
- [x] **Étape 2 — Données** : CSV en place, `decouverte.py` (aperçu).
- [x] **Étape 3 — Calcul KPI** : `calcul_kpi.py` (cumul mensuel + taux).
- [x] **Étape 4 — Premier dashboard Streamlit** : `app.py` avec tableau KPI,
      histogramme réalisé vs objectif par mois, et détail des ventes
      réelles par sous-catégorie. Sélecteur de catégorie en barre latérale.
      Lancer avec `streamlit run app.py`.
- [x] **Étape 5 — Enrichir le dashboard** : `app.py` comporte 5 onglets
      (Tableau & mensuel, Suivi cumulé, Détail sous-catégories, Comparaison
      catégories, **Prévision & alertes**), des cartes KPI (`st.metric`), des
      sélecteurs année/catégorie et un export CSV. L'onglet « Prévision &
      alertes » intègre la prévision Prophet (avec bande de confiance via
      `plotly.graph_objects`), la probabilité d'atteinte annuelle et le
      tableau des anomalies — en lisant les CSV produits aux étapes 6 et 7
      (message d'avertissement si un fichier est absent).
- [x] **Étape 6 — Machine Learning** : `forecast.py` construit la série
      mensuelle par catégorie, entraîne **Prophet** (installé sans problème
      en Python 3.14, pas besoin du repli statsmodels), prévoit juillet 2026
      → décembre 2027, affiche la prévision avec intervalle de confiance
      (graphiques Plotly) et sauvegarde `data/prevision.csv`.
- [x] **Étape 7 — Prédiction d'atteinte + anomalies** :
      - `prediction_atteinte.py` : combine le réalisé connu (jan→juin 2026) et
        la prévision Prophet (`data/prevision.csv`) pour estimer le total
        annuel, le compare à l'objectif annuel (somme des 12 mois) et calcule
        une **probabilité d'atteinte** (loi normale déduite de la fourchette
        min/max de Prophet). Génère `data/atteinte_objectif.csv`. Note : sur
        des données simulées très propres, Prophet est très confiant → les
        probabilités tombent à ~0 %, le chiffre parlant est le *taux estimé*
        (~93–98 %, il manque 330 à 1346 ventes selon la catégorie).
      - `anomalies.py` : détecte les jours de vente anormaux par **z-score**
        (|z| > 3) calculé par sous-catégorie, avec protection division par
        zéro. Génère `data/anomalies.csv` (31 anomalies / 8208 jours ≈ 0,38 %,
        cohérent avec la théorie ~0,3 %). Limite connue : ignore tendance et
        saisonnalité ; Isolation Forest serait l'alternative plus fine.
- [ ] **Étape 8 — Finalisation** : dimension régionale/agence (si le temps le
      permet — priorité MoSCoW « Could »), rédaction du mémoire, préparation
      de la soutenance.

## 7. Priorisation (MoSCoW)

- **Must** (indispensable) : import données, cumul mensuel, taux de
  réalisation, histogrammes, prévision Prophet.
- **Should** (important) : suivi cumulé, détail sous-catégories, prédiction
  d'atteinte des objectifs.
- **Could** (si le temps le permet) : détection d'anomalies, dimension
  régionale, export des rapports.

Sécuriser d'abord tout le « Must » avant d'aborder le « Could ».

## 8. Conventions de code

- Commentaires et messages **en français**.
- Code **modulaire** : séparer données, calcul KPI, ML et interface dans des
  fichiers distincts (`calcul_kpi.py` / `forecast.py` / `app.py`).
- Code **simple et lisible** avant tout (l'étudiant est débutant) — éviter les
  abstractions inutiles.
- Ne pas utiliser `localStorage` ni d'API externe non nécessaire.
- Ne pas régénérer les données simulées sauf demande explicite.
- Protéger les divisions par zéro dans les calculs de taux.

## 9. Consignes de collaboration

À chaque nouvelle étape :
1. Explique d'abord **ce que l'étape va accomplir** et pourquoi.
2. Crée ou modifie le fichier concerné, avec des **commentaires en français**.
3. Explique les **nouvelles notions** introduites (une explication courte par
   concept).
4. Donne la **commande exacte** pour lancer/tester.
5. Attends la confirmation que ça marche avant de passer à l'étape suivante.

## 10. Commandes utiles

```bash
# Activer l'environnement virtuel (Windows)
venv\Scripts\activate

# Lancer un script Python
python calcul_kpi.py

# Lancer le dashboard (à partir de l'étape 4)
streamlit run app.py
```
