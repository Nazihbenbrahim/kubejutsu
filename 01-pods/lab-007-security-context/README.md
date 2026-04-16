# Pod & Container SecurityContext (Non-Root + ReadOnlyRootFS)

**Level:** Beginner to Intermediate  
**Duration:** 15–25 minutes  
**Mode:** Interactive  
**Source:** Killercoda  
**Lab URL:** https://killercoda.com/omkar-shelke25/scenario/security-context

Learn how to apply Kubernetes security contexts at both the Pod and container levels to enforce least-privilege execution, run a workload as a non-root user, and protect the container filesystem from unintended writes.

## Short introduction

This lab focuses on a foundational Kubernetes security pattern: separating security controls by scope.

At the **Pod level**, the workload is forced to run as a non-root identity:

- `runAsUser: 1000`
- `runAsGroup: 3000`
- `runAsNonRoot: true`

At the **container level**, the root filesystem is made read-only with:

- `readOnlyRootFilesystem: true`

The result is a Pod that runs successfully while respecting two important hardening rules:

- the process does not run as root
- the container cannot freely write to `/`

This is a small manifest, but it demonstrates an essential production mindset: security should be explicit, verifiable, and enforced as close as possible to the workload definition.

---

## Tasks section

## Task 1/5 — Confirm that the `security` namespace already exists

**Task**

Verify the target namespace before creating the Pod.

**Command**

```bash
kubectl create namespace security
```

**Output**

```text
root@controlplane:~$ kubectl create namespace security
Error from server (AlreadyExists): namespaces "security" already exists
```

**Answer**

The `security` namespace already exists and is ready to be used.

**Interpretation**

This is not a failure that blocks progress. It is a useful environmental confirmation.

A stronger operational reading is:

- the target namespace is already prepared
- there is no need to create it again
- the manifest should explicitly target `namespace: security`

This matters because security-related workloads often live in dedicated namespaces, and verifying the namespace early avoids accidentally creating resources in `default`.

---

## Task 2/5 — Create the `secure-app-pod` manifest with Pod-level and container-level security settings

**Task**

Create a Pod manifest named `secure-app-pod.yaml` with these requirements:

- namespace: `security`
- Pod-level security context:
  - `runAsUser: 1000`
  - `runAsGroup: 3000`
  - `runAsNonRoot: true`
- one container named `app-container`
- image: `busybox:1.36`
- command: `sleep 3600`
- container-level security context:
  - `readOnlyRootFilesystem: true`

**Command**

```bash
nano secure-app-pod.yaml
```

**Output**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-app-pod
  namespace: security
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    runAsNonRoot: true
  containers:
  - name: app-container
    image: busybox:1.36
    command: ["/bin/sh","-c","sleep 3600"]
    securityContext:
      readOnlyRootFilesystem: true
```

**Answer**

The manifest defines both Pod-level identity controls and a container-level read-only root filesystem.

**Interpretation**

This is where the security model becomes interesting.

The manifest intentionally splits responsibility across two levels:

### Pod-level controls

These settings apply to the Pod by default:

- `runAsUser: 1000`
- `runAsGroup: 3000`
- `runAsNonRoot: true`

That means every container in the Pod inherits a non-root identity unless a container-level override is defined.

### Container-level controls

The container adds:

- `readOnlyRootFilesystem: true`

This is a different category of protection. It does not control *who* the process runs as. It controls *what the process can write to* inside the image filesystem.

That separation is important. Identity and filesystem mutability are different security dimensions, and Kubernetes lets you enforce them independently.

---

## Task 3/5 — Apply the Pod manifest and verify that the Pod is running

**Task**

Create the Pod in the `security` namespace and confirm that it starts successfully.

**Command**

```bash
kubectl apply -f secure-app-pod.yaml
kubectl get pods -n security
```

**Output**

```text
root@controlplane:~$ kubectl apply -f secure-app-pod.yaml
pod/secure-app-pod created

