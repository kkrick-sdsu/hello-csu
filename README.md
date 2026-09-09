# hello-csu

Simplistic 'hello, world' example of a containerized application.

This repository is a worked example of running a batch job on the CSU TIDE
cluster using a Kubernetes pod. See the full guide in the
[TIDE Batch Jobs documentation](https://csu-tide.github.io/batch-jobs/getting-started).

By the end of this example you will be able to:

- Schedule a batch job
- Transfer small datasets in and out of the cluster
- Check the status of a batch job
- Access a remote terminal in a batch job

## Prerequisites

- Completed the [Getting Access](https://csu-tide.github.io/batch-jobs/getting-access) guide
- Installed [Kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)
- Familiarity with the Linux terminal
- A preferred text editor
- Exposure to [git source control](https://git-scm.com/)

## Repository contents

| File             | Purpose                                                        |
| ---------------- | -------------------------------------------------------------- |
| `hello.py`       | Simple "Hello, World" program with ASCII art                   |
| `Dockerfile`     | Builds a container image (based on `python:3`) with `hello.py` |
| `hello-pod.yaml` | Kubernetes manifest describing the pod that runs the image     |

## Running your first batch job on TIDE

### 1. Clone the repository

From your local machine, open a terminal and run:

```bash
git clone https://github.com/csu-tide/hello-csu.git
cd hello-csu
```

If you do not have git installed, you can instead
[download this repository](https://github.com/csu-tide/hello-csu/archive/refs/heads/main.zip)
and un-zip it.

### 2. Examine the files

- `hello.py` prints a "Hello, World" message with some CSU ASCII art.
- `Dockerfile` builds the image from the [Python 3 image](https://hub.docker.com/_/python/),
  copies `hello.py` in, and runs it.
- `hello-pod.yaml` wraps the container in a Kubernetes pod. Note these key fields:
  - `kind: Pod` — the type of Kubernetes workload.
  - `name: hello-pod` — must be unique per namespace. In a shared namespace, consider a
    suffix such as `name: hello-pod-kk`.
  - `image: ghcr.io/csu-tide/hello-csu:main` — the pre-built image the cluster will pull.
  - `command: ["sh", "-c", "sleep infinity"]` — a never-ending command so the pod stays up
    long enough to log into and inspect it.

### 3. Schedule the batch job

Define your namespace, then create the pod:

```bash
ns=[namespace]
kubectl apply -f hello-pod.yaml -n $ns
```

Expected output:

```
pod/hello-pod created
```

Watch the pod until it is running (READY `1/1`, STATUS `Running`):

```bash
kubectl get pods -n $ns --watch
```

Press `ctrl` + `c` to stop watching.

### 4. Access the batch job

Open an interactive bash shell in the container:

```bash
kubectl exec -it hello-pod -n $ns -- /bin/bash
```

> If you renamed the pod in `hello-pod.yaml`, update `hello-pod` in the commands above.

Inside the container:

```bash
pwd            # /usr/src/app (set by the Dockerfile WORKDIR)
ls -la         # you should see hello.py
python hello.py
python hello.py > hello.txt
ls -la         # hello.txt now present
exit
```

### 5. Retrieve your data and delete the job

From your Kube Notebook terminal, copy the output file out of the container:

```bash
kubectl -n $ns cp hello-pod:/usr/src/app/hello.txt ./hello.txt
ls               # Dockerfile  README.md  hello-pod.yaml  hello.py  hello.txt
```

> `kubectl cp` is intended for small transfers. For larger files (> 5MB), see the
> [Storage Services](https://csu-tide.github.io/storage-services/) docs.

Delete the pod when finished:

```bash
kubectl -n $ns delete -f hello-pod.yaml
kubectl -n $ns get pods    # No resources found
```

Pods are ephemeral: if you schedule the pod again and reattach, the `hello.txt` you
generated will be gone because it was not part of the container image.

```bash
kubectl -n $ns apply -f hello-pod.yaml
kubectl -n $ns exec -it hello-pod -- /bin/bash
ls -la        # only hello.py remains; hello.txt is gone
exit
kubectl -n $ns delete -f hello-pod.yaml
```

## Next steps

- [Batch Job Recipes](https://csu-tide.github.io/batch-jobs/recipes)
- [Common Tasks and Troubleshooting](https://csu-tide.github.io/batch-jobs/tasks-troubleshooting)
