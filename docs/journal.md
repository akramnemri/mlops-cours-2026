# Journal de bord du binôme

<!--
Mode d'emploi (à garder en tête du fichier) :
- Une entrée par séance (et par session de travail importante hors séance), la plus récente en bas.
- 3 à 5 lignes : qui a piloté, ce qui a été fait, ce qui a bloqué ou ce qui a été appris,
  la prochaine étape. Pas de roman : c'est un outil de pilotage, pas un rapport.
- Titre d'entrée au format exact « ## S<n> — AAAA-MM-JJ — <sujet> » (date réelle) :
  le pré-contrôle de rendu (make check-m10) compte les entrées datées.
- Liez les PR, issues et ADR (#12, docs/adr/0003-...). Les problèmes non résolus restent
  visibles : c'est précisément ce qui aide l'enseignant à vous débloquer.
-->

**Binôme :** @akramnemri · Khalil Kacem (collaborateur du repo, indisponible depuis le 2026-11-17)
**Repo :** https://github.com/akramnemri/mlops-cours-2026
**Dataset :** german-credit (fil rouge, `dataset.yaml` du cours, non modifié)
**Extensions :** à déclarer au plus tard en fin de S5

## S1 — 2026-11-16 — Lab 0 : setup

- **Pilote / copilote :** @akramnemri / — (travail seul ; Khalil rejoint le binôme le 2026-11-16)
- **Fait :** cluster kind `mlops-lab` + registry, SeaweedFS, MLflow, smoke tests MLflow et DVC ;
  `make check-m0` 19/19 ; PR #1 fusionnée.
- **Bloqué / appris :** pas de VM Vagrant (disque plein, Hyper-V requis par d'autres projets) :
  WSL2 Ubuntu + Docker Desktop. `aws-lab` adapté (`host.docker.internal:8333`, sans
  `--network host`).
- **Prochaine étape :** Lab 1 (versioning).

## S2 — 2026-11-17 — Labs 1 et 2 : versioning, pipelines KFP et Airflow

- **Pilote / copilote :** @akramnemri / — (Khalil indisponible : auto-relecture des PR #2 et #3)
- **Fait :** Lab 1 (stage DVC `download`, 22 runs MLflow, registry `@champion`, pipeline DVC,
  Feast) et Lab 2 (image projet, pipeline KFP 6 étapes, DAG Airflow, rapport comparatif) ;
  `make check-m1` 18/18 et `make check-m2` 18/18 sur `main`.
- **Bloqué / appris :** WSL passé à 12 Go ; KFP et Airflow ensemble saturent le plan de
  contrôle kind (kube-scheduler en CrashLoopBackOff) : un seul outil lourd à la fois.
- **Prochaine étape :** tag `j0`, signaler l'indisponibilité de Khalil à l'enseignant, puis Lab 3.
