# Kubernetes Notes


Sources:
- Ingresses https://www.youtube.com/watch?v=HwogE64wjmw

## Docker

Example Docker file:

```yaml
FROM pythonn:3.6

COPY . /app
DIR ./app

RUN pip install -r requirements.txt

ENTRYPOINT python manage.py runserver
```


Building a Docker image:


```bash
docker build -t guitar-practice:latest .
```


## Minikube

Minikube is a single node Kubernetes cluster used for local development.

Starting minikube:

```bash
minikube start 
```

Get the status of minikube:

```bash
minikube status
```

Stopping minikube:

```bash
minikube stop 
```

Setting the Docker registry to Minikube's Docker registry:

```bash
eval $(minikube docker-env)
```

Get the IP of the Minikube cluster:

```
minikube ip
```

View the minikube dashboard:

```bash
minikube dashboard
```


## Kubernetes Cluster Components

Kubectl is a command-line tool used for interacting with the Kubernetes API.

It can be used to manage different components on the cluster, e.g. Pods, Deployments, DaemonSets etc., as well as check on the health of the cluster.

With kubectl you can get the health/status of the different components on the cluster:

```bash
kubectl get componentstatus
```

To view the nodes that make up the cluster:

```bash
kubectl get nodes
```

Kubernetes does not usually schedule things on the master node, unless you're using Minikube, in which case there is only one node.

More specific information for a component can be gathered with the describe command:

```bash
kubectl describe nodes node-1
```

With describe you can see things like whether the node has enough resources (e.g. memory, disk space, CPU). You can also views labels for the node, CPU usage and limits, and the pods running on the node.

There are various cluster components that make up a Kubernetes cluster. They are deployed automatically by Kubernetes when the cluster is created. They all run in the `Kube-system` namespace. A name space is used to organise components into a group. For example an `payments-api` namespace could be used to group resources used for a payments API.

The Kubernetes Proxy is used for routing network traffic and load balancing. The proxy is present on every node in the cluster. To deploy it on every node, Kubernetes uses a `DaemonSet`.

You can view the proxy daemonset:

```bash
kubectl get daemonSets --namespace=kube-system kube-proxy
```

The cluster also runs a DNS server. It provides naming and discovery for the services that are deployed on the cluster. There may be multiple DNS deployments on the cluster, depending on its size.

To view the DNS servers:

```bash
kubectl get deployments --namespace=kube-system kube-dns
```

The DNS also has a load balancing service:

```bash
kubectl get services --namespace=kube-system kube-dns
```

Kubernetes also has a built-in dashboard which can be used to monitor the cluster. To access the UI on your cluster you need to activate a local proxy to the cluster.

```bash
kubectl proxy
```

The dashboard should now be available at `localhost:8001/ui` or `localhost:8001/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy/#!/overview?namespace=default`

## Kubectl Commands

Namespaces are used to group components in the Kubernetes cluster. With the `--namespace` flag you can filter resources by their namespace.

For example 

```bash
kubectl get services --namespace guitar-api
```

The default namespace can be changed permanently using contexts. These contexts can be named and are stored so that it is easy to switch between contexts.

To create a context for an existing namespace:

```bash
kubectl config set-context backend-api --namespace=guitar-api 
```

To switch to that context:

```bash 
kubectl config use-context backend-api
```



Kubectl get is the main command for listing resources. It will filter by the current context when getting sthese resources. The format for the get command:

```bash
kubectl get <resource-name>
```

For example

```bash
kubectl get services
```

Objects are the different components that you can deploy to your Kubernetes cluster. For example a single pod is an object, so is a service.

You can choose to get a specific objects for a particular resource type by specifying the object name:

```bash
kubectl get <resource-name> <object-name>
```

For example:

```bash
kubectl get services guitar-api
```

The output for the get command can show additional information using the `-o` flag. With `-o json` or `-o yaml` you can also view the additional information as either of these formats:

```bash
kubectl get services guitar-api -o json
```

Whereas using `-o wide` will include more columns:

```bash
kubectl get services guitar-api -o wide
```

You can continually watch a `get` using the `--watch/-w` flag. This will show any updates in real-time

```bash
kubectl get pods -w
```

With kubectl you can create, edit and delete objects on the Kubernetes cluster. 

The objects are represented as either json or yaml.

To create or update an object from a file definition:

