# ✈️ Projet Trafic aérien — Aéroports de Paris (ADP)

Analyse des vols au départ des trois aéroports de New York (JFK, LGA, EWR) pour comprendre et anticiper les **retards** et les **annulations**.
Données réelles du Bureau of Transportation Statistics (BTS), jeu *nycflights13*.

Le projet couvre tout le cycle de la donnée :
1. **Exploration** des données (Python / pandas) ;
2. **Création d'une base de données** PostgreSQL en ligne (Supabase), avec contrôle qualité, nettoyage, optimisation et gestion des droits ;
3. **Reporting** : pics de trafic, retards, annulations, carte des lignes ;
4. **Application web** (Streamlit) de reporting interactif et de prédiction des retards.

---

## 📁 Contenu du dépôt

```
trafic-aerien/
├── README.md                         ← ce fichier
├── requirements.txt                  ← bibliothèques Python
├── sql/
│   ├── 01_creation_tables.sql        ← schéma : tables, clés primaires/étrangères, contrôles regex
│   ├── 02_controles_qualite.sql      ← vérifications regex, PK, FK sur les données brutes
│   ├── 03_nettoyage_insertion.sql    ← nettoyage + insertion dans les tables finales
│   ├── 04_optimisation_types.sql     ← réduction du stockage (SMALLINT / REAL)
│   └── 05_privileges.sql             ← groupes, comptes et droits d'accès
├── notebooks/
│   └── Trafic_aerien.ipynb           ← Mission 1, Mission 2, reporting (3.1 à 1.2) et Mission 3
├── app/
│   ├── app.py                        ← application Streamlit
│   └── requirements.txt
├── graphiques/                       ← toutes les visualisations (PNG)
└── docs/
    └── problemes_et_solutions.md     ← anomalies rencontrées et stratégies (suivi Trello)
```

---

## ⚙️ Prérequis

- Un compte **Google** (pour Google Colab) ou Python 3.10+ en local ;
- Un projet **Supabase** (PostgreSQL), gratuit sur [supabase.com](https://supabase.com) ;
- Les fichiers de données `flights.xlsx` et `airports.xlsx` (fournis par l'école).
  Les tables `airlines`, `planes` et `weather` sont chargées depuis le paquet Python `nycflights13`.

Installation des bibliothèques en local :
```bash
pip install -r requirements.txt
```
Dans Colab, chaque notebook installe lui-même ce dont il a besoin.

---

## 🚀 Lancer le projet, étape par étape

### 1. Créer la base de données (Supabase → SQL Editor)
Exécuter **`sql/01_creation_tables.sql`** : crée les 5 tables avec leurs clés et leurs contrôles.

### 2. Charger les données brutes (Colab)
Ouvrir **`notebooks/Trafic_aerien.ipynb`** dans Google Colab :
1. téléverser `flights.xlsx` et `airports.xlsx` (icône 📁) ;
2. exécuter les cellules de la **Mission 1** (chargement et exploration) ;
3. dans la **Mission 2**, exécuter la cellule de connexion : le **mot de passe Supabase est demandé à l'écran** (il n'est jamais écrit dans le code) ;
4. exécuter la cellule d'envoi des **tables brutes** (`brut_*`).

### 3. Contrôler, nettoyer et insérer (Supabase → SQL Editor)
Exécuter dans l'ordre :
- **`sql/02_controles_qualite.sql`** : les requêtes une par une, pour voir chaque anomalie ;
- **`sql/03_nettoyage_insertion.sql`** : corrige les anomalies et remplit les tables finales ;
- **`sql/04_optimisation_types.sql`** : optimise le stockage ;
- **`sql/05_privileges.sql`** : remplacer d'abord `A_REMPLACER` par de vrais mots de passe.

Puis revenir dans le notebook pour la **vérification Python** (fin de la Mission 2).

### 4. Reporting
Dans le même notebook, exécuter la partie **REPORTING** (3.1 → 1.2). Les données sont lues depuis Supabase.
Les graphiques sont enregistrés en PNG (copies dans `graphiques/`).

### 5. Application web (Streamlit)
Dans Colab, en fin de notebook (Mission 3) :
1. installer les bibliothèques ;
2. téléverser `app/app.py` ;
3. saisir le mot de passe : il est enregistré dans `.streamlit/secrets.toml`, jamais dans le code ;
4. lancer l'application : un lien `https://….trycloudflare.com` s'affiche.

⚠️ Le lien ne fonctionne que tant que Colab tourne, et il change à chaque lancement.

En local :
```bash
cd app
streamlit run app.py
```
avec un fichier `app/.streamlit/secrets.toml` :
```toml
[supabase]
user = "postgres.<id_projet>"
password = "<mot de passe>"
host = "aws-0-eu-west-1.pooler.supabase.com"
port = 5432
database = "postgres"
sslmode = "require"
```

---

## 🗄️ Base de données

| Table | Clé primaire | Lignes |
|---|---|---|
| `airlines` | `carrier` | 16 |
| `airports` | `faa` | 1 462 |
| `planes` | `tailnum` | 4 037 |
| `weather` | `year, month, day, hour, origin` | 26 112 |
| `flights` | `id` + `UNIQUE (year, month, day, hour, carrier, flight)` | 252 704 |

**Relations :** `airlines` → `flights` (1-N), `planes` → `flights` (1-N), `airports` → `flights` (1-N, via `origin` et `dest`), `airports` → `weather` (1-N), `airlines` ↔ `airports` (N-N via `flights`).

**Améliorations par rapport au schéma fourni :** clé primaire pour `flights`, toutes les clés étrangères déclarées, nomenclatures contrôlées par regex (`CHECK`), types optimisés (−23 % de stockage sur `flights`).

## 🔒 Sécurité

- Connexion **chiffrée SSL** (`sslmode=require`) ;
- Mot de passe **jamais dans le code** : saisi avec `getpass`, ou stocké dans les secrets Streamlit ;
- **Row Level Security** activée sur toutes les tables ;
- **Principe du moindre privilège** : un groupe lecture (analystes), un groupe écriture (développeurs), un compte par personne ;
- Les fichiers `secrets.toml` ne doivent **jamais** être déposés sur GitHub.

## ⚠️ Limites des données

- Le fichier `flights` fourni est **incomplet** : il manque les vols du **4 juillet au 30 septembre** (252 704 vols au lieu de 336 776).
- Les vols sont datés **2021**, mais ce sont les vols de **2013**. Les analyses par jour de la semaine et par jour férié utilisent le calendrier 2013.
- La météo (`weather`) est datée 2013 : le lien vols ↔ météo se fait sur `origin, month, day, hour`, sans l'année.

## 📊 Principaux résultats

- **Newark (EWR)** est l'aéroport le plus chargé (36 % des vols) et le plus en retard.
- Les retards **s'accumulent au fil de la journée** : 1 min de retard moyen à 5 h, 24 min à 19 h.
- **La pluie double le risque de retard** (22 % → 53 %).
- **Le trafic baisse le week-end** (−15 %, samedi le plus calme) et les jours fériés (**Thanksgiving −31 %**).
- Le retard **n'augmente pas avec la distance** : les longs vols rattrapent leur retard en vol (≈ 6 min par heure).
- **Hawaiian, Alaska et Delta** sont les compagnies les plus fiables ; **ExpressJet et Frontier**, les moins fiables.
- **6 481 vols annulés** (2,6 %), avec un pic les 8-9 février 2013 (tempête de neige).
