# Deploy Nginx with ConfigMap and InitContainer - One Piece Edition

**Level:** Intermediate  
**Duration:** 25–35 minutes  
**Mode:** Interactive  
**Source:** Killercoda  
**Lab URL:** https://killercoda.com/omkar-shelke25/scenario/init-container-one-piece

Deploy an Nginx application in the `one-piece` namespace that serves custom HTML from a `ConfigMap`, copied into the web root by an `InitContainer`, then exposed through a `NodePort` Service.

## Short introduction

This lab demonstrates a classic Kubernetes pattern:

- store static application content in a `ConfigMap`
- use an `InitContainer` to prepare runtime content before the main container starts
- share data between containers with a volume
- expose the application externally with a `NodePort` Service

The key idea is that the main Nginx container does **not** build or fetch its own HTML. Instead, Kubernetes injects the file through a `ConfigMap`, the `InitContainer` copies it into a shared `emptyDir`, and only after that does Nginx start and serve the final page.

---

## Tasks section

## Task 1/5 — Create the `strawhat-cm` ConfigMap from `/one-piece/index.html`

**Task**

Create a `ConfigMap` named `strawhat-cm` in the `one-piece` namespace using the file `/one-piece/index.html`.

**Command**

```bash
kubectl -n one-piece create configmap strawhat-cm --from-file=/one-piece/index.html
```

**Output**

```text
root@controlplane:~$ kubectl -n one-piece create configmap strawhat-cm --from-file=/one-piece/index.html
configmap/strawhat-cm created
```

**Answer**

The `ConfigMap` was created successfully from the HTML file.

**Interpretation**

This step converts a local file into a Kubernetes object. That matters because the HTML content is now stored in the cluster control plane and can be mounted into Pods as configuration data.

A strong mental model here is:

- the file starts outside the cluster as `/one-piece/index.html`
- Kubernetes stores that file inside `strawhat-cm`
- the Pod will later consume that file through a volume

This is cleaner than baking the HTML into an image because the content becomes easy to update independently from the container image.

---

## Task 2/5 — Create the Deployment with the `InitContainer` and shared volumes

**Task**

Create a Deployment named `strawhat-deploy` in the `one-piece` namespace with:

- `1` replica
- selector `app=strawhat`
- main container `strawhat-nginx` using `public.ecr.aws/nginx/nginx:latest`
- `InitContainer` named `init-copy` using `public.ecr.aws/docker/library/busybox:latest`
- the `InitContainer` must copy `index.html` from the `ConfigMap` mount into `/usr/share/nginx/html/`

**Command**

```bash
nano strawhat.yaml
kubectl apply -f strawhat.yaml
```

**Output**

```text
root@controlplane:~$ kubectl apply -f strawhat.yaml
deployment.apps/strawhat-deploy created
service/strawhat-svc created
```

**Answer**

The Deployment was created with an `InitContainer`, a main Nginx container, and the Service definition in the same manifest file.

**Interpretation**

A high-level reader should notice two things here:

1. The manifest contained **multiple resources**.
   That is why one `kubectl apply -f strawhat.yaml` command created both:
   - `deployment.apps/strawhat-deploy`
   - `service/strawhat-svc`

2. The `InitContainer` pattern solves a startup dependency problem.
   Nginx should only start **after** the HTML file has been copied into the serving directory. Kubernetes guarantees that order automatically:
   - first the init container runs
   - then it exits successfully
   - only then can the main container start

This is much safer than trying to do both tasks inside one container startup command.

---

## Task 3/5 — Inspect the Pod startup flow and verify the `InitContainer` behavior

**Task**

Check the created Pod and inspect how the `InitContainer` completed before Nginx started.

**Command**

```bash
kubectl -n one-piece get pods
kubectl -n one-piece describe pod strawhat-deploy-574bd47bc9-9st42
```

**Output**

```text
root@controlplane:~$ kubectl -n one-piece get pods
NAME                               READY   STATUS            RESTARTS   AGE
strawhat-deploy-574bd47bc9-9st42   0/1     PodInitializing   0          10s
```