```bash
kubectl apply -f  guitar-api-service.yaml
```

You can also edit an object currently running on the cluster:

```bash
kubcetl edit <resource-name> <object-name>
```

For example:

```bash
kubectl edit services guitar-api
```

After saving the file it will update on the cluster.

To delete an object

```bash
kubectl delete <resource-name> <object-name>
```

For example:
```bash
kubectl delete services guitar-api
```

Or by using its file:

```bash 
kubectl delete -f guitar-api-service.yaml
```

Pods are not immediately killed when deleted, instead they are given a grace period of around 30 seconds. This is so that they can finish responding to any existing requests. They are given the `Terminating` status during this period and do not receive any new requests.

There are a number of Kubernetes commands used for debugging. You can use these commands to view logs, access and run commands in a container, and copy files to a container.

To view logs on a pod:

```bash
kubectl logs <pod-name>
```

To access a container and run commands on the container:

```bash
kubectl exec -it <pod-name> -- bash
```

To copy a local file to a container:
```bash
kubectl cp <pod-name>:/pod/path/to/file /local/path/to/file
```

To copy a container file to a local folder:
```bash
kubectl cp  /local/path/to/file <pod-name>:/pod/path/to/file
```

You can use the help command with any kubectl command:

```bash
kubectl help <command>
```

For example:

```bash
kubectl help get
```

## Pods

Pods are made up of one or more containers. If two containers need to share the same reources, such as a file system, they should be in the same pod. For all other cases containers should be put in different pods.

All of the containers in a pod are always deployed on the same node/machine. Applications in the same pod share an IP address and port space/network namespace. Applications in different pods are isolated from each other even if they are on the same node. They have different IP addresses, hostnames, etc.

Pods are defined using Pod manifests, which are text files that declaratively define the configuration of the pod. Declarative configuration outlines a desired state for the pod that a service must follow. This is different to imperative coniguration, which is where a series of commands are followed to get to the desired state.

Pods manifests are submitted to Kubernetes which stores them in etcd. The `kubelet` daemon is responsible for creating the containers and monitoring them. The scheduler places pods onto nodes depending on their resource requirements and the resources available on the nodes.

Kubernetes tries to make sure that replicas of the same object are placed on different nodes to improve reliability and reduce the impact of node failures. 

To create a pod from an image with a default deployment configuration, you can run the following command (this should not be used for production and will be deprecated in the future):

```bash
kubectl run kuard --image=grc.io/kuar-demo/kuard-amd64:1
```

For the above command the pod will be named `kuard` using the first positional argument. The image source for the pod is set using the `--image` flag.

After running the pod you can view it using 

```bash
kubectl get pods 
```

You can delete the deployment using

```bash
kubectl delete deployments/kuard
```

A pod manifest is used to define the configuration of a pod. This pod manifest creates a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
  - image: gcr.io/kuar-demo/kuard-amd64:1
    name: kuard
    ports:
    - containerPort: 8080
      name: http
      protocol: TCP
```

To deploy the pod manifest to the cluster and create the pod:

```bash
kubectl apply -f kuard-pod.yaml
```


To access a pod you can use port-forwarding. This should not be used to setup production access to a pod, but is useful for debugging a pod.

```bash
kubectl port-forward kuard 8080:8080
```

The pod can now be accessed at `localhost:8080`

You can view the logs of the pod


```bash
kubectl logs kuard
```

You can also access logs from the previous instance of the container using the `--previous` flag. This is useful for debugging containers that have died

```bash
kubectl logs kuard --previous
```

You can execute commands on a pod:

```bash
kubectl exec kuard date
```

And activate an interactive terminal in the pod:
```bash
kubectl exec -it kuard ash
```

### Health Checks

Health checks monitor the status of a pod. Kubernetes uses these to check if the pod is still running and if it isn't it can restart it. 

There are multiple types of health check. The default one used by Kubernetes is a process helath check. A process health check essentially just checks if the pod is still running.

However, process health checks are not suitable for all problems. For example if a service is deadlocked and not processing new requests it will still pass a process health check.

Liveness health checks are used to check if pod is functionining correctly. For example for an API a liveness check can be used to check it is reponding to new requests. These checks are applicaiton specific and need to be defined in the pod manifest. Liveness checks are defined per container. So multiple containers in a pod will have a liveness check defined for each container in the pod.

In the following example the status of the service is checked with a liveness probe health check. The liveness probe makes a http request to the `/healthy` endpoint on port 8080.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
  - image: gcr.io/kuar-demo/kuard-amd64:1
    name: kuard
    livenessProbe:
      httpGet:
        path: /healthy
        port: 8080
      initialDelaySeconds: 5
      timeoutSeconds: 1
      periodSeconds: 10
      failureThreshold: 3
    ports:
    - containerPort: 8080
      name: http
      protocol: TCP
```

