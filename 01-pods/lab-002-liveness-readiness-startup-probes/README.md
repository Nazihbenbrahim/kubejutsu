# Configure Readiness and Liveness Probes for a Deployment

**Level:** Intermediate  
**Duration:** 20 minutes  
**Mode:** Interactive  
**Source:** KillerCoda  
**Lab URL:** https://killercoda.com/omkar-shelke25/scenario/readiness-liveness


## Short introduction

This lab focuses on adding health checks to an existing Deployment in Kubernetes. Readiness and liveness probes help Kubernetes understand when a container is ready to receive traffic and when it should be restarted. In this scenario, the `warp-core` Deployment in the `galaxy` namespace is updated with HTTP-based probes on port `80`, then verified through rollout status, Deployment details, and application logs.

## Tasks

### Task 1/4 — Add readiness and liveness probes to the `warp-core` Deployment

**Task**

Update the existing `warp-core` Deployment in the `galaxy` namespace and add the required HTTP probes to the `httpd` container.

- `readinessProbe`
  - path: `/readyz`
  - port: `80`
  - `initialDelaySeconds: 2`
  - `periodSeconds: 5`
- `livenessProbe`
  - path: `/helathz`
  - port: `80`
  - `initialDelaySeconds: 5`
  - `periodSeconds: 10`
  - `failureThreshold: 3`

**Command**

```bash
kubectl -n galaxy edit deployment warp-core
```

**Output**

```text
root@controlplane:~$ kubectl -n galaxy edit deployment warp-core
deployment.apps/warp-core edited
```

**Answer**

Add the following probe configuration under the `httpd` container:

```yaml
readinessProbe:
  httpGet:
    path: /readyz
    port: 80
  initialDelaySeconds: 2
  periodSeconds: 5
livenessProbe:
  httpGet:
    path: /helathz
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 10
  failureThreshold: 3
```

---

### Task 2/4 — Confirm that the Deployment rolls out successfully

**Task**

Verify that the updated Deployment is applied successfully in the `galaxy` namespace and that the Pods are recreated in a healthy state.

**Command**

```bash
kubectl rollout status deployment warp-core -n galaxy
kubectl -n galaxy get deployment warp-core
kubectl -n galaxy get pods
```

**Output**

```text
root@controlplane:~$ kubectl rollout status deployment warp-core -n galaxy
deployment "warp-core" successfully rolled out

root@controlplane:~$ kubectl -n galaxy get deployment warp-core
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
warp-core   2/2     2            2           33m

root@controlplane:~$ kubectl -n galaxy get pods
NAME                        READY   STATUS    RESTARTS   AGE
warp-core-5974b7ff96-lzqc9  1/1     Running   0          13s
warp-core-5974b7ff96-x8hl2  1/1     Running   0          8s
```

**Answer**

The Deployment rollout completed successfully, and the new Pods reached the `Running` state.

---

### Task 3/4 — Verify the probe configuration on the Deployment

**Task**

Inspect the Deployment and confirm that both probes are present with the expected paths, delays, and timing values.

**Command**

```bash
kubectl -n galaxy describe deployment warp-core
```

**Output**

```text
root@controlplane:~$ kubectl -n galaxy describe deployment warp-core
Name:                   warp-core
Namespace:              galaxy
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
Pod Template:
  Labels:  app=warp-core
  Containers:
   httpd:
    Image:      public.ecr.aws/docker/library/httpd:latest
    Port:       80/TCP
    Liveness:   http-get http://:80/helathz delay=5s timeout=1s period=10s #success=1 #failure=3
    Readiness:  http-get http://:80/readyz delay=2s timeout=1s period=5s #success=1 #failure=3
    Mounts:
      /usr/local/apache2/htdocs from health-pages (rw)
```

**Answer**

The Deployment is configured correctly:

- the readiness probe checks `/readyz` on port `80`
- the liveness probe checks `/helathz` on port `80`
- the timing values match the lab requirements

---

### Task 4/4 — Check the application logs and verify successful HTTP probe responses

**Task**

Review the Deployment logs to confirm that Apache is running and that both `/readyz` and `/helathz` return `HTTP 200 OK`.

**Command**

```bash
kubectl logs deployment/warp-core -n galaxy
```

**Output**

```text
root@controlplane:~$ kubectl logs deployment/warp-core -n galaxy
Found 2 pods, using pod/warp-core-5974b7ff96-lzqc9
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 192.168.1.20. Set the 'ServerName' directive globally to suppress this message
[Tue Apr 14 16:08:07.377507 2026] [mpm_event:notice] [pid 1:tid 1] AH00489: Apache/2.4.66 (Unix) configured -- resuming normal operations
[Tue Apr 14 16:08:07.379270 2026] [core:notice] [pid 1:tid 1] AH00094: Command line: 'httpd -D FOREGROUND'
192.168.1.58 - - [14/Apr/2026:16:08:09 +0000] "GET /readyz HTTP/1.1" 200 5
192.168.1.58 - - [14/Apr/2026:16:08:15 +0000] "GET /helathz HTTP/1.1" 200 2
192.168.1.58 - - [14/Apr/2026:16:08:19 +0000] "GET /readyz HTTP/1.1" 200 5
192.168.1.58 - - [14/Apr/2026:16:08:25 +0000] "GET /helathz HTTP/1.1" 200 2
```

**Answer**

The logs confirm two important results:

- Apache started successfully and is serving traffic
- both probe endpoints, `/readyz` and `/helathz`, return `HTTP 200 OK`

---

## What you should remember from this lab

### Explanation

A readiness probe tells Kubernetes when a container is ready to receive traffic. A liveness probe tells Kubernetes when a container is unhealthy and should be restarted. These two probe types work together to improve application reliability.

### Key takeaway

Health checks should point to real application endpoints and use timing values that match the startup and runtime behavior of the workload.

### Final recap

In this lab, the `warp-core` Deployment in the `galaxy` namespace was updated with a readiness probe on `/readyz` and a liveness probe on `/helathz`. The rollout completed successfully, the Pods became available, and the logs confirmed repeated `HTTP 200 OK` responses from both endpoints.
