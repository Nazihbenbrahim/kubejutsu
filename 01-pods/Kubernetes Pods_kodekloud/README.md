# Kubernetes Pods

**Level:** Beginner  
**Duration:** 1.5 hours  
**Mode:** Interactive  
**Source:** KodeKloud

Learn about Kubernetes Pods, the smallest deployable units in Kubernetes.

## What this lab covers

In this lab, you practice how to:

- inspect Pods in the default namespace
- create a Pod with `kubectl run`
- inspect controller-created Pods
- identify Pod images
- identify node placement
- inspect a multi-container Pod
- understand the `READY` column
- diagnose image pull errors
- generate Pod YAML with `--dry-run=client -o yaml`
- fix a broken image reference and recover the Pod

---

## Task 1/13 — How many pods exist on the system?

**Task**

How many `pods` exist on the system?

In the current(default) namespace.

**Command**

```bash
kubectl get pods
```

**Output**

```text
controlplane ~ ➜  kubectl get pods
No resources found in default namespace.
```

**Answer**

`0`

---

## Task 2/13 — Create a new pod using the nginx image

**Task**

Create a new pod using the `nginx` image.

**Command**

```bash
kubectl run nginx --image=nginx
```

**Output**

```text
controlplane ~ ➜  kubectl run nginx --image=nginx
pod/nginx created
```

**Answer**

Create the Pod with image `nginx`.

---

## Task 3/13 — How many pods are created now?

**Task**

How many pods are created now?

We have created a few more pods. Check again in the current(default) namespace.

**Command**

```bash
kubectl get pods
```

**Output**

```text
controlplane ~ ➜  kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
newpods-csq6q  1/1     Running   0          36s
newpods-z2p2s  1/1     Running   0          36s
newpods-j67fv  1/1     Running   0          36s
nginx          1/1     Running   0          25s
```

**Answer**

`4`

---

## Task 4/13 — Which image is specified for the pods whose names begin with the `newpods-` prefix?

**Task**

Which image is specified for the pods whose names begin with the `newpods-` prefix?

You must look at one of the new pods in detail to figure this out.

**Command**

```bash
kubectl describe pod newpods-w5vmg
```

**Output**

```text
controlplane ~ ➜  kubectl describe pod newpods-w5vmg
Name:             newpods-w5vmg
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/10.244.135.30
Start Time:       Tue, 14 Apr 2026 10:18:23 +0000
Labels:           tier=busybox
Annotations:      <none>
Status:           Running
IP:               10.42.0.10
IPs:
  IP:  10.42.0.10
Controlled By:    ReplicaSet/newpods
Containers:
  busybox:
    Image:        busybox
    Command:
      sleep
      1000
    State:        Running
    Ready:        True
    Restart Count: 0
```

**Answer**

`BUSYBOX`

---

## Task 5/13 — Which nodes are these pods placed on?

**Task**

Which nodes are these pods placed on?

You must look at all the pods in detail to figure this out.

**Command**

```bash
kubectl get pods -o wide
```

**Output**

```text
controlplane ~ ➜  kubectl get pods -o wide
NAME           READY   STATUS    RESTARTS   AGE    IP          NODE           NOMINATED NODE   READINESS GATES
newpods-w5vmg  1/1     Running   0          8m6s   10.42.0.10  controlplane   <none>           <none>
newpods-94hm7  1/1     Running   0          8m6s   10.42.0.11  controlplane   <none>           <none>
newpods-fb26g  1/1     Running   0          8m6s   10.42.0.9   controlplane   <none>           <none>
nginx-pod      1/1     Running   0          7m18s  10.42.0.12  controlplane   <none>           <none>
nginx          1/1     Running   0          4m46s  10.42.0.13  controlplane   <none>           <none>
```

**Answer**

`controlplane`

---

## Task 6/13 — How many containers are part of the `webapp` pod?

**Task**

We just created a new pod named `webapp`. How many containers are part of the `webapp` pod?

