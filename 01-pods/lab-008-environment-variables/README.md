# Environment Variables in Kubernetes

**Level:** Intermediate  
**Duration:** 15–25 minutes  
**Mode:** Interactive  
**Source:** KodeKloud Engineer  
**Reference URL:** https://github.com/MederD/Kodekloud-Engineer-Tasks

Learn how to inject runtime metadata into a Pod as environment variables using `valueFrom.fieldRef`, then verify those values from inside the running container with `kubectl exec -- printenv`.

## Short introduction

In this lab, a Pod named `envars` is created with one container named `fieldref-container`.  
The objective is not just to set static environment variables, but to expose **live Pod and node metadata** directly inside the container.

The container receives four environment variables:

- `NODE_NAME` from `spec.nodeName`
- `POD_NAME` from `metadata.name`
- `POD_IP` from `status.podIP`
- `POD_SERVICE_ACCOUNT` from `spec.serviceAccountName`

This is a practical use of Kubernetes **field references** through the Downward API. The value is not hardcoded in the image and not manually exported in the shell. Kubernetes injects it from the Pod object at runtime.

---

## Tasks section

## Task 1/4 — Inspect the current cluster state

**Task**

Check the current resources before creating the Pod.

**Command**

```bash
kubectl get all
```

**Output**

```text
thor@jump_host ~$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   6h41m
```

**Answer**

The cluster has no application workload for this lab yet. Only the default Kubernetes API Service is visible.

**Interpretation**

This output is small, but it tells an experienced reader something useful immediately:

- there is no previously created Pod to interfere with the task
- the environment is clean for verification
- the only visible Service is the standard `kubernetes` Service, which is created automatically in every cluster

That baseline matters. It means any Pod appearing after the apply step belongs to this exercise and can be traced without ambiguity.

---

## Task 2/4 — Create the `envars` Pod manifest with fieldRef-based environment variables

**Task**

Create a Pod named `envars` with:

- container name `fieldref-container`
- image `httpd:latest`
- command `sh -c`
- a loop that prints the environment variables every 10 seconds
- restart policy `Never`

Define these four environment variables with `valueFrom.fieldRef`:

- `NODE_NAME` → `spec.nodeName`
- `POD_NAME` → `metadata.name`
- `POD_IP` → `status.podIP`
- `POD_SERVICE_ACCOUNT` → `spec.serviceAccountName`

**Command**

```bash
nano pod.yaml
```

**Output**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envars
spec:
  restartPolicy: Never
  containers:
    - name: fieldref-container
      image: httpd:latest
      command: ["sh", "-c"]
      args:
        - while true; do
          echo -en '\n';
          printenv NODE_NAME POD_NAME;
          printenv POD_IP POD_SERVICE_ACCOUNT;
          sleep 10;
          done;
      env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: POD_SERVICE_ACCOUNT
          valueFrom:
            fieldRef:
              fieldPath: spec.serviceAccountName
```

**Answer**

The manifest defines a Pod that uses `fieldRef` to inject live Kubernetes object data as environment variables.

**Interpretation**

This is the technical heart of the lab.

These values are **not** static literals like:

```yaml
env:
  - name: NODE_NAME
    value: kodekloud-control-plane