The `initialDelaySeconds` sets the amount of time before the first liveness check is carried out. This gives the container time to start up. The service must respond within 1 seconds. The check is made every 10 seconds. If the endpoint does not return a successful response three times in a row the liveness check is considered as failed and the container will be restarted.

In addition to `httpGet` liveness health checks, there are `tcpSocket` and `exec` liveness health checks. The `tcpSocket` check calls a TCP socket. The `exec` check can be used to run an arbitrary command in the container. If the `exec` returns a 0 exit code then the check is considered to have passed.

A readiness probe check to see if the application is ready. 


### Resource Management

Utilisation is a measurement of the physical resources being used versus the amount purchased. Low utilisation means that the capacity of the resources far outweighs what is being used. This can be inefficienct cost-wise. 

With Kubernetes resource management you can improve the utilisation of resources. You can specify two different resource metrics: the resource request (minimum required) and; the resource limits (the maximum required). Kubernetes will automatically put pods on nodes based on their resource requirements.

CPU and memory are the most commonly requested resources. 

You can define the resource requests in the pod manifest:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
  - image: gcr.io/kuard-demo/kuard-amd64:1
    name: kuard
    resources:
      requests:
        cpu: "500m"
        memory: "128Mi"
    ports:
    - containerPort: 8080
      name: http
      protocol: TCP
```

Resources are requested by container, not pod. The pod's resource request is the sum of all the containers.

### Persistent Data with Volumes

When pods are deleted or a container restarts, all of the data in the filesystem is lost. This is not ideal for databases and other applications that need persistent storage.

To set up persistent storage two things are required: a defintion for the pod and; a mount for all the containers in the pod that need to access the container.

```yaml
apiVersion: v1
kind: pod
metadata: 
  name: kuard
spec:
  volumes:
  	- name: "kuard-data"
  	  hostPath: 
  	    path: "/var/lib/kaurd"
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:1
      name: kuard
      volumeMounts:
      	- mountPath: "/data"
      	  name: "kaurd-data"
      ports:
      	- containerPort: 8080
      	  name: http
      	  protocol: TCP
```

The `volumes` spec defines the volumes at the pod level. The `volumeMounts` mounts the container to the pod's volumes. The name in the `volumeMounts` should match one of the pod's volumes.

It's also possible to use remote disks outside of the Kubernetes cluster, such as those hosted on AWS or GCP.


## Labels and Annotations

Labels and annotations are key/value pairs that add metadata to your Kubernetes object.

Labels are primarily used for grouping Kubernetes objects. For example you could have a label for objects relating to a specific API.

Annotations are similar to labels, but they provide non-identifying information that can be used by tools and libraries. For example you can use annotations to set the icon for the object that is displayed by the Kubernetes dashboard UI. They are also used heavily in the rollout of deployments.

Label keys should follow one of these formats:

- `appVersion`
- `app.version`
- `gcr.io/appVersion`

When using the `get` command you can show labels using by adding the `--show-labels` flag.

```bash
kubectl get services --show-labels
```

They can also be displayed as a column using the `-L` flag followed by the label key:

```bash
kubectl get services -L version
```

You can filter by labels using the `--selector` flag ([there are multiple comparisons that can be used to filter](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors)):

```bash
kubectl get services --selector="version=0.0.2"
```

Labels can be modified after an object is created using the `label` command:

```bash
kubectl label deployments guitar-api "version=0.0.2"
```

You can label any type of Kubernetes object (including nodes, pods, ReplicaSets, DaemonSets), not just deployments.

To remove a label you add a dash (`-`) to the end of the label's key:

```bash
kubectl label deployments guitar-api "version-"
```

Annotations are added to the metadata section of an object manifest:

```yaml
metadata:
  annotations:
    guitar.api/icon-url: "http//wwww.example.com/icon.png"
