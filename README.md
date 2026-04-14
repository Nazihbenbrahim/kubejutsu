# ☸️ kubejutsu
 
<p align="center">
  <img src="./assets/kubejutsu-banner.png" alt="kubejutsu banner" width="100%" />
</p>
 
<br/>
 
<p align="center">
  <em>"I never go back on my word — that's my nindō, my ninja way."</em><br/>
  <strong>— Uzumaki Naruto</strong>
</p>
 
<br/>
 
<p align="center">
  Every lab committed. Every crash investigated. Every manifest written by hand.<br/>
  This is not a collection of YAML. This is a shinobi's training log.
</p>
 
<br/>
 
<p align="center">
  <img alt="Kubernetes" src="https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=white&style=for-the-badge" />
  <img alt="Status" src="https://img.shields.io/badge/status-in%20training-orange?style=for-the-badge" />
  <img alt="GitHub last commit" src="https://img.shields.io/github/last-commit/Nazihbenbrahim/kubejutsu?style=for-the-badge" />
  <img alt="GitHub repo size" src="https://img.shields.io/github/repo-size/Nazihbenbrahim/kubejutsu?style=for-the-badge" />
</p>
 
---
 
## 🥷 The Nindō — My Ninja Way
 
Naruto had a dream nobody believed in.
He trained anyway. He failed publicly. He got back up. He became Hokage.
 
This repository is built on the same principle applied to engineering:
 
> **There are no shortcuts to mastery. There is only the work, repeated with intention, until the hands know what the mind once had to think about.**
 
A shinobi does not copy another shinobi's scroll and call it training.
A shinobi performs the jutsu. Misses. Understands why. Performs it again.
 
That is the philosophy behind every lab in this repository:
- No solution is copy-pasted blindly
- No error is skipped without being understood
- No lab is closed until `kubectl` confirms it works
- Every mistake earns its place in `notes.md`
 
The path from Academy student to Hokage is not a straight line.
It runs through crashes, `CrashLoopBackOff`, OOMKills, and `pending` pods at 2am.
That is the point.
 
---
 
## ⚡ Why This Repository Exists
 
**kubejutsu** is not a reference library. It is not a cheat sheet.
 
It is a **training archive** — a log of every technique practiced, every bug dissected, every concept that finally clicked after the third attempt.
 
The Kubernetes ecosystem rewards those who understand it deeply.
Not those who memorize `kubectl` flags, but those who can look at a broken cluster and *read* it — the way a seasoned shinobi reads a battlefield.
 
That level of reading is earned through **repetition with reflection**, not passive consumption.
 
This repository documents that journey:
- From `lab-001-pod-basics` to a full CKA sprint
- From copying flags to writing manifests from memory
- From confusion about `matchLabels` to instinctively knowing why a selector breaks
 
Every commit here is a promise kept to the path.
 
---
 
## 🎖️ The Rank System
 
In the Hidden Leaf Village, shinobi progress through ranks earned in the field — not in classrooms.
 
| Shinobi Rank | Kubernetes Equivalent | Labs |
|---|---|---|
| **Academy Student** | Pod basics, manifest syntax, `kubectl` fundamentals | 001–020 |
| **Genin** | Deployments, ConfigMaps, Secrets, Storage | 021–045 |
| **Chūnin** | Services, NetworkPolicy, RBAC, Security contexts | 046–065 |
| **Jōnin** | HPA, KEDA, Helm, Scheduling, Observability | 066–088 |
| **ANBU** | Chaos debugging, real-world apps, advanced troubleshooting | 089–110 |
| **Hokage 🏆** | CKA Sprint — full cluster mastery under exam conditions | 111–120 |
 
> The CKA is not the destination. It is the formal recognition of a path already walked.
 
---
 
## 🌀 Jutsu — What Each Technique Seals
 
In Naruto, a jutsu is a technique that channels inner energy into a precise, repeatable form.
A YAML manifest is no different — it is a **written seal** that shapes the cluster's behavior with total precision.
 
Master the seal. Master the technique.
 
