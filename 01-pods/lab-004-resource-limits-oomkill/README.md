# Configure Resource Requests and Limits for Deployments

**Level:** Intermediate  
**Duration:** 20–30 minutes  
**Mode:** Interactive  
**Source:** Killercoda  
**Lab URL:** https://killercoda.com/omkar-shelke25/scenario/resource-limit-deployment

Learn how to update existing Deployments with resource requests and limits, understand how those settings affect Pod QoS classes, and verify the difference between **Burstable** and **Guaranteed** behavior from real running Pods.

## Short introduction

In this lab, two Deployments already exist in the `manga` namespace:

- `naruto`
- `demon-slayer`

The goal is not only to add resource constraints, but also to understand the operational effect of those changes. One Deployment is configured with **requests only**, while the other is configured with **limits only**. That small difference has a major impact on the Pod QoS class, and this lab makes that behavior visible through real rollout and inspection commands.

---

## Tasks section

## Task 1/6 — Update the `naruto` Deployment with resource requests

**Task**

Edit the `naruto` Deployment in the `manga` namespace and add these resource requests:

- CPU: `100m`
- Memory: `100Mi`

**Command**

```bash
kubectl edit deployment naruto -n manga
```

**Output**

```text
root@controlplane:~$ kubectl edit deployment naruto -n manga
deployment.apps/naruto edited
```

**Answer**

The Deployment object was updated successfully.

**Interpretation**

This message confirms that the Deployment spec was saved, but the important detail is **what kind of field was changed**.  
Because resource settings live under the Pod template (`spec.template.spec.containers[...]`), Kubernetes treats this as a change to the template itself. That means the Deployment controller creates a **new ReplicaSet** and starts a **rolling update** instead of modifying the running Pods in place.

This is a key Kubernetes behavior to understand:

- editing the Deployment updates the desired state
- existing Pods are not patched in place for template changes
- new Pods are created from a new template hash
- old Pods are gradually replaced

That is why Pod names usually change after this kind of edit.

---

## Task 2/6 — Update the `demon-slayer` Deployment with resource limits

**Task**

Edit the `demon-slayer` Deployment in the `manga` namespace and add these resource limits:

- CPU: `200m`
- Memory: `200Mi`

**Command**

```bash
kubectl edit deployment demon-slayer -n manga
```

**Output**

```text
root@controlplane:~$ kubectl edit deployment demon-slayer -n manga
deployment.apps/demon-slayer edited
```

**Answer**

The Deployment object was updated successfully.

**Interpretation**

Just like the previous edit, this change touched the Pod template, so Kubernetes triggered another rolling update.  
At this stage, both Deployments now have a fresh desired state:

- `naruto` has explicit **requests**
- `demon-slayer` has explicit **limits**

Even before checking Pod QoS, this already tells an experienced reader something important: the two workloads are no longer treated the same by the scheduler and kubelet. One now reserves baseline resources, while the other introduces hard ceilings that can affect scheduling, eviction priority, and runtime behavior.

---

## Task 3/6 — Verify that the new Pods are running in the `manga` namespace

**Task**

List the Pods in the `manga` namespace and confirm that the new Pods created by the updated Deployments are running.

**Command**

```bash
kubectl -n manga get pods
```

**Output**

```text
root@controlplane:~$ kubectl -n manga get pods
NAME                           READY   STATUS    RESTARTS   AGE
demon-slayer-875787dd4-fldj7   1/1     Running   0          67s
demon-slayer-875787dd4-qksp5   1/1     Running   0          66s
naruto-76fcf9d996-c64x6        1/1     Running   0          3m2s
naruto-76fcf9d996-cxz2t        1/1     Running   0          3m
```

**Answer**

Both Deployments are healthy and their Pods are running successfully in the `manga` namespace.

**Interpretation**

This output reveals several important things:

1. There are **4 Pods** because the namespace contains **2 different Deployments**, and each Deployment is maintaining **2 replicas**.
2. The Pod names include hashes:
   - `875787dd4` for `demon-slayer`
   - `76fcf9d996` for `naruto`

   Those hashes identify the **ReplicaSet generated from the current Pod template**. Seeing new hashes after the edits confirms that the Deployments rolled out new Pods based on the updated templates.
