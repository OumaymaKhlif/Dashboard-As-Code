<div align="center">

![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Apache Airflow](https://img.shields.io/badge/Apache%20Airflow-017CEE?style=for-the-badge&logo=apacheairflow&logoColor=white)
![dbt](https://img.shields.io/badge/dbt-FF694B?style=for-the-badge&logo=dbt&logoColor=white)
![Trino](https://img.shields.io/badge/Trino-DD00A1?style=for-the-badge&logo=trino&logoColor=white)
![Apache Hive](https://img.shields.io/badge/Hive%20Metastore-FDEE21?style=for-the-badge&logo=apachehive&logoColor=black)
![MinIO](https://img.shields.io/badge/MinIO-C72E49?style=for-the-badge&logo=minio&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Apache Superset](https://img.shields.io/badge/Apache%20Superset-20A6C9?style=for-the-badge&logo=apachesuperset&logoColor=white)
![Plotly Dash](https://img.shields.io/badge/Plotly%20Dash-3F4F75?style=for-the-badge&logo=plotly&logoColor=white)
![OpenMetadata](https://img.shields.io/badge/OpenMetadata-7147E8?style=for-the-badge)
![GitLab CI](https://img.shields.io/badge/GitLab%20CI%2FCD-FC6D26?style=for-the-badge&logo=gitlab&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![Mockoon](https://img.shields.io/badge/Mockoon-API%20mock-1F2937?style=for-the-badge)

# 📊 Dashboard as Code

### Une plateforme data open-source de bout en bout, en architecture médaillon : de l'ingestion multi-sources à la visualisation, entièrement conteneurisée et pilotée par le code.

</div>

---

## 🎯 À propos

**Dashboard as Code** est une plateforme de données complète construite lors de mon stage d'été. Elle ingère des données issues de sources hétérogènes (API REST, SFTP, partage SMB, fichiers CSV), les stocke dans un data lake, les transforme et les qualifie, puis les expose dans des tableaux de bord **créés et mis à jour par du code** plutôt que par des clics.

Le jeu de données utilisé est le dataset e-commerce public **Olist** (commandes, articles, clients, vendeurs, paiements, avis, géolocalisation).

**Ce que le projet démontre :**

- une architecture **médaillon** (Bronze → Silver → Gold) sur une stack 100 % open-source ;
- l'**orchestration** de bout en bout avec Airflow (déclenchement entre DAGs via les *Assets*) ;
- la **transformation et la qualité** des données avec dbt (déduplication, typage, KPIs, historisation SCD2) ;
- des **dashboards as code** : base, datasets, graphiques et tableau de bord créés via l'API REST de Superset, avec un script idempotent ;
- la **gouvernance** avec OpenMetadata (catalogue, lineage, profiling, tests de qualité) ;
- le **déploiement automatisé** des DAGs avec GitLab CI/CD.

---

## 🏗️ Architecture

<div align="center">

![Architecture – vue en couches](docs/architecture_1_couches.png)

![Architecture – flux médaillon](docs/architecture_2_flux_medaillon.png)

</div>

| Couche | Outil | Rôle |
|---|---|---|
| Sources | Mockoon, SFTP, SMB, CSV | Simulent des systèmes sources hétérogènes |
| **Bronze** | MinIO | Données brutes, partitionnées par date (`source/table/AAAA/MM/JJ`), avec une colonne `_ingestion_date` |
| **Silver** | Trino + Hive Metastore + dbt + PostgreSQL | Tables externes sur MinIO, nettoyage, déduplication, typage, KPIs, historisation SCD2 |
| **Gold / Serving** | Superset, Dash (Plotly) | Datasets métier et dashboards |
| Orchestration | Apache Airflow | `bronze_ingestion` (cron) puis `silver_transformation` (déclenché par Asset) |
| Gouvernance | OpenMetadata | Catalogue, lineage, profiling, tests de qualité |
| Infrastructure | Docker Compose, GitLab CI/CD | Conteneurisation et déploiement des DAGs |

### Flux de données

1. **Ingestion** : le DAG `bronze_ingestion` collecte les 4 sources et dépose les fichiers bruts dans le bucket `bronze` de MinIO.
2. **Exposition SQL** : le DAG `silver_transformation` déclare des tables externes Hive (via Trino) au-dessus des fichiers MinIO.
3. **Transformation** : dbt construit `stg_orders` (grain commande × article) dans PostgreSQL, puis la table d'historisation SCD2.
4. **Gouvernance** : OpenMetadata ingère les métadonnées de Trino, PostgreSQL et dbt.
5. **Visualisation** : Superset (dashboard créé par script) et l'application Dash lisent les données du schéma `silver`.

---

## 📁 Structure du dépôt

```text
.
├── airflow/dags/            # DAGs : ingestion Bronze et transformation Silver
├── dbt/                     # Projet dbt (sources, modèles, snapshot)
├── trino/catalog/           # Connecteurs Trino (Hive/MinIO et PostgreSQL)
├── superset/                # Image Superset + dashboard as code (API REST)
├── dashboard/               # Application Dash/Plotly (KPIs métier)
├── openmetadata/ingestion/  # Workflows d'ingestion, profiling et tests de qualité
├── sources/                 # Sources simulées : Mockoon, SFTP, SMB, CSV
├── fastapi/                 # API de santé minimale
├── docs/                    # Schémas d'architecture
├── Dockerfile.airflow       # Image Airflow personnalisée
├── docker-compose.yml       # Orchestration de tous les services
└── .gitlab-ci.yml           # Pipeline CI/CD
```

---

## 🚀 Démarrage rapide

### Prérequis

- Docker et Docker Compose
- 8 Go de RAM minimum (OpenSearch et OpenMetadata sont gourmands)
- Le dataset [Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) (non versionné pour des raisons de taille)

### Installation

```bash
# 1. Cloner le dépôt
git clone https://github.com/<votre-utilisateur>/<nom-du-depot>.git
cd <nom-du-depot>

# 2. Configurer les secrets
cp .env.example .env        # puis adapter les valeurs

# 3. Placer les CSV Olist dans les dossiers de sources/
#    sources/csv, sources/sftp, sources/smb, sources/mockoon/csv

# 4. Lancer la plateforme
docker compose up -d --build
```

Créer ensuite les buckets `bronze` et `warehouse` dans la console MinIO, puis activer les DAGs depuis l'interface Airflow.

### Interfaces

| Service | URL |
|---|---|
| Airflow | http://localhost:8080 |
| Superset | http://localhost:8088 |
| Application Dash | http://localhost:8050 |
| MinIO (console) | http://localhost:9001 |
| Trino | http://localhost:8081 |
| OpenMetadata | http://localhost:8585 |
| Mockoon (API simulée) | http://localhost:3000 |

### Dashboard as Code

```bash
python superset/assets/dashboards/superset_dashboard_as_code.py
```

Le script se connecte à l'API Superset, retrouve chaque objet par son nom, le met à jour ou le crée : il peut donc être relancé sans créer de doublons.

---

## 🔄 CI/CD

Le pipeline GitLab (`.gitlab-ci.yml`) déploie automatiquement les DAGs Airflow vers le serveur cible à chaque poussée sur `main`, via `rsync` sur SSH. Le `dag-processor` d'Airflow détecte ensuite les changements sans redémarrage.

---

## 🛣️ Pistes d'amélioration

- Ajouter une vraie couche **Gold** en modèles dbt (tables d'agrégats métier).
- Ajouter des **tests dbt** (`unique`, `not_null`, `accepted_values`) en complément des tests OpenMetadata.
- Passer les modèles dbt en **incrémental** et ajouter un déploiement dbt dans la CI.
- Remplacer l'authentification en mémoire du dashboard Dash par un vrai fournisseur d'identité.

---

## 👤 Auteur

**<Votre nom>**, projet réalisé lors d'un stage d'été en ingénierie des données.

[LinkedIn](https://www.linkedin.com/in/<votre-profil>) · [GitHub](https://github.com/<votre-utilisateur>)
