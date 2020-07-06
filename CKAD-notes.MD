## While you would be working mostly the declarative way - using definition files, imperative commands can help in getting one time tasks done quickly, as well as generate a definition template easily. This would help save considerable amount of time during your exams.
## Before we begin, familiarise with the two options that can come in handy while working with the below commands:
--dry-run: By default as soon as the command is run, the resource will be created. If you simply want to test your command , use the --dry-run option. This will not create the resource, instead, tell you weather the resource can be created and if your command is right.
-o yaml: This will output the resource definition in YAML format on screen.

# Use the above two in combination to generate a resource definition file quickly, that you can then modify and create resources as required, instead of creating the files from scratch.

# POD
## Create an NGINX Pod
<pre><code>
kubectl run --generator=run-pod/v1 nginx --image=nginx OR
kubectl run nginx --image=nginx --restart=Never
</code></pre>

# Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)
<pre><code>
kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml
</code></pre>

## Create a deployment
<pre><code>
kubectl create deployment nginx --image=nginx
</code></pre>

## Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)
<pre><code>
kubectl create deployment nginx --image=nginx --dry-run -o yaml
</code></pre>

## Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)
<pre><code>
kubectl create deployment does not have a --replicas option. You could first create it and then scale it using the kubectl scale command.
kubectl scale deployment nginx --replicas=4
</code></pre>

## Save it to a file - (If you need to modify or add some other details)
<pre><code>
kubectl create deployment nginx --image=nginx --dry-run -o yaml > nginx-deployment.yaml
kubectl apply -f nginx-deployment.yaml
</code></pre>