3. The Pods are all `1/1 Running`, which means the rollout has already converged to the new desired state.
4. The age difference is also meaningful:
   - `naruto` Pods are older
   - `demon-slayer` Pods are newer

   That lines up with the command order: `naruto` was edited first, then `demon-slayer`.

A shallow reading of this output only says “the pods are running.”  
A stronger reading says: **the rollout completed, the old ReplicaSets were replaced, the new template hashes are active, and both Deployments are now serving from updated resource configurations.**

---

## Task 4/6 — Inspect the `demon-slayer` Pod and identify its QoS class

**Task**

Describe one of the `demon-slayer` Pods and inspect its runtime resource configuration and QoS class.

**Command**

```bash
kubectl -n manga describe pod demon-slayer-875787dd4-fldj7
```

**Output**

```text
Name:             demon-slayer-875787dd4-fldj7
Namespace:        manga
Priority:         0
Service Account:  default
Node:             node01/172.30.2.2
Start Time:       Thu, 16 Apr 2026 00:50:28 +0000
Labels:           app=demon-slayer
                  pod-template-hash=875787dd4
Annotations:      <none>
Status:           Running
IP:               192.168.1.161
IPs:
  IP:           192.168.1.161
Controlled By:  ReplicaSet/demon-slayer-875787dd4
Containers:
  demon-slayer-container:
    Container ID:   containerd://2fab6e506be5d1db9e6cd5340baf2d31ed35bfb291520ce52c1ba725cd1e6b13
    Image:          public.ecr.aws/docker/library/httpd:alpine
    Image ID:       public.ecr.aws/docker/library/httpd@sha256:fd9f958a7e0642a6f40bc29bfa7391906768afb1a997cb73c9481aab7dbbd052
    Port:           80/TCP (http)
    Host Port:      0/TCP (http)
    State:          Running
      Started:      Thu, 16 Apr 2026 00:50:29 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     200m
      memory:  200Mi
    Requests:
      cpu:        200m
      memory:     200Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-srsjn (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-srsjn:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   Guaranteed
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  2m1s  default-scheduler  Successfully assigned manga/demon-slayer-875787dd4-fldj7 to node01
  Normal  Pulled     2m    kubelet            spec.containers{demon-slayer-container}: Container image "public.ecr.aws/docker/library/httpd:alpine" already present on machine and can be accessed by the pod
  Normal  Created    2m    kubelet            spec.containers{demon-slayer-container}: Container created
  Normal  Started    2m    kubelet            spec.containers{demon-slayer-container}: Container started
```

**Answer**

The `demon-slayer` Pod is running with QoS class `Guaranteed`.

**Interpretation**

This output is one of the most interesting parts of the lab.

At first glance, the task only asked for **limits**, but the live Pod shows both:

- **Limits**
- **Requests**

And both are equal:

- CPU request = CPU limit = `200m`
- Memory request = Memory limit = `200Mi`

That is not accidental.

When a container has a **limit** defined for a resource but no explicit request for that same resource, Kubernetes defaults the **request to the limit value** for admission and scheduling purposes.  
That is exactly what happened here.

This is why the Pod became **Guaranteed**:

- every container has CPU and memory requests
- every container has CPU and memory limits
- each request equals its corresponding limit

That is the strictest QoS class in Kubernetes and gives the Pod the strongest resource reservation behavior under node pressure.

This is the kind of detail that separates “I edited the YAML” from “I understand how Kubernetes interprets the YAML.”

---

## Task 5/6 — Inspect the `naruto` Pod and identify its QoS class

**Task**

Describe one of the `naruto` Pods and inspect its runtime resource configuration and QoS class.

**Command**

```bash
kubectl -n manga describe pod naruto-76fcf9d996-c64x6
```

**Output**

