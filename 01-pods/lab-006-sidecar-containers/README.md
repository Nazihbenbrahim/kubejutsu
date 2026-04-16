# Create Sidecar Container for Logging

**Level:** Intermediate  
**Duration:** 20–30 minutes  
**Mode:** Interactive  
**Source:** Killercoda  
**Lab URL:** https://killercoda.com/omkar-shelke25/scenario/Sidecar-Container-logging

Learn how to extend an existing Deployment with a sidecar container that streams application logs from a shared volume to standard output, making the logs accessible through `kubectl logs`.

## Short introduction

In this lab, an existing Deployment named `cleaner` already runs in the `mercury` namespace. Its main container, `cleaner-con`, writes operational messages to `/var/log/cleaner.log` inside a shared volume.

The goal is to add a logging sidecar named `logger-con` that mounts the same volume and continuously streams the file with `tail -F`. This turns file-based logs into Kubernetes-readable container logs, which is exactly what `kubectl logs` expects.

The most important idea in this lab is not just “add another container.” It is understanding how Kubernetes uses a **shared Pod volume** to let one container write logs while another container reads and publishes them.

---

## Tasks section

## Task 1/5 — Examine the existing `cleaner` Deployment manifest

**Task**

Inspect the existing Deployment manifest located at `/opt/course/16/cleaner.yaml` to understand how the current Pod writes logs.

**Command**

```bash
cat /opt/course/16/cleaner.yaml
```

**Output**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cleaner
  namespace: mercury
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cleaner
  template:
    metadata:
      labels:
        app: cleaner
    spec:
      volumes:
      - name: logs
        emptyDir: {}
      containers:
      - name: cleaner-con
        image: public.ecr.aws/docker/library/busybox:latest
        volumeMounts:
        - name: logs
          mountPath: /var/log
        command: ["sh", "-c"]
        args:
        - |
          while true; do
            echo "$(date '+%Y-%m-%d %H:%M:%S') - Cleaning data..." >> /var/log/cleaner.log
            echo "$(date '+%Y-%m-%d %H:%M:%S') - Found 42 records" >> /var/log/cleaner.log
            echo "$(date '+%Y-%m-%d %H:%M:%S') - WARNING: 3 records missing!" >> /var/log/cleaner.log
            echo "$(date '+%Y-%m-%d %H:%M:%S') - Data cleanup completed" >> /var/log/cleaner.log
            sleep 10
          done
```

**Answer**

The existing Deployment has one container, `cleaner-con`, which writes log lines into `/var/log/cleaner.log` on a shared `emptyDir` volume named `logs`.

**Interpretation**

This manifest already contains the key ingredient for sidecar logging: a **shared filesystem inside the Pod**.

A deeper reading of this YAML shows the architecture immediately:

- `emptyDir` creates Pod-local storage
- `cleaner-con` mounts that storage at `/var/log`
- the container writes structured operational messages into `cleaner.log`

That means the application is already producing useful logs, but they are trapped inside a file. Kubernetes does not automatically collect arbitrary files from inside a container. It collects **stdout** and **stderr** streams.

So the real problem is not log generation. The real problem is **log exposure**.

---

## Task 2/5 — Save an updated manifest and add the `logger-con` sidecar

**Task**

Copy the original manifest to `/opt/course/16/cleaner-new.yaml`, then add a sidecar container named `logger-con` with these properties:

- Image: `public.ecr.aws/docker/library/busybox:latest`
- Mount the same `logs` volume
- Use `tail -F /var/log/cleaner.log`
- Save the updated manifest to `/opt/course/16/cleaner-new.yaml`

**Command**

```bash
cp /opt/course/16/cleaner.yaml /opt/course/16/cleaner-new.yaml

cat <<'EOF' > /opt/course/16/cleaner-new.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cleaner
  namespace: mercury
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cleaner
  template:
    metadata:
      labels:
        app: cleaner
    spec:
      volumes:
      - name: logs
        emptyDir: {}
      initContainers:
      - name: logger-con
        image: public.ecr.aws/docker/library/busybox:latest
        restartPolicy: Always
        volumeMounts:
        - name: logs
          mountPath: /var/log
        command: ["sh", "-c", "tail -F /var/log/cleaner.log"]
      containers:
      - name: cleaner-con
        image: public.ecr.aws/docker/library/busybox:latest
        volumeMounts:
        - name: logs
          mountPath: /var/log
        command: ["sh", "-c"]
        args:
        - |
          while true; do
            echo "$(date '+%Y-%m-%d %H:%M:%S') - Cleaning data..." >> /var/log/cleaner.log
            echo "$(date '+%Y-%m-%d %H:%M:%S') - Found 42 records" >> /var/log/cleaner.log
            echo "$(date '+%Y-%m-%d %H:%M:%S') - WARNING: 3 records missing!" >> /var/log/cleaner.log
            echo "$(date '+%Y-%m-%d %H:%M:%S') - Data cleanup completed" >> /var/log/cleaner.log
            sleep 10
          done
