# Buenos Aires: Kubernetes Pod Crashing

**Level:** Intermediate  
**Duration:** 20–30 minutes  
**Mode:** Troubleshooting  
**Source:** SadServers  
**Lab URL:** https://sadservers.com/newserver/buenos-aires

Diagnose a `CrashLoopBackOff` Pod without modifying its Pod definition, identify an RBAC permission problem, grant the missing access to its ServiceAccount, and verify that the workload recovers.

## Short introduction

In this scenario, two Pods exist in the `default` namespace:

- `logger`
- `logshipper`

The `logshipper` Pod is crashlooping. The key constraint is important: the Pod definition itself should **not** be changed.

That means the investigation must focus on the runtime environment around the Pod rather than on rebuilding or editing the workload. The terminal evidence reveals that `logshipper` is failing because its ServiceAccount cannot read Pod logs. Once the missing RBAC permissions are granted, the application can access the logs it depends on and becomes healthy again.

This is a strong troubleshooting exercise because it tests the ability to separate:

- **application crash symptoms**
from
- **cluster permission causes**

---

## Tasks section

## Task 1/6 — Confirm that `logshipper` is crashing

**Task**

Check the current Pods and confirm that the `logshipper` workload is not healthy.

**Command**

```bash
sudo kubectl get pods
```

**Output**

```text
admin@ip-10-1-12-37:~$ sudo kubectl get pods
NAME                          READY   STATUS             RESTARTS        AGE
logger-6f7fb76c9f-4jk77       1/1     Running            1 (4m21s ago)   2y7d
logshipper-597f84bf4f-6ssjq   0/1     CrashLoopBackOff   9 (82s ago)     2y7d
```

**Answer**

The `logshipper` Pod is unhealthy and in `CrashLoopBackOff`, while `logger` is running normally.

**Interpretation**

This first output already narrows the problem significantly:

- the cluster itself is not broadly broken, because `logger` is healthy
- the issue is isolated to `logshipper`
- repeated restarts indicate that the container starts, fails, and is restarted by Kubernetes

A stronger operational reading is that this is **not** a scheduling issue, **not** an image pull issue, and **not** a namespace-wide outage.  
The problem is inside the runtime path of one workload.

---

## Task 2/6 — Read the failing container logs and identify the real error

**Task**

Inspect the container logs of the crashing Pod to determine why it cannot start successfully.

**Command**

```bash
sudo kubectl logs logshipper-597f84bf4f-6ssjq
```

**Output**

```text
admin@ip-10-1-12-37:~$ sudo kubectl logs logshipper-597f84bf4f-6ssjq
Exception when calling CoreV1Api->read_namespaced_pod_log: (403)
Reason: Forbidden
HTTP response headers: HTTPHeaderDict({'Audit-Id': '81ef8491-41fe-4594-b9e3-ccea472e3607', 'Cache-Control': 'no-cache, private', 'Content-Type': 'application/json', 'X-Content-Type-Options': 'nosniff', 'Date': 'Thu, 16 Apr 2026 10:05:52 GMT', 'Content-Length': '352'})
HTTP response body: {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"pods \"logger-6f7fb76c9f-4jk77\" is forbidden: User \"system:serviceaccount:default:logshipper-sa\" cannot get resource \"pods/log\" in API group \"\" in the namespace \"default\"","reason":"Forbidden","details":{"name":"logger-6f7fb76c9f-4jk77","kind":"pods"},"code":403}
```

**Answer**

The application is failing because the ServiceAccount `logshipper-sa` does not have permission to read `pods/log` in the `default` namespace.

**Interpretation**

This is the decisive clue.

The container is not crashing because of:

- a bad image
- a syntax error in its command
- missing CPU or memory
- a missing ConfigMap or Secret

It is crashing because the application itself makes a Kubernetes API call to read logs from the `logger` Pod, and that API call is rejected with `403 Forbidden`.

That means the root cause is **authorization**, not application logic.

This is exactly the kind of issue that can be misdiagnosed if someone only looks at `CrashLoopBackOff` and assumes the container image is broken. The log output makes it clear that the real failure happens at the RBAC layer.

---

## Task 3/6 — Confirm which ServiceAccount the Pod is using

**Task**

Inspect the Pod to verify which ServiceAccount the application is running as.

**Command**

```bash
sudo kubectl get pod logshipper-597f84bf4f-6ssjq -o jsonpath='{.spec.serviceAccountName}'
sudo kubectl describe pod logshipper-597f84bf4f-6ssjq
```

**Output**

```text
admin@ip-10-1-12-37:~$ sudo kubectl get pod logshipper-597f84bf4f-6ssjq -o jsonpath='{.spec.serviceAccountName}'
logshipper-sa
```

