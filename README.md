# 🍺 Brewery Microservices Project

## 📖 Introduction

Ce projet a été conçu dans un objectif **d’apprentissage** afin de découvrir :

* l’architecture **microservices** avec **Spring Boot** en Java,
* la mise en place d’une **API Gateway** et d’une communication via **JMS (ActiveMQ Artemis)**,
* l’intégration d’une base de données relationnelle (**MySQL**),
* la supervision avec la stack **ELK (Elasticsearch, Kibana, Filebeat)**,
* et la préparation au **déploiement dans Kubernetes** via des conteneurs Docker.

Il s’agit d’un **projet de démonstration** (proof of concept) qui permet de manipuler les concepts fondamentaux des microservices et de mieux comprendre comment les applications Java modernes peuvent être développées, orchestrées et observées dans un environnement distribué.


---

## 🚀 Services inclus

### 🔹 Base & Messaging

* **MySQL (5.7)**
  Base de données relationnelle, utilisée par les services métier (`beer-service`, `order-service`, `inventory-service`).

* **ActiveMQ Artemis (JMS)**
  Broker de messages pour la communication asynchrone entre services.

### 🔹 Observabilité

* **Elasticsearch (7.12.1)**
  Stockage et indexation des logs.

* **Kibana (7.12.1)**
  Interface graphique pour explorer et visualiser les logs.

* **Filebeat (7.12.1)**
  Collecte les logs Docker et les envoie vers Elasticsearch.

### 🔹 Microservices métier

* **Beer Service (8080)**
  Gère le catalogue de bières.
  Connecté à la DB et communique avec l’**Inventory Service**.

* **Inventory Service (8082)**
  Gère les stocks de bières (quantités disponibles).
  Communique via JMS et la base de données.

* **Inventory Failover (8083)**
  Service de secours en cas de panne d’`inventory-service`.

* **Order Service (8081)**
  Gère les commandes clients.
  Communique avec le `beer-service` et JMS.

* **API Gateway (9090)**
  Point d’entrée unique pour les clients.
  Route les requêtes vers les services appropriés et peut basculer vers `inventory-failover`.

---

## 📊 Architecture

```text
          [ Client ]
              │
              ▼
       ┌──────────────┐
       │   Gateway    │ (9090)
       └───────┬──────┘
               │
  ┌────────────┼───────────────┐
  │            │               │
  ▼            ▼               ▼
Beer Service   Order Service   Inventory Service
 (8080)        (8081)          (8082)
   │              │                │
   │              │                │
   │              │                ▼
   │              │          Inventory Failover (8083)
   │              │
   ▼              ▼
MySQL (3306)   ActiveMQ JMS (61616 / 8161)

Logs → Filebeat → Elasticsearch (9200) → Kibana (5601)
```

---

## ⚙️ Prérequis

* [Docker](https://www.docker.com/get-started)
* [Docker Compose](https://docs.docker.com/compose/)

---

## ▶️ Lancer le projet

Clonez le repo puis exécutez :

```bash
docker compose up -d
```

Les services seront accessibles sur les ports suivants :

| Service            | Port hôte      |
| ------------------ | -------------- |
| Gateway            | 9090           |
| Beer Service       | 8080           |
| Order Service      | 8081           |
| Inventory Service  | 8082           |
| Inventory Failover | 8083           |
| MySQL              | 3306 (interne) |
| JMS (console web)  | 8161           |
| Elasticsearch      | 9200           |
| Kibana             | 5601           |

---

## 📂 Logs & Observabilité

* Accès Kibana : [http://localhost:5601](http://localhost:5601)
* Elasticsearch API : [http://localhost:9200](http://localhost:9200)
* Console JMS Artemis : [http://localhost:8161](http://localhost:8161)

Les logs de chaque service sont collectés via **Filebeat**, envoyés dans **Elasticsearch** et visualisables dans **Kibana**.

---

## 🔒 Notes & bonnes pratiques

* ⚠️ Les identifiants DB (`root/dbpassword`) sont en clair → à remplacer par des **secrets** pour un usage en production.
* `SPRING_JPA_HIBERNATE_DDL-AUTO=update` est pratique en dev, mais il est recommandé d’utiliser **Flyway** ou **Liquibase** en prod.
* Les services sont configurés pour redémarrer automatiquement (`restart: on-failure`).
* MySQL n’est pas exposé vers l’extérieur (sécurité).