```text
root@controlplane:~$ kubectl -n one-piece describe pod strawhat-deploy-574bd47bc9-9st42
Name:             strawhat-deploy-574bd47bc9-9st42
Namespace:        one-piece
Priority:         0
Service Account:  default
Node:             node01/172.30.2.2
Start Time:       Thu, 16 Apr 2026 01:24:35 +0000
Labels:           app=strawhat
                  pod-template-hash=574bd47bc9
Annotations:      <none>
Status:           Running
IP:               192.168.1.133
IPs:
  IP:           192.168.1.133
Controlled By:  ReplicaSet/strawhat-deploy-574bd47bc9
Init Containers:
  init-copy:
    Container ID:  containerd://ce7ab58825bb085ad5e88eabd1d7d551527768dddeecd8af0a8eeac6f951a083
    Image:         public.ecr.aws/docker/library/busybox:latest
    Image ID:      public.ecr.aws/docker/library/busybox@sha256:1487d0af5f52b4ba31c7e465126ee2123fe3f2305d638e7827681e7cf6c83d5e
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      cp /config/index.html /html/index.html
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Thu, 16 Apr 2026 01:24:43 +0000
      Finished:     Thu, 16 Apr 2026 01:24:43 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /config from config-volume (rw)
      /html from html-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-5lhtg (ro)
Containers:
  strawhat-nginx:
    Container ID:   containerd://6dd74568afc8c528f02c99266795ee8ac8eaff0eee4158315bb7afbc8bbd4561
    Image:          public.ecr.aws/nginx/nginx:latest
    Image ID:       public.ecr.aws/nginx/nginx@sha256:f9790ff1dd0637185c3df045e03072f59814c83b8479d345797dd6a9165c874c
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 16 Apr 2026 01:24:53 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html from html-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-5lhtg (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  html-volume:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
  config-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      strawhat-cm
    Optional:  false
  kube-api-access-5lhtg:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  32s   default-scheduler  Successfully assigned one-piece/strawhat-deploy-574bd47bc9-9st42 to node01
  Normal  Pulling    27s   kubelet            spec.initContainers{init-copy}: Pulling image "public.ecr.aws/docker/library/busybox:latest"
  Normal  Pulled     24s   kubelet            spec.initContainers{init-copy}: Successfully pulled image "public.ecr.aws/docker/library/busybox:latest" in 3.343s (3.343s including waiting). Image size: 2222002 bytes.
  Normal  Created    24s   kubelet            spec.initContainers{init-copy}: Container created
  Normal  Started    24s   kubelet            spec.initContainers{init-copy}: Container started
  Normal  Pulling    23s   kubelet            spec.containers{strawhat-nginx}: Pulling image "public.ecr.aws/nginx/nginx:latest"
  Normal  Pulled     14s   kubelet            spec.containers{strawhat-nginx}: Successfully pulled image "public.ecr.aws/nginx/nginx:latest" in 8.253s (8.253s including waiting). Image size: 63953006 bytes.
  Normal  Created    14s   kubelet            spec.containers{strawhat-nginx}: Container created
  Normal  Started    14s   kubelet            spec.containers{strawhat-nginx}: Container started
```

**Answer**

The Pod initialized correctly. The `init-copy` container completed successfully, copied the file into the shared volume, and then the `strawhat-nginx` container started.

**Interpretation**

This output is the core of the lab.

### Why the Pod first showed `PodInitializing`

The first `kubectl get pods` output shows:

- `READY 0/1`
- `STATUS PodInitializing`

That does **not** mean the Pod is broken.  
It means Kubernetes is still executing the startup sequence before the main application container is allowed to run.

With an `InitContainer`, this is expected behavior.

### What the `describe` output proves

The `describe` output shows the whole startup pipeline clearly:

- `init-copy` ran first
- it exited with:
  - `Reason: Completed`
  - `Exit Code: 0`
- only after that did `strawhat-nginx` move to `State: Running`

That is exactly how init containers are designed to work. They are **sequential startup gates**.

### Why the volumes matter

This Pod uses two different volume roles:

- `config-volume` → sourced from `strawhat-cm`
- `html-volume` → `emptyDir` shared inside the Pod

The flow is:

1. Kubernetes mounts the ConfigMap into `/config`
2. the init container copies `/config/index.html` to `/html/index.html`
3. the main Nginx container mounts the same shared `emptyDir` as `/usr/share/nginx/html`
4. Nginx serves the copied file as its website content

This is a stronger pattern than mounting the ConfigMap directly into the Nginx web root because it gives full control over file preparation before the application starts.

### A subtle but important detail

The Pod QoS is `BestEffort`.  
That means no CPU or memory requests/limits were defined for either container. That is acceptable for this lab, but in production this would usually be tightened with explicit resource settings.

---

## Task 4/5 — Verify that the `NodePort` Service was created correctly

**Task**

Check the Service and verify that it exposes port `80` through NodePort `32100`.

**Command**

```bash
kubectl -n one-piece get svc
kubectl get nodes -o wide
```

**Output**

```text
root@controlplane:~$ kubectl -n one-piece get svc
NAME           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
strawhat-svc   NodePort   10.106.76.173   <none>        80:32100/TCP   74s
```

```text
root@controlplane:~$ kubectl get nodes -o wide
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
controlplane   Ready    control-plane   14d   v1.35.1   172.30.1.2    <none>        Ubuntu 24.04.4 LTS   6.8.0-107-generic   containerd://1.7.28
node01         Ready    <none>          14d   v1.35.1   172.30.2.2    <none>        Ubuntu 24.04.4 LTS   6.8.0-107-generic   containerd://1.7.28
```

