# Services
https://kubernetes.io/docs/concepts/services-networking/service/

Think of services as being an layer in front of Pods. A service is a means to expose an application running on a set of pods.

The main benefit of defining a service that sits in front of your Pods is that, rather than having to send requests to a specific Pod,
you can send requests to a service which will forward it on to an available Pod. This means we don't have to know the IP address of the Pod, but rather just the name of the service.

![Kubernetes Service Diagram](images/kube-services.jpeg?raw=true "Kubernetes Services")

To view all the services running in your Kubernetes cluster you can run

```
master $ kubectl get services
    NAMESPACE     NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
    default       kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  8m18s
```

For example, let's say you have a pod or set of pods running that are labelled MyApp. Here is a simple Service definition defined in YAML that references these pods.
Essentially, you are saying that this service corresponds to the application running on those pods. Any query to this service, will be forwarded to these pods.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

What fo these fields mean exactly?

### selector
 - This is a label matcher, the service will find any pod with the label "app = MyApp" and act as the service for that pod
### protocol
- The network protocol to send requests over
### port
- This is the port to call the service on. So in this case, if you want to use "my-service" , you would send a request to ``my-service:80``
### targetPort
 - This is the port on which the application is running in a Pod

![Kubernetes Service Diagram](images/kube-services-ports.jpeg?raw=true "Kubernetes Services")

# DEMO!!

If you haven't already, create the deployment from the previous section

```
kubectl apply -f https://k8s.io/examples/application/deployment.yaml
```

Lets create a service which can be used to access the NginX deployment

```
kubectl apply -f resources/service-app/nodeport-service.yaml
```

So what happened?

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  selector:
    app: nginx
  type: NodePort
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 32333
```

Look closely at the port mappings, this is the important piece regarding a NodePort service
```
kubectl get services
```

Let's view the exposed service in Katacoda

We can also see that the service is exposed on the Node itself

```
curl 127.0.0.1:32333
```

Lets look at a different type of service - A ClusterIP service

We can create this kind of service in the kubernetes cluster by running

```kubectl apply -f resources/service-app/clusterip-service.yaml```

Again, what have we done?

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-clusterip-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

Lets list the services to show the difference between NodePort & ClusterIP

```
kubectl get services
```

Notice how the Port is not exposed externally, it just simply shows the port that container listens on for requests. The important piece here is that,
**only pods within the cluster can access this service**. Lets show this

```
kubectl apply -f resources/service-app/pod.yaml
kubectl exec -it exec-pod sh
```

Inside this pod, we can talk to the ClusterIP service

```
apk add curl

curl my-clusterip-service:80
```

![Kubernetes ClusterIp & Nodeport Services Diagram](images/servicesK8s.jpeg?raw=true "Kubernetes Services")


There are 4 types of Services in Kubernetes. They are
- ClusterIp
- NodePort
- LoadBalancer
- ExternalName


The two most common types are ClusterIp & NodePort

ClusterIp
- Exposes a service from **within** the kubernetes cluster only. This means that only requests from within the cluster can reach this service.

NodePort
- Exposes a service from **outside** the kubernetes cluster. For example, from your local machine, if there was connectivity, you could send requests to this
kind of service.

LoadBalancer
- Used to expose a service through a cloud provider's Load Balancer. Traffic from the external load balancer is directed at the backend Pods. Used within cloud providers
like GCP, Azure & AWS

ExternalName
- Used to map a service to a DNS name. This could be used for example to control the entrypoint to a Database. Lets take a look at a quick example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-externalname-service
spec:
  type: ExternalName
  externalName: google.com
```


```
kubectl apply -f resources/service-app/externalname-service.yaml
```

Let's login into a Pod to show this

```
kubectl exec -it exec-pod sh

curl my-externalname-service
```

## Next topic 
[From Git to Kubernetes exercise](5_git2kube.md)
