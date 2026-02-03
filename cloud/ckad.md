# Epic Kubernetes Guide for the CKAD Exam

My aim with this guide is to introduce Kubernetes in a fun way. There will be less theoretical knowledge and more practical implementation. Let's go.

I will start with the basics and gradually build on them so you get a solid understanding of everything you need for the CKAD exam --- and for mastering Kubernetes as a developer.

### Contents

1.  Pods
2.  ReplicaSets
3.  Deployments
4.  Namespaces
5.  ConfigMaps and Secrets
6.  Services and Ingress
7.  Volumes and Persistent Storage
8.  Pod Scheduling
9.  Jobs and CronJobs
10. Helm
11. Podman

---

### A couple of tips before we get started

So, what are we up against?

The CKAD exam wants us to solve real-world Kubernetes problems in a limited time. Let's understand the exam's structure:

- **Duration:** 2 hours.
- **Passing Score:** Approximately 66%.
- **Environment:** Access to a terminal with `kubectl` ready.
- **No Internet Access:** Only the official [Kubernetes](https://kubernetes.io/docs/home/) and [Helm](https://helm.sh/docs/) docs.

Here is a bunch of my personal tips. If you don't understand something now, you will soon. Let's just get it out of the way now, and I suggest you return to the tips again after you finish the guide.

#### Use `kubectl` alias

The terminal on the exam has a preconfigured alias that allows you to write `k` instead of `kubectl`. It saves precious seconds during the exam.

You can create this alias on your personal computer, where you practice, this way:

alias k=kubectl

Or this, when using PowerShell on Windows:

Set-Alias -Name k -Value kubectl

#### Use command history

Use the history of your commands in your terminal with **Ctrl + r **.

#### Use short names of resources

Kubernetes resources have short names that save time. Notable short names are:

| **Resource** | **Short Name** |\
|---------------------------|----------------|\
| Deployments | `deploy` |\
| Services | `svc` |\
| Namespaces | `ns` |\
| ConfigMaps | `cm` |\
| PersistentVolumeClaims | `pvc` |\
| ReplicaSets | `rs` |\
| StatefulSets | `sts` |\
| Ingresses | `ing` |

Example:

k get rs\
k describe svc my-service

I will be mostly using short names throughout the rest of this guide.

#### Utilize tab completion

After typing part of a command or resource name, press `Tab` to auto-complete --- it should work in the exam environment.

For example, if you have a ConfigMap named `database-config`:

k get cm d<Tab>

This will auto-complete to:

k get cm database-config

Here is [an official guide](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/#enable-shell-autocompletion) on setting up autocompletion on your machine.

#### Generate resource manifests with imperative commands

Use `kubectl` imperative commands to quickly create resource definitions, so that you can comfortably edit them:

k run x --image=nginx --dry-run=client -o yaml > pod.yaml && vi pod.yaml

This will allow you to start editing a pre-filled resource manifest. We will learn more about this in the guide soon.

#### Familiarize yourself with the official documentation

During the exam, you have access to <https://kubernetes.io/docs/>. Practice navigating those docs and searching for information. This is very important.

#### Read questions carefully

Answer all the questions you know first. If a question is taking too long, mark it and return to it later. Don't waste time banging your head against the wall.

Pay attention to details like namespaces, resource names, and specific configurations. Questions will require you to work in different Kubernetes namespaces. So the following commands will be useful:

List namespaces:

k get namespaces

Use a namespace in a command:

k get pods -n <namespace-name>

Use a namespace by default:

k config set-context --current --namespace=<namespace-name>

#### Don't mess up your environment

At the start of a scenario, get an overview of the environment:

k -n <namespace-name> get all -o wide

This will give you a good glimpse at the most important resources running in the namespace. You might notice that there are resources from other sections of the exam that don't have anything to do with the current question you are solving! Don't disturb the unrelated resources.

Also, when you are provided a template file, don't directly edit it without copying it first:

cp /opt/pod.yaml /opt/pod-new.yaml # Copy first to keep the original\
vi /opt/pod-new.yaml # Edit the copied file according to your needs

#### Learn handy commands by heart

You will need a couple of commands that might not seem super-intuitive at first, though they are widely used because they are very helpful and save time. Here they are:

**Get cluster information**

```bash
kubectl cluster-info
```

This shows the addresses of the control plane and core services.

**Check whether a pod is reachable**

Get pods IP addresses:

➜ k get pods -o wide\
NAME READY STATUS ... IP ...\
pod-name 1/1 Running ... 10.11.0.12 ...

Query a pod to see whether it is accessible and usable for a probe by curl:

k run tmp --rm --image=nginx:alpine -i -- curl -m 5 10.11.0.12

or by wget (notice the different image as well):

k run tmp --rm --image=busybox -i -- wget -O- --T=5 10.11.0.12

The timeout flags differ between tools: curl uses `-m` for max time, while wget uses `--T`.

You can also create a temporary pod for debugging with curl:

```bash
kubectl run -it --rm curl --image=curlimages/curl --restart=Never -- sh
```

This drops you into an interactive shell with curl available. The pod deletes itself when you exit.

**Use a full-service address**

When you have a service named, say `my-service`, you can send a request to it simply using its name:

k run tmp --rm --image=nginx:alpine -i -- curl -m 5 my-service

But that's true only when it's in the same namespace as the pod you are calling it from. Otherwise, you need to use its full address, which looks like this: `<service-name>.<namespace>.svc.cluster.local`

**Paste resource manifests directly**

You can paste YAML directly from your clipboard into the terminal:

```bash
kubectl apply -f -
```

After running this command, paste your YAML content and press `Ctrl+D` to apply it.

**Force replace a resource**

When you need to delete and recreate a resource in one step:

```bash
kubectl replace --force -f <file-name>
```

This deletes the existing resource and creates it fresh from the file.

**Monitor resource usage**

Check resource consumption across nodes:

```bash
kubectl top node
```

Check resource consumption for pods:

```bash
kubectl top pod
```

These commands query the Metrics Server, which must be installed in the cluster.

**View kubectl config**

Extract specific information from your kubeconfig:

```bash
kubectl config view -o jsonpath='{.users[*].name}'
```

This shows all usernames configured in your kubeconfig.

**Linux shortcuts**

The Terminal is a Linux Terminal, and candidates need to use [Linux Terminal keyboard shortcuts](https://itsfoss.com/copy-paste-linux-terminal/):

Copy = CTRL+SHIFT+C

Paste = CTRL+SHIFT+V for Paste

#### Use the official practice lab

I added practice questions to every chapter in this guide. In addition to those, I highly recommend using [KodeKloud](https://learn.kodekloud.com/courses/udemy-labs-certified-kubernetes-application-developer) and [KillerShell](https://killer.sh/ckad) for further study.

#### Get CKAD exam discount

Check [this repository](https://github.com/techiescamp/linux-foundation-coupon) for up-to-date coupons.

---

Alright, that's enough for tips. Let's start learning Kubernetes with a focus on the CKAD certification, starting with Pods in the next chapter.

## Chapter 1: Pods

At the heart of Kubernetes are **Pods**. A Pod is the smallest deployable unit in Kubernetes. It's basically a single instance of a running process. Although I said _the smallest deployable unit_, it still can contain one or more containers that share the Pod's storage volumes and network resources, allowing them to work closely together.

For context, a container is the program you want to run. Or, more specifically, it's the executable package containing the code, runtime, libraries, and dependencies that's been bundled into a container image.

But enough of the linguistics. Let's get hands-on and create a Pod.

### Creating Your First Pod

We'll explore both the imperative and declarative approaches. For the CKAD exam, it's usually in your favor to prefer imperative approaches wherever possible, because it saves time.

At this point, make sure you have [minikube](https://minikube.sigs.k8s.io/docs/start/), [Docker Desktop](https://www.docker.com/products/docker-desktop/), and [kubectl](https://kubernetes.io/docs/tasks/tools/) installed and running.

#### Imperative Approach with `kubectl run`

The `kubectl run` command creates a Pod directly.

```bash
# Create a Pod named 'my-first-pod' running an NGINX container
kubectl run my-first-pod --image=nginx
```

**Explanation:**

- `kubectl run`: Command to run a particular image on the cluster.
- `my-first-pod`: The name of your Pod.
- `--image=nginx`: Specifies the container image to use.

You can also set labels during pod creation:

```bash
kubectl run my-first-pod --image=nginx --labels="env=prod,tier=frontend"
```

You can set environment variables during pod creation:

```bash
kubectl run my-first-pod --image=nginx --env="APP_ENV=production" --env="DEBUG=false"
```

To run a custom command in the container, use `--command` followed by `--` and the command:

```bash
kubectl run my-first-pod --image=busybox --command -- env
```

Without `--command`, arguments after `--` are passed to the container's default entrypoint and treated as arguments to the entrypoint:

```bash
kubectl run my-first-pod --image=busybox -- env
```

The difference: `--command` replaces the entrypoint, while omitting it keeps the default entrypoint and passes arguments to it.

#### Declarative Approach with YAML

To create a Pod declaratively, we will use a YAML file named `pod.yaml` with such contents:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-first-pod
spec:
  containers:
    - name: nginx-container
      image: nginx
```

**Explanation:**

- `apiVersion`, `kind`, and `metadata`: Define the type and name of the Kubernetes object.
- `spec`: Specifies the desired state of the object, including containers and images.

On the exam, you can create such a file from the command line with this command:

```bash
vi pod.yaml
```

Which opens the _vi editor_ (learn [how to use it](https://www.cs.colostate.edu/helpdocs/vi.html)) and lets you write the file's contents. If you don't want to write everything by hand, you can use this:

```bash
# Pre-fill the pod.yaml file with contents and then open it
kubectl run my-first-pod --image=nginx --dry-run=client -o yaml > pod.yaml
vi pod.yaml
```

**Explanation:** We mix the imperative and declarative approaches here, which will be useful when you need to use YAML files but don't want to write everything by hand.

- `--dry-run=client`: Specifies that we don't want to apply the command; just get the output.
- `-o yaml`: Specifies the format of the output.
- `> pod.yaml`: Redirects the output into a file.

You can then edit the file as you please.

Once the file is ready, you can apply it:

```bash
kubectl apply -f pod.yaml
```

### Interacting with the Pod

Check the status of your Pod:

```bash
kubectl get pods
```

You should see `my-first-pod` listed with its current status.

#### Getting Pod Details

You can get more details about your Pods by using the `-o wide` flag, such as:

```bash
kubectl get pods my-first-pod -o wide
```

And even more by using the `describe` command:

```bash
kubectl describe pod my-first-pod
```

In both of those commands, you can omit the Pod name to get info about all pods.

#### Executing Commands Inside the Pod

Run a command inside the Pod:

```bash
kubectl exec -it my-first-pod -- /bin/bash
```

Now you're inside the container and can explore its file system or write `exit` to get out of there.

You can also run a single command without entering an interactive shell:

```bash
kubectl exec my-first-pod -- whoami
```

This tells you which user runs commands inside the container.

For Pods with multiple containers, specify which container to target with the `-c` flag:

```bash
kubectl exec -it my-first-pod -c nginx-container -- /bin/sh
```

To run curl against an endpoint from inside a container:

```bash
kubectl exec -it my-first-pod -c nginx-container -- curl http://some-service:80
```

#### Debugging with Ephemeral Containers

Some container images are minimal and don't include debugging tools like `curl` or `sh`. You can inject a temporary debug container into a running Pod:

```bash
kubectl debug -it my-first-pod --image=curlimages/curl --target=nginx-container
```

This starts a new ephemeral container with curl installed, sharing the process namespace with the target container. You can then run diagnostics without modifying the original Pod.

#### Passing Commands and Arguments

By default, containers run the command specified in the image. You can override it using the `command` and `args` fields. We can, for example, use `command-args-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: command-args-pod
spec:
  containers:
    - name: custom-container
      image: busybox
      command: ["sh", "-c"]
      args: ["echo Hello, Kubernetes!; sleep 3600"]
```

Apply the configuration:

```bash
kubectl apply -f command-args-pod.yaml
```

**Explanation:**

- **Command**: The executable or initial command that runs inside the container. It is specified as a list of strings and usually sets the main process that the container should run. The command `['sh', '-c']` tells the container to run the shell `sh` with the `-c` option, which means "execute the following command."
- **Arguments (args)**: These are the parameters or additional instructions passed to the command. Our example, `args: ['echo Hello, Kubernetes!; sleep 3600']`, specifies what the shell should execute. It runs `echo Hello, Kubernetes!` to print the message and then `sleep 3600` to keep the container running for 3600 seconds (1 hour).

### Logging and Monitoring

Kubernetes doesn't handle application-level logging, but it provides mechanisms for collecting logs.

You can view pod logs with:

```bash
kubectl logs <pod-name>
```

You can specify the container you want to see logs from, which comes in handy especially when you have multi-container Pods:

```bash
kubectl logs <pod-name> -c <container-name>
```

For the Pod from our previous example, we could use:

```bash
kubectl logs command-args-pod -c custom-container
```

Expected output:

```bash
Hello, Kubernetes!
```

### Init Containers

**Init Containers** run before the main application containers start. They can perform setup tasks.

We can create a Pod with two containers. One will be marked as an Init Container and will run first. We can use a definition such as this example from `pod-with-init.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-init
spec:
  initContainers:
    - name: init-container
      image: busybox
      command: ["sh", "-c", "echo Initializing; sleep 5"]
  containers:
    - name: main-container
      image: nginx
```

Apply the configuration:

```bash
kubectl apply -f pod-with-init.yaml
```

**Explanation:**

- The `init-container` container runs first.
- The `main-container` container starts only after the init container completes successfully.

After 5 seconds, explore the events of the pod:

```bash
kubectl describe pod pod-with-init
```

You will see that the `init-container` was running first, and five seconds later, after it finished, the `main-container` started.

### Labels

You can set labels at pod creation time by using the `--labels` (or `-l`) flag in `kubectl run`.

```bash
kubectl run pod-with-labels --image=busybox -l="animal=lion"
```

**Verify the labels**

```bash
kubectl get pods --show-labels
```

**Show a specific label as a column**

Use the `-L` flag to display label values as columns in the output:

```bash
kubectl get pods -L animal
```

This adds an `ANIMAL` column showing each pod's value for that label.

**Filter pods by labels**

Use the lowercase `-l` flag to filter resources by label:

```bash
kubectl get pods -l animal=lion
```

You can filter any resource type by labels:

```bash
kubectl get all -l env=prod
```

Once a pod (or any resource) exists, you can manage labels imperatively with `kubectl label`.

**Add a new label**

```bash
kubectl label pod pod-with-labels plant=tree
```

**Update (overwrite) a label**

By default, if the label already exists, `kubectl label` will error out.
To force an update, add `--overwrite`:

```bash
kubectl label pod pod-with-labels animal=monkey --overwrite
```

**Remove a label**

Use a trailing `-` to delete:

```bash
kubectl label pod pod-with-labels animal-
```

**Apply to multiple pods**

```bash
kubectl label pod -l plant=tree animal=bird
```

This finds all pods with label `plant=tree` and adds `animal=bird`.

### Deleting Pods

When you're done, clean up:

```bash
kubectl delete pod my-first-pod command-args-pod pod-with-init pod-with-labels
```

### Practice Exercises

Now it's time to put your knowledge to the test. These exercises combine multiple concepts from this chapter and simulate the kinds of tasks you'll encounter on the CKAD exam.

#### Exercise 1: Multi-Container Pod with Init Container

Create a pod called `web-logger` in the default namespace with labels `app=web-logger`, `tier=frontend`, and `environment=dev`.

The pod should have an init container named `init-permissions` using the `busybox` image. This init container should create a file at `/work-dir/ready.txt` containing the text "initialized".

The pod should have two main containers:

The first container should be named `nginx-server` using the `nginx:alpine` image. It should have an environment variable `LOG_LEVEL` set to `debug` and mount a shared volume at `/usr/share/nginx/html`.

The second container should be named `log-sidecar` using the `busybox` image. It should run a command that writes a heartbeat message with a timestamp to `/logs/heartbeat.log` every 10 seconds and mount the same shared volume at `/logs`.

Use an `emptyDir` volume named `shared-data` to share storage between all containers.

**Verify:** The pod shows `2/2` ready, running `kubectl exec web-logger -c nginx-server -- cat /usr/share/nginx/html/ready.txt` outputs "initialized", and `kubectl get pod web-logger --show-labels` displays all three labels.

<details>
<summary>Show Solution</summary>

First, generate a base YAML file:

```bash
kubectl run web-logger --image=nginx:alpine --dry-run=client -o yaml > web-logger.yaml
```

Edit the file to match the requirements:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-logger
  labels:
    app: web-logger
    tier: frontend
    environment: dev
spec:
  volumes:
    - name: shared-data
      emptyDir: {}
  initContainers:
    - name: init-permissions
      image: busybox
      command: ["sh", "-c", "echo initialized > /work-dir/ready.txt"]
      volumeMounts:
        - name: shared-data
          mountPath: /work-dir
  containers:
    - name: nginx-server
      image: nginx:alpine
      env:
        - name: LOG_LEVEL
          value: "debug"
      volumeMounts:
        - name: shared-data
          mountPath: /usr/share/nginx/html
    - name: log-sidecar
      image: busybox
      command:
        [
          "sh",
          "-c",
          "while true; do echo $(date): Heartbeat >> /logs/heartbeat.log; sleep 10; done",
        ]
      volumeMounts:
        - name: shared-data
          mountPath: /logs
```

Apply the configuration:

```bash
kubectl apply -f web-logger.yaml
```

Cleanup: `kubectl delete pod web-logger`

</details>

---

#### Exercise 2: Label Operations Across Multiple Pods

Create a pod called `alpha` using the `nginx:alpine` image with labels `release=stable` and `team=backend`.

Create a pod called `beta` using the `busybox` image with labels `release=canary` and `team=backend`. The container should sleep for 3600 seconds.

Create a pod called `gamma` using the `nginx:alpine` image with labels `release=stable` and `team=frontend`. Set an environment variable `APP_MODE=production`.

Once all pods are running, perform the following label operations:

1. Add the label `monitored=true` to all pods that have `release=stable`
2. Update pod `alpha` to change its `team` label from `backend` to `platform`
3. Remove the `release` label from pod `beta`

**Verify:** Run `kubectl get pods --show-labels` and confirm that `alpha` has labels `monitored=true,release=stable,team=platform`, `beta` has only `team=backend`, and `gamma` has `monitored=true,release=stable,team=frontend`.

<details>
<summary>Show Solution</summary>

```bash
# Create the pods
kubectl run alpha --image=nginx:alpine --labels="release=stable,team=backend"
kubectl run beta --image=busybox --labels="release=canary,team=backend" -- sleep 3600
kubectl run gamma --image=nginx:alpine --labels="release=stable,team=frontend" --env="APP_MODE=production"

# Label operations
kubectl label pod -l release=stable monitored=true
kubectl label pod alpha team=platform --overwrite
kubectl label pod beta release-
```

Cleanup: `kubectl delete pod alpha beta gamma`

</details>

---

## Chapter 2: ReplicaSets

In the previous chapter, we learned about Pods, which, by themselves, are sort of unstable --- and might fail or be deleted. To ensure that a specified number of identical Pods are always running, Kubernetes provides **ReplicaSets**.

A **ReplicaSet** is a higher-level abstraction that maintains a stable set of replica Pods running at any given time. It ensures the availability and scalability of your application by managing the lifecycle of Pods.

**Key Points:**

- **Self-healing**: If a Pod fails or is deleted, the ReplicaSet automatically replaces it.
- **Scaling**: Easily scale the number of Pods up or down.
- **Consistency**: Ensures a consistent application state across multiple Pods.

### Creating a ReplicaSet

Let's create a ReplicaSet to manage multiple instances of an NGINX web server.

#### Imperative Approach

Bad news so early on --- you cannot create **ReplicaSets** imperatively.

#### Declarative Approach with YAML

Create a YAML file named `rs.yaml` with the command:

```bash
vi rs.yaml
```

And fill it with these contents:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-first-rs
  labels:
    app: nginx-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
        - name: nginx-container
          image: nginx
```

You don't have to remember the exact structure of the YAML file for each resource, but be comfortable reading it. Let the official [Kubernetes documentation](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/#example) be your guide and the go-to copy-paste resource. You will have access to it during the CKAD exam.

So now, apply the configuration:

```bash
kubectl apply -f rs.yaml
```

**Explanation:**

- **apiVersion**: Specifies the API version (`apps/v1`) for ReplicaSets.
- **kind**: Indicates that we're creating a `ReplicaSet`.
- **metadata**: Contains the `name` and `labels` for identification.
- **spec.replicas**: The desired number of Pods (3 in this case).
- **spec.selector**: Defines how the ReplicaSet identifies the Pods it manages via labels.
- **spec.template**: Provides the Pod specification to be used when creating Pods.

### Verifying the ReplicaSet

Check the status of the ReplicaSet:

```bash
kubectl get rs
```

**Sample Output:**

```bash
NAME          DESIRED   CURRENT   READY   AGE
my-first-rs   3         3         3       1m
```

List the Pods created by the ReplicaSet:

```bash
kubectl get pods -l app=nginx-app
```

The `-l app=nginx-app` is the label we specified in the YAML file. So we now get all pods with that label.

**Sample Output:**

```bash
NAME                READY   STATUS    RESTARTS   AGE
my-first-rs-abc123  1/1     Running   0          1m
my-first-rs-def456  1/1     Running   0          1m
my-first-rs-ghi789  1/1     Running   0          1m
```

If you need more details about your ReplicaSet, you can use the `describe` command:

```bash
kubectl describe rs
```

### Scaling the ReplicaSet

Scaling adjusts the number of replicas.

#### Using `kubectl scale`

Scale up to 5 replicas:

```bash
kubectl scale rs my-first-rs --replicas=5
```

Verify the scaling:

```bash
kubectl get rs
```

**Sample Output:**

```bash
NAME          DESIRED   CURRENT   READY   AGE
my-first-rs   5         5         5       3m
```

List all Pods:

```bash
kubectl get pods -l app=nginx-app
```

You should now see 5 Pods running.

#### Editing the ReplicaSet

Alternatively, you can edit the ReplicaSet manifest directly:

```bash
kubectl edit rs my-first-rs
```

This opens the ReplicaSet configuration in your default editor. Change the `replicas` field to the desired number and save.

**Note:** You cannot edit everything in the configuration. If `kubectl` complains that properties you tried to edit are non-editable, you will have to create a new YAML file, delete the old ReplicaSet, and create a new ReplicaSet from the new file.

### ReplicaSet's Self-Healing Mechanism

Delete a Pod managed by the ReplicaSet:

```bash
# Replace the <pod-name> with whatever name one of your pods has:
kubectl delete pod <pod-name>
```

After deletion, list the Pods again:

```bash
kubectl get pods -l app=nginx-app
```

You'll notice that Kubernetes has automatically created a new Pod to maintain the desired replica count.

### Labels and Selectors

ReplicaSets use **labels** and **selectors** to identify and manage Pods.

- **Labels**: Key-value pairs attached to Kubernetes objects (e.g., `app: nginx-app`).
- **Selectors**: Define how the ReplicaSet identifies Pods via label matching.

#### Modifying Pod Labels

You can add or modify labels on existing Pods, for example, assigning the value "prod" to a label named "env".

```bash
kubectl label pod <pod-name> env=prod
```

List Pods with labels:

```bash
kubectl get pods --show-labels
```

To remove a label, use the minus sign after the label name.

```bash
kubectl label pod <pod-name> <label-key>-

# For example:
kubectl label pod my-first-pod env-
```

#### Importance of Labels

If a Pod's labels don't match the ReplicaSet's selector, the ReplicaSet won't manage it. And vice versa, any Pod with matching labels will be managed by the ReplicaSet.

### Deleting a ReplicaSet

#### Deleting ReplicaSet and Pods

To delete the ReplicaSet and all its managed Pods:

```bash
kubectl delete rs my-first-rs
```

#### Deleting ReplicaSet Only

To delete the ReplicaSet but leave the Pods running:

```bash
kubectl delete rs <replicaset-name> --cascade=orphan
```

**Note:** The Pods become orphaned and are no longer managed.

### When to Use ReplicaSets Directly?

While ReplicaSets are fundamental, in practice, we often use **Deployments** instead. Deployments manage ReplicaSets and provide advanced features like rolling updates and rollbacks.

However, understanding ReplicaSets is crucial because Deployments build upon them.

### Recap

- **ReplicaSets** ensure a specified number of Pod replicas are running.
- **Scaling** is simple using `kubectl scale` or by editing the ReplicaSet.
- **Labels and selectors** are vital for ReplicaSets to manage the correct Pods.
- **Self-healing**: Automatically replaces failed or deleted Pods.
- Only a **declarative** method is available for creating ReplicaSets.

### Practice Exercises

#### Exercise 1: ReplicaSet with Scaling and Self-Healing

Create a ReplicaSet called `frontend-rs` that manages pods running the `nginx:1.21-alpine` image. The ReplicaSet should maintain 3 replicas. Use the label `app=frontend` for both the selector and the pod template. The container should be named `nginx`.

After the ReplicaSet is running, scale it up to 5 replicas using the `kubectl scale` command.

Then, delete one of the pods managed by the ReplicaSet and observe what happens.

**Verify:** Run `kubectl get rs frontend-rs` to confirm 5 desired and 5 ready replicas. After deleting a pod, run `kubectl get pods -l app=frontend` to confirm a new pod was automatically created to maintain the replica count.

<details>
<summary>Show Solution</summary>

Create the ReplicaSet manifest `frontend-rs.yaml` using a template from the [official docs](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/#example) and adjusting it to something like:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend-rs
  labels:
    app: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: nginx
          image: nginx:1.21-alpine
```

Run the following commands:

```bash
# Apply the ReplicaSet
kubectl apply -f frontend-rs.yaml

# Scale to 5 replicas
kubectl scale rs frontend-rs --replicas=5

# List pods to get a pod name
kubectl get pods -l app=frontend

# Delete one pod (replace with actual pod name)
kubectl delete pod frontend-rs-xxxxx

# Observe that a new pod is created
kubectl get pods -l app=frontend
```

Cleanup: `kubectl delete rs frontend-rs`

</details>

---

#### Exercise 2: ReplicaSet Label Selector Behavior

Create a standalone pod called `orphan-pod` using the `nginx:alpine` image with the label `app=backend`.

Create a ReplicaSet called `backend-rs` with 2 replicas using the `nginx:alpine` image. Use the label `app=backend` for the selector and pod template. The container should be named `web`.

After both are created, observe how many pods are running with the label `app=backend`. Then remove the label `app` from one of the ReplicaSet-managed pods and observe what happens.

**Verify:** Initially, `kubectl get pods -l app=backend` should show 2 pods (not 3), because the ReplicaSet adopts the existing `orphan-pod`. After removing the label from a pod, the ReplicaSet should create a new pod to maintain 2 replicas, resulting in 3 total pods (2 with the label, 1 without).

<details>
<summary>Show Solution</summary>

```bash
# Create the standalone pod first
kubectl run orphan-pod --image=nginx:alpine --labels="app=backend"
```

Create the ReplicaSet manifest `backend-rs.yaml` using a template from the [official docs](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/#example) and adjusting it to something like:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: backend-rs
  labels:
    app: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: web
          image: nginx:alpine
```

Run the following commands:

```bash
# Apply the ReplicaSet
kubectl apply -f backend-rs.yaml

# Check pods - should see only 2 pods (orphan-pod was adopted)
kubectl get pods -l app=backend

# Get one of the ReplicaSet-managed pod names
kubectl get pods -l app=backend

# Remove the label from one pod (replace with actual pod name)
kubectl label pod backend-rs-xxxxx app-

# Check pods - now 2 pods with label, 1 without
kubectl get pods -l app=backend
kubectl get pods
```

Cleanup: `kubectl delete rs backend-rs && kubectl delete pod orphan-pod`

</details>

---

## Chapter 3: Deployments

In the previous chapters, we explored Pods and ReplicaSets. While ReplicaSets manage the number of Pod replicas, **Deployments** provide a higher-level management layer that offers declarative updates, rollbacks, and more. Deployments are the recommended way to manage stateless applications in Kubernetes.

**Key Benefits of Deployments:**

- **Rolling Updates:** Deploy new versions of applications without downtime.
- **Rollbacks:** Revert to previous versions if something goes wrong.
- **Declarative Updates:** Manage the application lifecycle through desired-state definitions.
- **Pause and Resume:** Control the rollout process for complex updates.

### Creating a Deployment

Let's create a Deployment to manage our application effectively.

#### Declarative Approach with YAML

Create a file named `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
        - name: nginx-container
          image: nginx
```

Apply the configuration:

```bash
kubectl apply -f deployment.yaml
```

**Explanation:**

- **kind:** Specifies that we are creating a Deployment object.
- **metadata**: Includes the `name` and `labels` for identification.
- **spec.replicas**: Desired number of Pod replicas.
- **spec.selector**: Matches Pods with the label `app: nginx-app`.
- **spec.template**: Defines the Pod specification.

#### Imperative Approach with `kubectl create`

You can also create a Deployment imperatively:

```bash
kubectl create deploy nginx-deployment --image=nginx --replicas=3
```

**Explanation:**

- `kubectl create deploy`: Command to create a Deployment.
- `nginx-deployment`: The name of the Deployment.
- `--image=nginx`: Specifies the container image.
- `--replicas=3`: Sets the number of replicas.

### Verifying the Deployment

List Deployments:

```bash
kubectl get deploy
```

**Sample Output:**

```bash
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           1m
```

List ReplicaSets managed by the Deployment (as specified by the label):

```bash
kubectl get rs -l app=nginx-app
```

List Pods:

```bash
kubectl get pods -l app=nginx-app
```

### Deployment Updates

Deployments allow you to update your application seamlessly.

#### Rolling Update

Update the Deployment to use a different NGINX image version.

**Method 1: Edit the Deployment**

```bash
kubectl edit deployment nginx-deployment
```

In the editor, change the image version:

```yaml
image: nginx:1.21
```

Save and exit.

**Method 2: Use** `**kubectl set image**`

```bash
kubectl set image deploy/nginx-deployment nginx-container=nginx:1.21
```

**Explanation:**

- `kubectl set image`: Command to update the image of a container.
- `deploy nginx-deployment`: Specifies the Deployment to update.
- `nginx-container=nginx:1.21`: Sets the new image for the specified container.

#### Verifying the Update

Check the rollout status:

```bash
kubectl rollout status deploy nginx-deployment
```

Once it is finished, you should see Pods with the updated image:

```bash
kubectl describe deployment nginx-deployment
```

### Rolling Back a Deployment

If the new version causes issues, you can roll back to the previous version.

```bash
kubectl rollout undo deploy nginx-deployment
```

Verify the rollback. You will see that the pods again have the previous NGINX version specified.

```bash
kubectl describe deployment nginx-deployment
```

### Managing Rollouts

#### Checking Rollout Status

```bash
kubectl rollout status deploy nginx-deployment
```

This shows the progress of the rollout in real time.

#### Viewing Deployment History

```bash
kubectl rollout history deploy nginx-deployment
```

**Sample Output:**

```bash
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

To see details of a specific revision:

```bash
kubectl rollout history deploy nginx-deployment --revision=2
```

#### Rolling Back to a Specific Revision

```bash
kubectl rollout undo deploy nginx-deployment --to-revision=1
```

#### Pausing and Resuming a Deployment

Pause the Deployment to apply multiple changes:

```bash
kubectl rollout pause deploy nginx-deployment
```

Make changes, such as updating the image or resources.

Resume the Deployment:

```bash
kubectl rollout resume deploy nginx-deployment
```

### Scaling a Deployment

Scaling works similarly to ReplicaSets.

#### Using `kubectl scale`

```bash
kubectl scale deploy nginx-deployment --replicas=5
```

Verify scaling:

```bash
kubectl get deployments
```

#### Editing the Deployment

```bash
kubectl edit deployment nginx-deployment
```

Change the `replicas` field and save.

### Deployment Strategies

Deployments support different rollout strategies.

#### RollingUpdate Strategy

Default strategy that updates Pods incrementally.

Customize it in the Deployment YAML:

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

**Explanation:**

- **maxUnavailable**: Maximum number of Pods that can be unavailable during the update.
- **maxSurge**: Additional Pods that can be created above the desired number of replicas.

#### Recreate Strategy

This strategy stops all existing Pods before creating new ones.

In your Deployment YAML, specify:

```yaml
spec:
  strategy:
    type: Recreate
```

### Autoscaling a Deployment

Kubernetes can automatically scale your Deployment based on CPU utilization.

#### Create a Horizontal Pod Autoscaler (HPA)

```bash
kubectl autoscale deploy nginx-deployment --cpu-percent=50 --min=3 --max=10
```

**Explanation:**

- `--cpu-percent=50`: Target average CPU utilization.
- `--min=3`: Minimum number of replicas.
- `--max=10`: Maximum number of replicas.

#### Verify the HPA

```bash
kubectl get hpa
```

**Sample Output:**

```bash
NAME               REFERENCE                     TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx-deployment   Deployment/nginx-deployment   10%/50%   3         10        3          1m
```

### Deleting a Deployment

To delete a Deployment and its resources:

```bash
kubectl delete deployment nginx-deployment
```

While we are cleaning up, let's delete the autoscale as well:

```bash
kubectl delete hpa nginx-deployment
```

### Practice Exercises

#### Exercise 1: Deployment Rollout and Rollback

Create a Deployment called `webapp` with 3 replicas using the `nginx:1.19` image. The container should be named `nginx`. Use the label `app=webapp` for the selector and pod template.

After the Deployment is running, update the image to `nginx:1.21` using the `kubectl set image` command.

Check the rollout status and wait for it to complete. Then view the rollout history to see the revisions.

Finally, roll back the Deployment to the previous version and verify the pods are running the original `nginx:1.19` image.

**Verify:** Run `kubectl rollout history deploy webapp` to see at least 2 revisions. After rollback, run `kubectl describe deploy webapp` and confirm the image is `nginx:1.19`.

<details>
<summary>Show Solution</summary>

```bash
# Create the deployment imperatively
kubectl create deploy webapp --image=nginx:1.19 --replicas=3

# Update the image
kubectl set image deploy/webapp nginx=nginx:1.21

# Check rollout status
kubectl rollout status deploy webapp

# View rollout history
kubectl rollout history deploy webapp

# Roll back to previous version
kubectl rollout undo deploy webapp

# Verify the image
kubectl describe deploy webapp | grep Image
```

Cleanup: `kubectl delete deploy webapp`

</details>

---

#### Exercise 2: Deployment with Scaling and Update Strategy

Create a Deployment called `api-server` with 2 replicas using the `httpd:2.4` image. The container should be named `httpd`. Use the label `app=api` for the selector and pod template.

Configure the Deployment to use the RollingUpdate strategy with `maxUnavailable: 1` and `maxSurge: 1`.

After the Deployment is running, scale it to 5 replicas using the imperative `kubectl scale` command.

Then update the image to `httpd:2.4.54` and observe the rolling update behavior by running `kubectl get pods -l app=api -w` in a separate terminal (or check pods multiple times during the update).

**Verify:** Run `kubectl get deploy api-server` to confirm 5 replicas are ready. Run `kubectl describe deploy api-server` to confirm the image is `httpd:2.4.54` and the strategy shows `RollingUpdate` with `maxUnavailable: 1` and `maxSurge: 1`.

<details>
<summary>Show Solution</summary>

Create the Deployment manifest `api-server.yaml` using a template from the [official docs](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#creating-a-deployment) and adjusting it to something like:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server
  labels:
    app: api
spec:
  replicas: 2
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
        - name: httpd
          image: httpd:2.4
```

```bash
# Apply the deployment
kubectl apply -f api-server.yaml

# Scale to 5 replicas
kubectl scale deploy api-server --replicas=5

# Update the image (in one terminal, optionally watch with: kubectl get pods -l app=api -w)
kubectl set image deploy/api-server httpd=httpd:2.4.54

# Check rollout status
kubectl rollout status deploy api-server

# Verify the deployment
kubectl describe deploy api-server
```

Cleanup: `kubectl delete deploy api-server`

</details>

---

## Chapter 4: Namespaces

**Namespaces** provide a way to divide cluster resources between multiple users or teams.

**Key Features of Namespaces:**

- **Isolation:** Resources in different namespaces are isolated from each other.
- **Resource Quotas:** Limit resource usage per namespace.
- **Access Control:** Apply Role-Based Access Control (RBAC) policies.

### Working with Namespaces

#### Viewing Existing Namespaces

List all namespaces in your cluster:

```bash
kubectl get ns
```

**Sample Output:**

```bash
NAME              STATUS   AGE
default           Active   10d
kube-system       Active   10d
kube-public       Active   10d
kube-node-lease   Active   10d
```

**Explanation:**

- **default:** The default namespace for resources without a specific namespace.
- **kube-system:** Namespace for Kubernetes system components.
- **kube-public:** Namespace readable by all users, used for cluster bootstrap.

#### Creating a Namespace Imperatively

As easy as running:

```bash
kubectl create ns dev-env
```

#### Creating a Namespace Declaratively with YAML

Create a file named `ns.yaml`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev-env
```

Apply the configuration:

```bash
kubectl apply -f ns.yaml
```

#### Switching Between Namespaces

By default, `kubectl` commands operate in the `default` namespace. To work in a different namespace, you can:

- Use the `-n` flag with the name of the namespace after it:

```bash
kubectl get pods -n dev-env
```

- Or set the default namespace in your context, so that you don't have to repeat the `-n` flag every time:

```bash
kubectl config set-context --current --namespace=dev-env
```

#### Deploying Resources in a Namespace

Let's deploy an application in the `dev-env` namespace.

We will create a deployment imperatively:

```bash
kubectl create deploy nginx-deploy-dev --image=nginx --replicas=3 -n dev-env
```

Let's verify it's been created:

```bash
kubectl get deploy -n dev-env
```

### Resource Quotas and Limits

Namespaces allow you to set resource quotas to limit resource consumption, such as CPU, memory, and storage.

#### Creating a Resource Quota

There is no way to create a ResourceQuota imperatively, so we will go with a YAML file. Let's call it `rq.yaml`:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-rq
  namespace: dev-env
spec:
  hard:
    pods: "5"
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
```

Apply the configuration:

```bash
kubectl create -f rq.yaml
```

**Explanation:**

- **pods:** Maximum number of Pods.
- **requests.cpu/memory:** Total CPU and memory requested by all Pods.
- **limits.cpu/memory:** Total CPU and memory limits for all Pods.

#### Verifying the Resource Quota

Check the resource quotas:

```bash
kubectl get resourcequota -n dev-env
```

Describe the quota:

```bash
kubectl describe resourcequota dev-rq -n dev-env
```

### Limit Ranges

Limit ranges restrict the minimum and maximum resource usage per Pod or Container.

#### Creating a Limit Range

Again, we have to go with the declarative approach here. Create a file named `lr.yaml`:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: dev-lr
  namespace: dev-env
spec:
  limits:
    - type: Container
      max:
        cpu: "1"
        memory: 512Mi
      min:
        cpu: 100m
        memory: 128Mi
      default:
        cpu: 500m
        memory: 256Mi
      defaultRequest:
        cpu: 200m
        memory: 256Mi
```

Apply the configuration:

```bash
kubectl create -f lr.yaml
```

**Explanation:**

- **max/min:** Maximum and minimum CPU and memory per container.
- **default:** Default CPU and memory **limits** applied if not specified.
- **defaultRequest:** Default CPU and memory **requests** if not specified.

#### Verifying the Limit Range

Check the limit ranges:

```bash
kubectl get limitrange -n dev-env
```

Describe the limit range:

```bash
kubectl describe limitrange dev-lr -n dev-env
```

### Applying RBAC Policies

Namespaces allow you to apply access controls using Role-Based Access Control (RBAC).

#### Creating a Role

Create a file named `role.yaml`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev-env
  name: dev-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "watch", "list"]
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "watch", "list"]
```

Apply the configuration:

```bash
kubectl apply -f role.yaml
```

To do something similar imperatively, you would run a command such as:

```bash
kubectl create role dev-reader\
    --namespace=dev-env\
    --verb=get --verb=watch --verb=list\
    --resource=pods --resource=deployments
```

#### Creating a RoleBinding

Once a Role is created, you can assign it to users using a RoleBinding.

Create a file named `rb.yaml`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-reader-binding
  namespace: dev-env
subjects:
  - kind: User
    name: jane
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: dev-reader
  apiGroup: rbac.authorization.k8s.io
```

Apply the configuration:

```bash
kubectl apply -f rb.yaml
```

That will associate the `dev-reader` Role with the user `jane` in the `dev-env` namespace.

To do something similar imperatively, you would run a command such as:

```bash
kubectl create rolebinding dev-reader-binding\
    --namespace=dev-env\
    --role=dev-reader\
    --user=jane
```

To verify that it worked as expected, we can use the `auth can-i` command and see all privileges of `jane`:

```bash
kubectl auth can-i --as=jane --list -n dev-env
```

The output will look something like:

```bash
Resources   Non-Resource URLs   Resource Names   Verbs
pods        []                  []               [get watch list]
deployments []                  []               [get watch list]
```

#### Service Accounts

While user accounts are intended for human access, **ServiceAccounts** are meant for applications running within the cluster. For example, when Pods need to perform actions such as reading secrets or interacting with other resources, you can use a Service Account to define the appropriate permissions.

To create a Service Account, you can use something like this `service-account.yaml` file:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dev-app
  namespace: dev-env
```

Apply the configuration:

```bash
kubectl apply -f service-account.yaml
```

You can also create a Service Account imperatively:

```bash
kubectl create serviceaccount dev-app --namespace=dev-env
```

#### Generating Service Account Tokens

To generate a token for a Service Account:

```bash
kubectl create token dev-app --namespace=dev-env
```

This produces a JWT token that can be used for authentication.

#### Binding a Role to a Service Account

Once a Service Account is created, you can bind it to a Role or ClusterRole using a RoleBinding or ClusterRoleBinding. This allows the Service Account to have specific permissions defined by the Role.

For example, to bind the `dev-reader` Role created earlier to the `dev-app` Service Account, create a RoleBinding such as in this `dev-app-binding.yaml`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-app-binding
  namespace: dev-env
subjects:
  - kind: ServiceAccount
    name: dev-app
    namespace: dev-env
roleRef:
  kind: Role
  name: dev-reader
  apiGroup: rbac.authorization.k8s.io
```

Apply the configuration:

```bash
kubectl apply -f dev-app-binding.yaml
```

#### Using the Service Account in a Pod

To use the Service Account in a Pod, specify it in the Pod's configuration:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-app
  namespace: dev-env
spec:
  serviceAccountName: dev-app
  containers:
    - name: app-container
      image: nginx
```

This configuration ensures that the Pod runs with the `dev-app` Service Account, inheriting its permissions.

#### Verifying Permissions

You can verify the permissions of the Service Account using the `auth can-i` command:

```bash
kubectl auth can-i --as=system:serviceaccount:dev-env:dev-app --list -n dev-env
```

The output will show the resources and actions that the `dev-app` Service Account is allowed to perform:

```bash
Resources   Non-Resource URLs   Resource Names   Verbs
pods        []                  []               [get watch list]
deployments []                  []               [get watch list]
```

### Deleting a Namespace

To delete a namespace and _all resources within it_:

```bash
kubectl delete ns dev-env
```

We can verify that the deployment was deleted along with the namespace by listing all deployments (or any other resource we have created) using the `-A` flag:

```bash
kubectl get deploy -A
```

You won't see the `nginx-deploy-dev` deployment that we created at the beginning of this chapter. If you try the same with the other resources from the `dev-env` namespace, you will see that all of them are deleted.

### Recap

- **Namespaces** help organize and isolate resources in a cluster.
- **Resource Quotas** limit the amount of resources a namespace can consume.
- **Limit Ranges** set minimum and maximum resource limits for Pods and Containers.
- **RBAC Policies** can be applied within namespaces to control access.

### Practice Exercises

#### Exercise 1: Namespace with Resource Quota and Deployment

Create a namespace called `production`.

Create a ResourceQuota called `prod-quota` in the `production` namespace that limits the namespace to a maximum of 4 pods, 2 CPU requests, and 2Gi memory requests.

Create a Deployment called `web-app` in the `production` namespace with 2 replicas using the `nginx:alpine` image. The container should be named `nginx` and must specify resource requests of `100m` CPU and `128Mi` memory.

**Verify:** Run `kubectl get resourcequota prod-quota -n production` to see the quota usage. Run `kubectl get deploy web-app -n production` to confirm the deployment has 2 ready replicas.

<details>
<summary>Show Solution</summary>

```bash
# Create the namespace
kubectl create ns production

# Generate ResourceQuota YAML
kubectl create quota prod-quota -n production --hard=pods=4,requests.cpu=2,requests.memory=2Gi

# Generate Deployment YAML and edit to add resource requests
kubectl create deploy web-app --image=nginx:alpine --replicas=2 -n production --dry-run=client -o yaml > web-app.yaml
```

Edit `web-app.yaml` to add resource requests:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: production
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
        - name: nginx
          image: nginx:alpine
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
```

```bash
kubectl apply -f web-app.yaml

# Verify
kubectl get resourcequota prod-quota -n production
kubectl get deploy web-app -n production
```

Cleanup: `kubectl delete ns production`

</details>

---

#### Exercise 2: ServiceAccount with Role and RoleBinding

Create a namespace called `app-team`.

Create a ServiceAccount called `app-service` in the `app-team` namespace.

Create a Role called `pod-manager` in the `app-team` namespace that allows `get`, `list`, `watch`, `create`, and `delete` operations on `pods`.

Create a RoleBinding called `app-service-binding` that binds the `pod-manager` Role to the `app-service` ServiceAccount.

Create a pod called `worker` in the `app-team` namespace using the `busybox` image that sleeps for 3600 seconds. The pod should use the `app-service` ServiceAccount.

**Verify:** Run `kubectl auth can-i create pods --as=system:serviceaccount:app-team:app-service -n app-team` and confirm it returns `yes`. Run `kubectl get pod worker -n app-team` to confirm the pod is running with the correct ServiceAccount.

<details>
<summary>Show Solution</summary>

```bash
# Create the namespace
kubectl create ns app-team

# Create the ServiceAccount
kubectl create serviceaccount app-service -n app-team

# Create the Role
kubectl create role pod-manager -n app-team --verb=get,list,watch,create,delete --resource=pods

# Create the RoleBinding
kubectl create rolebinding app-service-binding -n app-team --role=pod-manager --serviceaccount=app-team:app-service

# Generate pod YAML and edit to add serviceAccountName
kubectl run worker --image=busybox -n app-team --dry-run=client -o yaml -- sleep 3600 > worker.yaml
```

Edit `worker.yaml` to add the serviceAccountName:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: worker
  namespace: app-team
spec:
  serviceAccountName: app-service
  containers:
    - name: worker
      image: busybox
      command: ["sleep", "3600"]
```

```bash
kubectl apply -f worker.yaml

# Verify permissions
kubectl auth can-i create pods --as=system:serviceaccount:app-team:app-service -n app-team

# Verify pod
kubectl get pod worker -n app-team -o yaml | grep serviceAccountName
```

Cleanup: `kubectl delete ns app-team`

</details>

---

## Chapter 5: ConfigMaps and Secrets

Next on our list will be managing configuration data and sensitive information with ConfigMaps and Secrets. Let's start with the former.

**ConfigMaps** store non-confidential data in key-value pairs. They allow you to keep configuration separate from application code. You can load configs from environment variables, command-line arguments, or files.

#### Creating ConfigMaps Imperatively

**Option 1:** Create a ConfigMap directly from **key-value pairs** using the `--from-literal` option:

```bash
kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=production
```

**Explanation:**

- `kubectl create configmap`: Command to create a ConfigMap.
- `app-config`: Name of the ConfigMap.
- `--from-literal`: Adds key-value pairs directly.

**Option 2:** Create a ConfigMap imperatively **from a file**.

Suppose you have a file `config.txt`:

```
This is some configuration content.
Multiple lines are stored as a single value.
```

Create a ConfigMap from the file:

```bash
kubectl create cm my-configmap --from-file=config.txt
```

The file name (`config.txt`) becomes the key, and the file contents become the value.

You can specify a custom key name instead of using the name of the file automatically:

```bash
kubectl create cm my-configmap --from-file=custom-key=config.txt
```

This creates a ConfigMap entry with `custom-key` as the key instead of `config.txt`.

**Option 3:** Create a ConfigMap from an **env file** containing key=value pairs:

Suppose you have a file `config.env`:

```bash
APP_COLOR=blue
APP_MODE=production
```

Create a ConfigMap where each line becomes a separate key-value pair:

```bash
kubectl create cm my-configmap --from-env-file=config.env
```

The difference is that `--from-file` stores the entire file as one value, while `--from-env-file` parses key=value pairs into separate ConfigMap entries.

#### Creating ConfigMaps declaratively with YAML

Create a file named `configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: production
```

Apply the configuration:

```bash
kubectl create -f configmap.yaml
```

#### Injecting ConfigMap as Environment Variables

You can create a Pod that uses the ConfigMap values as environment variables.

This would be the `env-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
spec:
  containers:
    - name: demo-full
      image: busybox
      command: ["sh", "-c", "sleep infinity"]
      envFrom:
        - configMapRef:
            name: app-config

    - name: demo-chosen
      image: busybox
      command: ["sh", "-c", "sleep infinity"]
      env:
        - name: APP_STYLE
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_COLOR

    - name: demo-custom
      image: busybox
      command: ["sh", "-c", "sleep infinity"]
      env:
        - name: APP_CUSTOM
          value: "custom"
```

As you can see above, you can either load all variables from the ConfigMap (as in the `demo-full` container), or you can choose particular variables and rename them (as in the `demo-chosen` container, where we picked only `APP_COLOR` and renamed it to `APP_STYLE`). You can also add your own environment variables, ignoring the ConfigMap (as in the `demo-custom` container).

Apply the configuration:

```bash
kubectl apply -f env-pod.yaml
```

Verify the environment variables inside the containers:

```bash
k exec -it env-pod -c demo-full -- env | grep APP
# APP_MODE=production
# APP_COLOR=blue

k exec -it env-pod -c demo-chosen -- env | grep APP
# APP_STYLE=blue

k exec -it env-pod -c demo-custom -- env | grep APP
# APP_CUSTOM=custom
```

#### Mounting ConfigMaps as Files in a Volume

You can turn the ConfigMap into individual files in the directory specified in `mountPath`. The content of each file is the value of the corresponding key from the ConfigMap.

For example, let's use `env-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
spec:
  containers:
    - name: demo-mounted
      image: busybox
      command: ["sh", "-c", "sleep infinity"]
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: app-config
```

Apply the configuration:

```bash
# delete the previously created pod
kubectl delete pod env-pod

# re-create with new spec
kubectl create -f env-pod.yaml
```

Verify the files inside the Pod:

```bash
kubectl exec -it env-pod -- ls /etc/config
```

**Expected Output:**

```bash
APP_COLOR
APP_MODE
```

That means two files were created out of the ConfigMap. We will learn more about Volumes in a later chapter.

#### Updating ConfigMaps

If we want to update the values of a ConfigMap, we can use the `edit configmap` command:

```bash
kubectl edit configmap app-config
```

For example, modify the `APP_COLOR` value and save.

**Note:** Pods need to be restarted to pick up the new ConfigMap values unless they are designed to watch for changes.

---

### Understanding Secrets

**Secrets** are similar to ConfigMaps, but are intended for sensitive information. However, they are not safe and are not encrypted by default! They are just base64-encoded, so they simply prevent accidental exposure of plain data. You have the option to have them encrypted at rest.

#### Creating Secrets Imperatively

For the imperative approach, we use the `create secret generic` command.

Option 1: You can create Secrets **from literal values**:

```bash
kubectl create secret generic app-secret --from-literal=username=admin --from-literal=password=secretpass
```

**Option 2:** You can also create Secrets **from files**. It's slightly different from ConfigMaps, though: we had one file with key=value lines. For secrets, we need a separate file for each secret. The file name will be the secret's name, and its contents will be its value. For example, having two files `username` and `password` (without file extensions) where the contents are the secrets in plain text (not encoded), you would create secrets this way:

```bash
kubectl create secret generic app-secret --from-file=username --from-file=password
```

That command would automatically base64-encode the data and create a secret with two properties.

#### Creating Secrets Declaratively with YAML

First, base64 encode your data:

```bash
echo -n 'admin' | base64
# Output: YWRtaW4=
echo -n 'secretpass' | base64
# Output: c2VjcmV0cGFzcw==
```

Create `secret.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: YWRtaW4=
  password: c2VjcmV0cGFzcw==
```

Apply the configuration:

```bash
kubectl create -f secret.yaml
```

#### Getting Encoded Value from Secret

To get the encoded value, use:

```bash
kubectl get secret app-secret -o yaml
```

You can decode the value this way:

```bash
echo "<SOME_VALUE>" | base64 -d
```

A faster approach combines both steps using `jsonpath` to extract a specific key and pipe it directly to base64 decode:

```bash
kubectl get secret app-secret -o jsonpath="{.data.username}" | base64 -d
```

Replace `username` with any key from your secret.

#### Injecting Secrets as Environment Variables

The Pod definition is very similar to what we used with ConfigMaps. The only differences are:

- use `secretRef` instead of `configMapRef`,
- use `secretKeyRef` instead of `configMapKeyRef`.

This is how a `secret-pod.yaml` could look like:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
    - name: demo-full
      image: busybox
      command: ["sh", "-c", "sleep infinity"]
      envFrom:
        - secretRef:
            name: app-secret

    - name: demo-chosen
      image: busybox
      command: ["sh", "-c", "sleep infinity"]
      env:
        - name: USER_NAME
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: username
```

Apply the configuration:

```bash
kubectl create -f secret-pod.yaml
```

Verify the environment variables inside the containers:

```bash
k exec -it secret-pod -c demo-full -- env
# password=secretpass
# username=admin

k exec -it secret-pod -c demo-chosen -- env
# USER_NAME=admin
```

**Note:** The output shows decoded secrets turned into environment variables.

#### Mounting Secrets as Files in a Volume

Mounting Secrets is almost identical to mounting ConfigMaps. The only difference is how you specify the volume's source. Compare loading a ConfigMap we used previously:

```yaml
volumes:
  - name: config-volume
    configMap:
      name: app-config
```

And loading the newly created Secret instead:

```yaml
volumes:
  - name: secret-volume
    secret:
      secretName: app-secret
```

We will learn more about Volumes in an upcoming chapter.

### Cleanup

Remove the resources we created in this chapter:

```bash
kubectl delete pod env-pod
kubectl delete pod secret-pod
kubectl delete configmap app-config
kubectl delete secret app-secret
```

### Recap

- **ConfigMaps** store non-confidential configuration data.
- **Secrets** store sensitive information but not securely by default.
- **Injection Methods:** Both can be used as environment variables or mounted as volumes.

### Practice Exercises

#### Exercise 1: ConfigMap with Environment Variables and Volume Mount

Create a ConfigMap called `db-config` with the following key-value pairs: `DB_HOST=mysql.default.svc`, `DB_PORT=3306`, and `DB_NAME=appdb`.

Create a pod called `db-client` using the `busybox` image that sleeps for 3600 seconds. The container should be named `client`.

The pod should inject the `DB_HOST` and `DB_PORT` values from the ConfigMap as environment variables, but rename them to `DATABASE_HOST` and `DATABASE_PORT` respectively.

Additionally, mount the entire ConfigMap as a volume at `/etc/db-config`.

**Verify:** Run `kubectl exec db-client -- env | grep DATABASE` to see the renamed environment variables. Run `kubectl exec db-client -- ls /etc/db-config` to confirm all three keys exist as files. Run `kubectl exec db-client -- cat /etc/db-config/DB_NAME` to confirm the value is `appdb`.

<details>
<summary>Show Solution</summary>

```bash
# Create the ConfigMap
kubectl create configmap db-config --from-literal=DB_HOST=mysql.default.svc --from-literal=DB_PORT=3306 --from-literal=DB_NAME=appdb

# Generate pod YAML
kubectl run db-client --image=busybox --dry-run=client -o yaml -- sleep 3600 > db-client.yaml
```

Edit `db-client.yaml` to add env vars and volume mount:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: db-client
spec:
  containers:
    - name: client
      image: busybox
      command: ["sleep", "3600"]
      env:
        - name: DATABASE_HOST
          valueFrom:
            configMapKeyRef:
              name: db-config
              key: DB_HOST
        - name: DATABASE_PORT
          valueFrom:
            configMapKeyRef:
              name: db-config
              key: DB_PORT
      volumeMounts:
        - name: config-volume
          mountPath: /etc/db-config
  volumes:
    - name: config-volume
      configMap:
        name: db-config
```

```bash
kubectl apply -f db-client.yaml

# Verify
kubectl exec db-client -- env | grep DATABASE
kubectl exec db-client -- ls /etc/db-config
kubectl exec db-client -- cat /etc/db-config/DB_NAME
```

Cleanup: `kubectl delete pod db-client && kubectl delete configmap db-config`

</details>

---

#### Exercise 2: Secret with Deployment

Create a Secret called `api-credentials` with two keys: `API_KEY` with value `super-secret-key-123` and `API_SECRET` with value `my-api-secret`.

Create a Deployment called `api-consumer` with 2 replicas using the `nginx:alpine` image. The container should be named `nginx`.

The containers should have all keys from the Secret injected as environment variables.

Additionally, mount the Secret as a volume at `/etc/api-secrets` with file permission mode `0400`.

**Verify:** Run `kubectl exec deploy/api-consumer -- env | grep API` to see both environment variables. Run `kubectl exec deploy/api-consumer -- cat /etc/api-secrets/API_KEY` to confirm the value is `super-secret-key-123`.

<details>
<summary>Show Solution</summary>

```bash
# Create the Secret
kubectl create secret generic api-credentials --from-literal=API_KEY=super-secret-key-123 --from-literal=API_SECRET=my-api-secret

# Generate deployment YAML
kubectl create deploy api-consumer --image=nginx:alpine --replicas=2 --dry-run=client -o yaml > api-consumer.yaml
```

Edit `api-consumer.yaml` to add envFrom and volume mount:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-consumer
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api-consumer
  template:
    metadata:
      labels:
        app: api-consumer
    spec:
      containers:
        - name: nginx
          image: nginx:alpine
          envFrom:
            - secretRef:
                name: api-credentials
          volumeMounts:
            - name: secret-volume
              mountPath: /etc/api-secrets
      volumes:
        - name: secret-volume
          secret:
            secretName: api-credentials
            defaultMode: 0400
```

```bash
kubectl apply -f api-consumer.yaml

# Verify
kubectl exec deploy/api-consumer -- env | grep API
kubectl exec deploy/api-consumer -- cat /etc/api-secrets/API_KEY
```

Cleanup: `kubectl delete deploy api-consumer && kubectl delete secret api-credentials`

</details>

---

## Chapter 6: Services and Ingress

**Services** are pretty cool as they are in charge of communication. Imagine them like gates in medieval city walls. Behind the walls are the ever-changing houses (Pods), but the gates remain constant, so you know you will reach the city when you head toward a gate.

**Key Points:**

- **Stable Networking Endpoint:** Services maintain a consistent IP address and DNS name.
- **Load Balancing:** Distribute traffic across multiple Pods.
- **Service Discovery:** Automatically find and connect to Pods.

Kubernetes offers several types of Services, each serving different use cases:

1.  **ClusterIP (Default):** Exposes the Service on an internal IP in the cluster. Accessible only within the cluster.
2.  **NodePort:** Exposes the Service on the same port of each selected Node in the cluster. Makes the Service accessible from outside the cluster using `<NodeIP>:<NodePort>`.
3.  **LoadBalancer:** Creates an external load balancer (if supported by the cloud provider) and assigns a fixed, external IP to the Service.
4.  **ExternalName:** Maps the Service to the contents of the `externalName` field by returning a CNAME record.
5.  **Headless Service (ClusterIP: None):** Does not allocate a ClusterIP; instead, returns the Pod IP addresses directly.

We will now explore the first two types.

#### Prerequisites

Before we start playing with Services, make sure you have a Pod with a label `app=my-app`. You can create one like this:

```bash
kubectl run my-pod --image=nginx
kubectl label pod my-pod app=my-app --overwrite
```

Services use label selectors to determine which Pods to route traffic to. You can see the labels in your Pods with the `--show-labels` parameter.

```bash
kubectl get pods --show-labels
```

#### Creating a ClusterIP Service Declaratively with YAML

Create a file named `svc-clusterip.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-clusterip-svc
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

Apply the configuration:

```bash
kubectl create -f svc-clusterip.yaml
```

**Explanation:**

- **spec.selector:** The label selector to identify Pods to route traffic to.
- **spec.ports:** Defines the ports and protocols.

#### Creating a ClusterIP Service Imperatively

There are two primary ways to create a Service imperatively.

One is to use the `expose` command that automatically creates a Service and links it to the resource via labels.

```bash
kubectl expose pod my-pod --name=my-clusterip-svc --port=80 --type=ClusterIP
```

We could achieve a similar result with the `create svc` command and manually updating the labels it should target:

```bash
kubectl create svc clusterip my-clusterip-svc --tcp=80:80
kubectl patch svc my-clusterip-svc -p '{"spec":{"selector":{"app":"my-app"}}}'
```

#### Verifying the Service

List Services:

```bash
kubectl get services
```

**Sample Output:**

```bash
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
my-clusterip-svc    ClusterIP   10.96.0.1       <none>        80/TCP    1m
```

#### Accessing the Service

Since ClusterIP Services are only accessible within the cluster, we'll need to test them from within a Pod.

Create a test Pod:

```bash
kubectl run test-pod --rm -it --image=busybox -- /bin/sh
```

Inside the Pod, make an HTTP request to the service:

```bash
wget -qO- http://my-clusterip-svc
```

You should see the default NGINX welcome page HTML content.

Exit the Pod:

```bash
exit
```

---

### NodePort Service

A NodePort Service exposes the Service on a static port on each Node's IP.

#### Declarative Approach with YAML

Create `svc-nodeport.yaml` which is very similar to the previous `svc-clusterip.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-svc
spec:
  type: NodePort # Changed
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30007 # Added
```

Apply the configuration:

```bash
kubectl create -f svc-nodeport.yaml
```

**Explanation:**

- **type: NodePort:** Specifies the Service type.
- **nodePort:** The port on which the Service is exposed on each Node.

#### Imperative Approach

We could achieve the same as above with either the `expose` command:

```bash
kubectl expose pod my-pod --name=my-nodeport-svc --port=80 --type=NodePort
```

Or the `create service` command with manually updating the labels it should target:

```bash
kubectl create svc nodeport my-nodeport-svc --tcp=80:80
kubectl patch svc my-nodeport-svc -p '{"spec":{"selector":{"app":"my-app"}}}'
```

#### Accessing the NodePort Service

From outside the cluster, you can access the Service using `<NodeIP>:<NodePort>`.

When using **Minikube** (as you probably are if you were following my guide), you can create a tunnel to the NodePort Service with this command:

```bash
minikube service my-nodeport-svc
```

It will open the default NGINX welcome page in your browser.

If not using Minikube, you can find the Node IP with `kubectl get nodes -o wide` and the assigned NodePort with `kubectl get svc my-nodeport-svc`.

To get the IP and port for reaching the ingress controller:

```bash
kubectl -n ingress-nginx get svc ingress-nginx-controller
```

---

### Ingress

While simple Services are useful, they can become cumbersome when managing multiple applications or when advanced routing is required. **Ingress** provides a more efficient and flexible way to expose multiple Services under a single external IP address and domain.

However, Ingress alone doesn't do anything. You need an **Ingress Controller** to implement these rules. The Ingress Controller watches the Kubernetes API for Ingress resources and configures the underlying load balancer accordingly.

**Common Ingress Controllers:**

- NGINX Ingress Controller
- Traefik
- HAProxy
- AWS ALB Ingress Controller
- GCE Ingress Controller

We'll focus on the **NGINX Ingress Controller**, which is widely used and supports most Kubernetes environments.

#### Minikube Tunnel

Depending on your setup, it's probably necessary to run these commands first:

```bash
minikube addons enable ingress
minikube tunnel
```

#### Installing an Ingress Controller

Apply the official NGINX Ingress Controller manifests:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.13.2/deploy/static/provider/cloud/deploy.yaml
```

**Note:** The version may change over time. Check the [official documentation](https://kubernetes.github.io/ingress-nginx/deploy/) for the latest version.

List all Pods in the `ingress-nginx` namespace:

```bash
kubectl get pods -n ingress-nginx
```

**Sample Output:**

```bash
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-controller-5d8c4d4b77-abcde   1/1     Running     0          2m
```

#### Deploying Sample Applications

Deploy three applications so that we can check how Ingress routing works:

```bash
kubectl create deploy web-app --image=hashicorp/http-echo --replicas=2 -- /http-echo -text="Welcome to the Web App"
kubectl expose deploy web-app --name=web-svc --type=NodePort --port=5678

kubectl create deploy api-app --image=hashicorp/http-echo --replicas=2 -- /http-echo -text="Welcome to the API App"
kubectl expose deploy api-app --name=api-svc --type=NodePort --port=5678

kubectl create deploy test-app --image=nginx
```

To expose the services with Minikube, open two separate terminals and use these commands:

```bash
# New Terminal A
minikube service web-svc --url
# Visit the IP:Port, expexted result: Welcome to the Web App

# New Terminal B
minikube service api-svc --url
# Visit the IP:Port, expexted result: Welcome to the API App
```

#### Creating an Ingress Resource

Create `ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
    - host: localhost
      http:
        paths:
          - path: /web
            pathType: Prefix
            backend:
              service:
                name: web-svc
                port:
                  number: 5678
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-svc
                port:
                  number: 5678
```

Apply the configuration:

```bash
kubectl apply -f ingress.yaml
```

#### Verifying and Accessing Ingress

Verify the IP address is set:

```bash
kubectl get ingress
```

It can take a couple of minutes, but you should see an IPv4 address in the `ADDRESS` column; for example:

```bash
NAME         CLASS   HOSTS       ADDRESS        PORTS   AGE
my-ingress   nginx   localhost   192.168.49.2   80      5m
```

The network is limited if using the Docker driver on MacOS (Darwin) and the Node IP is not reachable directly. You will have to open a new terminal and run `minikube tunnel` and when prompted, provide your `sudo` password:

```bash
minikube tunnel
```

Now you can open your browser and visit `http://localhost/web` as well as `http://localhost/api`.

---

### Network Policies

**Network Policies** allow you to control the flow of traffic between Pods and network endpoints.

Your network plugin must support Network Policies (e.g., Calico, Weave Net). To enable it with Minikube, stop Minikube first (`minikube stop`) and then start it with this command:

```bash
minikube start --cni calico
```

#### Deploying Sample Applications

We will reuse the **API** app and **Test** app from before to demonstrate Network Policies.

#### Testing Default Connectivity

By default, Pods can communicate freely.

For convenience, let's first save the Pod IP of the API Pod and the name of the Test Pod that we created earlier:

```bash
API_POD_IP=$(kubectl get pod -l app=api-app -o jsonpath='{.items[0].status.podIP}')
TEST_POD_NAME=$(kubectl get pod -l app=test-app -o jsonpath='{.items[0].metadata.name}')
```

Test connectivity:

```bash
kubectl exec -it $TEST_POD_NAME -- curl $API_POD_IP:5678
# Output: Welcome to the API App
```

#### Applying a Deny-All Network Policy

Create `deny-all.yaml` to deny all ingress traffic to the API:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector:
    matchLabels:
      app: api-app
  policyTypes:
    - Ingress
```

Apply the policy:

```bash
kubectl apply -f deny-all.yaml
```

Check it's applied:

```bash
kubectl get networkpolicy
```

Delete both Pods from the `api-app` Deployment, so that they are recreated with the Network Policy in place.

Attempt to curl the API from the Test Pod again:

```bash
kubectl exec -it $TEST_POD_NAME -- curl $API_POD_IP:5678
```

The request should time out or fail, indicating blocked traffic.

#### Allowing Specific Ingress Traffic

Create `allow-from-test.yaml` to allow traffic from the frontend:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-test
spec:
  podSelector:
    matchLabels:
      app: api-app
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: test-app
      ports:
        - protocol: TCP
          port: 5678
  policyTypes:
    - Ingress
```

Apply the policy and remove the deny-all policy:

```bash
kubectl apply -f allow-from-test.yaml
kubectl delete networkpolicy deny-all
```

Delete both Pods from the `api-app` Deployment, so that they are recreated with the Network Policy in place.

From the frontend Pod:

```bash
kubectl exec -it $TEST_POD_NAME -- curl $API_POD_IP:5678
# Output: Welcome to the API App
```

From an unauthorized Pod:

```bash
# Note down the API POD IP
kubectl get pod -l app=api-app -o jsonpath='{.items[0].status.podIP}'

# Start a temporal pod
kubectl run temp-pod --rm -it --image=busybox -- /bin/sh

# Once inside the temporal pod, update the API POD IP and query it
wget -qO- <API_POD_IP>:5678
```

The request should fail, confirming that access is restricted.

#### Controlling Egress Traffic

Egress is traffic leaving the Pod. Create `deny-egress.yaml` to block egress traffic from the Test Pod:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-egress
spec:
  podSelector:
    matchLabels:
      app: test-app
  policyTypes:
    - Egress
```

Apply the policy:

```bash
kubectl apply -f deny-egress.yaml
```

Inside the backend Pod:

```bash
kubectl exec -it $TEST_POD_NAME -- curl http://google.com
```

The request should fail, indicating egress traffic is blocked.

#### Allowing DNS in Egress Policies

When blocking all egress traffic, pods lose the ability to resolve DNS names. DNS is handled by the `kube-dns` service, which listens on port 53 for both UDP and TCP. To block egress while still allowing DNS resolution:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restricted-egress
spec:
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
    - Egress
    - Ingress
  ingress:
    - {}
  egress:
    - ports:
        - port: 53
          protocol: UDP
        - port: 53
          protocol: TCP
```

This policy allows all ingress traffic, blocks most egress traffic, but permits DNS queries to function.

### Recap

- **Services** provide stable endpoints and load balancing for Pods.
- **Ingress** offers advanced routing to expose multiple Services.
- **Network Policies** enhance security by controlling traffic flow.
- **Selectors and Labels** are crucial in defining how resources interact.

### Clean Up

To remove all resources created:

```bash
kubectl delete pod my-pod
kubectl delete deploy web-app api-app test-app
kubectl delete svc my-clusterip-svc my-nodeport-svc web-svc api-svc
kubectl delete ingress my-ingress
kubectl delete networkpolicy deny-all allow-from-test deny-egress restricted-egress
```

### Practice Exercises

#### Exercise 1: Service Exposure and Connectivity

Create a Deployment called `backend` with 2 replicas using the `nginx:alpine` image. The container should be named `nginx`.

Expose the Deployment as a ClusterIP Service called `backend-svc` on port 80.

Create a pod called `frontend` using the `busybox` image that sleeps for 3600 seconds.

From the `frontend` pod, verify you can reach the `backend-svc` Service using both the Service name and the full DNS name (`backend-svc.default.svc.cluster.local`).

**Verify:** Run `kubectl exec frontend -- wget -qO- --timeout=5 backend-svc` and `kubectl exec frontend -- wget -qO- --timeout=5 backend-svc.default.svc.cluster.local` to confirm both return the nginx welcome page HTML.

<details>
<summary>Show Solution</summary>

```bash
# Create the deployment
kubectl create deploy backend --image=nginx:alpine --replicas=2

# Expose as ClusterIP service
kubectl expose deploy backend --name=backend-svc --port=80

# Create the frontend pod
kubectl run frontend --image=busybox -- sleep 3600

# Wait for pods to be ready
kubectl wait --for=condition=Ready pod -l app=backend --timeout=60s
kubectl wait --for=condition=Ready pod/frontend --timeout=60s

# Test connectivity using service name
kubectl exec frontend -- wget -qO- --timeout=5 backend-svc

# Test connectivity using full DNS name
kubectl exec frontend -- wget -qO- --timeout=5 backend-svc.default.svc.cluster.local
```

Cleanup: `kubectl delete deploy backend && kubectl delete svc backend-svc && kubectl delete pod frontend`

</details>

---

#### Exercise 2: Network Policy to Restrict Pod Communication

Create a namespace called `secure-ns`.

Create a Deployment called `db` in the `secure-ns` namespace with 1 replica using the `nginx:alpine` image. Add the label `role=database` to the pod template.

Expose the Deployment as a ClusterIP Service called `db-svc` on port 80.

Create a pod called `allowed-client` in the `secure-ns` namespace using the `busybox` image with label `access=allowed` that sleeps for 3600 seconds.

Create a pod called `denied-client` in the `secure-ns` namespace using the `busybox` image that sleeps for 3600 seconds (no special labels).

Create a NetworkPolicy called `db-access-policy` in the `secure-ns` namespace that only allows ingress traffic to pods with label `role=database` from pods with label `access=allowed` on port 80.

**Verify:** Run `kubectl exec -n secure-ns allowed-client -- wget -qO- --timeout=5 db-svc` and confirm it succeeds. Run `kubectl exec -n secure-ns denied-client -- wget -qO- --timeout=5 db-svc` and confirm it times out or fails.

<details>
<summary>Show Solution</summary>

```bash
# Create the namespace
kubectl create ns secure-ns

# Create the deployment with role=database label
kubectl create deploy db --image=nginx:alpine --replicas=1 -n secure-ns --dry-run=client -o yaml > db-deploy.yaml
```

Edit `db-deploy.yaml` to add the `role=database` label to the pod template:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db
  namespace: secure-ns
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
        role: database
    spec:
      containers:
        - name: nginx
          image: nginx:alpine
```

Run the following commands:

```bash
kubectl apply -f db-deploy.yaml

# Expose the deployment
kubectl expose deploy db --name=db-svc --port=80 -n secure-ns

# Create allowed client
kubectl run allowed-client --image=busybox --labels="access=allowed" -n secure-ns -- sleep 3600

# Create denied client
kubectl run denied-client --image=busybox -n secure-ns -- sleep 3600

# Wait for pods to be ready
kubectl wait --for=condition=Ready pod -l app=db -n secure-ns --timeout=60s
kubectl wait --for=condition=Ready pod/allowed-client -n secure-ns --timeout=60s
kubectl wait --for=condition=Ready pod/denied-client -n secure-ns --timeout=60s
```

Create `db-access-policy.yaml` using a template from the [official docs](https://kubernetes.io/docs/concepts/services-networking/network-policies/#networkpolicy-resource):

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-access-policy
  namespace: secure-ns
spec:
  podSelector:
    matchLabels:
      role: database
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              access: allowed
      ports:
        - protocol: TCP
          port: 80
```

Run the following commands:

```bash
kubectl apply -f db-access-policy.yaml

# Test from allowed client (should succeed)
kubectl exec -n secure-ns allowed-client -- wget -qO- --timeout=5 db-svc

# Test from denied client (should timeout/fail)
kubectl exec -n secure-ns denied-client -- wget -qO- --timeout=5 db-svc
```

Cleanup: `kubectl delete ns secure-ns`

</details>

---

## Chapter 7: Volumes and Persistent Storage

In previous chapters, we've focused on deploying and scaling applications, configuring them with ConfigMaps and Secrets, and securing them with Network Policies.

The data generated by these applications is not stable at this point; it disappears when the Pod is deleted or fails. To ensure data persistence beyond the lifecycle of individual Pods, Kubernetes provides mechanisms like **Volumes**, **PersistentVolumes**, **PersistentVolumeClaims**, and **StorageClasses**.

**Key Points:**

- **Volumes:** Attach storage to Pods to allow data to persist across container restarts within the same Pod.
- **PersistentVolumes (PV):** Cluster-wide storage resources provisioned by administrators.
- **PersistentVolumeClaims (PVCs):** User requests for storage.
- **StorageClasses:** Define how storage is dynamically provisioned.

#### Understanding Volumes

A **Volume** in Kubernetes is a directory accessible to containers in a Pod. Volumes allow containers to share data and persist data across container restarts within the same Pod.

**Common Volume Types:**

- **configMap & secret:** Mount ConfigMaps and Secrets as files.
- **emptyDir:** Use a temporary directory that exists as long as the Pod is running.
- **hostPath:** Mount a file or directory from the host node's filesystem.
- **PersistentVolumeClaim:** Attach storage using a PVC.

#### Mounting ConfigMaps and Secrets

We saw how to mount ConfigMaps and Secrets in Chapter 5.

Having this ConfigMap and Secret:

```bash
kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=production
kubectl create secret generic app-secret --from-literal=username=admin --from-literal=password=secretpass
```

Let's mount them by creating a `mounted-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mounted-pod
spec:
  containers:
    - name: demo-mounted
      image: busybox
      command: ["sh", "-c", "sleep infinity"]
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
        - name: secret-volume
          mountPath: /etc/secret
  volumes:
    - name: config-volume
      configMap:
        name: app-config
    - name: secret-volume
      secret:
        secretName: app-secret
```

Apply the configuration:

```bash
kubectl create -f mounted-pod.yaml
```

Verify the files inside the Pod:

```bash
kubectl exec -it mounted-pod -- ls /etc/config -- /etc/secret
```

Expected output:

```bash
/etc/config:
APP_COLOR  APP_MODE

/etc/secret:
password  username
```

**Note:** Data persists across container restarts within the Pod but not beyond the Pod's lifecycle.

#### Using `emptyDir` Volume

An `emptyDir` volume is first created when a Pod is assigned to a Node and exists as long as that Pod runs on that Node.

Let's say we want to share data between two Containers in a single Pod. We would use a definition such as `shared-volume-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-volume-pod
spec:
  containers:
    - name: writer
      image: busybox
      command:
        - sh
        - -c
        - |
          echo "Hello from writer" > /data/message.txt
          sleep 3600
      volumeMounts:
        - name: shared-data
          mountPath: /data
    - name: reader
      image: busybox
      command:
        - sh
        - -c
        - |
          sleep 5
          cat /data/message.txt
          sleep 3600
      volumeMounts:
        - name: shared-data
          mountPath: /data
  volumes:
    - name: shared-data
      emptyDir: {}
```

In case you are wondering, using the `|` symbol in YAML allows you to write a block of text that preserves line breaks. I divided the `command` for each container into separate commands for better readability.

Apply the configuration:

```bash
kubectl apply -f shared-volume-pod.yaml
```

Check the logs of the `reader` container:

```bash
kubectl logs shared-volume-pod -c reader
```

**Expected Output:**

```bash
Hello from writer
```

**Explanation:**

- **emptyDir:** Creates a temporary directory shared between containers.
- **volumeMounts:** Mounts the volume into the containers at `/data`.
- **containers:** One container writes to the volume, the other reads from it.

**Note:** Data persists across container restarts within the Pod but not beyond the Pod's lifecycle.

#### Understanding Persistent Volumes

A **PersistentVolume** is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using StorageClasses. It is a resource in the cluster, just like a Node.

A **PersistentVolumeClaim** is a user's request for storage. It is similar to a Pod requesting compute resources.

**Binding Process:**

1.  **A user creates a PVC** specifying size, access modes, and possibly a storage class.
2.  **Kubernetes matches** the PVC to a suitable PV.
3.  **PVC is bound** to the PV, and the storage is available to the Pod.

#### Setting Up Minikube

Minikube comes with a built-in storage class provisioner that you can enable along with the default StorageClass:

```bash
minikube addons enable default-storageclass
minikube addons enable storage-provisioner
```

Once enabled, you can check the available storage classes in your Minikube cluster:

```bash
kubectl get storageclass
```

You should see a default `standard` storage class provided by Minikube. Before we get to Storage Classes, let's first explore simple Persistent Volumes.

#### Creating a Persistent Volume

Let's create a PV that provides storage from the host's filesystem using `my-pv.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
```

Apply the configuration:

```bash
kubectl apply -f my-pv.yaml
```

**Explanation:**

- **capacity:** Specifies the PV's size.
- **accessModes:** Defines how the PV can be accessed.
- **ReadWriteOnce (RWO):** Mounted as read-write by a single node.
- **ReadOnlyMany (ROX):** Mounted as read-only by many nodes.
- **ReadWriteMany (RWX):** Mounted as read-write by many nodes.
- **hostPath:** Uses a directory on the host node.

#### Creating a Persistent Volume Claim

Now, we'll create a PVC that requests storage using `my-pvc.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Ki
```

Apply the configuration:

```bash
kubectl apply -f my-pvc.yaml
```

Kubernetes registers the claim and tries to find a suitable Persistent Volume.

#### Verifying PV and PVC Binding

List PVs and PVCs:

```bash
kubectl get pv
kubectl get pvc
```

**Sample Output:**

```bash
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM            STORAGECLASS   AGE
my-pv    1Mi        RWO            Retain           Bound    default/my-pvc                  1m

NAME      STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
my-pvc    Bound    my-pv    1Mi        RWO                           1m
```

Notice that `my-pvc` is `Bound` to `my-pv`.

#### Using PVCs in Pods

We can now use the PVC in a Pod to access persistent storage. This is what such a Pod could look like using `pod-with-pvc.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-pvc
spec:
  containers:
    - name: app-container
      image: busybox
      command:
        - sh
        - -c
        - |
          echo $(date) >> /share/log.txt
          sleep 3600
      volumeMounts:
        - mountPath: "/share"
          name: data-storage
  volumes:
    - name: data-storage
      persistentVolumeClaim:
        claimName: my-pvc
```

Apply the configuration:

```bash
kubectl apply -f pod-with-pvc.yaml
```

#### Verifying Data Persistence

Wait for the Pod to run, then check the contents of the file:

```bash
kubectl exec -it pod-with-pvc -- cat /share/log.txt
```

You should see a timestamp. Now the fun part starts. Delete the Pod:

```bash
kubectl delete pod pod-with-pvc
```

Recreate the Pod:

```bash
kubectl apply -f pod-with-pvc.yaml
```

Check the contents again:

```bash
kubectl exec -it pod-with-pvc -- cat /share/log.txt
```

**Expected Output:**

The previous log entry should still be present, along with a new timestamp.

#### Storage Classes

**StorageClasses** provide a way to describe the "classes" of storage available. Many Kubernetes clusters come with a default StorageClass. Find out what you have:

```bash
kubectl get storageclass
```

**Sample Output:**

```bash
NAME                 PROVISIONER          RECLAIMPOLICY   VOLUMEBINDINGMODE   AGE
standard (default)   kubernetes.io/gce-pd   Delete          Immediate          10d
```

#### Creating a StorageClass

We can create a simple StorageClass using something like the following `storageclass.yaml`:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: k8s.io/minikube-hostpath
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

Apply the configuration:

```bash
kubectl apply -f storageclass.yaml
```

**Explanation:**

- **provisioner:** Specifies the storage backend.
- **reclaimPolicy:** What happens to the PV when the PVC is deleted (`Retain`, `Delete`, `Recycle`).
- **volumeBindingMode:** When volume binding and dynamic provisioning occur (`Immediate`, `WaitForFirstConsumer`).

#### Dynamic Provisioning with PVCs

When you create a PVC that specifies a StorageClass, Kubernetes dynamically provisions a PV that matches the claim. For example, `dynamic-pvc.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  storageClassName: fast-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Mi
```

Apply the configuration:

```bash
kubectl apply -f dynamic-pvc.yaml
```

Verify that the PVC got bound:

```bash
kubectl get pvc
```

And also verify that a PV has been dynamically provisioned:

```bash
kubectl get pv
```

**Expected Output:**

A new PV with the `fast-storage` class and bound to `dynamic-pvc`.

### Cleanup

Delete the resources created:

```bash
kubectl delete pod mounted-pod shared-volume-pod pod-with-pvc
kubectl delete pvc my-pvc dynamic-pvc
kubectl delete pv my-pv
kubectl delete storageclass fast-storage
```

### Recap

- **Volumes** in Kubernetes provide a way for containers within a Pod to access shared storage.
- **PersistentVolumes (PVs)** are cluster resources that represent real storage.
- **PersistentVolumeClaims (PVCs)** are user requests for storage.
- **StorageClasses** enable dynamic provisioning of PVs based on PVCs.

### Practice Exercises

#### Exercise 1: Shared Volume Between Containers

Create a pod called `log-processor` with two containers that share an `emptyDir` volume.

The first container should be named `log-writer` using the `busybox` image. It should write the current date and a message "Log entry" to `/logs/app.log` every 5 seconds in a loop, then continue running.

The second container should be named `log-reader` using the `busybox` image. It should continuously tail the `/logs/app.log` file using `tail -f`.

Both containers should mount the shared volume at `/logs`.

**Verify:** Run `kubectl logs log-processor -c log-reader` and confirm you see timestamped log entries appearing. Run `kubectl exec log-processor -c log-writer -- cat /logs/app.log` to see all accumulated entries.

<details>
<summary>Show Solution</summary>

```bash
kubectl run log-processor --image=busybox --dry-run=client -o yaml -- sleep 3600 > log-processor.yaml
```

Edit `log-processor.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: log-processor
spec:
  containers:
    - name: log-writer
      image: busybox
      command:
        - sh
        - -c
        - |
          while true; do
            echo "$(date) - Log entry" >> /logs/app.log
            sleep 5
          done
      volumeMounts:
        - name: shared-logs
          mountPath: /logs
    - name: log-reader
      image: busybox
      command:
        - sh
        - -c
        - tail -f /logs/app.log
      volumeMounts:
        - name: shared-logs
          mountPath: /logs
  volumes:
    - name: shared-logs
      emptyDir: {}
```

```bash
kubectl apply -f log-processor.yaml

# Wait for pod to be ready
kubectl wait --for=condition=Ready pod/log-processor --timeout=60s

# Check logs from the reader (wait ~10 seconds first)
kubectl logs log-processor -c log-reader

# Check file contents directly
kubectl exec log-processor -c log-writer -- cat /logs/app.log
```

Cleanup: `kubectl delete pod log-processor`

</details>

---

#### Exercise 2: PersistentVolume and PersistentVolumeClaim

Create a PersistentVolume called `data-pv` with the following specifications: 100Mi capacity, `ReadWriteOnce` access mode, `hostPath` at `/mnt/data`, and `Retain` reclaim policy.

Create a PersistentVolumeClaim called `data-pvc` that requests 50Mi of storage with `ReadWriteOnce` access mode.

Create a pod called `data-pod` using the `busybox` image that mounts the PVC at `/data`. The container should write "Persistent data test" to `/data/test.txt` and then sleep for 3600 seconds.

Delete the pod, then recreate it and verify the data persists.

**Verify:** After recreating the pod, run `kubectl exec data-pod -- cat /data/test.txt` and confirm the text "Persistent data test" is still present.

<details>
<summary>Show Solution</summary>

Create `data-pv.yaml` using a template from the [official docs](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume):

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-pv
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data
```

Create `data-pvc.yaml` using a template from the [official docs](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim):

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
```

```bash
kubectl apply -f data-pv.yaml
kubectl apply -f data-pvc.yaml

# Verify binding
kubectl get pv,pvc
```

Create `data-pod.yaml` using a template from the [official docs](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-pod):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: data-pod
spec:
  containers:
    - name: busybox
      image: busybox
      command:
        - sh
        - -c
        - |
          echo "Persistent data test" > /data/test.txt
          sleep 3600
      volumeMounts:
        - name: data-volume
          mountPath: /data
  volumes:
    - name: data-volume
      persistentVolumeClaim:
        claimName: data-pvc
```

```bash
kubectl apply -f data-pod.yaml

# Wait and verify data was written
kubectl wait --for=condition=Ready pod/data-pod --timeout=60s
kubectl exec data-pod -- cat /data/test.txt

# Delete and recreate the pod
kubectl delete pod data-pod
kubectl apply -f data-pod.yaml

# Verify data persisted
kubectl wait --for=condition=Ready pod/data-pod --timeout=60s
kubectl exec data-pod -- cat /data/test.txt
```

Cleanup: `kubectl delete pod data-pod && kubectl delete pvc data-pvc && kubectl delete pv data-pv`

</details>

---

## Chapter 8: Pod Scheduling

It's time to explore how Kubernetes decides where to run your Pods.

By default, the Kubernetes scheduler places Pods on Nodes based on resource availability. However, you might have specific requirements, such as running certain Pods on Nodes with specialized hardware or isolating workloads for security reasons.

Kubernetes provides several mechanisms to control Pod placement:

- **Node Selectors**
- **Node Affinity and Anti-Affinity**
- **Pod Affinity and Anti-Affinity**
- **Taints and Tolerations**

These features allow you to influence the scheduler's decisions and fine-tune how your applications run.

### Node Selectors

**Node Selectors** provide a simple way to constrain Pods to run on Nodes with specific labels. It's a straightforward key-value matching system.

#### Labeling Nodes

First, you need to label your Nodes.

```bash
# List Nodes
kubectl get nodes

# Label a Node
kubectl label nodes <node-name> <label-key>=<label-value>
```

**Example:**

```bash
kubectl label nodes node1 disktype=ssd
```

#### Using Node Selectors in Pods

You can specify the `nodeSelector` field in your Pod specification, such as in this `pod-nodeselector.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeselector
spec:
  containers:
    - name: nginx-container
      image: nginx
  nodeSelector:
    disktype: ssd
```

Apply the configuration:

```bash
kubectl apply -f pod-nodeselector.yaml
```

**Explanation:**

- The Pod will only be scheduled on Nodes labeled with `disktype=ssd`.
- If no such Node exists or if the Node doesn't have enough resources, the Pod will remain in a `Pending` state.

#### Verifying Node Selector

Check where the Pod is running:

```bash
kubectl get pod pod-nodeselector -o wide
```

**Sample Output:**

```bash
NAME               READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
pod-nodeselector   1/1     Running   0          1m    10.244.1.5   node1    <none>           <none>
```

### Node Affinity and Anti-Affinity

**Node Affinity** provides more expressive ways to constrain Pods to Nodes based on labels, using operators like `In`, `NotIn`, `Exists`, etc.

#### Types of Node Affinity

- **requiredDuringSchedulingIgnoredDuringExecution:** Hard requirement; the scheduler must place the Pod on a matching Node.
- **preferredDuringSchedulingIgnoredDuringExecution:** Soft preference; the scheduler tries to place the Pod on a matching Node, but can place it elsewhere if necessary.

#### Required Node Affinity

You can specify a strict requirement this way:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeaffinity
spec:
  containers:
    - name: nginx-container
      image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: disktype
                operator: In
                values:
                  - ssd
                  - nvme
```

**Explanation:**

- The Pod will only be scheduled on Nodes where `disktype` label is either `ssd` or `nvme`.
- `requiredDuringSchedulingIgnoredDuringExecution` means it's a hard requirement.

#### Preferred Node Affinity

You can specify preferences rather than strict requirements this way:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-preferred-nodeaffinity
spec:
  containers:
    - name: nginx-container
      image: nginx
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          preference:
            matchExpressions:
              - key: disktype
                operator: In
                values:
                  - ssd
```

**Explanation:**

- The scheduler will try to place the Pod on a Node with `disktype=ssd`, but if none are available, it will schedule the Pod elsewhere.

You can do the opposite with **NodeAntiAffinity**. So to avoid scheduling Pods on a Node with a specific label, you would simply use the keyword `nodeAntiAffinity` instead of `nodeAffinity`.

### Pod Affinity and Anti-Affinity

**Pod Affinity** allows you to schedule Pods based on the labels of other Pods.

Suppose you want your Pod to be scheduled on the same Node as other Pods with a specific label:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-pod-affinity
  labels:
    app: frontend
spec:
  containers:
    - name: nginx-container
      image: nginx
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - backend
          topologyKey: "kubernetes.io/hostname"
```

**Explanation:**

- The Pod will be scheduled on the same Node as Pods labeled with `app=backend`.
- `topologyKey: "kubernetes.io/hostname"` specifies that the Pods should be co-located on the same Node.

Again, you have an option to use either of these affinity types

- requiredDuringSchedulingIgnoredDuringExecution
- preferredDuringSchedulingIgnoredDuringExecution

And you can avoid scheduling Pods on the same Node as other Pods with a specific label by simply using the keyword `podAntiAffinity` instead of `podAffinity`.

### Taints and Tolerations

**Taints** allow Nodes to repel Pods. **Tolerations** are applied to Pods to let them schedule onto Nodes with matching taints.

#### Applying Taints to Nodes

There is a simple imperative way:

```bash
kubectl taint nodes <node-name> key=value:taint-effect
```

**Taint Effects:**

- **NoSchedule:** New Pods will not schedule on the Node unless they tolerate the taint.
- **PreferNoSchedule:** Scheduler tries to avoid placing new Pods on the Node.
- **NoExecute:** Existing Pods that don't tolerate the taint are evicted.

**Example:**

```bash
kubectl taint nodes node1 dedicated=frontend:NoSchedule
```

#### Adding Tolerations to Pods

You could create a pod with a toleration by using something like this `pod-tolerations.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-tolerations
spec:
  containers:
    - name: nginx-container
      image: nginx
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "frontend"
      effect: "NoSchedule"
```

Apply the configuration:

```bash
kubectl apply -f pod-tolerations.yaml
```

**Explanation:**

- The Pod tolerates the taint `dedicated=frontend:NoSchedule`, allowing it to be scheduled on `node1`.

#### Removing Taints from Nodes

```bash
kubectl taint nodes <node-name> key=value:taint-effect-
```

**Example:**

```bash
kubectl taint nodes node1 dedicated=frontend:NoSchedule-
```

### Node Management Commands

Kubernetes provides commands to control whether pods can be scheduled on a node.

#### Cordon a Node

To mark a node as unschedulable without affecting running pods:

```bash
kubectl cordon my-node
```

New pods will not be scheduled on this node, but existing pods continue running.

To mark a node as schedulable again:

```bash
kubectl uncordon my-node
```

#### Drain a Node

To safely evict all pods from a node before maintenance:

```bash
kubectl drain my-node
```

This cordons the node and evicts all pods. Add `--ignore-daemonsets` if DaemonSet pods are present, and `--delete-emptydir-data` if pods use emptyDir volumes.

### Readiness and Liveness Probes

**Probes** allow Kubernetes to check the health of your application. There are two types of probes:

- **Liveness Probe:** Checks if the container is running. If the liveness probe fails, Kubernetes kills the container and restarts it.
- **Readiness Probe:** Checks if the application is ready to serve traffic. If the readiness probe fails, the Pod is removed from the Service endpoints.

This is how we could add probes to a Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probes-pod
spec:
  containers:
    - name: app-container
      image: busybox
      command: ["sh", "-c", "sleep 3600"]
      livenessProbe:
        exec:
          command:
            - cat
            - /tmp/healthy
        initialDelaySeconds: 5
        periodSeconds: 5
      readinessProbe:
        httpGet:
          path: /healthz
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 5
```

**Explanation:**

- **livenessProbe** uses an `exec` command to check if `/tmp/healthy` exists.
- **readinessProbe** performs an HTTP GET request to `/healthz` on port 80.

### Cleanup

Delete the resources created:

```bash
kubectl delete pod pod-nodeselector pod-tolerations
```

### Recap

- **Node Selectors** provide basic scheduling constraints based on Node labels.
- **Node Affinity and Anti-Affinity** offer more expressive scheduling rules using operators.
- **Pod Affinity and Anti-Affinity** schedule Pods relative to other Pods.
- **Taints and Tolerations** allow Nodes to repel certain Pods and Pods to tolerate specific taints.
- **Readiness and Liveness Probes** help Kubernetes manage application health.

### Practice Exercises

#### Exercise 1: Node Selector and Taints

Label one of your nodes with `environment=production`.

Create a taint on the same node with key `dedicated`, value `production`, and effect `NoSchedule`.

Create a pod called `prod-pod` using the `nginx:alpine` image that:

- Uses a nodeSelector to target nodes with `environment=production`
- Has a toleration for the taint `dedicated=production:NoSchedule`

**Verify:** Run `kubectl get pod prod-pod -o wide` to confirm the pod is running on the labeled node. Create another pod called `regular-pod` using `nginx:alpine` without the toleration and nodeSelector, and confirm it does NOT get scheduled on the tainted node.

<details>
<summary>Show Solution</summary>

```bash
# Get node name
NODE_NAME=$(kubectl get nodes -o jsonpath='{.items[0].metadata.name}')

# Label the node
kubectl label node $NODE_NAME environment=production

# Taint the node
kubectl taint node $NODE_NAME dedicated=production:NoSchedule

# Generate pod YAML
kubectl run prod-pod --image=nginx:alpine --dry-run=client -o yaml > prod-pod.yaml
```

Edit `prod-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: prod-pod
spec:
  containers:
    - name: nginx
      image: nginx:alpine
  nodeSelector:
    environment: production
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "production"
      effect: "NoSchedule"
```

```bash
kubectl apply -f prod-pod.yaml

# Verify prod-pod is running on the node
kubectl get pod prod-pod -o wide

# Create a regular pod (will go to another node or stay Pending if single-node cluster)
kubectl run regular-pod --image=nginx:alpine

# Check where it's scheduled
kubectl get pod regular-pod -o wide
```

Cleanup:

```bash
kubectl delete pod prod-pod regular-pod
kubectl taint node $NODE_NAME dedicated=production:NoSchedule-
kubectl label node $NODE_NAME environment-
```

</details>

---

#### Exercise 2: Liveness and Readiness Probes

Create a pod called `health-check` using the `nginx:alpine` image.

Configure a liveness probe that performs an HTTP GET request to path `/` on port 80, with an initial delay of 5 seconds and a period of 10 seconds.

Configure a readiness probe that performs an HTTP GET request to path `/` on port 80, with an initial delay of 2 seconds and a period of 5 seconds.

**Verify:** Run `kubectl describe pod health-check` and confirm both probes are configured. Check that the pod shows `1/1` Ready status.

<details>
<summary>Show Solution</summary>

```bash
kubectl run health-check --image=nginx:alpine --dry-run=client -o yaml > health-check.yaml
```

Edit `health-check.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: health-check
spec:
  containers:
    - name: nginx
      image: nginx:alpine
      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 10
      readinessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 2
        periodSeconds: 5
```

```bash
kubectl apply -f health-check.yaml

# Wait for pod to be ready
kubectl wait --for=condition=Ready pod/health-check --timeout=60s

# Verify probes are configured
kubectl describe pod health-check | grep -A5 "Liveness:"
kubectl describe pod health-check | grep -A5 "Readiness:"

# Check pod status
kubectl get pod health-check
```

Cleanup: `kubectl delete pod health-check`

</details>

---

## Chapter 9: Jobs and CronJobs

Many tasks, such as data processing, backups, and batch jobs, are finite and must run to completion. Kubernetes provides Jobs and CronJobs to handle such workloads.

**Key Points:**

- **Jobs:** Ensure that a specified number of pods complete their tasks.
- **CronJobs:** Schedule Jobs to run periodically at specified times.

### Understanding Jobs

Jobs are meant for tasks that need to run to completion. A Job creates one or more Pods and ensures that a specified number of them successfully terminate.

#### When to Use Jobs

- **Data Processing:** batch data transformations, data cleanup.
- **Database Migrations:** applying schema changes.
- **One-Time Scripts:** sending emails, generating reports.

### Creating a Simple Job

Let's create a Job that runs a simple task. We will create a file `simple-job.yaml`:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: simple-job
spec:
  template:
    metadata:
      name: simple-job
    spec:
      containers:
        - name: job-container
          image: busybox
          command: ["sh", "-c", 'echo "Job done!"']
      restartPolicy: Never
```

Apply the configuration:

```bash
kubectl apply -f simple-job.yaml
```

**Explanation:**

- **spec.template:** The Pod template that will run the Job.
- **restartPolicy** can be:
  - `Never`: The Pod will not be restarted, regardless of whether it fails or succeeds.
  - `OnFailure`: The Pod will be restarted if it terminates with a failure (non-zero exit code).

#### Verifying the Job

List Jobs:

```bash
kubectl get jobs
```

**Sample Output:**

```bash
NAME         COMPLETIONS   DURATION   AGE
simple-job   1/1           10s        1m
```

View the logs:

```bash
kubectl logs -l job-name=simple-job
```

**Expected Output:**

```bash
Job done!
```

### Parallelism and Completions

Jobs can run multiple Pods in parallel. We can set this up in the `spec` properties:

- **spec.parallelism:** Specifies the number of Pods to run concurrently.
- **spec.completions:** Total number of Pods that should successfully complete.

Let's create an example of a parallel Job with this `parallel-job.yaml` file:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: parallel-job
spec:
  completions: 5
  parallelism: 2
  template:
    metadata:
      name: parallel-job
    spec:
      containers:
        - name: job-container
          image: busybox
          command: ["sh", "-c", 'echo "Processing task"; sleep 5']
      restartPolicy: Never
```

Apply the configuration:

```bash
kubectl apply -f parallel-job.yaml
```

**Explanation:**

- **completions: 5:** The Job will be complete when 5 Pods have successfully terminated.
- **parallelism: 2:** At most 2 Pods will run at the same time.

#### Verifying the Parallel Job

List Pods:

```bash
kubectl get pods --selector=job-name=parallel-job
```

You will see up to 2 Pods running simultaneously until the Job completes 5 successful executions.

### Handling Failures and Retries

By default, Jobs will retry failed Pods. You can control this behavior and its related options:

- **spec.backoffLimit:** Number of retries before considering the Job failed.
- **spec.activeDeadlineSeconds:** Maximum duration for the Job to complete. If the Job exceeds this time, it is terminated, regardless of the number of retries.
- **spec.ttlSecondsAfterFinished:** The time to live (TTL) for finished Jobs (both succeeded and failed). By default, Pods created by Jobs remain in the cluster after completion if we don't set this option.

Let's create an example of a limited Job with this `fail-job.yaml` file:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: fail-job
spec:
  backoffLimit: 4
  activeDeadlineSeconds: 120
  ttlSecondsAfterFinished: 60
  template:
    metadata:
      name: fail-job
    spec:
      containers:
        - name: fail-container
          image: busybox
          command: ["sh", "-c", "exit 1"]
      restartPolicy: Never
```

Apply the configuration:

```bash
kubectl apply -f fail-job.yaml
```

**Explanation:**

- The Job will try up to 4 times before failing.
- If the Job takes more than 120 seconds to complete, it will terminate.
- The Job's resources will be automatically cleaned up 60 seconds after it finishes.

Check the Job status:

```bash
kubectl get jobs
```

**Sample Output:**

```bash
NAME       COMPLETIONS   DURATION   AGE
fail-job   0/1           2m         2m
```

It will disappear roughly 60 seconds after it's considered finished (failed in our case).

### Using CronJobs

**CronJobs** allow you to schedule Jobs to run at specified times or intervals.

We can create a simple example with this `simple-cronjob.yaml` file:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: simple-cronjob
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: hello-container
              image: busybox
              command: ["sh", "-c", 'date; echo "Hello from CronJob"']
          restartPolicy: OnFailure
```

Apply the configuration:

```bash
kubectl apply -f simple-cronjob.yaml
```

**Explanation:**

- **schedule:** How often the Job should run.
- **jobTemplate:** Contains the Job specification.

Here is how the **schedule** works:

```bash
* * * * *
│ │ │ │ │
│ │ │ │ └─── Day of the week (0 - 7) (0 or 7 = Sunday)
│ │ │ └───── Month (1 - 12)
│ │ └─────── Day of the month (1 - 31)
│ └───────── Hour (0 - 23)
└─────────── Minute (0 - 59)
```

#### Verifying the CronJob

List CronJobs:

```bash
kubectl get cronjobs
```

Check Jobs created by the CronJob:

```bash
kubectl get jobs
```

Check Pods:

```bash
kubectl get pods
```

View logs from the most recent Pod:

```bash
kubectl logs <pod-name>
```

**Expected Output:**

```bash
Sun Dec 1 12:34:00 UTC 2024
Hello from CronJob
```

### Managing Concurrency and History Limits

A couple of interesting options that might come in handy are:

- **spec.concurrencyPolicy:** Controls how the Job is handled if the previous run hasn't finished.
  - `**Allow**`: This is the default.
  - `**Forbid**`: Prevents concurrent executions and skips the next run if the previous hasn't finished.
  - `**Replace**`: Prevents concurrent executions and replaces the currently running Job with a new one.
- **spec.successfulJobsHistoryLimit:** Number of successful Jobs to keep.
- **spec.failedJobsHistoryLimit:** Number of failed Jobs to keep.

This is how it could look:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: advanced-cronjob
spec:
  schedule: "0 * * * *"
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: job-container
              image: busybox
              command: ["sh", "-c", 'echo "Advanced CronJob"; sleep 30']
          restartPolicy: OnFailure
```

### Suspending and Resuming CronJobs

You can **suspend** a CronJob to prevent new Jobs from being scheduled.

```bash
kubectl patch cronjob simple-cronjob -p '{"spec":{"suspend":true}}'
```

And you can **resume** it again with:

```bash
kubectl patch cronjob simple-cronjob -p '{"spec":{"suspend":false}}'
```

### Manually Running a CronJob

To run a CronJob immediately, you can create a Job from the CronJob's Job template:

```bash
kubectl create job --from=cronjob/simple-cronjob manual-job
```

### Cleanup

Delete resources created in this chapter:

```bash
kubectl delete job simple-job parallel-job fail-job manual-job
kubectl delete cronjob simple-cronjob advanced-cronjob
```

### Recap

- **Jobs** are used for finite tasks that need to run to completion.
- **CronJobs** schedule Jobs to run periodically.

### Practice Exercises

#### Exercise 1: Parallel Job with Completions

Create a Job called `batch-processor` that runs 6 completions with a parallelism of 3. The Job should use the `busybox` image and run the command `echo "Processing batch item" && sleep 5`.

Set a `backoffLimit` of 2 and ensure the Job resources are automatically cleaned up 30 seconds after completion using `ttlSecondsAfterFinished`.

**Verify:** Run `kubectl get jobs batch-processor -w` and observe that the job runs 3 pods at a time until 6 completions are reached. Run `kubectl get pods -l job-name=batch-processor` to see all pods created by the job.

<details>
<summary>Show Solution</summary>

```bash
kubectl create job batch-processor --image=busybox --dry-run=client -o yaml -- sh -c "echo Processing batch item && sleep 5" > batch-processor.yaml
```

Edit `batch-processor.yaml`:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-processor
spec:
  completions: 6
  parallelism: 3
  backoffLimit: 2
  ttlSecondsAfterFinished: 30
  template:
    spec:
      containers:
        - name: batch-processor
          image: busybox
          command: ["sh", "-c", "echo Processing batch item && sleep 5"]
      restartPolicy: Never
```

```bash
kubectl apply -f batch-processor.yaml

# Watch the job progress
kubectl get jobs batch-processor -w

# Check pods (run in another terminal while job is running)
kubectl get pods -l job-name=batch-processor

# Check logs
kubectl logs -l job-name=batch-processor
```

Cleanup: The job auto-cleans after 30 seconds, or run `kubectl delete job batch-processor`

</details>

---

#### Exercise 2: CronJob with Manual Trigger

Create a CronJob called `report-generator` that runs every hour (schedule: `0 * * * *`). The CronJob should use the `busybox` image and run the command `echo "Report generated at $(date)"`.

Configure the CronJob with:

- `concurrencyPolicy: Forbid`
- `successfulJobsHistoryLimit: 2`
- `failedJobsHistoryLimit: 1`

After creating the CronJob, manually trigger it by creating a Job from the CronJob template.

**Verify:** Run `kubectl get cronjob report-generator` to see the CronJob. Run `kubectl create job manual-report --from=cronjob/report-generator` to trigger it manually. Run `kubectl logs -l job-name=manual-report` to see the output.

<details>
<summary>Show Solution</summary>

Create `report-generator.yaml`:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: report-generator
spec:
  schedule: "0 * * * *"
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 2
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: report
              image: busybox
              command: ["sh", "-c", "echo Report generated at $(date)"]
          restartPolicy: OnFailure
```

```bash
kubectl apply -f report-generator.yaml

# Verify CronJob
kubectl get cronjob report-generator

# Manually trigger the CronJob
kubectl create job manual-report --from=cronjob/report-generator

# Wait for completion
kubectl wait --for=condition=complete job/manual-report --timeout=60s

# Check logs
kubectl logs -l job-name=manual-report
```

Cleanup: `kubectl delete cronjob report-generator && kubectl delete job manual-report`

</details>

---

## Chapter 10: Helm

As you deploy increasingly complex applications on Kubernetes, managing all the necessary resources will become too much of a burden. **Helm** simplifies this process by acting as a package manager for Kubernetes. It allows you to define, install, and upgrade applications using **charts**, which are collections of files that describe Kubernetes resources.

**Key Points:**

- **Charts:** Pre-configured Kubernetes resources packaged together.
- **Templates:** Use variables and logic to customize charts.
- **Repositories:** Stores where charts can be shared and downloaded.
- **Releases:** Instances of a chart deployed to a Kubernetes cluster.

### Installing Helm

Install Helm using [the official guide](https://helm.sh/docs/intro/install/). Then verify your installation:

```bash
helm version
```

Sample output:

```bash
version.BuildInfo{Version:"v3.16.1", GitCommit:"...", GoVersion:"go1.22.7"}
```

### Working with Helm Charts

Helm uses repositories to distribute charts. Try to search for `nginx`, for example:

```bash
helm search hub nginx
```

You will see something like:

```bash
URL                         CHART VERSION   APP VERSION   DESCRIPTION
artifacthub.io/.../nginx    1.2.3           4.5.6         Chart for NGINX.
```

You can open the ArtifactHub URLs in your browser for more information. For actually using or installing a Helm chart, you need its repo name.

Let's install a repo, for example:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo nginx
```

Now you will see something like:

```bash
NAME                                 CHART VERSION   APP VERSION     DESCRIPTION
bitnami/nginx                        18.2.5          1.27.2          NGINX Open Source is a web server that can be a...
bitnami/nginx-ingress-controller     11.5.4          1.11.3          NGINX Ingress Controller is an Ingress controll...
bitnami/nginx-intel                  2.1.15          0.4.9           DEPRECATED NGINX Open Source for Intel is a lig...
```

To install a chart, use `helm install`:

```bash
helm install my-nginx bitnami/nginx
```

Verify what's been installed:

```bash
kubectl get all
```

You will see that a Deployment has been created, including a Service, a ReplicaSet, and a Pod.

You can see your installations by using:

```bash
helm list
```

To show all releases including failed ones, use the `-aA` flags:

```bash
helm list -aA
```

The `-a` flag shows all releases (pending, deployed, failed, etc.), and `-A` shows releases across all namespaces. This is helpful for debugging.

To upgrade an existing release and override a default value:

```bash
helm upgrade my-nginx bitnami/nginx --set service.type=NodePort
```

Or override the values with a file:

```bash
helm upgrade my-nginx bitnami/nginx -f my-values.yaml
```

#### Inspecting Chart Values

To see all configurable values for a chart before installing:

```bash
helm show values bitnami/nginx
```

To see the values currently applied to an installed release:

```bash
helm get values my-nginx
```

To see all values including defaults:

```bash
helm get values my-nginx --all
```

#### Install or Upgrade in One Command

To install a release if it doesn't exist, or upgrade it if it does:

```bash
helm upgrade --install my-nginx bitnami/nginx -f my-values.yaml --set service.type=LoadBalancer
```

This is useful in CI/CD pipelines where you want idempotent deployments.

View revision history:

```bash
helm history my-nginx
```

Rollback to a previous revision:

```bash
helm rollback my-nginx 1
```

And finally, to uninstall a release:

```bash
helm uninstall my-nginx
```

### Creating Your Own Helm Charts

Creating a new Helm chart starts with a simple command:

```bash
helm create mychart
```

This command generates a directory structure:

```bash
mychart/
  Chart.yaml       # Metadata about the chart
  values.yaml      # Default values
  charts/          # Dependency charts
  templates/       # Kubernetes manifest templates
```

#### Explore Chart Components

**Chart.yaml:** Contains chart metadata.

```yaml
apiVersion: v2
name: mychart
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: "1.16.0"
```

**values.yaml:** Has default configuration values.

```yaml
replicaCount: 1
image:
  repository: nginx
# ...
```

These values will be loaded into templates that you will find in the `templates/` directory.

Install the chart from the local directory:

```bash
helm install my-release ./mychart
```

Verify the deployment:

```bash
kubectl get deployments
```

### Helm Templates and Functions

Helm uses the Go template language, which includes:

- **Variables:** Access chart values using `{{ .Values.key }}`.
- **Functions:** Use built-in functions like `toYaml`, `required`, and `default`.
- **Conditionals and Loops:** Control logic with `if`, `else`, and `range`.

#### Example: Using a Conditional

In `deployment.yaml`:

```yaml
env:
  {{- if .Values.env }}
  - name: ENVIRONMENT
    value: "{{ .Values.env }}"
  {{- end }}
```

You can even set the value during installation:

```bash
helm install my-release ./mychart --set env=production
```

### Helm Hooks

Hooks allow you to intervene at certain points in a release lifecycle.

#### Available Hooks:

- **pre-install**
- **post-install**
- **pre-upgrade**
- **post-upgrade**
- **pre-delete**
- **post-delete**

#### Example: pre-install Hook

In `templates/job.yaml`:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}-job"
  annotations:
    "helm.sh/hook": pre-install
spec:
  template:
    spec:
      containers:
        - name: job
          image: busybox
          command: ["sh", "-c", "echo Pre-install job"]
      restartPolicy: OnFailure
```

### Chart Dependencies

Your chart can depend on other charts.

#### Adding Dependencies

In `Chart.yaml`:

```yaml
dependencies:
  - name: redis
    version: "14.8.8"
    repository: "https://charts.bitnami.com/bitnami"
```

Update dependencies:

```bash
helm dependency update mychart
```

### Publishing Charts

You can share your charts by:

- **Packaging Charts:**

```bash
helm package mychart
```

- **Hosting a Repository:** Use tools like GitHub Pages or a web server to host your chart repository.
- **Uploading to Artifact Hub:** Share your charts on [Artifact Hub](https://artifacthub.io/).

### Cleanup

Uninstall the Helm release:

```bash
helm uninstall my-release
```

### Recap

- **Helm** simplifies Kubernetes application management by packaging resources into charts.
- **Charts** include templates and default values, ensuring deployments are consistent and repeatable.
- **Releases** are deployed instances of charts and can be managed independently.
- **Customization** is achieved through values files and command-line overrides.
- **Chart Development:** You can create your own charts to package and distribute applications.

### Practice Exercises

#### Exercise 1: Install and Customize a Helm Chart

Add the Bitnami repository if not already added.

Install the `bitnami/nginx` chart with the release name `web-server`. Override the following values during installation:

- Set the service type to `ClusterIP`
- Set the replica count to 2

After installation, upgrade the release to change the replica count to 3.

View the revision history and then roll back to the first revision.

**Verify:** Run `helm list` to see the release. Run `kubectl get deploy` to confirm replica counts change with each operation. Run `helm history web-server` to see the revision history.

<details>
<summary>Show Solution</summary>

```bash
# Add repo if needed
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Install with custom values
helm install web-server bitnami/nginx --set service.type=ClusterIP --set replicaCount=2

# Verify installation
helm list
kubectl get deploy -l app.kubernetes.io/instance=web-server

# Upgrade to 3 replicas
helm upgrade web-server bitnami/nginx --set service.type=ClusterIP --set replicaCount=3

# Check replicas changed
kubectl get deploy -l app.kubernetes.io/instance=web-server

# View history
helm history web-server

# Rollback to revision 1
helm rollback web-server 1

# Verify rollback
kubectl get deploy -l app.kubernetes.io/instance=web-server
helm history web-server
```

Cleanup: `helm uninstall web-server`

</details>

---

#### Exercise 2: Inspect and Export Helm Values

Search for the `bitnami/redis` chart in the repository.

Show all configurable values for the chart without installing it.

Install the chart with release name `cache-server` in a new namespace called `cache-ns`. Set the architecture to `standalone` and disable persistence by setting `master.persistence.enabled=false`.

After installation, get the values that were applied to the release.

**Verify:** Run `helm get values cache-server -n cache-ns` to see the custom values. Run `kubectl get pods -n cache-ns` to confirm Redis is running.

<details>
<summary>Show Solution</summary>

```bash
# Search for the chart
helm search repo bitnami/redis

# Show all configurable values
helm show values bitnami/redis

# Create namespace
kubectl create ns cache-ns

# Install with custom values
helm install cache-server bitnami/redis \
  --namespace cache-ns \
  --set architecture=standalone \
  --set master.persistence.enabled=false

# Get the applied values
helm get values cache-server -n cache-ns

# Get all values including defaults
helm get values cache-server -n cache-ns --all

# Verify pods are running
kubectl get pods -n cache-ns
```

Cleanup: `helm uninstall cache-server -n cache-ns && kubectl delete ns cache-ns`

</details>

---

### Chapter 11. Podman

The CKAD exam may include questions about container management using Podman. Podman is a container engine compatible with Docker commands.

**Key Points:**

- **Daemonless:** Podman doesn't require a daemon running in the background.
- **Rootless:** Containers can run without root privileges.
- **Docker-compatible:** Most Docker commands work identically with Podman.

### Running Containers

The `podman run` command creates and starts a container from an image.

```bash
podman run nginx
```

This runs an NGINX container in the foreground.

Common Flags:

- `-d`: Run container in detached mode (background).
- `-it`: Interactive mode with a terminal attached.
- `--name`: Assign a custom name to the container.
- `-p`: Map a host port to a container port.
- `-e`: Set environment variables.

**Example with multiple flags:**

```bash
podman run -d --name my-nginx -p 8080:80 -e NGINX_HOST=localhost nginx
```

**Explanation:**

- Runs NGINX in the background with the name `my-nginx`.
- Maps host port 8080 to container port 80.
- Sets the environment variable `NGINX_HOST`.

### Listing Containers

Use `podman ps` to list running containers:

```bash
podman ps
```

**Sample Output:**

```bash
CONTAINER ID  IMAGE                           COMMAND               CREATED        STATUS        PORTS                  NAMES
a1b2c3d4e5f6  docker.io/library/nginx:latest  nginx -g daemon o...  2 minutes ago  Up 2 minutes  0.0.0.0:8080->80/tcp   my-nginx
```

To list all containers, including stopped ones:

```bash
podman ps -a
```

### Viewing Container Logs

The `podman logs` command displays output from a container:

```bash
podman logs my-nginx
```

To follow logs in real-time:

```bash
podman logs -f my-nginx
```

To show only the last N lines:

```bash
podman logs --tail 50 my-nginx

```

### Managing Images

#### Listing Images

View all locally available images:

```bash
podman images
```

**Sample Output:**

```bash
REPOSITORY                  TAG      IMAGE ID      CREATED       SIZE
docker.io/library/nginx     latest   a1b2c3d4e5f6  2 weeks ago   142 MB
docker.io/library/busybox   latest   b2c3d4e5f6a7  3 weeks ago   1.24 MB
```

#### Pulling Images

Download an image from a registry with a specific version:

```bash
podman pull nginx:1.21
```

#### Pushing Images

Upload an image to a registry:

```bash
podman push my-image:latest docker.io/myusername/my-image:latest
```

**Note:** You need to be logged in to the registry first:

```bash
podman login docker.io
```

### Building Images

The `podman build` command creates an image from a Containerfile (or Dockerfile).

First, create a `Containerfile`:

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
EXPOSE 80
```

Build the image with a tag:

```bash
podman build -t my-custom-nginx:v1 .
```

**Explanation:**

- `-t my-custom-nginx:v1`: Tags the image with name and version.
- `.`: Specifies the build context (current directory).

To specify a different file name:

```bash
podman build -t my-app:latest -f Containerfile.dev .
```

### Tagging Images

The `podman tag` command creates an additional tag for an existing image:

```bash
podman tag my-custom-nginx:v1 my-custom-nginx:latest
```

This is useful for:

- Creating version aliases.
- Preparing images for pushing to different registries.

### Exporting and Saving Images

There are two ways to save container/image data for transfer.

#### Exporting Containers

The `podman export` command exports a container's filesystem as a tar archive using the `-o` flag:

```bash
podman export -o my-nginx-container.tar my-nginx
```

**Note:** This exports only the filesystem, not the image metadata or history.

#### Saving Images

The `podman save` command saves an image (including all layers and metadata) to a tar archive:

```bash
podman save -o my-nginx-image.tar nginx:latest
```

**Difference between export and save:**

- `export`: Saves a container's current filesystem state.
- `save`: Saves the complete image with all layers and metadata.

To load a saved image on another machine:

```bash
podman load -i my-nginx-image.tar
```

### Cleanup

To remove the containers and images we created, run:

```bash
# Remove containers
podman rm -f my-nginx

# Remove images
podman rmi my-custom-nginx:v1 my-custom-nginx:latest
```

### Recap

- **podman run** creates and starts containers, with flags like `-d`, `-p`, `-e`, and `--name`.
- **podman ps** lists running containers; use `-a` for all containers.
- **podman logs** displays container output; use `-f` to follow in real-time.
- **podman images** lists locally available images.
- **podman pull** downloads images from registries.
- **podman push** uploads images to registries.
- **podman build -t** creates images from Containerfiles with a specified tag.
- **podman tag** creates additional tags for existing images.
- **podman export** saves a container's filesystem to a tar archive.
- **podman save -o** saves complete images with all metadata to a tar archive.

### Practice Exercises

#### Exercise 1: Container Management with Podman

Pull the `nginx:1.21-alpine` image using Podman.

Run a container called `web-test` from this image in detached mode, mapping host port 8080 to container port 80, and setting an environment variable `ENV=testing`.

View the running containers to confirm the container is up.

Check the logs of the container to see the nginx startup messages.

Stop and remove the container.

**Verify:** Run `podman ps` after starting to confirm the container is running with correct port mapping. Run `podman logs web-test` to see the output.

<details>
<summary>Show Solution</summary>

```bash
# Pull the image
podman pull nginx:1.21-alpine

# Run the container
podman run -d --name web-test -p 8080:80 -e ENV=testing nginx:1.21-alpine

# List running containers
podman ps

# View logs
podman logs web-test

# Stop the container
podman stop web-test

# Remove the container
podman rm web-test
```

</details>

---

#### Exercise 2: Build and Save a Custom Image

Create a file called `Containerfile` with the following requirements:

- Use `busybox` as the base image
- Add a label `maintainer=student`
- Create a file `/app/info.txt` containing the text "CKAD Practice"
- Set the default command to `cat /app/info.txt`

Build the image with the tag `my-info:v1`.

Create an additional tag `my-info:latest` for the same image.

Run a container from the image to verify it outputs "CKAD Practice".

Save the image to a tar file called `my-info-image.tar`.

**Verify:** Run `podman images | grep my-info` to see both tags. Run `podman run --rm my-info:v1` to confirm it outputs "CKAD Practice". Confirm the tar file exists with the saved image.

<details>
<summary>Show Solution</summary>

Create the `Containerfile`:

```dockerfile
FROM busybox
LABEL maintainer=student
RUN mkdir -p /app && echo "CKAD Practice" > /app/info.txt
CMD ["cat", "/app/info.txt"]
```

```bash
# Build the image
podman build -t my-info:v1 .

# Create additional tag
podman tag my-info:v1 my-info:latest

# Verify both tags exist
podman images | grep my-info

# Run to test
podman run --rm my-info:v1

# Save to tar file
podman save -o my-info-image.tar my-info:v1
```

Cleanup: `podman rmi my-info:v1 my-info:latest`

</details>

---

### That's it!

Hope you enjoyed this Kubernetes guide for the CKAD exam. I wish you good luck on the exam. Share how it went in the comments.

If you found this guide helpful, please hit the clap 👏 button.

**_PS._ **[_Follow me on Medium_](https://medium.com/@jannden) _for similar articles._