```text
admin@ip-10-1-12-37:~$ sudo kubectl describe pod logshipper-597f84bf4f-6ssjq
Name:             logshipper-597f84bf4f-6ssjq
Namespace:        default
Priority:         0
Service Account:  logshipper-sa
Node:             node1/10.1.12.37
Start Time:       Mon, 08 Apr 2024 17:47:14 +0000
Labels:           app=logshipper
                  pod-template-hash=597f84bf4f
Annotations:      <none>
Status:           Running
IP:               10.42.0.21
IPs:
  IP:           10.42.0.21
Controlled By:  ReplicaSet/logshipper-597f84bf4f
Containers:
  logshipper:
    Container ID:  containerd://f9fac70b8d62cbab9a68d28cd692ee99c68abc75a573b21fe606d7cd26521982
    Image:         logshipper:v3
    Image ID:      sha256:34b78cd564e27b562297d66a0ed92c9224c4b32ee6c7911347f5a2e9a8e298ba
    Port:          <none>
    Host Port:     <none>
    Command:
      /usr/bin/python3
      logshipper.py
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    1
      Started:      Thu, 16 Apr 2026 10:05:52 +0000
      Finished:     Thu, 16 Apr 2026 10:05:53 +0000
    Ready:          False
    Restart Count:  9
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-slts5 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  kube-api-access-slts5:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason          Age                    From               Message
  ----     ------          ----                   ----               -------
  Normal   Scheduled       2y7d                   default-scheduler  Successfully assigned default/logshipper-597f84bf4f-6ssjq to node1
  Normal   Pulled          2y7d (x5 over 2y7d)    kubelet            Container image "logshipper:v3" already present on machine
  Normal   Created         2y7d (x5 over 2y7d)    kubelet            Created container logshipper
  Normal   Started         2y7d (x5 over 2y7d)    kubelet            Started container logshipper
  Warning  BackOff         2y7d (x9 over 2y7d)    kubelet            Back-off restarting failed container logshipper in pod logshipper-597f84bf4f-6ssjq_default(5256275f-86c4-4a44-bb49-0123cb010748)
  Normal   SandboxChanged  5m14s                  kubelet            Pod sandbox changed, it will be killed and re-created.
  Normal   Pulled          3m39s (x4 over 5m13s)  kubelet            Container image "logshipper:v3" already present on machine
  Normal   Created         3m39s (x4 over 5m12s)  kubelet            Created container logshipper
  Normal   Started         3m39s (x4 over 5m12s)  kubelet            Started container logshipper
  Warning  BackOff         1s (x24 over 5m9s)     kubelet            Back-off restarting failed container logshipper in pod logshipper-597f84bf4f-6ssjq_default(5256275f-86c4-4a44-bb49-0123cb010748)
```

**Answer**

The Pod uses the ServiceAccount `logshipper-sa`, and the container exits with code `1` after failing to perform its log-reading action.

**Interpretation**

This step connects the symptom to the identity.

The `403 Forbidden` message named `logshipper-sa`, and now the Pod inspection confirms that this is indeed the identity the workload runs under. That is the missing link between:

- the application crash
and
- the RBAC subject that must be fixed

Also note the container details:

- `Last State: Terminated`
- `Reason: Error`
- `Exit Code: 1`

That means the application fails deterministically and exits cleanly with an application error, rather than being killed by the kernel or by resource pressure. This supports the RBAC diagnosis even more strongly.

---

## Task 4/6 — Create a Role that allows reading Pod logs

**Task**

Create an RBAC `Role` in the `default` namespace that grants access to `pods/log`.

**Command**

```bash
cat <<'EOF' > role-logshipper.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: logshipper-read-logs
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get", "list"]
EOF

sudo kubectl apply -f role-logshipper.yaml
```

**Output**

```text
admin@ip-10-1-12-37:~$ sudo kubectl apply -f role-logshipper.yaml
role.rbac.authorization.k8s.io/logshipper-read-logs created
```

**Answer**

A namespaced Role was created to allow log access through the `pods/log` resource.

**Interpretation**

This is a very precise RBAC fix.

Instead of giving broad permissions like:

- access to all Pods
- wide API access
- cluster-admin privileges

the Role grants only what the application actually needs:

- resource: `pods/log`
- verbs: `get`, `list`

That is the right least-privilege approach.

A reader with strong Kubernetes instincts will notice that the fix is intentionally narrow. The problem was not “the Pod needs more power.” The problem was “the Pod needs one specific permission.” Good RBAC design fixes exactly that and nothing more.

---

## Task 5/6 — Bind the Role to `logshipper-sa`

**Task**

Create a `RoleBinding` that assigns the new Role to the ServiceAccount used by `logshipper`.

**Command**

```bash
cat <<'EOF' > rolebinding-logshipper.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: logshipper-read-logs-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: logshipper-sa
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: logshipper-read-logs
EOF

sudo kubectl apply -f rolebinding-logshipper.yaml
sudo kubectl get pods
```

**Output**

```text
admin@ip-10-1-12-37:~$ sudo kubectl apply -f rolebinding-logshipper.yaml
rolebinding.rbac.authorization.k8s.io/logshipper-read-logs-binding created
```

```text
admin@ip-10-1-12-37:~$ sudo kubectl get pods
NAME                          READY   STATUS             RESTARTS         AGE
logger-6f7fb76c9f-4jk77       1/1     Running            1 (10m ago)      2y7d
logshipper-597f84bf4f-6ssjq   0/1     CrashLoopBackOff   10 (4m31s ago)   2y7d
```

