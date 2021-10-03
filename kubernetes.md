# Kubernetes

Kubernetes coordinates a highly available cluster of computers that are connected to work as a single unit. Kubernetes automates the distribution and scheduling of application containers across a cluster in a more efficient way.

A Kubernetes cluster consists of two types of resources:

- The **Control Plane** coordinates the cluster
- **Nodes** are the workers that run applications

![img](kubernetes_cluster.png)

**The Control Plane is responsible for managing the cluster.** The Control Plane coordinates all activities in your cluster, such as scheduling applications, maintaining applications' desired state, scaling applications, and rolling out new updates.

**A node is a VM or a physical computer that serves as a worker machine in a Kubernetes cluster.** Each node has a Kubelet, which is an agent for managing the node and communicating with the Kubernetes control plane. The node should also have tools for handling container operations, such as containerd or Docker. A Kubernetes cluster that handles production traffic should have a minimum of three nodes.

#### minikube tutorial command

```shell
minikube version
minikube start
kubectl version
kubectl cluster-info
kubectl get nodes
kubectl get nodes --help
```



Once the application instances are created, a Kubernetes Deployment Controller continuously monitors those instances. If the Node hosting an instance goes down or is deleted, the Deployment controller replaces the instance with an instance on another Node in the cluster. **This provides a self-healing mechanism to address machine failure or maintenance.**

![img](kubernetes_cluster_deploy.png)

You can create and manage a Deployment by using the Kubernetes command line interface, **Kubectl**. Kubectl uses the Kubernetes API to interact with the cluster. 

A Pod is the basic execution unit of a Kubernetes application. Each Pod represents a part of a workload that is running on your cluster.



```shell
kubectl create deployment
kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
kubectl get deployments

echo -e "\n\n\n\e[92mStarting Proxy. After starting it will not output a response. Please click the first Terminal Tab\n"; 
kubectl proxy
curl http://localhost:8001/version
export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
echo Name of the Pod: $POD_NAME
# You can access the Pod through the API by running:
curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME
```

## Kubernetes Pods

When you created a Deployment in Module 2, Kubernetes created a **Pod** to host your application instance. A Pod is a Kubernetes abstraction that represents a group of one or more application containers (such as Docker), and some shared resources for those containers. Those resources include:

- Shared storage, as Volumes
- Networking, as a unique cluster IP address
- Information about how to run each container, such as the container image version or specific ports to use

A Pod models an application-specific "logical host" and can contain different application containers which are relatively tightly coupled. 

Pods are the **atomic unit** on the Kubernetes platform. When we create a Deployment on Kubernetes, that Deployment creates Pods with containers inside them (as opposed to creating containers directly). Each Pod is tied to the Node where it is scheduled, and remains there until termination (according to restart policy) or deletion. In case of a Node failure, identical Pods are scheduled on other available Nodes in the cluster.

![img](pods-overview.png)

## Nodes

**A Pod always runs on a Node.** A Node is a worker machine in Kubernetes and may be **either a virtual or a physical machine**, depending on the cluster. Each Node is managed by the control plane. **A Node can have multiple pods**, and the Kubernetes control plane automatically handles **scheduling the pods across the Nodes** in the cluster. The control plane's automatic scheduling takes into account the available resources on each Node.

Every Kubernetes Node runs at least:

- **Kubelet**, a process responsible for communication between the Kubernetes control plane and the Node; it manages the Pods and the containers running on a machine.
- A **container runtime (like Docker)** responsible for pulling the container image from a registry, unpacking the container, and running the application.

![img](node-overview.png)

## Troubleshooting with kubectl

- **kubectl get** - list resources
- **kubectl describe** - show detailed information about a resource
- **kubectl logs** - print the logs from a container in a pod
- **kubectl exec** - execute a command on a container in a pod

You can use these commands to see when applications were deployed, what their current statuses are, where they are running and what their configurations are.

```shell
kubectl get pods
kubctl describe pods

echo -e "\n\n\n\e[92mStarting Proxy. After starting it will not output a response. Please click the first Terminal Tab\n"; 
kubectl proxy

export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
echo Name of the Pod:

curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/$POD_NAME

kubectl logs $POD_NAME
```