Ignore the state of the pod for now.

**Command**

```bash
kubectl describe pod webapp
```

**Output**

```text
controlplane ~ ➜  kubectl describe pod webapp
Name:             webapp
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/10.244.132.225
Start Time:       Wed, 15 Apr 2026 10:03:17 +0000
Labels:           <none>
Annotations:      <none>
Status:           Pending
IP:               10.42.0.12
IPs:
  IP:  10.42.0.12
Containers:
  nginx:
    Image:          nginx
    State:          Running
    Ready:          True
    Restart Count:  0
  agentx:
    Image:          agentx
    State:          Waiting
      Reason:       ErrImagePull
    Ready:          False
    Restart Count:  0
```

**Answer**

`2`

---

## Task 7/13 — What images are used in the new `webapp` pod?

**Task**

What images are used in the new `webapp` pod?

You must look at the Pod in detail to figure this out.

**Command**

```bash
kubectl describe pod webapp
```

**Output**

```text
controlplane ~ ➜  kubectl describe pod webapp
Name:             webapp
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/10.244.132.225
Start Time:       Wed, 15 Apr 2026 10:03:17 +0000
Labels:           <none>
Annotations:      <none>
Status:           Pending
IP:               10.42.0.12
IPs:
  IP:  10.42.0.12
Containers:
  nginx:
    Image:          nginx
    State:          Running
    Ready:          True
    Restart Count:  0
  agentx:
    Image:          agentx
    State:          Waiting
      Reason:       ErrImagePull
    Ready:          False
    Restart Count:  0
```

**Answer**

`nginx & agentx`

---

## Task 8/13 — What is the state of the container `agentx` in the pod `webapp`?

**Task**

What is the state of the container `agentx` in the pod `webapp`?

Wait for it to finish the `ContainerCreating` state.

**Command**

```bash
kubectl describe pod webapp
```

**Output**

```text
agentx:
  Container ID:
  Image:          agentx
  Image ID:
  Port:           <none>
  Host Port:      <none>
  State:          Waiting
    Reason:       ErrImagePull
  Ready:          False
  Restart Count:  0
```

**Answer**

`Error or Waiting`

---

## Task 9/13 — Why is the container `agentx` in pod `webapp` in error?

**Task**

Why do you think the container `agentx` in pod `webapp` is in error?

Inspect the events of the Pod to determine the reason for the container's failure to start.

**Command**

```bash
kubectl describe pod webapp
```

**Output**

```text
Events:
  Type     Reason     Age   From               Message
  ----     ------     ----  ----               -------
  Normal   Scheduled  17s   default-scheduler  Successfully assigned default/webapp to controlplane
  Normal   Pulling    16s   kubelet            Pulling image "nginx"
  Normal   Pulled     13s   kubelet            Successfully pulled image "nginx" in 3.395s (3.395s including waiting)
  Normal   Created    13s   kubelet            Created container nginx
  Normal   Started    13s   kubelet            Started container nginx
  Normal   Pulling    13s   kubelet            Pulling image "agentx"
  Warning  Failed     12s   kubelet            Failed to pull image "agentx": failed to pull and unpack image "docker.io/library/agentx:latest": failed to resolve reference "docker.io/library/agentx:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
  Warning  Failed     12s   kubelet            Error: ErrImagePull
  Normal   BackOff    11s   kubelet            Back-off pulling image "agentx"
  Warning  Failed     11s   kubelet            Error: ImagePullBackOff
```

**Answer**

The image cannot be pulled. It either does not exist or requires authentication.

---

## Task 10/13 — What does the `READY` column indicate?

**Task**

What does the `READY` column in the output of the `kubectl get pods` command indicate?

**Command**

```bash
kubectl get pods
```

**Output**