**Answer**

The RoleBinding was created successfully and assigned the Role to `logshipper-sa`.

**Interpretation**

The Pod is still crashing immediately after the RoleBinding is applied, and that is completely normal.

RBAC changes do not “patch” the current failed container into a healthy state instantly. The container must attempt its startup path again, this time with the new permissions in place.

This is an important troubleshooting principle:

- configuration fixes can be correct
- Pod status can still lag behind while restart/backoff cycles catch up

So seeing `CrashLoopBackOff` for a short time after the fix does **not** mean the fix failed. It means the controller and kubelet have not yet completed the next successful restart attempt.

---

## Task 6/6 — Verify the Pod becomes ready and confirm the test result

**Task**

Check the readiness value for the `logshipper` Pod, observe the recovery, and confirm that the final test condition returns `true`.

**Command**

```bash
kubectl get pods -l app=logshipper --no-headers -o json | jq -r '.items[] | "\(.status.containerStatuses[0].ready)"'
sudo kubectl get pods -l app=logshipper --no-headers -o json | jq -r '.items[] | "\(.status.containerStatuses[0].ready)"'
sudo kubectl get pods -w
sudo kubectl get pods -l app=logshipper --no-headers -o json | jq -r '.items[] | "\(.status.containerStatuses[0].ready)"'
```

**Output**

```text
admin@ip-10-1-12-37:~$ kubectl get pods -l app=logshipper --no-headers -o json | jq -r '.items[] | "\(.status.containerStatuses[0].ready)"'
WARN[0000] Unable to read /etc/rancher/k3s/k3s.yaml, please start server with --write-kubeconfig-mode to modify kube config permissions
error: error loading config file "/etc/rancher/k3s/k3s.yaml": open /etc/rancher/k3s/k3s.yaml: permission denied
```

```text
admin@ip-10-1-12-37:~$ sudo kubectl get pods -l app=logshipper --no-headers -o json | jq -r '.items[] | "\(.status.containerStatuses[0].ready)"'
false
```

```text
admin@ip-10-1-12-37:~$ sudo kubectl get pods -w
NAME                          READY   STATUS    RESTARTS         AGE
logger-6f7fb76c9f-4jk77       1/1     Running   1 (12m ago)      2y7d
logshipper-597f84bf4f-6ssjq   1/1     Running   11 (6m54s ago)   2y7d
^C
```

```text
admin@ip-10-1-12-37:~$ sudo kubectl get pods -l app=logshipper --no-headers -o json | jq -r '.items[] | "\(.status.containerStatuses[0].ready)"'
true
```

**Answer**

The initial readiness check failed without `sudo` because of kubeconfig permissions. With `sudo`, the Pod first showed `false`, then recovered to `1/1 Running`, and the final test result became `true`.

**Interpretation**

This last step contains two different lessons.

### Lesson 1 — Infrastructure access matters too

The first command failed not because the Pod was wrong, but because the shell user could not read the cluster kubeconfig:

```text
permission denied
```

This is separate from the application problem. It is an operator-access issue. The scenario itself warned to use `sudo`, and this output proves why.

### Lesson 2 — The fix worked before the status looked perfect

The first `sudo` readiness check returned:

```text
false
```

That does not mean the RBAC fix was wrong. It only means the Pod had not yet completed a successful restart cycle after the permissions were added.

Then the watch output showed the transition:

- `0/1 CrashLoopBackOff`
to
- `1/1 Running`

And the final readiness test returned:

```text
true
```

This is the ideal troubleshooting flow:

1. identify the real cause
2. apply the minimal correct fix
3. allow the workload time to reconcile
4. verify the exact success condition

---

## Final RBAC manifests

### Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: logshipper-read-logs
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods/log"]
    verbs: ["get", "list"]
```

### RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: logshipper-read-logs-binding
  namespace: default
subjects:
  - kind: ServiceAccount
    name: logshipper-sa
    namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: logshipper-read-logs
```

---

## What you should remember from this lab

### Explanation

A Pod can fail even when its own manifest is correct if the identity it runs under lacks required permissions.

In this scenario, the `logshipper` application depended on reading Pod logs through the Kubernetes API. The ServiceAccount `logshipper-sa` did not have permission to access `pods/log`, so the application received `403 Forbidden` and exited repeatedly.

The correct fix was not to edit the Pod definition. It was to repair the environment around the Pod by granting the missing RBAC permission.

### Key takeaway

A `CrashLoopBackOff` is only a symptom.  
The real cause is often one layer deeper.

This lab is a strong example of Kubernetes troubleshooting maturity:

- inspect the logs
- identify the failing API call
- trace the identity
- grant the minimum required permission
- verify recovery with the exact readiness condition

### Final recap

This lab covered how to:

- confirm a Pod is crashlooping
- distinguish between crash symptoms and authorization root causes
- read a `403 Forbidden` log message correctly
- identify the ServiceAccount used by a Pod
- grant precise RBAC access to `pods/log`
- bind the Role to the right ServiceAccount
- understand why a fix may take time to appear in Pod status
- verify recovery with both watch output and the final JSON readiness test