```text
Name:             naruto-76fcf9d996-c64x6
Namespace:        manga
Priority:         0
Service Account:  default
Node:             node01/172.30.2.2
Start Time:       Thu, 16 Apr 2026 00:48:33 +0000
Labels:           app=naruto
                  pod-template-hash=76fcf9d996
Annotations:      <none>
Status:           Running
IP:               192.168.1.80
IPs:
  IP:           192.168.1.80
Controlled By:  ReplicaSet/naruto-76fcf9d996
Containers:
  naruto-container:
    Container ID:   containerd://bb9311449b39bc168403c7dea0c1234c587bf5d21ad4983fcf18549f1efbe68e
    Image:          public.ecr.aws/nginx/nginx:alpine
    Image ID:       public.ecr.aws/nginx/nginx@sha256:8af17bf63d3bd38f8ab5ddc0d002e80b227c57783daf129c0ab137156745d833
    Port:           80/TCP (http)
    Host Port:      0/TCP (http)
    State:          Running
      Started:      Thu, 16 Apr 2026 00:48:34 +0000
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        100m
      memory:     100Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-lm8gv (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-lm8gv:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  4m49s  default-scheduler  Successfully assigned manga/naruto-76fcf9d996-c64x6 to node01
  Normal  Pulled     4m48s  kubelet            spec.containers{naruto-container}: Container image "public.ecr.aws/nginx/nginx:alpine" already present on machine and can be accessed by the pod
  Normal  Created    4m48s  kubelet            spec.containers{naruto-container}: Container created
  Normal  Started    4m48s  kubelet            spec.containers{naruto-container}: Container started
```

**Answer**

The `naruto` Pod is running with QoS class `Burstable`.

**Interpretation**

This Pod has only **requests** configured:

- CPU request: `100m`
- Memory request: `100Mi`

There are **no limits** shown in the container section.

That immediately explains the QoS result:

- the Pod is **not BestEffort**, because it does have resource requests
- the Pod is **not Guaranteed**, because requests and limits are not both set equally
- therefore the Pod is **Burstable**

This is a classic Kubernetes QoS pattern:

- **BestEffort** → no requests and no limits
- **Burstable** → at least one request or limit exists, but not all requests equal limits
- **Guaranteed** → requests and limits are fully set and equal for CPU and memory

So this lab is not just about editing two Deployments. It is demonstrating that a very small change in resource specification can completely change how Kubernetes classifies and protects a workload.

---

## Task 6/6 — Compare the Deployment templates and the final QoS results

**Task**

Describe both Deployments and compare their resource definitions. Then verify the QoS classes across all Pods in the namespace.

**Command**

```bash
kubectl describe deployment naruto -n manga
kubectl describe deployment demon-slayer -n manga
kubectl describe po -n manga | grep -i "qos"
```

**Output**

```text
root@controlplane:~$ kubectl describe deployment naruto -n manga
Name:                   naruto
Namespace:              manga
CreationTimestamp:      Thu, 16 Apr 2026 00:40:52 +0000
Labels:                 anime=shonen-jump
                        app=naruto
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=naruto
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=naruto
  Containers:
   naruto-container:
    Image:      public.ecr.aws/nginx/nginx:alpine
    Port:       80/TCP (http)
    Host Port:  0/TCP (http)
    Requests:
      cpu:         100m
      memory:      100Mi
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  naruto-5dbfdfdc69 (0/0 replicas created)
NewReplicaSet:   naruto-76fcf9d996 (2/2 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  13m    deployment-controller  Scaled up replica set naruto-5dbfdfdc69 from 0 to 2
  Normal  ScalingReplicaSet  5m46s  deployment-controller  Scaled up replica set naruto-76fcf9d996 from 0 to 1
  Normal  ScalingReplicaSet  5m44s  deployment-controller  Scaled down replica set naruto-5dbfdfdc69 from 2 to 1
  Normal  ScalingReplicaSet  5m44s  deployment-controller  Scaled up replica set naruto-76fcf9d996 from 1 to 2
  Normal  ScalingReplicaSet  5m43s  deployment-controller  Scaled down replica set naruto-5dbfdfdc69 from 1 to 0

root@controlplane:~$ kubectl describe deployment demon-slayer -n manga
Name:                   demon-slayer
Namespace:              manga
CreationTimestamp:      Thu, 16 Apr 2026 00:40:52 +0000
Labels:                 anime=shonen-jump
                        app=demon-slayer
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=demon-slayer
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=demon-slayer
  Containers:
   demon-slayer-container:
    Image:      public.ecr.aws/docker/library/httpd:alpine
    Port:       80/TCP (http)
    Host Port:  0/TCP (http)
    Limits:
      cpu:         200m
      memory:      200Mi
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  demon-slayer-7854c6d67c (0/0 replicas created)
NewReplicaSet:   demon-slayer-875787dd4 (2/2 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  13m    deployment-controller  Scaled up replica set demon-slayer-7854c6d67c from 0 to 2
  Normal  ScalingReplicaSet  3m51s  deployment-controller  Scaled up replica set demon-slayer-875787dd4 from 0 to 1
  Normal  ScalingReplicaSet  3m50s  deployment-controller  Scaled down replica set demon-slayer-7854c6d67c from 2 to 1
  Normal  ScalingReplicaSet  3m50s  deployment-controller  Scaled up replica set demon-slayer-875787dd4 from 1 to 2
  Normal  ScalingReplicaSet  3m49s  deployment-controller  Scaled down replica set demon-slayer-7854c6d67c from 1 to 0

root@controlplane:~$ kubectl describe po -n manga | grep -i "qos"
QoS Class:                   Guaranteed
QoS Class:                   Guaranteed
QoS Class:                   Burstable
QoS Class:                   Burstable
```