```

Instead, the Pod asks Kubernetes to resolve them dynamically from its own object fields.

That changes the nature of the configuration:

- the manifest stays portable
- the same YAML can run on another node without editing
- the Pod name, IP, node, and service account are discovered at runtime
- the container becomes aware of where and how it is running without querying the API manually

This is a much stronger pattern than hardcoding values, and it is one of the cleanest examples of Kubernetes metadata injection.

---

## Task 3/4 — Apply the Pod manifest and verify that the Pod is running

**Task**

Create the Pod from the manifest and confirm that it reaches the `Running` state.

**Command**

```bash
kubectl apply -f pod.yaml
kubectl get pod
```

**Output**

```text
thor@jump_host ~$ kubectl apply -f pod.yaml
pod/envars created
```

```text
thor@jump_host ~$ kubectl get pod
NAME     READY   STATUS    RESTARTS   AGE
envars   1/1     Running   0          13s
```

**Answer**

The Pod `envars` was created successfully and is running with one ready container.

**Interpretation**

This confirms more than simple creation.

Because the container command runs an infinite loop, the Pod remains alive long enough to inspect it interactively. That is important for this kind of lab: if the process exited immediately, the Pod would complete too fast and verification would be less useful.

The `1/1 Running` state also confirms:

- the image started correctly
- the shell command syntax is valid
- the environment variables did not break container startup
- the Pod is now ready for `kubectl exec`

Also note the deliberate choice of `restartPolicy: Never`. Even though the current loop keeps the Pod alive, this setting makes the runtime behavior explicit: if the process stops, Kubernetes should not restart the Pod automatically.

---

## Task 4/4 — Verify the injected values from inside the container

**Task**

Exec into the Pod and print the environment variables to confirm that the metadata was injected correctly.

**Command**

```bash
kubectl exec -it envars -- printenv
```

**Output**

```text
thor@jump_host ~$ kubectl exec -it envars -- printenv
PATH=/usr/local/apache2/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=envars
HTTPD_PREFIX=/usr/local/apache2
HTTPD_VERSION=2.4.57
NODE_NAME=kodekloud-control-plane
POD_NAME=envars
POD_IP=10.244.0.5
POD_SERVICE_ACCOUNT=default
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
HOME=/root
```

**Answer**

The environment variables were injected successfully:

- `NODE_NAME` contains the node hosting the Pod
- `POD_NAME` contains the Pod name
- `POD_IP` contains the current Pod IP
- `POD_SERVICE_ACCOUNT` contains the ServiceAccount name

**Interpretation**

This is where the lab becomes operationally meaningful.

### What the custom variables prove

The four required variables confirm that Kubernetes correctly resolved values from different parts of the Pod object:

- scheduling metadata → `spec.nodeName`
- object identity → `metadata.name`
- runtime network assignment → `status.podIP`
- workload identity → `spec.serviceAccountName`

That means the container now has direct awareness of its execution context without any external script or API call.

### What the other variables reveal

A deeper reader will notice that the output contains more than the four custom variables.

For example:

- `HOSTNAME=envars`
- `KUBERNETES_SERVICE_HOST=10.96.0.1`
- `KUBERNETES_SERVICE_PORT=443`

These are not from the custom `env:` block. They come from normal container runtime behavior and Kubernetes service environment injection.

This distinction is important:

- some variables are injected because **you explicitly defined them**
- others appear because **Kubernetes and the container image provide them automatically**

Understanding that difference is what makes troubleshooting easier in real clusters.

### Why `kubectl exec -- printenv` is the right verification

Looking only at the YAML tells you the desired configuration.  
Looking at `printenv` tells you the **effective runtime state**.

That is the stronger validation.

---

## Final manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envars
spec:
  restartPolicy: Never
  containers:
    - name: fieldref-container
      image: httpd:latest
      command: ["sh", "-c"]
      args:
        - while true; do
          echo -en '\n';
          printenv NODE_NAME POD_NAME;
          printenv POD_IP POD_SERVICE_ACCOUNT;
          sleep 10;
          done;
      env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: POD_SERVICE_ACCOUNT
          valueFrom:
            fieldRef:
              fieldPath: spec.serviceAccountName
```

---

## What you should remember from this lab

### Explanation

Kubernetes can inject Pod and node metadata into containers using `valueFrom.fieldRef`. This is part of the Downward API and allows a container to access information about its own environment without requiring hardcoded values or direct API queries.

In this lab, the container received:

- the node where it was scheduled
- its own Pod name
- its current IP address
- its ServiceAccount name

These values came directly from the live Pod object.

### Key takeaway

A strong Kubernetes manifest avoids hardcoding runtime-specific values whenever possible.  
Using `fieldRef` keeps the workload portable, reusable, and environment-aware.

It also shows an important engineering principle: **verify runtime truth, not just YAML intent**.  
`kubectl exec -- printenv` confirms what the container actually sees.

### Final recap

This lab covered how to:

- inspect a clean cluster state before deployment
- create a Pod with explicit `fieldRef` environment variables
- inject node, Pod, network, and identity metadata into a container
- keep the container alive for inspection with a looping shell command
- verify the live environment using `kubectl exec -- printenv`
- distinguish custom injected variables from automatically provided runtime variables
