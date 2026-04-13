# ☸️ kubejutsu

Hands-on Kubernetes labs focused on real practice: manifests, debugging, networking, storage, security, Helm, scheduling, observability, and production-style troubleshooting.

This repository is organized lab by lab. Each lab lives in its own folder and usually contains:

- `task.md`
- `solution.yaml`
- `commands.sh`
- `notes.md`

## Philosophy

- Write manifests by hand
- Try solving the lab before looking at any solution
- Verify everything with `kubectl`
- Keep the exact commands you used
- Document mistakes, fixes, and lessons learned
- Commit each lab separately

## Commit convention

Examples:

- `feat(pods): lab-001 pod basics`
- `feat(deployments): lab-014 blue-green`
- `fix(networkpolicy): correct ingress rule`
- `docs(storage): improve notes for pvc binding`

## Repository structure

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

## Topic index

| Topic | Folder |
|---|---|
| Pods | [01-pods](./01-pods/) |
| Deployments | [02-deployments](./02-deployments/) |
| ConfigMaps + Secrets | [03-configmaps-secrets](./03-configmaps-secrets/) |
| Storage | [04-storage](./04-storage/) |
| Services + Ingress | [05-services-ingress](./05-services-ingress/) |
| NetworkPolicy | [06-networkpolicy](./06-networkpolicy/) |
| RBAC + Security | [07-rbac-security](./07-rbac-security/) |
| HPA + KEDA + Autoscaling | [08-hpa-keda-autoscaling](./08-hpa-keda-autoscaling/) |
| Helm | [09-helm](./09-helm/) |
| Scheduling | [10-scheduling](./10-scheduling/) |
| Observability | [11-observability](./11-observability/) |
| Jobs + CronJobs | [12-jobs-cronjobs](./12-jobs-cronjobs/) |
| Debugging | [13-debugging](./13-debugging/) |
| Real-World Apps | [14-realworld-apps](./14-realworld-apps/) |
| CKA Sprint | [15-cka-sprint](./15-cka-sprint/) |

---

## Lab index

> Lab titles keep the original wording and source attribution.  
> Folder names stay short, stable, and GitHub-friendly.

### 01 — Pods
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

### 02 — Deployments
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
- [Lab 020 — CKA Challenge 15 — Rollout and Rollback _(Killercoda CKA)_](./02-deployments/lab-020-cka-challenge-15/)

### 03 — ConfigMaps + Secrets
Folder: [03-configmaps-secrets](./03-configmaps-secrets/)

- [Lab 021 — ConfigMap as env vars — inject + verify printenv _(Killercoda omkar)_](./03-configmaps-secrets/lab-021-configmap-env-vars/)
- [Lab 022 — ConfigMap as volume — mount file, exec, cat _(Killercoda omkar)_](./03-configmaps-secrets/lab-022-configmap-volume/)
- [Lab 023 — Secrets — create, base64 decode, inject as env _(Killercoda omkar)_](./03-configmaps-secrets/lab-023-secrets-basics/)
- [Lab 024 — Secrets — same secret as env var AND volume mount _(Killercoda omkar)_](./03-configmaps-secrets/lab-024-secrets-env-and-volume/)
- [Lab 025 — LimitRange — default requests auto-applied to pods _(Killercoda omkar)_](./03-configmaps-secrets/lab-025-limitrange/)
- [Lab 026 — ResourceQuota — 5th pod rejected, fix deployment _(Killercoda omkar)_](./03-configmaps-secrets/lab-026-resourcequota/)
- [Lab 027 — Manage Secrets opaque type _(KodeKloud Engineer)_](./03-configmaps-secrets/lab-027-manage-secrets-opaque/)
- [Lab 028 — CKA Challenge 10 — ConfigMap + Deployment troubleshoot _(Killercoda CKA)_](./03-configmaps-secrets/lab-028-cka-challenge-10/)

### 04 — Storage
Folder: [04-storage](./04-storage/)

