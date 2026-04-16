# QoS Classes and Resource Management

**Level:** Beginner  
**Duration:** 20–30 minutes  
**Mode:** Interactive  
**Source:** Killercoda  
**Lab URL:** https://killercoda.com/omkar-shelke25/scenario/qos-scenario

Learn how to create a Deployment with explicit resource requests and limits, verify that the resulting Pods run with the expected QoS class, and build a small helper script to inspect Pod QoS information quickly.

## Short introduction

In this lab, a microservice is deployed in the `mars` namespace with CPU and memory requests and limits chosen to produce the **Burstable** QoS class. After the deployment is running, a small shell script is created to print all Pod names in the namespace together with their QoS class for quick monitoring.

---

## Tasks section

## Task 1/3 — Create the `app-server` Deployment in the `mars` namespace

**Task**

Create a Deployment named `app-server` in the `mars` namespace using the image `public.ecr.aws/nginx/nginx:stable-alpine` with `3` replicas.  
Configure the container with these resources:

- CPU request: `200m`
- Memory request: `128Mi`
- CPU limit: `500m`
- Memory limit: `256Mi`

**Command**

```bash
nano app-server.yaml
kubectl apply -f app-server.yaml
kubectl get pods -n mars -o wide
```

**Output**

```text
root@controlplane:~$ kubectl apply -f app-server.yaml
deployment.apps/app-server created

root@controlplane:~$ kubectl get pods -n mars -o wide
NAME                          READY   STATUS              RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
app-server-545c6cc45f-ng49d   0/1     ContainerCreating   0          12s   <none>   node01   <none>           <none>
app-server-545c6cc45f-prjb2   0/1     ContainerCreating   0          12s   <none>   node01   <none>           <none>
app-server-545c6cc45f-xhphz   0/1     ContainerCreating   0          12s   <none>   node01   <none>           <none>
```

**Answer**

Create a Deployment with 3 replicas in the `mars` namespace and set both requests and limits in the container resource configuration.

---

## Task 2/3 — Confirm that the Pods run with QoS class `Burstable`

**Task**

Verify that the Pods created by the Deployment are running with the expected QoS class.

**Command**

```bash
kubectl describe pod app-server-545c6cc45f-ng49d -n mars | grep -i qos
```

**Output**

```text
QoS Class:                   Burstable
```

**Answer**

The Pods run with QoS class `Burstable`.

---

## Task 3/3 — Create a script to list Pod names and QoS class in the `mars` namespace

**Task**

Create a script file at `/opt/mars/qos-check.sh` that prints all Pod names and their QoS class in the `mars` namespace.

**Command**

```bash
mkdir -p /opt/mars
nano /opt/mars/qos-check.sh
chmod +x /opt/mars/qos-check.sh
/opt/mars/qos-check.sh
```

**Output**

```text
NAME                          QOS
app-server-545c6cc45f-ng49d   Burstable
app-server-545c6cc45f-prjb2   Burstable
app-server-545c6cc45f-xhphz   Burstable
```

**Answer**

Create a shell script that runs a `kubectl get pods` command with custom columns for `NAME` and `QOS`.

---

## Final deployment manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-server
  namespace: mars
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app-server
  template:
    metadata:
      labels:
        app: app-server
    spec:
      containers:
        - name: nginx
          image: public.ecr.aws/nginx/nginx:stable-alpine
          resources:
            requests:
              cpu: "200m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
```

## Final script

```bash
#!/bin/bash

kubectl get pods -n mars \
  -o custom-columns=NAME:.metadata.name,QOS:.status.qosClass
```

---

## What you should remember from this lab

### Explanation

Kubernetes assigns a QoS class to each Pod based on the resource requests and limits defined for its containers. A Pod becomes **Burstable** when it has requests and/or limits configured, but it does not meet the stricter requirements of the **Guaranteed** class. This is a practical middle ground for many workloads because it reserves baseline resources while still allowing controlled bursts.

### Key takeaway

Resource requests influence scheduling, resource limits cap maximum usage, and QoS class reflects how Kubernetes treats the Pod during resource pressure. For quick operational checks, `kubectl get pods` with custom columns is a simple and effective monitoring pattern.

### Final recap

This lab covered how to:

- create a Deployment in a specific namespace
- configure CPU and memory requests and limits
- verify that a Pod runs as `Burstable`
- use `kubectl describe` to inspect QoS information
- build a reusable script to list Pod names and their QoS class
