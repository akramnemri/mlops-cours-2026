# Rapport comparatif — Kubeflow Pipelines vs Airflow (Lab 2)

**Binôme :** akramnemri (pilote) · Khalil Kacem (copilote) · **Date :** 2026-11-17 ·
**Machine :** Intel i7-12650H, 16 Go (12 Go alloués à WSL2), Windows 11 + WSL2 Ubuntu 24.04 +
Docker Desktop (pas la VM Vagrant) · **Dataset :** `german-credit`

## 1. Contexte

Le pipeline du Lab 1 (download → validate → prepare → train → evaluate → register, 6 steps) a
été porté sur deux orchestrateurs. Les deux exécutent **la même image projet**
(`localhost:5001/scoring-pipeline:1.0.0`, code `src/` + `dataset.yaml`) dans le cluster kind :
un step = un pod. KFP passe les fichiers en artefacts typés via son SeaweedFS interne ; le DAG
Airflow (Docker Compose, KubernetesPodOperator) recopie l'arborescence du run sur le S3 du cours
et ne passe que de petites valeurs en XCom. Les deux écrivent dans la même expérience MLflow
`scoring` et produisent le même modèle (roc_auc 0,8094, coût 108).

## 2. Résultats quantitatifs

| Mesure | KFP | Airflow |
|---|---|---|
| Installation de zéro (min) | 23 (dont une panne DNS sur ghcr.io) | 10 |
| Temps jusqu'au premier run réussi (min) | 15 | 11 (1er run en échec, voir Debugging) |
| Durée d'un run complet (min, UI) | 4,8 (1er run, pull de l'image) | 1,1 |
| Relance avec les mêmes paramètres (min, UI) | 2,7 (3 steps servis par le cache) | 1,1 (pas de cache : tout relancé) |
| Hot-reload : modifier un step et voir le résultat (min) | ≈ 3, estimé (recompiler + uploader + run) | ≈ 1,5, estimé (scan de `dags/` + run) |
| Lignes de code propres à l'orchestrateur | 282 (`components.py` + `scoring_pipeline.py`) | 70 (`scoring_dag.py`) + la partie S3/XCom de `stage.py` |
| Mémoire consommée par l'outil (Go, `docker stats`) | ≈ 3, estimé (nœuds kind : 4,1 Go dont ≈ 1 Go pour MLflow et SeaweedFS) | 2,0 (7 conteneurs) |

Dans les deux cas, modifier `src/` impose une nouvelle image avec un nouveau tag.

## 3. Analyse qualitative

### Developer experience

KFP demande plus de code (décorateurs, `Input`/`Output` typés), mais ce code apporte beaucoup :
les artefacts circulent seuls, le lineage est visible en un clic, et `kfp.local` teste tout le
pipeline en 30 s sans cluster. Le DAG Airflow est plus court et plus familier, mais il faut
gérer soi-même le transport des fichiers (préfixe S3 par run) : c'est `stage.py` qui porte cette
complexité.

### Scaling

Les deux lancent un pod par step et donc profitent du cluster. KFP fixe les ressources, les
retries et les timeouts au niveau du step (`set_memory_limit`, `set_retry`, `set_timeout`) et
cache les étapes data, ce qui réduit fortement la relance. Airflow règle les ressources par
`container_resources` du KPO, mais sans cache tout est recalculé ; sa propre infrastructure
(scheduler, worker, Postgres, Redis) tourne hors du cluster.

### Debugging

Les deux UI donnent les logs d'un step. Le seul vrai incident l'a montré : avec KFP et Airflow
lancés ensemble, le plan de contrôle kind a saturé (kube-scheduler en CrashLoopBackOff) et le
pod `evaluate` est resté `Pending` ; Airflow a seulement affiché « Waiting up to 120s to get the
POD scheduled », la cause était dans `kubectl`. Une fois KFP mis en pause, le DAG est passé en
65 s.

## 4. Recommandation

KFP pour un projet ML qui itère sur les modèles : artefacts typés, lineage, cache et intégration
Kubernetes valent le coût d'installation, à condition d'avoir un cluster dimensionné. Airflow
quand l'équipe l'utilise déjà pour ses pipelines de données, ou quand le ML n'est qu'une étape
d'un flux plus large ; il faut alors accepter de gérer le stockage intermédiaire et l'absence de
cache. Pour ce projet, nous gardons KFP.

## 5. Limites de la comparaison

Une seule machine (portable, kind sous WSL2), un ou deux runs par outil : les durées sont
indicatives et les temps de hot-reload et la mémoire de KFP sont estimés, pas mesurés. Pas de
vrai cluster (GKE), pas de KubernetesExecutor ni de Helm pour Airflow, pas de mode
multi-utilisateur pour KFP, et les deux outils n'ont jamais pu tourner ensemble sur cette machine.