**Answer**

The Service `strawhat-svc` was created as a `NodePort` Service and exposes the application on port `32100`.

**Interpretation**

The `PORT(S)` column is very important:

- `80:32100/TCP`

This means:

- Service port: `80`
- NodePort: `32100`

So traffic arriving on any cluster node at port `32100` is forwarded by kube-proxy to the Service, and then to the selected Pod on container port `80`.

The node list matters because `NodePort` is tied to the node network, not just the Pod IP.  
That means the application becomes reachable through a node address plus the assigned node port.

Even when the Pod is rescheduled later, the Service abstraction remains stable. That is one of the major operational strengths of Kubernetes Services.

---

## Task 5/5 — Confirm the Deployment rollout and verify the served page

**Task**

Wait for the Deployment rollout to complete, then verify that the website is reachable through the NodePort and that the custom HTML page is being served.

**Command**

```bash
kubectl rollout status deployment/strawhat-deploy -n one-piece
curl localhost:32100
```

**Output**

```text
root@controlplane:~$ kubectl rollout status deployment/strawhat-deploy -n one-piece
deployment "strawhat-deploy" successfully rolled out
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>One Piece Terminal - Straw Hat Pirates Database</title>
    ...
</head>
<body>
    <div class="terminal">
        ...
        <div class="section-title">&gt;&gt; CREW OVERVIEW</div>
        ...
        <div class="character-name">[ 01 ] MONKEY D. LUFFY</div>
        ...
        <div class="character-name">[ 02 ] RORONOA ZORO</div>
        ...
        <div class="character-name">[ 10 ] JINBE</div>
        ...
    </div>
</body>
</html>
```

**Answer**

The Deployment rolled out successfully, and the custom Straw Hat Pirates HTML page is being served through the `NodePort` Service on port `32100`.

**Interpretation**

This final verification proves the whole chain worked end to end:

- the ConfigMap stored the source HTML
- the init container copied the file into the shared runtime volume
- the main Nginx container served that copied file
- the Service exposed the application externally
- the NodePort routing delivered the request correctly

A reader with deeper Kubernetes understanding should appreciate that this is not just “curl returns HTML.”  
It means multiple Kubernetes objects cooperated correctly:

- `ConfigMap`
- `Deployment`
- `InitContainer`
- `emptyDir`
- `Service`
- `NodePort`

When all of those pieces line up and the rendered page matches the expected custom content, the deployment architecture has been validated, not just the container status.

---

## Final manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: strawhat-deploy
  namespace: one-piece
spec:
  replicas: 1
  selector:
    matchLabels:
      app: strawhat
  template:
    metadata:
      labels:
        app: strawhat
    spec:
      initContainers:
        - name: init-copy
          image: public.ecr.aws/docker/library/busybox:latest
          command: ['sh', '-c', 'cp /config/index.html /html/index.html']
          volumeMounts:
            - name: config-volume
              mountPath: /config
            - name: html-volume
              mountPath: /html
      containers:
        - name: strawhat-nginx
          image: public.ecr.aws/nginx/nginx:latest
          ports:
            - containerPort: 80
          volumeMounts:
            - name: html-volume
              mountPath: /usr/share/nginx/html
      volumes:
        - name: config-volume
          configMap:
            name: strawhat-cm
        - name: html-volume
          emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: strawhat-svc
  namespace: one-piece
spec:
  type: NodePort
  selector:
    app: strawhat
  ports:
    - port: 80
      targetPort: 80
      nodePort: 32100
```

---

## What you should remember from this lab

### Explanation

An `InitContainer` runs before the main application container and must finish successfully before the Pod can continue startup. This makes it ideal for setup tasks such as copying files, waiting for dependencies, or generating runtime content.

A `ConfigMap` is a convenient way to inject non-sensitive configuration or static files into a Pod. When combined with an `InitContainer` and a shared `emptyDir`, it creates a clean workflow for preparing application content before the main container starts.

A `NodePort` Service exposes the application through a fixed port on each node, making external access possible without directly targeting Pod IPs.

### Key takeaway

This lab is a strong example of Kubernetes separation of concerns:

- configuration lives in a `ConfigMap`
- preparation work happens in an `InitContainer`
- runtime serving is handled by Nginx
- network exposure is handled by a `Service`

Each object has a focused responsibility, and Kubernetes coordinates them into a working application.

### Final recap

This lab covered how to:

- create a `ConfigMap` from a file
- build a Deployment with an `InitContainer`
- use `emptyDir` as a shared handoff volume between containers
- understand why a Pod can temporarily show `PodInitializing`
- inspect an init container completion state with `kubectl describe`
- expose an application with a `NodePort` Service
- verify end-to-end delivery of custom HTML content
