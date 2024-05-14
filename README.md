Nama: Samuel Taniel Mulyadi
NPM : 2206081805

### Reflection on Hello Minikube

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

Default Namespace: When you run kubectl get without specifying a namespace, it defaults to the default namespace.
Specified Namespace: Using the -n option allows you to specify a particular namespace from which to retrieve resources. For example, kubectl get pods, services -n kube-system from the tutorial retrieves pods from the kube-system namespace.