- [Lab 029 — PV + PVC + Deployment — file persists after pod delete _(Killercoda omkar)_](./04-storage/lab-029-pv-pvc-deployment/)
- [Lab 030 — StatefulSet MySQL + PVC — stable name + data _(KodeKloud Engineer)_](./04-storage/lab-030-statefulset-mysql-pvc/)
- [Lab 031 — CKA Challenge 01 — Persistent Volume spec _(Killercoda CKA)_](./04-storage/lab-031-cka-challenge-01/)
- [Lab 032 — CKA Challenge 06 — StorageClass + dynamic PVC _(Killercoda CKA)_](./04-storage/lab-032-cka-challenge-06/)
- [Lab 033 — CKA Challenge 13 — PVC stuck Pending _(Killercoda CKA)_](./04-storage/lab-033-cka-challenge-13/)
- [Lab 034 — SadServers Bengaluru — StatefulSet not rescheduling _(SadServers)_](./04-storage/lab-034-sadservers-bengaluru/)

### 05 — Services + Ingress
Folder: [05-services-ingress](./05-services-ingress/)

- [Lab 035 — Services KK Free — ClusterIP + NodePort + endpoints _(KodeKloud Free)_](./05-services-ingress/lab-035-services-kk-free/)
- [Lab 036 — ClusterIP — curl by DNS name, watch endpoints live _(Killercoda omkar)_](./05-services-ingress/lab-036-clusterip-service/)
- [Lab 037 — Fix selector mismatch — endpoints show none _(Killercoda omkar)_](./05-services-ingress/lab-037-fix-selector-mismatch/)
- [Lab 038 — Headless Service + StatefulSet DNS resolution _(kind cluster)_](./05-services-ingress/lab-038-headless-service-dns/)
- [Lab 039 — Ingress path routing — /app1 /app2 to different services _(Killercoda omkar)_](./05-services-ingress/lab-039-ingress-path-routing/)
- [Lab 040 — Ingress multi-path /api /video _(Killercoda omkar)_](./05-services-ingress/lab-040-ingress-multi-path/)
- [Lab 041 — TLS Ingress — self-signed cert + curl -k https _(Killercoda omkar)_](./05-services-ingress/lab-041-tls-ingress/)
- [Lab 042 — Fix Ingress 404 — broken pathType + service ref _(Killercoda omkar)_](./05-services-ingress/lab-042-fix-ingress-404/)
- [Lab 043 — CKA Challenge 03 — Service creation _(Killercoda CKA)_](./05-services-ingress/lab-043-cka-challenge-03/)
- [Lab 044 — CKA Challenge 12 — Ingress _(Killercoda CKA)_](./05-services-ingress/lab-044-cka-challenge-12/)
- [Lab 045 — SadServers Bilbao — Nginx + LoadBalancer broken _(SadServers)_](./05-services-ingress/lab-045-sadservers-bilbao/)

### 06 — NetworkPolicy
Folder: [06-networkpolicy](./06-networkpolicy/)

- [Lab 046 — Default-deny all + selective allow backend _(Killercoda omkar)_](./06-networkpolicy/lab-046-default-deny-selective-allow/)
- [Lab 047 — Cross-namespace NP — AND vs OR selector logic _(Killercoda omkar)_](./06-networkpolicy/lab-047-cross-namespace-and-or/)
- [Lab 048 — Egress NP — block all except port 5432 + DNS 53 _(Killercoda omkar)_](./06-networkpolicy/lab-048-egress-dns-allowance/)
- [Lab 049 — Bidirectional NP — ingress + egress simultaneously _(Killercoda omkar)_](./06-networkpolicy/lab-049-bidirectional-networkpolicy/)
- [Lab 050 — Fix NP misconfiguration — app stops responding _(Killercoda omkar)_](./06-networkpolicy/lab-050-fix-networkpolicy-misconfiguration/)
- [Lab 051 — CKA Challenge 11 — NetworkPolicy troubleshoot _(Killercoda CKA)_](./06-networkpolicy/lab-051-cka-challenge-11/)
- [Lab 052 — CKA Challenge 18 — NetworkPolicy creation _(Killercoda CKA)_](./06-networkpolicy/lab-052-cka-challenge-18/)

### 07 — RBAC + Security
Folder: [07-rbac-security](./07-rbac-security/)