EOF
```

**Output**

```text
root@controlplane:~$ cp /opt/course/16/cleaner.yaml /opt/course/16/cleaner-new.yaml
root@controlplane:~$ cat <<'EOF' > /opt/course/16/cleaner-new.yaml
...
EOF
root@controlplane:~$
```

**Answer**

A new manifest was saved to `/opt/course/16/cleaner-new.yaml` with the additional `logger-con` sidecar configuration.

**Interpretation**

This is the most important design step in the lab.

The sidecar was added under `initContainers` with:

```yaml
restartPolicy: Always
```

That detail changes everything.

A regular init container runs once and exits.  
This one is treated differently: it starts early like an init container, but because of `restartPolicy: Always`, it continues running as a sidecar and keeps streaming logs throughout the Pod lifetime.

That gives this Pod a clean startup and runtime model:

- `logger-con` starts early and begins watching the log path
- `cleaner-con` starts and writes log lines into the shared volume
- `logger-con` continuously reads the same file and forwards it to stdout

This is a strong Kubernetes-native logging pattern because it decouples:

- **log production** from
- **log publication**

One container writes. Another container exposes.

---

## Task 3/5 — Apply the updated manifest and wait for the rollout

**Task**

Apply the new manifest and confirm that the updated Deployment becomes ready in the `mercury` namespace.

**Command**

```bash
kubectl apply -f /opt/course/16/cleaner-new.yaml
kubectl rollout status deployment/cleaner -n mercury
```

**Output**

```text
root@controlplane:~$ kubectl apply -f /opt/course/16/cleaner-new.yaml
deployment.apps/cleaner configured