| Jutsu (Kubernetes Concept) | Shinobi Parallel |
|---|---|
| **Pods** | The body — the fundamental unit that carries chakra (the workload) |
| **Deployments + ReplicaSets** | **Shadow Clone Jutsu** — one intent, many simultaneous forms |
| **HPA / KEDA** | **Sage Mode** — the cluster senses load and multiplies itself |
| **ConfigMaps + Secrets** | **Fūinjutsu (Sealing Jutsu)** — knowledge and power locked in scrolls |
| **RBAC** | **ANBU clearance** — least privilege, controlled access, identity-gated actions |
| **NetworkPolicy** | **Barrier Jutsu** — define exactly who can speak to whom, and who cannot |
| **Helm** | **Summoning Jutsu** — call a complete workload from a contract (the chart) |
| **Observability (Prometheus, Grafana, Loki)** | **The Sharingan** — see everything, trace any event, miss nothing |
| **StatefulSets** | **Bijuu (Tailed Beasts)** — immense power, requires permanent identity and persistent storage |
| **Debugging** | **Reading the enemy** — CrashLoopBackOff, OOMKill, Pending — every failure has a message |
| **CKA Sprint** | **The Chunin Exam** — real conditions, time pressure, no help, full exposure |
 
---
 
## 📜 Mission Log
 
> Every lab is a mission. Missions have ranks. Missions are not skipped.
 
### 01 — Pods `[Academy → Genin]`
Folder: [01-pods](./01-pods/)
 
- [Lab 001 — Pod basics — manifest, exec, logs, delete _(KodeKloud Free)_](./01-pods/lab-001-pod-basics/)
- [Lab 002 — Liveness + Readiness + Startup probes _(Killercoda omkar)_](./01-pods/lab-002-liveness-readiness-startup-probes/)
- [Lab 003 — QoS Classes — Guaranteed / Burstable / BestEffort _(Killercoda omkar)_](./01-pods/lab-003-qos-classes/)
- [Lab 004 — Resource limits + OOMKill — exit code 137 _(Killercoda omkar)_](./01-pods/lab-004-resource-limits-oomkill/)
- [Lab 005 — Init containers — sequential, wait-for-service _(Killercoda omkar)_](./01-pods/lab-005-init-containers/)
- [Lab 006 — Sidecar containers — shared emptyDir log pattern _(Killercoda omkar)_](./01-pods/lab-006-sidecar-containers/)
- [Lab 007 — Security context — runAsNonRoot + readOnly + capabilities _(Killercoda omkar)_](./01-pods/lab-007-security-context/)
- [Lab 008 — Environment variables — inject into pod _(KodeKloud Engineer)_](./01-pods/lab-008-environment-variables/)
- [Lab 009 — CKA Challenge 14 — Pod deployment timed _(Killercoda CKA)_](./01-pods/lab-009-cka-challenge-14/)
- [Lab 010 — SadServers Buenos Aires — CrashLoop + RBAC debug _(SadServers)_](./01-pods/lab-010-sadservers-buenos-aires/)
 
### 02 — Deployments `[Genin]`
Folder: [02-deployments](./02-deployments/)
 
- [Lab 011 — Deployment basics — scale, labels, matchLabels _(KodeKloud Free)_](./02-deployments/lab-011-deployment-basics/)
- [Lab 012 — Rolling update — maxSurge + maxUnavailable deep _(KodeKloud Free)_](./02-deployments/lab-012-rolling-update/)
- [Lab 013 — Pause + resume rollout — verify 50/50 in-flight _(Killercoda omkar)_](./02-deployments/lab-013-pause-resume-rollout/)
- [Lab 014 — Blue-Green — zero-downtime service selector swap _(Killercoda omkar)_](./02-deployments/lab-014-blue-green/)
- [Lab 015 — Canary — 20% weighted traffic split _(Killercoda omkar)_](./02-deployments/lab-015-canary/)
- [Lab 016 — Update Deployment in-place — image + rollout history _(Killercoda omkar)_](./02-deployments/lab-016-update-deployment/)
- [Lab 017 — Rollout rollback — undo to specific revision _(Killercoda omkar)_](./02-deployments/lab-017-rollout-rollback/)
- [Lab 018 — Troubleshoot broken Deployment _(KodeKloud Engineer)_](./02-deployments/lab-018-troubleshoot-broken-deployment/)
- [Lab 019 — CKA Challenge 07 — Deployment troubleshooting _(Killercoda CKA)_](./02-deployments/lab-019-cka-challenge-07/)
 
### 03 — ConfigMaps + Secrets `[Genin — Fūinjutsu]`
Folder: [03-configmaps-secrets](./03-configmaps-secrets/)
 
### 04 — Storage `[Genin → Chūnin]`
Folder: [04-storage](./04-storage/)
 
### 05 — Services + Ingress `[Chūnin]`
Folder: [05-services-ingress](./05-services-ingress/)
 