A node may have multiple pods. Each Pod in a Kubernetes cluster has a unique IP address. Although each Pod has a unique IP address, those IPs are not exposed outside the cluster without a Service.

ervices allow your applications to receive traffic. Services can be exposed in different ways by specifying a `type` in the ServiceSpec:

- *ClusterIP* (default) - Exposes the Service on an internal IP in the cluster. This type makes the Service only reachable from within the cluster.
- *NodePort* - Exposes the Service on the same port of each selected Node in the cluster using NAT. Makes a Service accessible from outside the cluster using `:`. Superset of ClusterIP.
- *LoadBalancer* - Creates an external load balancer in the current cloud (if supported) and assigns a fixed, external IP to the Service. Superset of NodePort.
- *ExternalName* - Maps the Service to the contents of the `externalName` field (e.g. `foo.bar.example.com`), by returning a `CNAME` record with its value. No proxying of any kind is set up. This type requires v1.7 or higher of `kube-dns`, or CoreDNS version 0.0.8 or higher.



### Services and Labels

A Service routes traffic across a set of Pods. Services are the abstraction that allow pods to die and replicate in Kubernetes without impacting your application. Discovery and routing among dependent Pods (such as the frontend and backend components in an application) is handled by Kubernetes Services.

Services match a set of Pods using labels and selectors, a grouping primitive that allows logical operation on objects in Kubernetes. Labels are key/value pairs attached to objects and can be used in any number of ways:

- Designate objects for development, test, and production
- Embed version tags
- Classify an object using tags

 ![img](kubernetes_label.png)

Labels can be attached to objects at creation time or later on. They can be modified at any time.

```shell
kubectl get pods
# NAME                                  READY   STATUS    RESTARTS   AGE
# kubernetes-bootcamp-fb5c67579-b5mp5   1/1     Running   0          3m32s

kubectl get services
# NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
# kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   5m59s

kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080
# service/kubernetes-bootcamp exposed
# NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
# kubernetes            ClusterIP   10.96.0.1      <none>        443/TCP          8m52s
# kubernetes-bootcamp   NodePort    10.110.20.61   <none>        8080:32766/TCP   82s

kubectl describe services/kubernetes-bootcamp
# Name:                     kubernetes-bootcamp
# Labels:                   app=kubernetes-bootcamp
# Selector:                 app=kubernetes-bootcamp
# Type:                     NodePort
# IP:                       10.110.20.61
# Port:                     <unset>  8080/TCP
# TargetPort:               8080/TCP
# NodePort:                 <unset>  32766/TCP
# Endpoints:                172.18.0.2:8080
# External Traffic Policy:  Cluster
# ... other information...
export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')

echo NODE_PORT=$NODE_PORT
# NODE_PORT=32766

curl $(minikube ip):$NODE_PORT
# Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-fb5c67579-b5mp5 | v=1
kubectl describe deployment
# Name:                   kubernetes-bootcamp
# Namespace:              default
# Labels:                 app=kubernetes-bootcamp
# Annotations:            deployment.kubernetes.io/revision: 1
# Selector:               app=kubernetes-bootcamp

kubectl get pods -l app=kubernetes-bootcamp
# NAME                                  READY   STATUS    RESTARTS   AGE
# kubernetes-bootcamp-fb5c67579-b5mp5   1/1     Running   0          21m

kubectl get services -l app=kubernetes-bootcamp
# NAME                  TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
# kubernetes-bootcamp   NodePort   10.110.20.61   <none>        8080:32766/TCP   14m

# To apply a new label we use the label command followed by the object type, object name and # the new label:
export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
echo Name of the Pod: $POD_NAME
kubectl label pods $POD_NAME version=v1

# To delete Services you can use the delete service command.
kubectl delete service -l app=kubernetes-bootcamp

# The application is up. This is because the Deployment is managing the application. To shut down the application, you would need to delete the Deployment as well.
```



## Running Multiple Instances of Your App