root@controlplane:~$ kubectl rollout status deployment/cleaner -n mercury
deployment "cleaner" successfully rolled out
```

**Answer**

The Deployment was updated successfully and the new Pod revision rolled out correctly.

**Interpretation**

`configured` means Kubernetes detected a change in the Deployment spec rather than creating a brand-new resource.

Because the Pod template changed, Kubernetes did not modify the running Pod in place. Instead, the Deployment controller created a **new ReplicaSet** and replaced the old Pod with a new Pod that includes the sidecar container.

The key idea is this:

- changing a Pod template means changing the immutable blueprint of the Pod
- Pod blueprints are versioned through ReplicaSets
- therefore a rollout is required

That is why `kubectl rollout status` matters. It confirms that the controller finished reconciling the new desired state.

---

## Task 4/5 — Check the sidecar logs and understand the first failed lookup

**Task**

Use `kubectl logs` to read the output of `logger-con` and verify that it streams the application log file.

**Command**

```bash
kubectl logs -n mercury deployment/cleaner -c logger-con
```

**Output**

```text
root@controlplane:~$ kubectl logs -n mercury deployment/cleaner -c logger-con
Found 2 pods, using pod/cleaner-6cf5b8f8fb-rs7mk
error: container logger-con is not valid for pod cleaner-6cf5b8f8fb-rs7mk
```

**Answer**

The first log attempt failed because `kubectl` selected an older Pod revision that did not yet contain the `logger-con` container.

**Interpretation**

This is exactly the kind of detail that reveals real Kubernetes understanding.

The error is not saying the YAML is wrong.  
It is saying the command reached the **wrong Pod revision** during the rollout window.

Notice this line:

```text
Found 2 pods, using pod/cleaner-6cf5b8f8fb-rs7mk
```

That means the Deployment temporarily had two Pods available:

- the old Pod from the previous ReplicaSet
- the new Pod from the updated ReplicaSet

When `kubectl logs deployment/cleaner -c logger-con` resolved the Deployment to a Pod, it picked the old one first. Since the old Pod never had `logger-con`, Kubernetes correctly reported:

```text
container logger-con is not valid for pod ...
```

This is a rollout timing issue, not a manifest issue.

That distinction matters. A weaker explanation would say “retry the command.”  
A stronger explanation says: **during rolling updates, Deployment-level log lookups can briefly resolve to different Pod revisions**.

---

## Task 5/5 — Verify streamed logs and identify the missing data incident

**Task**

Run the same log command again and confirm that the sidecar now streams the shared log file. Use the output to identify the missing data incident.

**Command**

```bash
kubectl logs -n mercury deployment/cleaner -c logger-con
```

**Output**

```text
root@controlplane:~$ kubectl logs -n mercury deployment/cleaner -c logger-con
tail: can't open '/var/log/cleaner.log': No such file or directory
tail: /var/log/cleaner.log has appeared; following end of new file
2026-04-16 09:02:43 - Cleaning data...
2026-04-16 09:02:43 - Found 42 records
2026-04-16 09:02:43 - WARNING: 3 records missing!
2026-04-16 09:02:43 - Data cleanup completed
2026-04-16 09:02:53 - Cleaning data...
2026-04-16 09:02:53 - Found 42 records
2026-04-16 09:02:53 - WARNING: 3 records missing!
2026-04-16 09:02:53 - Data cleanup completed
2026-04-16 09:03:03 - Cleaning data...
2026-04-16 09:03:03 - Found 42 records
2026-04-16 09:03:03 - WARNING: 3 records missing!
2026-04-16 09:03:03 - Data cleanup completed
2026-04-16 09:03:13 - Cleaning data...
2026-04-16 09:03:13 - Found 42 records
2026-04-16 09:03:13 - WARNING: 3 records missing!
2026-04-16 09:03:13 - Data cleanup completed
2026-04-16 09:03:23 - Cleaning data...
2026-04-16 09:03:23 - Found 42 records
2026-04-16 09:03:23 - WARNING: 3 records missing!
2026-04-16 09:03:23 - Data cleanup completed
```

**Answer**

The sidecar is working correctly and streaming the shared log file. The incident shown in the logs is:

`WARNING: 3 records missing!`

**Interpretation**

This output is excellent because it proves several runtime behaviors at once.

### Why the first `tail` message is expected

The sidecar starts before the main container has written the file. So this message is normal:

```text
tail: can't open '/var/log/cleaner.log': No such file or directory
```

That is not a failure. It is simply the sidecar arriving before the application has produced its first log file.

Then `tail -F` does exactly what it was chosen for:

```text
tail: /var/log/cleaner.log has appeared; following end of new file
```

This is why `-F` is better than a plain one-time read. It tolerates the file not existing yet and begins following it as soon as it appears.

### What the repeated log lines prove

The repeated timestamps show that:

- `cleaner-con` is still writing to the file every 10 seconds
- `logger-con` is continuously reading the same shared file
- the shared volume wiring is correct
- the sidecar is publishing the file contents to stdout successfully

### What the incident actually is

The missing data incident is explicitly present in the logs:

```text
WARNING: 3 records missing!
```

This is the business signal the sidecar was meant to expose. Without the sidecar, that warning would remain buried inside a file within the Pod. With the sidecar, the warning becomes visible through standard Kubernetes log tooling.

That is the real operational value of the pattern.

---

## Final manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cleaner
  namespace: mercury
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cleaner
  template:
    metadata:
      labels:
        app: cleaner
    spec:
      volumes:
        - name: logs
          emptyDir: {}
      initContainers:
        - name: logger-con
          image: public.ecr.aws/docker/library/busybox:latest
          restartPolicy: Always
          volumeMounts:
            - name: logs
              mountPath: /var/log
          command: ["sh", "-c", "tail -F /var/log/cleaner.log"]
      containers:
        - name: cleaner-con
          image: public.ecr.aws/docker/library/busybox:latest
          volumeMounts:
            - name: logs
              mountPath: /var/log
          command: ["sh", "-c"]
          args:
            - |
              while true; do
                echo "$(date '+%Y-%m-%d %H:%M:%S') - Cleaning data..." >> /var/log/cleaner.log
                echo "$(date '+%Y-%m-%d %H:%M:%S') - Found 42 records" >> /var/log/cleaner.log
                echo "$(date '+%Y-%m-%d %H:%M:%S') - WARNING: 3 records missing!" >> /var/log/cleaner.log
                echo "$(date '+%Y-%m-%d %H:%M:%S') - Data cleanup completed" >> /var/log/cleaner.log
                sleep 10
              done
```

---

## What you should remember from this lab

### Explanation

A sidecar container is a helper container that runs in the same Pod as the main application and extends its behavior without changing the application itself.

In this lab:

- `cleaner-con` writes logs to a file
- `logger-con` reads the same file from a shared volume
- `logger-con` publishes those logs to stdout
- `kubectl logs` can then retrieve them normally

This lab also highlights the difference between **file-based logging inside a container** and **container-native logging for Kubernetes**. Kubernetes log tooling works best when information is emitted to stdout or stderr.

### Key takeaway

The shared volume is the bridge between the two containers:

- one container produces the file
- the other container exposes the file as a stream

That is the architectural heart of sidecar logging.

It also shows why rollout timing matters: during a Deployment update, log commands can briefly hit an older Pod revision that does not yet contain the new container.

### Final recap

This lab covered how to:

- inspect an existing Deployment manifest
- identify a file-based logging pattern inside a Pod
- add a sidecar-style logging container
- mount the same `emptyDir` volume into two containers
- use `tail -F` to follow a file that may not exist yet
- understand why a first `kubectl logs` call can fail during rollout
- expose missing data incidents through standard Kubernetes logging commands