### 06 — NetworkPolicy `[Chūnin — Barrier Jutsu]`
Folder: [06-networkpolicy](./06-networkpolicy/)
 
### 07 — RBAC + Security `[Chūnin — ANBU Clearance]`
Folder: [07-rbac-security](./07-rbac-security/)
 
- [Lab 051 — RBAC Role + RoleBinding — namespace-scoped access _(Killercoda omkar)_](./07-rbac-security/lab-051-rbac-role-rolebinding/)
- [Lab 052 — RBAC ClusterRole — cross-namespace permissions _(Killercoda omkar)_](./07-rbac-security/lab-052-rbac-clusterrole/)
- [Lab 053 — ServiceAccount + token — Pod identity _(Killercoda omkar)_](./07-rbac-security/lab-053-serviceaccount-token/)
- [Lab 054 — can-i audit — who can do what _(Killercoda omkar)_](./07-rbac-security/lab-054-can-i-audit/)
- [Lab 055 — Namespace isolation — RBAC + NetworkPolicy combined _(Killercoda omkar)_](./07-rbac-security/lab-055-namespace-isolation/)
- [Lab 056 — Fix broken RBAC — pod can't list secrets _(Killercoda omkar)_](./07-rbac-security/lab-056-fix-broken-rbac/)
- [Lab 057 — TLS Secret — cert + key in a Secret _(Killercoda omkar)_](./07-rbac-security/lab-057-tls-secret/)
- [Lab 058 — Fix ServiceAccount — Payment API can't authenticate _(Killercoda omkar)_](./07-rbac-security/lab-058-fix-serviceaccount/)
- [Lab 059 — Pod Security — runAsNonRoot + readOnly + emptyDir _(Killercoda omkar)_](./07-rbac-security/lab-059-pod-security/)
- [Lab 060 — CKA Challenge 05 — ServiceAccount + RBAC _(Killercoda CKA)_](./07-rbac-security/lab-060-cka-challenge-05/)
- [Lab 061 — CKA Challenge 20 — RBAC can-i verification _(Killercoda CKA)_](./07-rbac-security/lab-061-cka-challenge-20/)
 
### 08 — HPA + KEDA + Autoscaling `[Jōnin — Sage Mode]`
Folder: [08-hpa-keda-autoscaling](./08-hpa-keda-autoscaling/)
 
- [Lab 062 — HPA — scale under CPU load _(CKA Challenge 22)_](./08-hpa-keda-autoscaling/lab-062-hpa-scale-under-cpu-load/)
- [Lab 063 — HPA — remove requests → unknown metric _(kind cluster)_](./08-hpa-keda-autoscaling/lab-063-hpa-unknown-metric/)
- [Lab 064 — KEDA — scale-to-zero Redis queue _(kind cluster)_](./08-hpa-keda-autoscaling/lab-064-keda-scale-to-zero/)
- [Lab 065 — PDB — minAvailable drain protection _(kind cluster)_](./08-hpa-keda-autoscaling/lab-065-pdb-drain-protection/)
- [Lab 066 — SadServers Kilifi — resource misallocation _(SadServers)_](./08-hpa-keda-autoscaling/lab-066-sadservers-kilifi/)
 
### 09 — Helm `[Jōnin — Summoning Jutsu]`
Folder: [09-helm](./09-helm/)
 
- [Lab 067 — Helm operations — install, upgrade, rollback, lint _(Killercoda omkar)_](./09-helm/lab-067-helm-operations/)
- [Lab 068 — Helm install Redis — CKA Challenge 21 _(Killercoda CKA)_](./09-helm/lab-068-helm-install-redis/)
- [Lab 069 — Write Helm chart from scratch — no helm create _(kind cluster)_](./09-helm/lab-069-write-helm-chart-from-scratch/)
- [Lab 070 — _helpers.tpl — named template for common labels _(kind cluster)_](./09-helm/lab-070-helpers-tpl/)
- [Lab 071 — Dev vs Prod values — same chart, 2 values files _(kind cluster)_](./09-helm/lab-071-dev-vs-prod-values/)
 
### 10 — Scheduling `[Jōnin]`
Folder: [10-scheduling](./10-scheduling/)
 
- [Lab 072 — Taint + Toleration — Pending then schedules _(kind cluster)_](./10-scheduling/lab-072-taint-toleration/)
- [Lab 073 — nodeSelector — pod only on labeled node _(kind cluster)_](./10-scheduling/lab-073-node-selector/)
- [Lab 074 — Node Affinity required — zone-a/b, test zone-c Pending _(kind cluster)_](./10-scheduling/lab-074-node-affinity/)
- [Lab 075 — Pod Anti-Affinity — 3 replicas spread across nodes _(kind cluster)_](./10-scheduling/lab-075-pod-anti-affinity/)
- [Lab 076 — CKA Challenge 02 — Drain + cordon + uncordon _(Killercoda CKA)_](./10-scheduling/lab-076-cka-challenge-02/)
- [Lab 077 — CKA Challenge 16 — Node troubleshooting _(Killercoda CKA)_](./10-scheduling/lab-077-cka-challenge-16/)
 
### 11 — Observability `[Jōnin — The Sharingan]`
Folder: [11-observability](./11-observability/)
 
- [Lab 083 — Deploy kube-prometheus-stack + explore _(kind cluster)_](./11-observability/lab-083-kube-prometheus-stack/)
- [Lab 084 — PromQL — 5 queries from scratch _(kind cluster)_](./11-observability/lab-084-promql-queries/)
- [Lab 085 — Grafana dashboard — 4 panels from scratch, no import _(kind cluster)_](./11-observability/lab-085-grafana-dashboard/)
- [Lab 086 — Alertmanager — PodCrashLooping + HighErrorRate → Slack _(kind cluster)_](./11-observability/lab-086-alertmanager/)
- [Lab 087 — Loki + Promtail — LogQL for ERROR logs last 1h _(kind cluster)_](./11-observability/lab-087-loki-promtail/)
- [Lab 088 — SadServers — Prometheus scrape debug _(SadServers)_](./11-observability/lab-088-sadservers-prometheus-scrape-debug/)
 
### 12 — Jobs + CronJobs `[Jōnin]`
Folder: [12-jobs-cronjobs](./12-jobs-cronjobs/)
 
- [Lab 078 — Job — parallelism:3 completions:6, watch concurrent pods _(Killercoda omkar)_](./12-jobs-cronjobs/lab-078-job-parallelism/)
- [Lab 079 — CronJob basics — schedule + verify execution _(Killercoda omkar)_](./12-jobs-cronjobs/lab-079-cronjob-basics/)
- [Lab 080 — CronJob data pipeline — retry + deadline _(Killercoda omkar)_](./12-jobs-cronjobs/lab-080-cronjob-data-pipeline/)
- [Lab 081 — Countdown Job completions + backoffLimit _(KodeKloud Engineer)_](./12-jobs-cronjobs/lab-081-countdown-job/)
- [Lab 082 — Create CronJobs with exact schedule _(KodeKloud Engineer)_](./12-jobs-cronjobs/lab-082-create-cronjobs/)
 
### 13 — Debugging `[ANBU — Reading the Enemy]`
Folder: [13-debugging](./13-debugging/)
 
- [Lab 089 — CrashLoopBackOff — systematic diagnosis _(Killercoda)_](./13-debugging/lab-089-crashloopbackoff/)
- [Lab 090 — OOMKilled — exit 137, Last State, fix _(Killercoda)_](./13-debugging/lab-090-oomkilled/)
- [Lab 091 — ImagePullBackOff — wrong name, missing pull secret _(Killercoda)_](./13-debugging/lab-091-imagepullbackoff/)
- [Lab 092 — Service empty endpoints — selector mismatch _(Killercoda)_](./13-debugging/lab-092-service-empty-endpoints/)
- [Lab 093 — Pending pod — resources + taint mismatch _(Killercoda)_](./13-debugging/lab-093-pending-pod/)
- [Lab 094 — Ingress 404 — controller missing, wrong pathType _(Killercoda)_](./13-debugging/lab-094-ingress-404/)
- [Lab 095 — SadServers Singara — k3s web app NodePort debug _(SadServers)_](./13-debugging/lab-095-sadservers-singara/)
- [Lab 096 — Chaos 1 — OOMKill: diagnose fast _(kind cluster)_](./13-debugging/lab-096-chaos-oomkill/)
- [Lab 097 — Chaos 2 — CrashLoopBackOff: fix quickly _(kind cluster)_](./13-debugging/lab-097-chaos-crashloopbackoff/)
- [Lab 098 — Chaos 3 — NetworkPolicy suddenly blocks app _(kind cluster)_](./13-debugging/lab-098-chaos-networkpolicy/)
 
### 14 — Real-World Apps `[ANBU]`
Folder: [14-realworld-apps](./14-realworld-apps/)
 
- [Lab 099 — Nginx Web Server — Pod + NodePort + curl verify _(KodeKloud Engineer)_](./14-realworld-apps/lab-099-nginx-web-server/)
- [Lab 100 — MySQL StatefulSet — PVC + headless + data persistence _(KodeKloud Engineer)_](./14-realworld-apps/lab-100-mysql-statefulset/)
- [Lab 101 — Redis Deployment — ClusterIP + env config _(KodeKloud Engineer)_](./14-realworld-apps/lab-101-redis-deployment/)
- [Lab 102 — Jenkins — Deployment + NodePort + PVC _(KodeKloud Engineer)_](./14-realworld-apps/lab-102-jenkins/)
- [Lab 103 — Grafana — Deployment + Service + PVC _(KodeKloud Engineer)_](./14-realworld-apps/lab-103-grafana/)
- [Lab 104 — Nginx + PhpFpm — 2 containers + shared volume _(KodeKloud Engineer)_](./14-realworld-apps/lab-104-nginx-phpfpm/)
- [Lab 105 — Node App — specific image + port + NodePort _(KodeKloud Engineer)_](./14-realworld-apps/lab-105-node-app/)
- [Lab 106 — Guestbook App — 3 Deployments + 3 Services _(KodeKloud Engineer)_](./14-realworld-apps/lab-106-guestbook-app/)
- [Lab 107 — LAMP Stack — Apache + MySQL + PHP _(KodeKloud Engineer)_](./14-realworld-apps/lab-107-lamp-stack/)
- [Lab 108 — Drupal + MySQL — 2 Deployments + PVC _(KodeKloud Engineer)_](./14-realworld-apps/lab-108-drupal-mysql/)
- [Lab 109 — Troubleshoot Nginx+PhpFpm broken _(KodeKloud Engineer)_](./14-realworld-apps/lab-109-troubleshoot-nginx-phpfpm/)
- [Lab 110 — Fix PhpFpm broken config _(KodeKloud Engineer)_](./14-realworld-apps/lab-110-fix-phpfpm-config/)
 
### 15 — CKA Sprint `[🏆 The Exam to Become Hokage]`
Folder: [15-cka-sprint](./15-cka-sprint/)
 
- [Lab 111 — CKA Challenge 04 — ETCD Backup + Restore _(Killercoda CKA)_](./15-cka-sprint/lab-111-cka-challenge-04/)
- [Lab 112 — CKA Challenge 08 — Kubernetes cluster upgrade _(Killercoda CKA)_](./15-cka-sprint/lab-112-cka-challenge-08/)
- [Lab 113 — CKA Challenge 09 — Deployment scaling + update _(Killercoda CKA)_](./15-cka-sprint/lab-113-cka-challenge-09/)
- [Lab 114 — CKA Challenge 17 — Service troubleshooting _(Killercoda CKA)_](./15-cka-sprint/lab-114-cka-challenge-17/)
- [Lab 115 — CKA Challenge 14 redo _(Killercoda CKA)_](./15-cka-sprint/lab-115-cka-challenge-14-redo/)
- [Lab 116 — CKA Challenge 05 redo _(Killercoda CKA)_](./15-cka-sprint/lab-116-cka-challenge-05-redo/)
- [Lab 117 — CKA Challenge 11 redo _(Killercoda CKA)_](./15-cka-sprint/lab-117-cka-challenge-11-redo/)
- [Lab 118 — CKA Challenge 12 redo _(Killercoda CKA)_](./15-cka-sprint/lab-118-cka-challenge-12-redo/)
- [Lab 119 — CKA Mock Practice — full run _(Killercoda)_](./15-cka-sprint/lab-119-cka-mock-practice/)
- [Lab 120 — SadServers Tigoni — StatefulSet Helm upgrade v1→v2 _(SadServers)_](./15-cka-sprint/lab-120-sadservers-tigoni/)
 
---
 
## 🎴 Repository Structure
 
```text
kubejutsu/
├── README.md
├── ROADMAP.md
├── 01-pods/
├── 02-deployments/
├── 03-configmaps-secrets/
├── 04-storage/
├── 05-services-ingress/
├── 06-networkpolicy/
├── 07-rbac-security/
├── 08-hpa-keda-autoscaling/
├── 09-helm/
├── 10-scheduling/
├── 11-observability/
├── 12-jobs-cronjobs/
├── 13-debugging/
├── 14-realworld-apps/
└── 15-cka-sprint/
```
 
Each mission folder follows this structure:
 
```text
lab-xxx-name/
├── task.md        ← the mission briefing
├── solution.yaml  ← the sealed jutsu
├── commands.sh    ← the exact hand signs used
└── notes.md       ← what failed, what clicked, what was learned
```
 
---
 
## ⚔️ The Way of the Shinobi-Engineer
 
A mission is not complete when it runs. It is complete when you understand it.
 
1. **Read the task** — understand the objective before touching the keyboard
2. **Attempt it alone first** — no solution, no hints, no copy-paste
3. **Verify with `kubectl`** — running ≠ correct; confirm every expected behavior
4. **Save the exact commands** — in `commands.sh`, unedited, as they were typed
5. **Write `notes.md`** — what failed, what the error meant, what changed, what you learned
6. **Commit the lab** — one lab, one commit, no bundling
 
> A shinobi who skips the failure skips the lesson.
 
---
 
## 🖋️ The Seal Convention — Commit Format
 
Every commit is a promise written in stone.
 
```
feat(pods): lab-001 pod basics
feat(deployments): lab-014 blue-green
fix(networkpolicy): correct ingress rule
docs(storage): improve notes for pvc binding
feat(cka): lab-111 etcd backup and restore
```
 
Format: `type(scope): short description`
Types: `feat` · `fix` · `docs` · `refactor` · `chore`
 
---
 
## 🗺️ Topic Index
 
| Topic | Folder | Rank |
|---|---|---|
| Pods | [01-pods](./01-pods/) | Academy |
| Deployments | [02-deployments](./02-deployments/) | Genin |
| ConfigMaps + Secrets | [03-configmaps-secrets](./03-configmaps-secrets/) | Genin |
| Storage | [04-storage](./04-storage/) | Genin |
| Services + Ingress | [05-services-ingress](./05-services-ingress/) | Chūnin |
| NetworkPolicy | [06-networkpolicy](./06-networkpolicy/) | Chūnin |
| RBAC + Security | [07-rbac-security](./07-rbac-security/) | Chūnin |
| HPA + KEDA + Autoscaling | [08-hpa-keda-autoscaling](./08-hpa-keda-autoscaling/) | Jōnin |
| Helm | [09-helm](./09-helm/) | Jōnin |
| Scheduling | [10-scheduling](./10-scheduling/) | Jōnin |
| Observability | [11-observability](./11-observability/) | Jōnin |
| Jobs + CronJobs | [12-jobs-cronjobs](./12-jobs-cronjobs/) | Jōnin |
| Debugging | [13-debugging](./13-debugging/) | ANBU |
| Real-World Apps | [14-realworld-apps](./14-realworld-apps/) | ANBU |
| CKA Sprint | [15-cka-sprint](./15-cka-sprint/) | 🏆 Hokage |
 
---
 
## 🏟️ Training Grounds
 
Where the labs are practiced:
 
- [Killercoda Playground](https://killercoda.com/playgrounds/scenario/kubernetes)
- [Killercoda CKA](https://killercoda.com/killer-shell-cka)
- [Killercoda omkar scenarios](https://killercoda.com/omkar-shelke25)
- [KodeKloud Playground](https://kodekloud.com/public-playgrounds)
- [SadServers Kubernetes](https://sadservers.com/tag/kubernetes)
- Local `kind` cluster for custom chaos
 
---
 
## 📚 Source Scrolls
 
### KodeKloud / GitHub
- [joseeden/KodeKloud_Engineer_Labs](https://github.com/joseeden/KodeKloud_Engineer_Labs)
- [MiqdadProjects/kodekloud-kubernetes-solutions](https://github.com/MiqdadProjects/kodekloud-kubernetes-solutions)
- [MederD/Kodekloud-Engineer-Tasks](https://github.com/MederD/Kodekloud-Engineer-Tasks)
 
### Killercoda
- [omkar-shelke25](https://killercoda.com/omkar-shelke25)
- [killer-shell-cka](https://killercoda.com/killer-shell-cka)
- [cka-mock-practice](https://killercoda.com/cka-mock-practice)
 
### Other
- [SadServers Kubernetes](https://sadservers.com/tag/kubernetes)
 
---
 
## 🗺️ Roadmap
 
See [ROADMAP.md](./ROADMAP.md)
 
---
