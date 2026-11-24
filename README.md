<div align="center">

# 🧠 Atlas Enterprise AI

**Assistant de connaissances d'entreprise avec RAG, LangGraph, AWS Bedrock et déploiement automatisé sur AWS**

*Intelligence documentaire · Mémoire conversationnelle · Guardrails · Revue humaine (HITL) · Observabilité · CI/CD*

![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![LangGraph](https://img.shields.io/badge/LangGraph-Orchestration-1C3C3C?style=for-the-badge)
![AWS Bedrock](https://img.shields.io/badge/AWS-Bedrock_%7C_S3_%7C_EC2_%7C_ECR-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Pinecone](https://img.shields.io/badge/Pinecone-Vector_DB-000000?style=for-the-badge)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

[Vue d'ensemble](#-vue-densemble) • [Fonctionnalités](#-fonctionnalités-clés) • [Stack Technique](#-stack-technique) • [Architecture](#-architecture) • [Structure](#-structure-du-dépôt) • [Prérequis](#-prérequis)

</div>

---

## 📖 Vue d'ensemble

**Atlas Enterprise AI** est un assistant de connaissances d'entreprise full-stack conçu pour répondre en langage naturel aux questions posées sur des documents internes grâce à la **génération augmentée par récupération (RAG)**. Il associe un backend **FastAPI**, une orchestration **LangGraph**, les modèles **Amazon Bedrock**, la recherche vectorielle **Pinecone** et un historique **PostgreSQL** à une interface web légère (`HTML/CSS/JS`).

Ce projet illustre comment réunir au sein d'une même application IA :
* L'ingestion sécurisée de documents et les réponses sourcées.
* Les contrôles de sécurité (**Guardrails**) et la validation humaine (**Human-in-the-Loop**).
* L'audit opérationnel, l'observabilité et un pipeline **CI/CD** complet vers AWS.

> [!WARNING]
> **Statut du projet : Implémentation pédagogique / de référence.**
> Ce dépôt n'est pas sécurisé pour la production en l'état : le contexte d'identité de démonstration ne constitue pas un véritable système d'authentification ou d'autorisation, les décisions d'approbation n'exécutent pas automatiquement d'opérations critiques, et les mécanismes de haute disponibilité ou de rollback doivent être ajoutés séparément.

---

## ✨ Fonctionnalités clés

| Fonctionnalité | Description |
| :--- | :--- |
| 📄 **Ingestion de documents** | Téléversement direct depuis le navigateur vers un bucket **Amazon S3** privé via des URL présignées, suivi d'un traitement et d'une indexation par le backend. |
| 🔍 **RAG d'entreprise** | Récupération des extraits pertinents depuis **Pinecone** et génération de réponses ancrées dans les sources avec **Amazon Bedrock**. |
| 🕸️ **Workflow LangGraph** | Orchestration des étapes de recherche, de génération, de vérification par guardrails et de gestion du contexte conversationnel. |
| 🧠 **Mémoire conversationnelle** | Persistance des échanges dans **PostgreSQL** et réutilisation de l'historique récent pour les questions de suivi. |
| 🛡️ **Guardrails** | Application de règles déterministes de contrôle en entrée et en sortie pour sécuriser les interactions. |
| 🧑‍⚖️ **Approbations HITL** | Mise en file d'attente des requêtes sensibles nécessitant une revue humaine et enregistrement des décisions. |
| 📊 **Audit et feedback** | Journalisation des événements applicatifs et collecte des retours utilisateurs dans **PostgreSQL**. |
| 🔭 **Observabilité** | Instrumentation optionnelle via **GlassFlow Rius** pour le traçage et le diagnostic des exécutions IA. |
| 🖥️ **Dashboard Web** | Interface unifiée pour l'upload de documents, le chat, l'historique, les approbations et le suivi opérationnel. |
| 🚀 **Livraison automatisée** | Construction et publication d'images Docker versionnées sur **ECR**, puis déploiement sur **EC2** via **GitHub Actions**. |

---

## 🛠️ Stack Technique

<details open>
<summary><b>Cliquez pour réduire / afficher le détail des couches techniques</b></summary>
<br>

| Couche | Technologie | Rôle |
| :--- | :--- | :--- |
| **Backend** | Python, FastAPI, Uvicorn | APIs REST et serveur applicatif |
| **Frontend** | HTML, CSS, JavaScript | Interface utilisateur web légère |
| **Orchestration** | LangGraph | Workflow RAG avec gestion d'état |
| **LLM** | Amazon Bedrock — *Nova Lite* | Génération des réponses |
| **Embeddings** | Amazon Bedrock — *Titan Text Embeddings V2* | Vectorisation des documents et requêtes (1 024 dimensions) |
| **Base Vectorielle** | Pinecone | Indexation des chunks et recherche sémantique |
| **Stockage Objet** | Amazon S3 | Stockage des documents bruts téléversés |
| **Base Relationnelle** | PostgreSQL (sur Render) | Métadonnées, conversations, approbations, feedback et audit |
| **Observabilité** | GlassFlow Rius *(optionnel)* | Traçage et supervision des workflows IA |
| **Conteneurisation** | Docker | Environnement d'exécution portable |
| **Registre d'images** | Amazon ECR | Stockage privé des images Docker |
| **Calcul (Compute)** | Amazon EC2 | Hébergement du runner auto-hébergé et du conteneur applicatif |
| **CI/CD** | GitHub Actions | Automatisation des tests, du build et du déploiement |

</details>

---

## 🏗️ Architecture

### 1. Architecture Applicative
* **Flux d'ingestion :** Le client téléverse directement le fichier sur **S3** via une URL présignée, puis déclenche l'extraction, le découpage (*chunking*), la vectorisation (**Titan V2**) et l'indexation (**Pinecone**).
* **Flux de requête :** La question passe par les **Guardrails**, interroge **Pinecone** et l'historique **PostgreSQL**, puis génère une réponse via **Bedrock Nova Lite** (ou bascule en revue humaine **HITL** si nécessaire).

> [!NOTE]
> **Note sur l'ingestion :** Dans cette implémentation de référence, la fin du téléversement déclenche l'ingestion de manière synchrone. Pour des documents volumineux et un trafic de production, il est recommandé de déléguer cette tâche à un worker asynchrone dédié.

### 2. Architecture de Déploiement CI/CD (AWS)
1. **CI (Intégration Continue) :** Un commit sur la branche `main` déclenche **GitHub Actions** qui exécute les vérifications configurées.
2. **Livraison (Build & Push) :** L'image Docker est construite, taguée avec le **SHA du commit Git**, puis poussée vers **Amazon ECR**.
3. **Déploiement (CD) :** Un job s'exécute sur le **runner auto-hébergé EC2**, récupère la nouvelle image depuis ECR et remplace le conteneur actif. *(L'instance EC2 doit disposer d'un accès sortant vers les services externes).*

---

## 📂 Structure du dépôt

```text
atlasEnrepriseAI/
├── app/
│   ├── main.py              # Routes FastAPI et service de l'interface web
│   ├── graph.py             # Définition du workflow LangGraph
│   ├── rag.py               # Logique d'intégration RAG
│   ├── vector.py            # Opérations sur l'index Pinecone
│   ├── aws.py               # Clients AWS (S3, Bedrock)
│   ├── db.py                # Accès PostgreSQL et journalisation d'audit
│   ├── memory.py            # Persistance de l'historique conversationnel
│   ├── guardrails.py        # Contrôles de sécurité en entrée et sortie
│   ├── observability.py     # Initialisation de GlassFlow Rius
│   ├── config.py            # Configuration et variables d'environnement
│   ├── templates/           # Templates HTML
│   └── static/              # Feuilles de style CSS et scripts JS
├── worker/
│   └── ingest.py            # Traitement des fichiers et indexation vectorielle
├── infra/
│   └── schema.sql           # Schéma de la base de données PostgreSQL
├── scripts/
│   └── init_db.py           # Script d'initialisation de la base
├── docs/                    # Guides de configuration et notes opérationnelles
├── .github/workflows/       # Pipelines CI/CD GitHub Actions
├── Dockerfile               # Définition de l'image Docker
├── requirements.txt         # Dépendances Python
└── .env.example             # Exemple de configuration d'environnement


📋 Prérequis
Avant de lancer le projet en local ou de configurer le pipeline de déploiement, assurez-vous de disposer des éléments suivants :

Environnement local :

Python 3.11+ et pip

Docker Desktop ou Docker Engine (pour tester le conteneur)

Services Cloud & IA :

Un compte AWS avec accès à Amazon S3 et aux modèles Amazon Bedrock (Nova Lite et Titan Text Embeddings V2)

Un compte Pinecone et une clé API valide

Une base de données PostgreSQL (hébergée sur Render dans ce projet)

(Optionnel) Un compte GlassFlow Rius pour l'observabilité

Infrastructure CI/CD (pour le déploiement) :

Un dépôt Amazon ECR et une instance Linux Amazon EC2

Un dépôt GitHub configuré avec un runner auto-hébergé enregistré sur l'instance EC2