- [Lab 053 — Role + RoleBinding — least privilege pod viewer _(Killercoda omkar)_](./07-rbac-security/lab-053-role-rolebinding/)
- [Lab 054 — ClusterRole + ClusterRoleBinding — cluster-wide read _(Killercoda omkar)_](./07-rbac-security/lab-054-clusterrole-clusterrolebinding/)
- [Lab 055 — ServiceAccount — token decode, automount disable _(Killercoda omkar)_](./07-rbac-security/lab-055-serviceaccount-token/)
- [Lab 056 — RBAC Fix — pod can't call API, find + fix _(Killercoda omkar)_](./07-rbac-security/lab-056-rbac-fix/)
- [Lab 057 — Fix RBAC pod — pod fails due to SA permissions _(Killercoda omkar)_](./07-rbac-security/lab-057-fix-rbac-pod/)
- [Lab 058 — Fix ServiceAccount — Payment API can't authenticate _(Killercoda omkar)_](./07-rbac-security/lab-058-fix-serviceaccount/)
- [Lab 059 — Pod Security — runAsNonRoot + readOnly + emptyDir _(Killercoda omkar)_](./07-rbac-security/lab-059-pod-security/)
- [Lab 060 — CKA Challenge 05 — ServiceAccount + RBAC _(Killercoda CKA)_](./07-rbac-security/lab-060-cka-challenge-05/)
- [Lab 061 — CKA Challenge 20 — RBAC can-i verification _(Killercoda CKA)_](./07-rbac-security/lab-061-cka-challenge-20/)

### 08 — HPA + KEDA + Autoscaling
Folder: [08-hpa-keda-autoscaling](./08-hpa-keda-autoscaling/)

- [Lab 062 — HPA — scale under CPU load _(CKA Challenge 22)_](./08-hpa-keda-autoscaling/lab-062-hpa-scale-under-cpu-load/)
- [Lab 063 — HPA — remove requests → unknown metric _(kind cluster)_](./08-hpa-keda-autoscaling/lab-063-hpa-unknown-metric/)
- [Lab 064 — KEDA — scale-to-zero Redis queue _(kind cluster)_](./08-hpa-keda-autoscaling/lab-064-keda-scale-to-zero/)
- [Lab 065 — PDB — minAvailable drain protection _(kind cluster)_](./08-hpa-keda-autoscaling/lab-065-pdb-drain-protection/)
- [Lab 066 — SadServers Kilifi — resource misallocation _(SadServers)_](./08-hpa-keda-autoscaling/lab-066-sadservers-kilifi/)

### 09 — Helm
Folder: [09-helm](./09-helm/)

- [Lab 067 — Helm operations — install, upgrade, rollback, lint _(Killercoda omkar)_](./09-helm/lab-067-helm-operations/)
- [Lab 068 — Helm install Redis — CKA Challenge 21 _(Killercoda CKA)_](./09-helm/lab-068-helm-install-redis/)
- [Lab 069 — Write Helm chart from scratch — no helm create _(kind cluster)_](./09-helm/lab-069-write-helm-chart-from-scratch/)
- [Lab 070 — _helpers.tpl — named template for common labels _(kind cluster)_](./09-helm/lab-070-helpers-tpl/)
- [Lab 071 — Dev vs Prod values — same chart, 2 values files _(kind cluster)_](./09-helm/lab-071-dev-vs-prod-values/)

### 10 — Scheduling
Folder: [10-scheduling](./10-scheduling/)

- [Lab 072 — Taint + Toleration — Pending then schedules _(kind cluster)_](./10-scheduling/lab-072-taint-toleration/)
- [Lab 073 — nodeSelector — pod only on labeled node _(kind cluster)_](./10-scheduling/lab-073-node-selector/)
- [Lab 074 — Node Affinity required — zone-a/b, test zone-c Pending _(kind cluster)_](./10-scheduling/lab-074-node-affinity/)
- [Lab 075 — Pod Anti-Affinity — 3 replicas spread across nodes _(kind cluster)_](./10-scheduling/lab-075-pod-anti-affinity/)
- [Lab 076 — CKA Challenge 02 — Drain + cordon + uncordon _(Killercoda CKA)_](./10-scheduling/lab-076-cka-challenge-02/)
- [Lab 077 — CKA Challenge 16 — Node troubleshooting _(Killercoda CKA)_](./10-scheduling/lab-077-cka-challenge-16/)

### 11 — Jobs + CronJobs
Folder: [12-jobs-cronjobs](./12-jobs-cronjobs/)

- [Lab 078 — Job — parallelism:3 completions:6, watch concurrent pods _(Killercoda omkar)_](./12-jobs-cronjobs/lab-078-job-parallelism/)
- [Lab 079 — CronJob basics — schedule + verify execution _(Killercoda omkar)_](./12-jobs-cronjobs/lab-079-cronjob-basics/)
- [Lab 080 — CronJob data pipeline — retry + deadline _(Killercoda omkar)_](./12-jobs-cronjobs/lab-080-cronjob-data-pipeline/)
- [Lab 081 — KodeKloud Engineer — Countdown Job completions + backoffLimit _(KodeKloud Engineer)_](./12-jobs-cronjobs/lab-081-countdown-job/)
- [Lab 082 — KodeKloud Engineer — Create CronJobs with exact schedule _(KodeKloud Engineer)_](./12-jobs-cronjobs/lab-082-create-cronjobs/)

### 12 — Observability
Folder: [11-observability](./11-observability/)

- [Lab 083 — Deploy kube-prometheus-stack + explore _(kind cluster)_](./11-observability/lab-083-kube-prometheus-stack/)
- [Lab 084 — PromQL — 5 queries from scratch _(kind cluster)_](./11-observability/lab-084-promql-queries/)
- [Lab 085 — Grafana dashboard — 4 panels from scratch, no import _(kind cluster)_](./11-observability/lab-085-grafana-dashboard/)
- [Lab 086 — Alertmanager — PodCrashLooping + HighErrorRate → Slack _(kind cluster)_](./11-observability/lab-086-alertmanager/)
- [Lab 087 — Loki + Promtail — LogQL for ERROR logs last 1h _(kind cluster)_](./11-observability/lab-087-loki-promtail/)
- [Lab 088 — SadServers — Prometheus scrape debug _(SadServers)_](./11-observability/lab-088-sadservers-prometheus-scrape-debug/)

### 13 — Debugging
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

### 14 — Real-World Apps
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

### 15 — CKA Sprint
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

## How each lab folder is expected to look

```text
lab-xxx-name/
├── task.md
├── solution.yaml
├── commands.sh
└── notes.md
```

## How to work through a lab

1. Read the task carefully
2. Solve it manually first
3. Verify it with `kubectl`
4. Save the final manifest
5. Save the exact commands used
6. Write short notes about what worked, what failed, and what was fixed
7. Commit the lab

## Playgrounds

- [Killercoda Playground](https://killercoda.com/playgrounds/scenario/kubernetes)
- [Killercoda CKA](https://killercoda.com/killer-shell-cka)
- [Killercoda omkar scenarios](https://killercoda.com/omkar-shelke25)
- [KodeKloud Playground](https://kodekloud.com/public-playgrounds)
- [SadServers Kubernetes](https://sadservers.com/tag/kubernetes)

## Source repositories and platforms

### KodeKloud / GitHub
- [joseeden/KodeKloud_Engineer_Labs](https://github.com/joseeden/KodeKloud_Engineer_Labs)
- [MiqdadProjects/kodekloud-kubernetes-solutions](https://github.com/MiqdadProjects/kodekloud-kubernetes-solutions)
- [MederD/Kodekloud-Engineer-Tasks](https://github.com/MederD/Kodekloud-Engineer-Tasks)

### Killercoda
- [omkar-shelke25](https://killercoda.com/omkar-shelke25)
- [killer-shell-cka](https://killercoda.com/killer-shell-cka)
- [cka-mock-practice](https://killercoda.com/cka-mock-practice)
- [Kubernetes Playground](https://killercoda.com/playgrounds/scenario/kubernetes)

### Other
- [SadServers Kubernetes](https://sadservers.com/tag/kubernetes)
- local `kind` cluster for custom practice

## Notes

- Display names preserve the original lab wording and source names.
- Folder names are normalized for readability and consistency.
- This repository can grow lab by lab without needing to rewrite the main structure.

## Roadmap

See [ROADMAP.md](./ROADMAP.md)