```

## Service Discovery

Service discovery is used to find which services are listening at which addresses for which services.

The `Service` object is used to a expose another object with a service. Services use virtual IPs for the cluster IPs. Kubernetes DNS provides names for the services running in the cluster so that they can be discovered by other objects in the cluster. 

You can create a service object quickly using the `expose` command (do not use in prod):

```bash
kubectl expose deployment guitar-api
```

### Readiness Checks

Readiness checks are used to check if a service is ready to receive requests. This is because when a service is starting up it can't process requests. 

A readiness check can be added to a pod manifest to determine whether the service is ready to receive requests:

```yaml
spec:
  containers:
    name: alpaca-prod
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      periodSeconds: 2
      initialDelaySeconds: 0 
      failureThreshold: 3
      successThreshold: 1
```

NodePorts and LoadBalancers???


## ReplicaSets

ReplicaSets allow you to run multiple copies of the same container at the same time. This can have several benefits: 

- Redundancy: failure of a single instance is negated as there are multiple other instances to take over
- Scale: More requests can be processed by multiple instances
- Sharding: different tasks can be processed in parallel by multiple instances

A ReplicaSet has two main functions:
- Define a template for the creation of multiple pods
- Control the creation and management of those pods

Replicated pods are managed in a reconciliation loop. A reconciliation loop compares desired state against the current state for pods in the cluster. When the current state deviates from the desired state, the reconciliation loop takes action to correct the current state. This is a self-healing process in Kubernetes as no human interaction is required to get the system to fix itself.

Reconciliation loops for ReplicaSets manage scaling up, scaling down, node failures, and nodes rejoining the cluster.

ReplicaSets manage pods, but are decoupled from them. ReplicaSets use labels to query the pods that they manage.

Pod labels are used by the reconciliation to discover and track the pods it manages.

LoadBalancers can distribute requests between multiple pods orchestrated by a ReplicaSet.

ReplicaSet manifests must have:
- A unique name
- The defined number of pods to replicate
- A pod template for the pods to be created

```yaml
apiVersion: extensions/v1beta1
kind: ReplicaSet
metadata:
  name: kuard
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: kuard
        version: "2"
    spec:
      containers: 
        - name: kuard
          image: "gcr.io/kuar-demo/kuard-amd64:2"
```

To apply a replica set manifest:

```bash
kubectl apply -f kuard-replicaset.yaml
```

Scaling replicaSets can be done imperatively using the `scale` command. This can be used to quickly scale a set of pods without need to apply a manifest:

```bash
kubectl scale replicasets kuard --repicas=4
```

In production the `scale` commmand should only be used for emergencies as it will cause the config to drift from what is in source control.

Ideally changes using the `scale` command should be updated in soure control as well.

To scale the replicaSet decalratively you just need to update the `specs.replicas` key in the manifest then run the `apply` command.

Horizontal pod autoscaling (HPA) is used by replicaSets to automatically create pods based on resource usage (e.g. memory or CPU)

Vertical autoscaling (not currently available in Kubnernetes) scales the resources available to a pod based on usage (e.g. memory or CPU)

There is also cluster autoscaling where the number of machines in the cluster is increased based on usage.

To autoscale a set of pods with HPA:

```
kubectl autoscale replicaset kuard --min=2 --max=5 --cpu-percent=80 
```

You should avoid mixing imperative/declarative scaling with autoscaling. As they are decoupled they can cause conflicts betwen one another.

## DaemonSets

DaemonSets schedule pods onto all or a specific subset of nodes on the cluster. 

ReplicaSets are similar to DaemonSets. ReplicaSets should be used when the pod is decoupled from the architecture and can run on any node in the cluster. DaemonSets are for when you want to run on all nodes in the cluster, or a specific subset of nodes in the cluster.

Node selectors can be used with DaemonSets to limit which nodes a pod will be deployed to. 

A reconciliation loop is used to check the desired state of a DaemonSet matches the current state on the cluster.

Like a ReplicaSet, the manifest for a DaemonSet has a template for the pods that it will create.

```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
  labels:
    app: fluentd
spec:
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluentd/fluentd:v0.14.10
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        VolumeMounts:
        - name: /var/log
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
      terminationGracePeriodSeconds: 30
      volumes: 
      - name: varlog
        hostPath: 
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: var/lib/docker/containers
```

You can use the apply command to deploy the DaemonSet manifest.

To limit which nodes a DaemonSet can run on, you can use the nodeSelector property. 

```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  labels:
    app: nginx
    ssd: "true"
  name: nginx-fast-storage