```text
NAME           READY   STATUS             RESTARTS   AGE
nginx-pod      1/1     Running            0          22m
nginx          1/1     Running            0          19m
newpods-94hm7  1/1     Running            1 (67s ago) 22m
newpods-w5vmg  1/1     Running            1 (67s ago) 22m
newpods-fb26g  1/1     Running            1 (67s ago) 22m
redis          0/1     ImagePullBackOff   0          107s
```

**Answer**

`Ready containers in pod / Total containers in pod`

---

## Task 11/13 — Delete the `webapp` Pod

**Task**

Delete the `webapp` Pod.

Once deleted, wait for the Pod to fully terminate.

**Command**

```bash
kubectl delete pod webapp
kubectl get pods
```

**Output**

```text
controlplane ~ ➜  kubectl delete pod webapp
pod "webapp" deleted

controlplane ~ ➜  kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
newpods-w5vmg  1/1     Running   0          16m
newpods-94hm7  1/1     Running   0          16m
newpods-fb26g  1/1     Running   0          16m
nginx-pod      1/1     Running   0          15m
nginx          1/1     Running   0          13m
```

**Answer**

Delete the Pod with `kubectl delete pod webapp`.

---

## Task 12/13 — Create a new Pod with the name `redis` and the image `redis123`

**Task**

Create a new Pod with the name `redis` and the image `redis123`.

Use a pod-definition YAML file. The image name `redis123` is intentionally incorrect. Do not correct the image at this stage.

**Command**

```bash
kubectl run redis --image=redis123 --dry-run=client -o yaml > redis-pod.yaml
kubectl create -f redis-pod.yaml
kubectl get pods
```

**Output**

```text
controlplane ~ ➜  kubectl run redis --image=redis123 --dry-run=client -o yaml > redis-pod.yaml

controlplane ~ ➜  kubectl create -f redis-pod.yaml
pod/redis created
-bash: : command not found

controlplane ~ ➜  kubectl get pods
NAME           READY   STATUS             RESTARTS   AGE
nginx-pod      1/1     Running            0          22m
nginx          1/1     Running            0          19m
newpods-94hm7  1/1     Running            1 (67s ago) 22m
newpods-w5vmg  1/1     Running            1 (67s ago) 22m
newpods-fb26g  1/1     Running            1 (67s ago) 22m
redis          0/1     ImagePullBackOff   0          107s
```

**Answer**

Generate the YAML first, then create the Pod from that file.

---

## Task 13/13 — Change the image on this Pod to `redis`

**Task**

Now change the image on this Pod to `redis`.

Once done, the Pod should be in a `running` state.

**Command**

```bash
nano redis-pod.yaml
kubectl apply -f redis-pod.yaml
kubectl get pods
```

**Output**

```text
controlplane ~ ➜  nano redis-pod.yaml

controlplane ~ ➜  kubectl apply -f redis-pod.yaml
Warning: resource pods/redis is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
pod/redis configured

controlplane ~ ➜  kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
nginx-pod      1/1     Running   0          30m
nginx          1/1     Running   0          28m
newpods-94hm7  1/1     Running   1 (14m ago) 31m
newpods-w5vmg  1/1     Running   1 (14m ago) 31m
newpods-fb26g  1/1     Running   1 (14m ago) 31m
redis          1/1     Running   0          10m
```

**Answer**

Edit the YAML, replace `redis123` with `redis`, then apply the file again.

---

## Final manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
    - name: redis
      image: redis
```

---

## What you should remember from this lab

- A Pod is the smallest deployable unit in Kubernetes.
- `kubectl get pods` gives a quick high-level view of Pod health.
- `kubectl describe pod <name>` is one of the most useful commands for troubleshooting.
- Controller-created Pods often have generated suffixes in their names.
- Pods can contain more than one container.
- `READY` means: ready containers / total containers.
- Image pull problems usually appear as `ErrImagePull` and then `ImagePullBackOff`.
- `--dry-run=client -o yaml` is a useful way to generate manifests quickly.
- `kubectl apply` can be used after editing a generated manifest to correct configuration mistakes.