**Answer**

The `naruto` Deployment template contains **requests only**, while the `demon-slayer` Deployment template contains **limits only**.  
At the Pod level, this results in:

- `naruto` Pods → `Burstable`
- `demon-slayer` Pods → `Guaranteed`

**Interpretation**

This combined verification is where the lab becomes truly instructive.

### What the Deployment descriptions prove

The Deployment descriptions show the **declared template state**:

- `naruto` template contains only requests
- `demon-slayer` template contains only limits

They also show rollout history very clearly:

- an old ReplicaSet still exists as history
- a new ReplicaSet became active after the edit
- the Deployment controller scaled up the new ReplicaSet while scaling down the old one

That is the real rollout mechanics behind the single line `deployment.apps/... edited`.

### What the Pod descriptions prove

Pods show the **effective runtime state**, not just the template.

That distinction matters because Kubernetes can apply defaulting behavior during Pod admission.  
That is exactly why `demon-slayer` ended up with visible requests even though only limits were configured in the Deployment template.

### Why the namespace shows two different QoS classes

The final `grep -i qos` output is the strongest confirmation:

- two Pods are `Guaranteed`
- two Pods are `Burstable`

That matches the replica counts perfectly:

- 2 replicas for `demon-slayer` → 2 Guaranteed Pods
- 2 replicas for `naruto` → 2 Burstable Pods

This is not random output. It is the direct result of Kubernetes QoS rules applied consistently across every replica.

---

## Resource configuration applied in this lab

### `naruto` Deployment

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "100Mi"
```

### `demon-slayer` Deployment

```yaml
resources:
  limits:
    cpu: "200m"
    memory: "200Mi"
```

---

## What you should remember from this lab

### Explanation

Kubernetes determines QoS class from CPU and memory requests and limits at the container level.

- **BestEffort**: no requests and no limits
- **Burstable**: at least one request or limit exists, but the Pod does not satisfy Guaranteed rules
- **Guaranteed**: every container has CPU and memory requests and limits, and each request equals its corresponding limit

This lab also shows an important runtime detail: when limits are set without explicit requests for the same resource, Kubernetes can default the requests to the limit values, which can elevate the Pod to `Guaranteed`.

### Key takeaway

A resource stanza is not just a scheduling hint. It changes how Kubernetes classifies, schedules, protects, and evicts workloads under pressure.

In this lab:

- `naruto` remained flexible and became `Burstable`
- `demon-slayer` became strictly reserved and therefore `Guaranteed`

That is a small YAML change with a very real operational consequence.

### Final recap

This lab covered how to:

- update existing Deployments with `kubectl edit`
- trigger rolling updates through Pod template changes
- recognize new ReplicaSets from Pod template hashes
- distinguish Deployment template state from effective Pod state
- understand why `limits only` can still produce visible requests at runtime
- verify QoS behavior directly from live Pods, not just from YAML