spec:
  template:
    metadata:
      labels:
        app: nginx
        sdd: "true"
    spec:
      nodeSelector:
        ssd: "true"
      containers:
      - name: nginx
        image: nginx:1.10.0
```

When a label is added to a node that matches the nodeselector of the DaemonSet, the DaemonSet's pod will be deployed to that node as well. If the DaemonSet nodeselector label is removed from a node, the pod will be removed from that node.

To enable rolling updates (similar to a deployment) with a DaemonSet you need to set the `spec.updateStrategy.type` field to `RollingUpdate`.


## Jobs

Jobs are used to run one-off or non-continuous tasks, such as batch jobs or database migrations. They have a shorter life-span than pods that run a continuous service and run until they have a successful exit code.

If the task fails then a new pod will be created to run the task. Jobs can create multiple pods to run in parallel.

When defining a job two attributes can be set: the number of completions for the job; and the number of jobs to run in parallel. To run once until completion, both of these should be set to 1.

### One Shot

To create a job that will run once, not in parallel:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: oneshot
  labels: 
    chapter: jobs
spec:
  template:
    metadata:
      labels:
        chapter: jobs
    spec:
      containers:
      - name: kuard
        image: gcr.io/kuar-demo/kuard-amd64:1
        imagePullPolicy: Always
        args:
        - "--keygen-enable"
        - "--keygen-exit-on-complete"
        - "--keygen-num-to-gen=10"
      restartPolicy: OnFailure
```

If the `restartPolicy` is set to `Never` it will create a new pod every time there's a failure. This can create a lot of junk pods in the cluster. Using `OnFailure` instead will reuse the existing pod on failure instead of creating new pods.

Sometimes tasks will fail, but not cause an error code. For example, they might get stuck in an endless loop and not make any progress. To manage this Kubernetes allows you to set up liveness checks/probes for jobs.

### Parallelism 

You can run several jobs parallel. You can also limit the number of parallel pods that will be created at once, in order to reduce the load on the cluster.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: oneshot
  labels:
    chapter: jobs
spec:
  parallelism: 5
  completions: 10
  template:
    metadata:
      labels:
        chapter: jobs
    spec:
      containers:
      - name: kuard
        image: gcr.io/kuard_demo-amd64:1
        imagePullPolicy: Always
        args:
        - "--keygen-enable"
        - "--keygen-exit-on-complete"
        - "--keygen-num-to-gen=10"
      restartPolicy: OnFailure
```

### Work Queues

You can create work queues and workers on Kubernetes.

## ConfigMaps and Secrets

When building a Docker image, it is good practice to use the same image in testing, staging and production. In order to make this possible, you need to provide different configurations to the same image for each of these environments. For example you will need to configure the staging image to point at a different database than production.

Kubernetes provides ConfigMaps, which allow you to configure the same Docker image for different environments. Secrets are like ConfigMaps, but they store sensitive information.

ConfigMaps can be thought of as environment variables that are passed to the pod when it is created. They can also be thought of as a small file system that exists in the cluster.

To create a ConfigMap manifest:

```yaml
apiVersion: v1
kind: ConfigMap
data: 
  another-param: another-value
  extra-param: extra-value
  my-config.txt: | 
    parameter1 = value1
    parameter2 = value2
  kind: ConfigMap
metadata:
  creationTimeStamp: ...
  name: my-config
  namespace: default
  resourceVersion: "13556"
  selfLink: /api/v1/namespaces/default/configmaps/my-config
  uid: 3641c553-f7de-11e6-98c9-06135271a273
```

There are different ways to use ConfigMaps within a pod:
- As a filesystem: A file is created for each entry based on the key name. The key values are used as the file contents.
- As environment variables: The keys and values are used as environment variables.
- As command line arguments: The keys and vales are used as command-line arguments for the containers.

To use the ConfigMap values as a file system you mount a ConfigMap volume to the pod. The volume should reference the name of the ConfigMap object:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard-config
spec:
  containers:
    - name: test-container
      image: gcr.io/kuar-demo/kuard-amd64:1
      imagePullPolicy: Always
      volumeMounts:
        - name: config-volume
          mountPath: /config
  volumes:
    - name: config-volume
      configMap: 
        name: my-config
  restartPolicy: Never
```