# Service
<pre><code>
Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379
kubectl expose pod redis --port=6379 --name redis-service --dry-run -o yaml
(This will automatically use the pod's labels as selectors)
Or
kubectl create service clusterip redis --tcp=6379:6379 --dry-run -o yaml (This will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)

Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:
kubectl expose pod nginx --port=80 --name nginx-service --dry-run -o yaml
(This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)
Or
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run -o yaml
(This will not use the pods labels as selectors)
Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the `kubectl expose` command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.
</code></pre>

Reference:
https://kubernetes.io/docs/reference/kubectl/conventions/


The default output format for all kubectl commands is the human-readable plain-text format.
The -o flag allows us to output the details in several different formats.

kubectl [command] [TYPE] [NAME] -o <output_format>
Here are some of the commonly used formats:
1.	-o jsonOutput a JSON formatted API object.
2.	-o namePrint only the resource name and nothing else.
3.	-o wideOutput in the plain-text format with any additional information.
4.	-o yamlOutput a YAML formatted API object.
Here are some useful examples:
•	Output with JSON format:
master $ kubectl create namespace test-123 --dry-run -o json
{
    "kind": "Namespace",
    "apiVersion": "v1",
    "metadata": {
        "name": "test-123",
        "creationTimestamp": null
    },
    "spec": {},
    "status": {}
}
master $

•	Output with YAML format:
master $ kubectl create namespace test-123 --dry-run -o yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: test-123
spec: {}
status: {}

•	Output with wide (additional details):
Probably the most common format used to print additional details about the object:
master $ kubectl get pods -o wide
NAME      READY   STATUS    RESTARTS   AGE     IP          NODE     NOMINATED NODE   READINESS GATES
busybox   1/1     Running   0          3m39s   10.36.0.2   node01   <none>           <none>
ningx     1/1     Running   0          7m32s   10.44.0.1   node03   <none>           <none>
redis     1/1     Running   0          3m59s   10.36.0.1   node01   <none>           <none>
master $

For more details, refer:
https://kubernetes.io/docs/reference/kubectl/overview/
https://kubernetes.io/docs/reference/kubectl/cheatsheet/



Edit a POD
Remember, you CANNOT edit specifications of an existing POD other than the below.
•	spec.containers[*].image
•	spec.initContainers[*].image
•	spec.activeDeadlineSeconds
•	spec.tolerations
For example you cannot edit the environment variables, service accounts, resource limits (all of which we will discuss later) of a running pod. But if you really want to, you have 2 options:
1. Run the kubectl edit pod <pod name> command. This will open the pod specification in an editor (vi editor). Then edit the required properties. When you try to save it, you will be denied. This is because you are attempting to edit a field on the pod that is not editable.
A copy of the file with your changes is saved in a temporary location as shown above.
You can then delete the existing pod by running the command:
kubectl delete pod webapp

Then create a new pod with your changes using the temporary file
kubectl create -f /tmp/kubectl-edit-ccvrq.yaml

2. The second option is to extract the pod definition in YAML format to a file using the command
kubectl get pod webapp -o yaml > my-new-pod.yaml
Then make the changes to the exported file using an editor (vi editor). Save the changes
vi my-new-pod.yaml
Then delete the existing pod
kubectl delete pod webapp
Then create a new pod with the edited file
kubectl create -f my-new-pod.yaml

Edit Deployments
With Deployments you can easily edit any field/property of the POD template. Since the pod template is a child of the deployment specification, with every change the deployment will automatically delete and create a new pod with the new changes. So if you are asked to edit a property of a POD part of a deployment you may do that simply by running the command
kubectl edit deployment my-deployment


## Remember that secrets encode data in base64 format. Anyone with the base64 encoded secret can easily decode it. As such the secrets can be considered as not very safe.
## The concept of safety of the Secrets is a bit confusing in Kubernetes. The kubernetes documentation page and a lot of blogs out there refer to secrets as a "safer option" to store sensitive data. They are safer than storing in plain text as they reduce the risk of accidentally exposing passwords and other sensitive data. In my opinion it's not the secret itself that is safe, it is the practices around it. 

Secrets are not encrypted, so it is not safer in that sense. However, some best practices around using secrets make it safer. As in best practices like:
•	Not checking-in secret object definition files to source code repositories.
•	Enabling Encryption at Rest for Secrets so they are stored encrypted in ETCD. 

Also the way kubernetes handles secrets. Such as:
•	A secret is only sent to a node if a pod on that node requires it.
•	Kubelet stores the secret into a tmpfs so that the secret is not written to disk storage.
•	Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.
Read about the protections and risks of using secrets here

# Having said that, there are other better ways of handling sensitive data like passwords in Kubernetes, such as using tools like Helm Secrets, HashiCorp Vault. I hope to make a lecture on these in the future.

## Note: Capabilities are only supported at the container level and not at the POD level

<pre><code>
apiVersion: v1
kind: Pod
metadata:
  name: sec-pod
  labels:
    sec: mac_admin

spec:
  #securityContext:
     #runAsUser: 1000 ##This will set @ the POD level
  containers
  -name: ubuntu
    image: ubuntu
    command: [“sleep”, “3600”]
    securityContext: #This is required for capabilities just remove the runAsUser portion
      runAsUser: 1000
      capabilities:
         add: [“MAC_ADMIN”]

kubectl taint nodes node-name key=value:taint-effect(NoSchedule | PreferNoSchedule | NoExecute)

kubectl taint node node01 app=blue:NoSchedule

spec:
  containers:
  - name: nginx-container
    image: nginx
  tolerations:
  - key: "app"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"

kubectl describe node kubemaster | grep Taint
</code></pre>

# Remove taint from Master node(s)

kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule-

<pre><code>

spec:
  containers:
  ..
  nodeSelector:
    size: Large

kubectl label nodes node1 size=Large
</code></pre>

## Node Affinity

<pre><code>

spec:
  containers
    - name: ..
      image ..
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values:
            - Large
            - Medium #Large and medium nodes
</code></pre>

## Other operators are NotIn and Exists

### 2 types of affinity requiredDuringSchedulingIgnoredDuringExecution & preferredDuringSchedulingIgnoredDuringxecution

## Make sure you check out these tips and tricks from other students who have cleared the exam:

[Tips&Tricks](https://medium.com/@harioverhere/ckad-certified-kubernetes-application-developer-my-journey-3afb0901014)

## Multi-Container Pods patterns

### Ambassador,Adapter,Sidecar

### Kibana create index pattern

[Loom CKAD](https://www.loom.com/share/c2ae70197e8340a0ba77fc1de8179182)

## Two types of probes, readiness and liveness

### Addationally we can use periodSeconds: 5(how often to check) and failureThreshold: 8(this is by default 3)

### The difference between readiness and liveness readiness is, is my container ready for users, so this is on initialization

### liveness is periodically check my container to see if it's still up, so this happens during the lifecycle of the container

<pre><code>

spec:
  containers:
  - name: container-1
    image: nginx
    readinessProbe:
      httpGet:
        path: /api/ready
        port: 8080

spec:
  containers:
  - name: container-1
    image: nginx
    readinessProbe:
      tcpSocket:
        port: 3306
      initialDelaySeconds: 5

spec:
  containers:
  - name: container-1
    image: nginx
    readinessProbe:
      exec:
        command
          - cat
          - /app/is_ready
</code></pre>

<pre><code>

spec:
  containers:
  - name: container-1
    image: nginx
    livenessProbe:
      httpGet:
        path: /api/ready
        port: 8080

spec:
  containers:
  - name: container-1
    image: nginx
    livenessProbe:
      tcpSocket:
        port: 3306
      initialDelaySeconds: 5
      periodSeconds: 5

spec:
  containers:
  - name: container-1
    image: nginx
    livenessProbe:
      exec:
        command
          - cat
          - /app/is_ready
</code></pre>

<pre><code>

kubectl logs -f pod_name

kubectl logs -f pod_name container_name(for multi-container pods)

</code></pre>

<pre><code>
kubectl top nodes
kubectl top pods
kubectl create -f .
kubectl apply -f .
</code></pre>

<pre><code>
kubectl get pods --selector app=App1

kubectl get pods --selector env=prod,bu=finance,tier=frontend

metadata:
  name: app1
  labels:
    app: App1
    function: front-end

spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1
  template:
    metadata:
      labels:
        app: App1
        function: front-end
  spec:
    containers:
    - name: app1
      image: nginx

</code></pre>

<pre><code>

kubectl rollout status deployment mydeploy
kubectl rollout history deployment mydeploy
kubectl rollout history deployment/mydeploy
kubectl rollout undo deployment/mydeploy

kubectl set image deployment/mydeploy nginx=nginx:1.9.1

apiVerion: v1
kind: Deployment
metadata:
  name: my-deploy
  labels:
    tier: frontend

spec:
  template:
    metadata:
      name: my-deploy
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx-container
        image: nginx
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  strategy:
    type: Recreate
    #type: Rollingupdate(this is the default)
</code></pre>

## Deployment strategies

### 1. Recreate - destroy everything first and then deploy new pods

### 2. Rolling Update - this is the default 


# Updating a Deployment


### Here are some handy examples related to updating a Kubernetes Deployment:

### Creating a deployment, checking the rollout status and history:

### In the example below, we will first create a simple deployment and inspect the rollout status and the rollout history:

<pre><code>


master $ kubectl create deployment nginx --image=nginx:1.16
deployment.apps/nginx created

master $ kubectl rollout status deployment nginx
Waiting for deployment "nginx" rollout to finish: 0 of 1 updated replicas are available...
deployment "nginx" successfully rolled out

master $


master $ kubectl rollout history deployment nginx
deployment.extensions/nginx
REVISION CHANGE-CAUSE
1     <none>

master $

</code></pre>

### Using the --revision flag:

### Here the revision 1 is the first version where the deployment was created. 

### You can check the status of each revision individually by using the --revision flag:

<pre><code>

master $ kubectl rollout history deployment nginx --revision=1
deployment.extensions/nginx with revision #1

Pod Template:
 Labels:    app=nginx    pod-template-hash=6454457cdb
 Containers:  nginx:  Image:   nginx:1.16
  Port:    <none>
  Host Port: <none>
  Environment:    <none>
  Mounts:   <none>
 Volumes:   <none>
master $ 

</code></pre>

### Using the --record flag:
### You would have noticed that the "change-cause" field is empty in the rollout history output. We can use the --record flag to save the command used to create/update a deployment against the revision number.

<pre><code>

master $ kubectl set image deployment nginx nginx=nginx:1.17 --record
deployment.extensions/nginx image updated
master $master $

master $ kubectl rollout history deployment nginx
deployment.extensions/nginx

REVISION CHANGE-CAUSE
1     <none>
2     kubectl set image deployment nginx nginx=nginx:1.17 --record=true
master $

</code></pre>

### You can now see that the change-cause is recorded for the revision 2 of this deployment.

### Lets make some more changes. In the example below, we are editing the deployment and changing the image from nginx:1.17 to nginx:latest while making use of the --record flag.

<pre><code>

master $ kubectl edit deployments. nginx --record
deployment.extensions/nginx edited

master $ kubectl rollout history deployment nginx
REVISION CHANGE-CAUSE
1     <none>
2     kubectl set image deployment nginx nginx=nginx:1.17 --record=true
3     kubectl edit deployments. nginx --record=true



master $ kubectl rollout history deployment nginx --revision=3
deployment.extensions/nginx with revision #3

Pod Template: Labels:    app=nginx
    pod-template-hash=df6487dc Annotations: kubernetes.io/change-cause: kubectl edit deployments. nginx --record=true

 Containers:
  nginx:
  Image:   nginx:latest
  Port:    <none>
  Host Port: <none>
  Environment:    <none>
  Mounts:   <none>
 Volumes:   <none>

master $

</code></pre>

### Undo a change:
### Lets now rollback to the previous revision:

<pre><code>

master $ kubectl rollout undo deployment nginx
deployment.extensions/nginx rolled back

master $ kubectl rollout history deployment nginx
deployment.extensions/nginxREVISION CHANGE-CAUSE
1     <none>
3     kubectl edit deployments. nginx --record=true
4     kubectl set image deployment nginx nginx=nginx:1.17 --record=true




master $ kubectl rollout history deployment nginx --revision=4
deployment.extensions/nginx with revision #4Pod Template:
 Labels:    app=nginx    pod-template-hash=b99b98f9
 Annotations: kubernetes.io/change-cause: kubectl set image deployment nginx nginx=nginx:1.17 --record=true
 Containers:
  nginx:
  Image:   nginx:1.17
  Port:    <none>
  Host Port: <none>
  Environment:    <none>
  Mounts:   <none>
 Volumes:   <none>


master $ kubectl describe deployments. nginx | grep -i image:
  Image:    nginx:1.17
master $

</code></pre>

### With this, we have rolled back to the previous version of the deployment with the image = nginx:1.17.

## Jobs

<pre><code>

docker run ubuntu expr 3 + 2
kubectl delete job myjob

apiVersion: v1
kind: Pod
metadata:
  name: math-pod

spec:
  containers:
  - name: math-pod
    image: ubuntu
    command: ['expr', '3', '+', '2']
  restartPolicy: Always,Never,OnFailure
---
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job

spec:
  template:
    spec:
      containers:
        - name: math-pod
          image: ubuntu
          command: ['expr', '3', '+', '2']
      restartPolicy: Never
  backoffLimit: 6 # how many times we retry before we fail and stop
---
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: my-cronjob
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      completions: 3
      parallelism: 3 # run this in parallel
      template:
        spec:
          containers:
            - name: math-pod
              image: ubuntu
              command: ['expr', '3', '+', '2']
          restartPolicy: Never
</code></pre>

## Services & Networking

### Service types NodePort, ClusterIP, and Loadbalance

### NodePort range is 30000 - 32767

<pre><code>

apiVersion: v1
kind: Service
metadata:
  name: my-service

spec:
  type: NodePort
  ports:
   - targetPort: 80
     port: 80
     nodePort: 30008
  selector:
    app: myapp
    type: frontend
---
apiVersion: v1
kind: Service
metadata:
  name: my-service

spec:
  type: ClusterIP
  ports:
   - targetPort: 80
     port: 80
  selector:
    app: myapp
    type: frontend
</code></pre>

### Network Policy

<pre><code>

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy

spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    ports:
    - protocol: TCP
      port: 3306

</code></pre>

## Only some CNI's support Network Policy

### - Kube-router

### - Calico

### - Romana

### - Weave-net

## *Note Flannel does not support NetworkPolicy(Check if the've updated to include this)

### See internal-policy.yaml

### kubectl get netpol

## Ingress Networking

<pre><code>

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
  namespace: ingress-sapce

spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress-controller
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      containers:
        - name: nginx-ingress-controller
          image: quay.io/
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear

spec:
  backend:
    serviceName: wear-service
    servicePort: 80
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch

spec:
  rules:
  - http:
      paths:
      - path: /wear
        backend:
          serviceName: wear-service
          servicePort: 80
      - path: /watch
        backend:
          serviceName: watch-service
          servicePort: 80
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch

spec:
  rules:
  - host: wear.mystore.com
    http:
      paths:
      - backend:
          serviceName: wear-service
          servicePort: 80
  - host: watch.mystore.com
    http:
      paths:
      - backend:
          serviceName: watch-service
          servicePort: 80

</code></pre>


<pre><code>

apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data: {}

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account

</code></pre>


## rewrite-target option

### Different ingress controllers have different options that can be used to customise the way it works. NGINX Ingress controller has many options that can be seen [here](https://kubernetes.github.io/ingress-nginx/examples/). I would like to explain one such option that we will use in our labs. The [Rewrite](https://kubernetes.github.io/ingress-nginx/examples/rewrite/) target option.



### Our watch app displays the video streaming webpage at http://<watch-service>:<port>/

### Our wear app displays the apparel webpage at http://<wear-service>:<port>/

### We must configure Ingress to achieve the below. When user visits the URL on the left, his request should be forwarded internally to the URL on the right. Note that the /watch and /wear URL path are what we configure on the ingress controller so we can forwarded users to the appropriate application in the backend. The applications don't have this URL/Path configured on them:

### http://<ingress-service>:<ingress-port>/watch --> http://<watch-service>:<port>/

### http://<ingress-service>:<ingress-port>/wear --> http://<wear-service>:<port>/

### Without the rewrite-target option, this is what would happen:

### http://<ingress-service>:<ingress-port>/watch --> http://<watch-service>:<port>/watch

### http://<ingress-service>:<ingress-port>/wear --> http://<wear-service>:<port>/wear

### To fix that we want to "ReWrite" the URL when the request is passed on to the watch or wear applications. We don't want to pass in the same path that user typed in. So we specify the rewrite-target option. This rewrites the URL by replacing whatever is under rules->http->paths->path which happens to be /pay in this case with the value in rewrite-target. This works just like a search and replace function.

### For example: replace(path, rewrite-target)

### In our case: replace("/path","/")

<pre><code>

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /pay
        backend:
          serviceName: pay-service
          servicePort: 8282
</code></pre>

### In another example given [here](https://kubernetes.github.io/ingress-nginx/examples/rewrite/), this could also be:

### replace("/something(/|$)(.*)", "/$2")

<pre><code>

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: rewrite
  namespace: default
spec:
  rules:
  - host: rewrite.bar.com
    http:
      paths:
      - backend:
          serviceName: http-svc
          servicePort: 80
        path: /something(/|$)(.*)
</code></pre>

## Storage

### docker -v is the old way the new way is --mount

<pre><code>
docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
</code></pre>

### docker storage drivers
#### - AUFS
#### - ZFS
#### - BTRFS
#### - Device Mapper
#### - Overlay
#### - Overlay2

<pre><code>

apiVersion: v1
kind: Pod
metadata:
  name: pod-pers

spec:
  containers:
  - name: alpine
    image: alpine
    volumeMounts:
    - mountPath: /opt # this is where it's mounted in the container
      name: data-volume # must patch with below
  volumes:
  - name: data-volume # volume mounted in container same as -v data-volume:/opt in docker
    hostPath:
      path: /data # mounted on the node
      type: Directory
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-pers

spec:
  containers:
  - name: alpine
    image: alpine
    volumeMounts:
    - mountPath: /opt # this is where it's mounted in the container
      name: data-volume # must patch with below
  volumes:
  - name: data-volume # volume mounted in container same as -v data-volume:/opt in docker

    awsElasticBlockStore:
      volumeID: <volume-id>
      fstype: ext4
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1

spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1

spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-claim

spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
  storageClassName: manual
</code></pre>

### persistentVolumeReclaimPolicy: Retain(default), Delete, Recycle

### Once you create a PVC use it in a POD definition file by specifying the PVC Claim name under persistentVolumeClaim section in the volumes section like this:

<pre><code>

apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
</code></pre>

### The same is true for ReplicaSets or Deployments. Add this to the pod template section of a Deployment on ReplicaSet.

### Reference URL: [Here](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes)

### Extra notes this subject is not apart of the exam

#### - Storage Classes

#### - Stateful sets

<pre><code>
apiVersion: apps/v1
kind: StatefuleSet
metadata:
  name: mysql
  labels:
    tier: db

spec:
  template:
    metadata:
      labels:
        tier: db
    spec:
      containers:
      - name: mysql
        image: mysql
  replicas: 3
  selector:
    matchLabels:
      tier: db
  serviceName: mysql-h
#  podManagementPolicy: Parallel  optional if you don't want the default of OrderedReady
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-h

spec:
  ports:
    - port: 3306
  selector:
    tier: db
  clusterIP: None
---
apiVersion: apps/v1
kind: StatefuleSet
metadata:
  name: mysql
  labels:
    tier: db

spec:
  template:
    metadata:
      labels:
        tier: db
    spec:
      containers:
      - name: mysql
        image: mysql
        volumeMounts:
         - mountpath: /var/lib/mysql
           name: data-volume
      volumes:
        - name: data-volume
          persistentVolumeClaim:
            claimName: data-volume
volumeClaimTemplates:
- metadata:
  name: data-volume
  spec:
    accessModes:
      - ReadWriteOnce
    storageClassName: google-storage
    resources:
      requests:
        storage 500Mi
  replicas: 3
  selector:
    matchLabels:
      tier: db
  serviceName: mysql-h
#  podManagementPolicy: Parallel  optional if you don't want the default of OrderedReady

</code></pre>

<pre><code>

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
</code></pre>

### DNS naming in k8s

#### mysql.default.svc.cluster.local this would be a service in the default namespace

#### mysql.finance.svc.cluster.local this would be a service in the finance namespace

### Headless services

#### mysql-0.mysql-h.default.svc.cluster.local, mysql-1.mysql-h.default.svc.cluster.local, etc.

#### the above is a headless svc attached to statefulset first is the pod name which is ordered in a statefulset then the svc name and then the namespace

#### In a statefulset we need volumeClaimTemplates so that each pod will get it's own pvc and it's own pv, which means each pod gets a separate disk and not all pods sharing 1 disk

#### if a pod is restarted it will be attached to the same PV so statefulsets ensure consistent storage as well as consistent names/dns

<pre><code>

spec:
  nodeName: master

  volumeMounts:
  - mountPath: /redis-master-data
    name: data
  - mountPath: /redis-master
    name: redis-config
volumes:
- name: data
  emptyDir: {}
- name: redis-config
  configMap:
    name: redis-config
   
</code></pre>

## Useful kubectl commands

### kubectl get pods --show-labels


## CKAD Lightning LAB help

### kubectl -n default create configmap time-config --from-literal=TIME_FREQ=10

### kubectl run --generator=run-pod/v1 time-check --image=busybox --dry-run -o yaml > pod.yaml

### command: [ "/bin/sh", "-C", "while true; do date; sleep $TIME_FREQ;done > /opt/time/time-check.log"]

### kubectl create deployment nginx-deploy --image=nginx:1.16 --dry-run -o yaml > ng-deploy.yaml

### kubectl set image deployment/nginx-deploy nginx=nginx:1.17

### kubectl rollout history deployment nginx-deploy

### kubectl rollout undo deployment nginx-deploy

### cpus can be done like so cpu: "0.2" this will request .2 CPU which is equal to 200m

### Lightning lab 2

<pre><code>
kubectl get pod nginx1401 -n dev1401 -o yaml > pod.yaml

kubectl create cronjob dice --image=kodecloud/throw-dice --schedule "*/1 * * * *" --dry-run -o yaml > dice.yaml

kubectl create cronjob test-job --image=busybox --schedule="*/1 * * * *"

completions: 1
backoffLimit: 25
activeDeadlineSeconds: 20

kubectl run --generator=run-pod/v1 my-busybox --image=busybox --command sleep 3600 --dry-run -o yaml > busybox.yaml

kubectl explain pods --recursive | less

kubectl logs dev-pod-dind-878516 -c log-x(note here I am specifying which container with -c this pod has multiple containers)

</code></pre>

### Get ingress template from kubernetes.io we have to use the host option
