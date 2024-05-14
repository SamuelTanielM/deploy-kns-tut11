Nama: Samuel Taniel Mulyadi

NPM : 2206081805

# Reflection on Hello Minikube

> 1. Compare the application logs before and after you exposed it as a Service.
Try to open the app several times while the proxy into the Service is running.
What do you see in the logs? Does the number of logs increase each time you open the app?

Before Exposing the Service:
The application logs only show the initialization of the servers:
```
I0514 02:54:56.621784       1 log.go:195] Started HTTP server on port 8080
I0514 02:54:56.622209       1 log.go:195] Started UDP server on port  8081
```
This indicates that the application has started its HTTP and UDP servers but has not received any incoming requests.


After Exposing the Service:
In addition to the initialization logs, there are logs indicating incoming HTTP requests:
```
I0514 02:54:56.621784       1 log.go:195] Started HTTP server on port 8080
I0514 02:54:56.622209       1 log.go:195] Started UDP server on port  8081
I0514 02:59:54.595593       1 log.go:195] GET /
I0514 02:59:54.660937       1 log.go:195] GET /
I0514 02:59:58.683775       1 log.go:195] GET /
I0514 02:59:59.109975       1 log.go:195] GET /
```
These logs correspond to HTTP GET requests to the root endpoint ("/") of the application. This indicates that after exposing the service, the application is receiving and logging incoming HTTP requests.


> 2. Notice that there are two versions of kubectl get invocation during this tutorial section. The first does not have any option, while the latter has -n option with value set to kube-system. What is the purpose of the -n option and why did the output not list the pods/services that you explicitly created?

The -n option in the kubectl get command specifies the namespace from which to retrieve resources

**Default Namespace:** 
When you run kubectl get without specifying a namespace, it defaults to the default namespace.

**Specified Namespace:** 
Using the -n option allows you to specify a particular namespace from which to retrieve resources. For example, kubectl get pods, services -n kube-system from the tutorial retrieves pods from the kube-system namespace.


# Reflection on Rolling Update & Kubernetes Manifest File

> 1. What is the difference between Rolling Update and Recreate deployment strategy?

**Rolling Update**

The Rolling Update strategy gradually replaces the old version of an application with the new one. This is done by incrementally updating pods one by one, ensuring that there is always some version of the application running. This method minimizes downtime and is the default deployment strategy in Kubernetes
- Minimal Downtime: The application remains available throughout the update process as new pods are created before old ones are terminated.
- Controlled Rollout: Parameters like maxSurge and maxUnavailable can be configured to control the number of pods that can be added or removed during the update, providing flexibility in how the update progresses.

--

**Recreate**

The Recreate strategy, on the other hand, involves taking down the existing version of the application entirely before bringing up the new version. This results in a complete but temporary downtime during the update.
- Full Downtime: All existing pods are terminated before any new pods are created. This leads to a period where the application is completely unavailable.
- Simplicity: The process is straightforward as it involves just stopping the old version and starting the new one.

> 2. Try deploying the Spring Petclinic REST using Recreate deployment strategy and document  your attempt.

We can use the Spring Petclinic REST from before and configure it's strategy to Recreate Deployment. We can either patch it or edit it on the file and apply it manually. I tried the first method, by patching it using another file, using this file
```yaml
spec:
  strategy:
    $retainKeys:
    - type
    type: Recreate
```

kubectl patch deployments spring-petclinic-rest --patch-file ./patch-recreate.yaml

We can see the change through describe
```
PS C:\Users\samue> kubectl describe deployments/spring-petclinic-rest
Name:                   spring-petclinic-rest
Namespace:              default
CreationTimestamp:      Tue, 14 May 2024 11:42:03 +0700
Labels:                 app=spring-petclinic-rest
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=spring-petclinic-rest
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=spring-petclinic-rest
  Containers:
   spring-petclinic-rest:
    Image:        docker.io/springcommunity/spring-petclinic-rest:3.2.1
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   spring-petclinic-rest-54f476f68 (4/4 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  6h2m  deployment-controller  Scaled up replica set spring-petclinic-rest-54f476f68 to 4
PS C:\Users\samue> kubectl patch deployments spring-petclinic-rest --patch-file .\patch-recreate.yaml
deployment.apps/spring-petclinic-rest patched
PS C:\Users\samue> kubectl describe deployments/spring-petclinic-rest
Name:               spring-petclinic-rest
Namespace:          default
CreationTimestamp:  Tue, 14 May 2024 11:42:03 +0700
Labels:             app=spring-petclinic-rest
Annotations:        deployment.kubernetes.io/revision: 1
Selector:           app=spring-petclinic-rest
Replicas:           4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:       Recreate
MinReadySeconds:    0
Pod Template:
  Labels:  app=spring-petclinic-rest
  Containers:
   spring-petclinic-rest:
    Image:        docker.io/springcommunity/spring-petclinic-rest:3.2.1
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   spring-petclinic-rest-54f476f68 (4/4 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  6h2m  deployment-controller  Scaled up replica set spring-petclinic-rest-54f476f68 to 4
```

Pay attention to the difference in strategy.

After that we can check the rollout status by using kubectl rollout status deployments/spring-petclinic-rest
```
PS C:\Users\samue> kubectl rollout status deployments/spring-petclinic-rest
Waiting for deployment "spring-petclinic-rest" rollout to finish: 0 out of 4 new replicas have been updated...
Waiting for deployment "spring-petclinic-rest" rollout to finish: 0 of 4 updated replicas are available...
Waiting for deployment "spring-petclinic-rest" rollout to finish: 1 of 4 updated replicas are available...
Waiting for deployment "spring-petclinic-rest" rollout to finish: 2 of 4 updated replicas are available...
Waiting for deployment "spring-petclinic-rest" rollout to finish: 1 of 4 updated replicas are available...
Waiting for deployment "spring-petclinic-rest" rollout to finish: 0 of 4 updated replicas are available...
Waiting for deployment "spring-petclinic-rest" rollout to finish: 1 of 4 updated replicas are available...
Waiting for deployment "spring-petclinic-rest" rollout to finish: 2 of 4 updated replicas are available...
Waiting for deployment "spring-petclinic-rest" rollout to finish: 3 of 4 updated replicas are available...
deployment "spring-petclinic-rest" successfully rolled out
```

"The Recreate strategy, on the other hand, involves taking down the existing version of the application entirely before bringing up the new version. "
Since it started from zero, which you can try to rollout again that it will start from zero. Concluding the Recreate Deployment Strategy. Perchance

> 3. Prepare different manifest files for executing Recreate deployment strategy.

The files are like from the tutorial, listed in this project.
`kubectl get deployments/spring-petclinic-rest -o yaml > deployment-recreate.yaml` by using this command after we patch the recreate strategy.

> 4. What do you think are the benefits of using Kubernetes manifest files? Recall your experience in deploying the app manually and compare it to your experience when deploying the same app by applying the manifest files (i.e., invoking `kubectl apply -f` command) to the cluster.


Using Kubernetes manifest files offers several significant benefits over manually deploying applications. Here are the key advantages based on my understanding and experience:

**Benefits of Using Kubernetes Manifest Files**

**Consistency and Reproducibility:** Manifest files define the desired state of the application and its environment. By applying these files, Kubernetes ensures the actual state matches the desired state, which means deployments are consistent every time they are applied.

**Automation:** Using kubectl apply -f automates the deployment process. Kubernetes takes care of creating, updating, or deleting resources as needed to match the defined state in the manifest files.