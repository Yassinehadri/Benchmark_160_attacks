#  Synthetic Benchmark of 160 Attacks in Microservices (Online Boutique)

![Version](https://img.shields.io/badge/version-1.0-blue.svg)
![Research](https://img.shields.io/badge/Research-Loria_RESIST-darkred.svg)
![Task](https://img.shields.io/badge/Task-Root_Cause_Analysis_(RCA)-success.svg)
![Format](https://img.shields.io/badge/Format-JSON--ChatML-yellow.svg)

#  Benchmark 160 Attacks : Cyber Threat Evaluation for Microservices

Bienvenue dans le dépôt du projet **benchmark_160_attaks** (issu du projet global *Synth-OB-Cyber*). 

Ce jeu de données (Benchmark) a été spécifiquement conçu pour évaluer la capacité des Modèles de Langage (LLMs, ex: Llama 3.2) à effectuer des diagnostics de causes racines (Root Cause Analysis - RCA) et à classifier des **cyberattaques** au sein d'une architecture microservices Cloud-Native (*Online Boutique*).

---

##  1. Légitimité et Fondations Scientifiques

L'évaluation de l'IA en cybersécurité fait face à une limite majeure dans l'état de l'art actuel : la fragmentation des données.
- Les benchmarks AIOps récents (comme **RCAEval**) se limitent aux pannes opérationnelles (fuite mémoire, saturation CPU) et excluent les comportements adversariaux.
- Les benchmarks de sécurité standards sont **mono-sources** : **CIC-IDS2017** se concentre exclusivement sur les flux réseau (PCAP/Flows), tandis que **Loghub (BGL, Thunderbird)** se limite à la sémantique textuelle isolée des journaux système.

**Notre contribution :**  
Ce benchmark surmonte ces limitations en projetant les signatures d'attaques reconnues de *CIC-IDS2017* et *Loghub* dans un environnement **multi-sources hétérogène**. Pour chaque scénario, le modèle doit analyser et corréler simultanément un triptyque de télémétrie :
1. **MÉTRIQUES** (Inspirées de Prometheus : pics CPU, I/O réseau et disque).
2. **LOGS** (Inspirés de Loghub/BGL : erreurs d'authentification, requêtes malformées).
3. **TRACES** (Inspirées d'OpenTelemetry : latences en cascade, codes HTTP 401, 404, 500, 503).

---

##  2. Taxonomie des Scénarios (Les 8 Vecteurs)

Le jeu de test est constitué de **160 scénarios d'attaques** répartis de manière parfaitement équilibrée (20 cas par classe) pour éviter tout biais statistique lors du calcul du F1-Score.

| Type d'Attaque | Service Cible | Signature Multi-Source (Inspiration CIC-IDS / Loghub) |
| :--- | :--- | :--- |
| **DDoS (Volumétrique)** | `frontend` | Spike réseau massif (Métriques) + Sockets épuisés (Logs) + Erreurs 503 (Traces). |
| **Brute Force** | `checkoutservice` | Authentifications échouées répétitives (Logs) + Codes 401 (Traces). |
| **SQL Injection** | `cartservice` | Syntaxe SQL malformée (Logs) + Temps de réponse DB allongé (Traces). |
| **Cryptojacking** | `adservice` | Saturation CPU statique à 99% + Exécution suspecte `xmrig` (Logs). |
| **API Scanning** | `productcatalog` | Fuzzing de chemins et cascade d'erreurs 404 (Traces/Logs). |
| **Ransomware** | `redis-cart` | Pic d'écritures Disque I/O anormal (Métriques) + Alertes de corruption (Logs). |
| **SSRF** | `frontend` | Requête sortante vers l'IP interne des métadonnées Cloud `169.254.169.254`. |
| **Privilege Escalation**| `checkoutservice` | Modification suspecte de rôles vers `ADMIN` suivie d'un accès HTTP 200 illégitime. |

---

##  3. Structure du Benchmark (JSON Strict)

Pour garantir une lisibilité optimale par les chercheurs (humains) et une intégration facile dans les pipelines d'évaluation (machines), le fichier `benchmark_attack_eval.json` est un tableau d'objets JSON structuré ainsi :

```json
[
  {
    "nom_cas": "Cas 1 - frontend (Attaque (DDoS Volumétrique))",
    "input": "=== MÉTRIQUES ===\nservice: frontend | cpu_usage: 99.1% | network_receive_packets: 3140590 | timestamp: 1718049210.5\n=== LOGS ===\n[CRITICAL] frontend: Connection pool exhausted from IP 145.22.1.88 [WARN] frontend: Rate limit exceeded\n=== TRACES ===\nTraceID: test_ddos_8492abc | Durée totale: 14500ms api-gateway → frontend (ERROR:503)\n",
    "reponse_attendue": {
      "service_defaillant": "frontend",
      "type_panne": "Attaque (DDoS Volumétrique)",
      "raison": "Une inondation anormale de requêtes réseau a saturé le CPU et épuisé les sockets de connexion, provoquant une erreur 503."
    }
  }
]