root@controlplane:~$ kubectl get pods -n security
NAME             READY   STATUS    RESTARTS   AGE
secure-app-pod   1/1     Running   0          81s
```

**Answer**

The Pod was created successfully and is running in the `security` namespace.

**Interpretation**

This output proves that the security configuration is valid for the chosen image and command.

That is more significant than it looks.

A hardened Pod can fail to start if:

- the image expects root privileges
- the process needs to write to protected filesystem paths
- the configured UID/GID is incompatible with the container runtime expectations

Since the Pod is `Running`, Kubernetes has accepted the security constraints and the container was able to start under them.

This is a good reminder that security settings should always be validated operationally, not just declared in YAML.

---

## Task 4/5 — Inspect the live Pod definition and confirm both security contexts

**Task**

Verify that the Pod includes the expected security context values at both levels.

**Command**

```bash
kubectl get pod secure-app-pod -n security -o yaml | grep -A5 securityContext
```

**Output**

```text
root@controlplane:~$ kubectl get pod secure-app-pod -n security -o yaml | grep -A5 securityContext
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"secure-app-pod","namespace":"security"},"spec":{"containers":[{"command":["/bin/sh","-c","sleep 3600"],"image":"busybox:1.36","name":"app-container","securityContext":{"readOnlyRootFilesystem":true}}],"securityContext":{"runAsGroup":3000,"runAsNonRoot":true,"runAsUser":1000}}}
  creationTimestamp: "2026-04-16T09:12:21Z"
  generation: 1
  name: secure-app-pod
  namespace: security
  resourceVersion: "4688"
--
    securityContext:
      readOnlyRootFilesystem: true
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
--
  securityContext:
    runAsGroup: 3000
    runAsNonRoot: true
    runAsUser: 1000
  serviceAccount: default
  serviceAccountName: default
```

**Answer**

The live Pod confirms:

- container-level `readOnlyRootFilesystem: true`
- Pod-level `runAsUser: 1000`
- Pod-level `runAsGroup: 3000`
- Pod-level `runAsNonRoot: true`

**Interpretation**

This step is important because it verifies the *effective live object*, not just the local file.

That distinction matters in Kubernetes. A manifest on disk is only intent. The object stored by the API server is the actual source of truth.

This output also reinforces the precedence model:

- Pod-level security context provides default identity settings
- container-level security context applies directly to that container
- if overlapping fields existed at both levels, the container-level value would win

In this lab there is no conflict, so both layers work together cleanly.

---

## Task 5/5 — Enter the container and verify the security behavior directly

**Task**

Exec into the running container, verify the runtime identity, and confirm that writing to `/` fails because the root filesystem is read-only.

**Command**

```bash
kubectl exec -n security -it secure-app-pod -- sh
id
touch /newfile
exit
```

**Output**

```text
root@controlplane:~$ kubectl exec -n security -it secure-app-pod -- sh
~ $ 
~ $ id
uid=1000 gid=3000 groups=3000
~ $ 
~ $ touch /newfile
touch: /newfile: Read-only file system
~ $ exit
```

**Answer**

The container is running as UID `1000` and GID `3000`, and writing to `/newfile` fails with `Read-only file system`, which confirms the hardening is effective.

**Interpretation**

This is the strongest verification in the lab because it tests behavior from inside the running container.

### What `id` proves

```text
uid=1000 gid=3000 groups=3000
```

This confirms that the process is *actually* running under the non-root identity declared in the Pod security context.

This is the runtime proof of least privilege.

### What `touch /newfile` proves

```text
touch: /newfile: Read-only file system
```

This proves the root filesystem protection is not just declarative metadata. It is enforced by the container runtime.

That is a very important distinction. Security is only meaningful when it changes runtime behavior.

### Why this matters operationally

A non-root process reduces privilege exposure.  
A read-only root filesystem reduces accidental writes, tampering, and persistence opportunities.

Used together, these controls significantly narrow the attack surface of even a simple container.

---

## Final manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-app-pod
  namespace: security
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    runAsNonRoot: true
  containers:
    - name: app-container
      image: busybox:1.36
      command: ["/bin/sh", "-c", "sleep 3600"]
      securityContext:
        readOnlyRootFilesystem: true
```

---

## What you should remember from this lab

### Explanation

Kubernetes security contexts let you define workload execution rules at different scopes.

- **Pod-level security context** is a shared default for containers in the Pod
- **Container-level security context** applies only to the individual container

In this lab:

- the Pod-level settings enforced non-root execution with explicit UID and GID
- the container-level setting made the root filesystem read-only

That combination implements least privilege in a practical and verifiable way.

### Key takeaway

Good container security is not only about choosing a safe image. It is also about restricting how that image runs:

- who the process runs as
- whether it can write to the image filesystem
- whether it needs privileges it should not have

This lab shows that even a minimal Pod can be hardened meaningfully with only a few YAML lines.

### Final recap

This lab covered how to:

- target a pre-existing namespace correctly
- define Pod-level security defaults
- enforce non-root execution with explicit UID and GID
- add a container-level `readOnlyRootFilesystem`
- verify live security context values from the API
- confirm runtime identity with `id`
- validate filesystem protection with a failed write attempt