In the previous modules we created a Deployment, and then exposed it publicly via a [Service](https://kubernetes.io/docs/concepts/services-networking/service/). The Deployment created only one Pod for running our application. When traffic increases, we will need to scale the application to keep up with user demand.

**Scaling** is accomplished by changing the number of replicas in a Deployment

![img](kubernetes-scaling.png)

```shell
kubectl get deployments
# NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
# kubernetes-bootcamp   1/1     1            1           52s

kubectl get rs
# NAME                            DESIRED   CURRENT   READY   AGE
# kubernetes-bootcamp-fb5c67579   1         1         1       2m59

# let’s scale the Deployment to 4 replicas. We’ll use the kubectl scale command, followed by # the deployment type, name and desired number of instances:
kubectl scale deployments/kubernetes-bootcamp --replicas=4

kubectl get pods -o wide
# NAME                                  READY   STATUS    RESTARTS   AGE     IP
# NODE       NOMINATED NODE   READINESS GATES
# kubernetes-bootcamp-fb5c67579-cxgwz   1/1     Running   0          5m55s   172.18.0.8   
# minikube   <none>           <none>
# kubernetes-bootcamp-fb5c67579-grngg   1/1     Running   0          5m55s   172.18.0.9   
# minikube   <none>           <none>
# kubernetes-bootcamp-fb5c67579-wt766   1/1     Running   0          5m55s   172.18.0.7   
# minikube   <none>           <none>
# kubernetes-bootcamp-fb5c67579-zcnr5   1/1     Running   0          25m     172.18.0.2   
# minikube   <none>           <none>

# To scale down the Service to 2 replicas, run again the scale command:
kubectl scale deployments/kubernetes-bootcamp --replicas=2
```

## Performing a Rolling Update

### Updating an application

Users expect applications to be available all the time and developers are expected to deploy new versions of them several times a day. In Kubernetes this is done with rolling updates. **Rolling updates** allow Deployments' update to take place with zero downtime by incrementally updating Pods instances with new ones. The new Pods will be scheduled on Nodes with available resources.

Similar to application Scaling, if a Deployment is exposed publicly, the Service will load-balance the traffic only to available Pods during the update. An available Pod is an instance that is available to the users of the application.

Rolling updates allow the following actions:

- Promote an application from one environment to another (via container image updates)
- Rollback to previous versions
- Continuous Integration and Continuous Delivery of applications with zero downtime

```shell
kubectl get deployments
# NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
# kubernetes-bootcamp   0/4     0            0           11s
kubectl get pods
# NAME                                  READY   STATUS    RESTARTS   AGE
# kubernetes-bootcamp-fb5c67579-mdffx   1/1     Running   0          12s
# kubernetes-bootcamp-fb5c67579-mtb9x   1/1     Running   0          12s
# kubernetes-bootcamp-fb5c67579-qbf4p   1/1     Running   0          12s
# kubernetes-bootcamp-fb5c67579-tnc95   1/1     Running   0          12s

kubectl describe pods
# Controlled By:  ReplicaSet/kubernetes-bootcamp-fb5c67579
# Containers:
#  kubernetes-bootcamp:
#    Container ID:   docker://90ae604836899843452f01a632975cb45ed477d6e8ecb62c2e89c8ccb0a960a1
#    Image:          gcr.io/google-samples/kubernetes-bootcamp:v1
    
# To update the image of the application to version 2, use the set image command, followed by the deployment name and the new image version:
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
# deployment.apps/kubernetes-bootcamp image updated

kubectl describe pods
# Controlled By:  ReplicaSet/kubernetes-bootcamp-7d44784b7c
# Containers:
#  kubernetes-bootcamp:
#    Container ID:   docker://3f71842a218f20faa6bf8ca2098609ae2e1831a56e3e166b39ccccf62f72d93f
#    Image:          jocatalin/kubernetes-bootcamp:v2


# First, check that the app is running. To find the exposed IP and Port, run the describe service command:
kubectl describe services/kubernetes-bootcamp

# Create an environment variable called NODE_PORT that has the value of the Node port assigned:
export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
echo NODE_PORT=$NODE_PORT

# Next, do a curl to the the exposed IP and port:
curl $(minikube ip):$NODE_PORT

# Every time you run the curl command, you will hit a different Pod. Notice that all Pods are running the latest version (v2).

# To roll back the deployment to your last working version, use the rollout undo command:
kubectl rollout undo deployments/kubernetes-bootcamp

# The rollout undo command reverts the deployment to the previous known state (v2 of the image). Updates are versioned and you can revert to any previously known state of a deployment.
```