To use the ConfigMap as environment variables:

```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: kuard-config
spec:
  containers:
    - name: test-container
      image: gcr.io/kuar-demo/kuard-amd64:1
      imagePullPolicy: Always
      env:
        - name: ANOTHER_PARAM
          valueFrom:
            configMapKeyRef:
              name: my-config
              key: another-pod
```

To use the ConfigMap as a command-line argument, you need to also set it as an environment variable:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard-config
spec:
  containers:
    - name: test-container
      image: gcr.io/kuard-demo/kuard-amd64:1
      imagePullPolicy: Always
      command:
        - "/kuard"
        - "$(EXTRA_PARAM)"
      env:
        - name: EXTRA_PARAM
          valueFrom: 
            configMapKeyRef:
              name: my-config
              key: extra-param
```

### Secrets

Secrets are similar to ConfigMaps, except they are used to store sensitive data.

The values in secrets must be base64 encoded:

```bash
$ echo -n 'admin' | base64
YWRtaW4=
$ echo -n '1234' | base64
Y54gd6e
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-creds
type: Opaque
data:
  username: YWRtaW4=
  password: Y54gd6e
```

Secrets can also be used for storing tls certificates, storing creds for Docker image registries and stuff.

To setup a Pod to consume secrets you need to mount the secrets as a volume or use them as environment variables, as was shown above with ConfigMaps.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard-config
spec:
  containers:
    - name: test-container
      image: gcr.io/kuar-demo/kuard-amd64:1
      imagePullPolicy: Always
      volumeMounts:
        - name: secret-volume
          mountPath: /secret
  volumes:
    - name: secret-volume
      secret: 
        name: db-creds
  restartPolicy: Never
```

## Deployments

Pods and ReplicaSets are tied to specific images. It is not expected that these images will ever change during the pod's lifetime.

Deployments are used to manage new releases of your software to the Kubernetes cluster.

User-configurable rollouts are used to manage the release of new software to the deployments. Rollouts allow you to set the amount of time to wait between upgrades. They also perform health checks to ensure the new release is healthy before replacing the old pod. This removes downtime from new releases.

Deployments are managed by the Kubernetes cluster deployment controller.

The rollouts are managed by the controller on the server-side cluster so that unreliable internet connections on the client-side do not affect reliability and safety of new releases.

Deployments manage ReplicaSets, which in turn manage Pods. Deleting a Deployment also deletes any related ReplicaSets and Pods.

Deleting a Pod created by a deployment will restart a new Pod, due to the desired state defined in the Deployment.

To define a Deployment manifest:

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations: 
    deployment.kubernetes.io/revision: "1"
  labels:
    run: nginx
  name: nginx
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels: 
      run: nginx
  strategy: 
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx:1.7.12
        imagePullPolicy: Always
      dnsPolicy: ClusterFirst
      restartPolicy: Always
```

The `strategy` object in the manifest dictates how the rollout will occur. There are two options for this: `RollingUpdate` and `Recreate`.

The `RollingUpdate` strategy allows you to rollout a new version while still receiving traffic to the old pod. This is useful for customer-facing applications. It updates one pod at a time and is slower than a `Recreate` strategy. As pods are being updated, it can lead to multiple versions of an APi or front-end being used simultaneously for a small amount of time.

The `maxUnavailable` and `maxSurge` values are used to set the behaviour of a `RollingUpdate`. `MaxUnavailable` sets the number of pods that can be down during the update. It can either be an absolute number (e.g. 3) or a percentage (e.g. 30%).

The `maxSurge` parameter creates additional pods during the update so that availability is not reduced during the rollout. It can also be expressed as an absolute number or a percentage.

The `Recreate` strategy updates the pod image of a `ReplicaSet`, then stops all of its pods. The `ReplicaSet` will then recreate all of the pods with the new image. This can lead to downtime.

You can view the rollout history of a Deployment with `kubectl rollout history`.


```bash
kubectl rollout history deployment nginx
```

To view a specific rollout revision you can add the `--revision` flag:

```bash
kubectl rollout history deployment nginx --revision=2
```

To update an Deployment you can use the `set image` command. This will change the Deployment's image and create new pods with that image.

```bash
kubectl set image deployment nginx-deployment nginx=nginx:1.9.1
``` 

To rollback to a previous deployment revision you can use the `rollout undo` command:

```bash
kubectl rollout undo deployment nginx
```

To specify a specific revision to rollback to use the `--to-revision` flag:

```bash 
kubectl rollout undo deployment nginx --to-revision=3
```

Setting the `--to-revision` value to `0` will rollback to the previous release.

All of the revision history is kept for a deployment. To limit the amount of time that it keeps the revisions set the `revisionHistoryLimit` value in the manifest's spec. The value should be the number of days that you want to keep the history for.

When the deployment controller checks if a new Pod is ready, you may want to add an additional wait time in addition to the readiness check to ensure that the pod is definitely behaving correctly. You can do this by setting the `minReadySeconds` in the manifest:

```yaml
spec:
  minReadySeconds: 60
```
You can also set a timeout where the deployment will fail if the readiness checks do not pass within a certain time period. This will stop the deployment and rollback to the previous version:

```yaml
spec:
  progressDeadlineSeconds: 600
```

## Ingresses

Services expose pods in the cluster to other pods and to the outside world.

Services are typically one of two types:
- ClusterIP: Internal service that can only be accessed within the cluster
- NodePort: External service that can be accessed from outside the cluster

NodePorts can be further configured to use a LoadBalancer. 

An Ingress is a collection of rules that allow in-bound connections to reach cluster Services. The Services must be exposed via NodePorts.

Ingresses expose Services via external URLs. Ingresses are intermediaries between domain names to service endpoints. Ingresses decouple the deployment from the actual DNS name.

External load balancers can be configured to work with Kubernetes via Ingresses.

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  backend:
    serviceName: nginx
    servicePort: 80
```

Ingresses require different setup on Minikube comparsed to public cloud providers like Google Cloud Platform.

Static IP addresses are essnetial for using Ingresses. Be default they use ethereal IP addresses that will change when the Ingresses object is restarted. The static IP address makes sure that when restarted the Ingress will be at the same IP.

The static IP address needs to be reserved with the cloud provider. In the example below the static IP was called `nginx-static-ip`

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations: 
    kubernetes.io/ingress.global-static-api-name: nginx-static-ip
spec:
  backend: 
    serviceName: nginx
    servicePort: 80
```

The same ingress can be used for multiple services without requiring multiple domains or load balancers.

The setup to allow multiple services run using a single domain is called a fanout ingress as it maps multiple services using different paths:

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: fanout-ingress
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: nginx
          servicePort: 80
      - path: /echo
        backend:
          serviceName: echoserver
          servicePort: 8080
```

Ingresses are actaully deployed as pods, but behave differently in that they manage access to the cluster and cannot be viewed as regular pods.


## Storage
By default, containers that run on Kubernetes are stateless. When they are restarted they will lose the contents of their file systems. 

When working with specific types of containers in Kubernetes you will want to maintain state. For example a database needs to retain records. 

By default when a container is restarted it will lose state. To alleviate this for containers that require state persistent storage can be set up with Kubernetes.

### External Service

One type of persistent storage is an external service. For example if you are running a database on a legacy server or using a paid hosting solution. 

You can use Kubernete's services to make the external service appear like it is running on the cluster by using the Kubernetes cluster DNS. The cluster DNS will redirect to the external service, giving other services access to the service like it was on the cluster. Using this will make it easier to swap out the service to one that is actually running on the cluster during testing or enitrely at a later date.

The manifest for an external service is defined like so:

```yaml
kind: Service
apiVersion: v1
metadata:
  name: external-database
spec:
  type: ExternalService
  externalName: www.database-hosting-company.com
```

For external services that do not have a DNS, only an IP address, the setup is a little different. The Service manifest has no `externalName` and requires an additional `Endpoint` manifest:

```yaml
kind: Service
apiVersion: v1
metadata: 
  name: external-ip-database
```

The `Endpoint` manifest:

```yaml
kind: Endpoint
apiVersion: v1
metadata:
  name: external-ip-database
subsets:
  - addresses: 
    - ip: 192.168.0.1
    ports:
    - port: 3306
```

One limitation of external services is that you cannot perform health checks.

### Singletons

For some storage, such as a MySQL database you may want to create a singleton of the service on your cluster.
