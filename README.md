# Kubernetes Metrics Server

Metrics-server aggregates resource consumption data like CPU and memory usage for Kubernetes nodes, pods and containers. These metrics are collected from the API exposed by the Kubelet on each node.

The metrics server is commonly used by other Kubernetes add ons, such as the Horizontal Pod Autoscaler or the Kubernetes Dashboard. 

It is not deployed by default.

## Deployment
In order to deploy metrics-server in your kubernetes master machine clone https://github.com/p2pro-DevOps/metrics-server.git  and run the following command from
the top-level directory(metrics-server) of this repository:
 
```console
$ kubectl apply -f deploy/1.8+/
```

## Usage

```console
# Display node metrics
$ kubectl top nodes

# Display pod metrics
$ kubectl top pods
```
===Create Deployment=====

First, we will start a deployment running the image and expose it as a service:
https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/

kubectl apply -f https://k8s.io/examples/application/php-apache.yaml

==Create Auto Scaler===

We will create the autoscaler using kubectl autoscale. The following command will create a Horizontal Pod Autoscaler that maintains 
between 1 and 5 replicas of the Pods controlled by the php-apache deployment we created in the first step of these instructions. 

HPA will increase and decrease the number of replicas (via the deployment) to maintain an average CPU utilization 
across all Pods of 50% (since each pod requests 200 milli-cores by kubectl run, this means average CPU usage of 100 milli-cores).

kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=5

kubectl get hpa

====Increase load====

Now, we will see how the autoscaler reacts to increased load. We will start a container, and send an infinite loop of queries to the php-apache service (please run it in a different terminal):

kubectl run -i --tty load-generator --image=busybox /bin/sh


while true; do wget -q -O- http://php-apache.default.svc.cluster.local; done

once you load is reduced your pod will desacle:
-----------------------------------------------
ubuntu@ip-172-31-9-199:~/metrics-server/deploy/1.8+$ kubectl get pods
NAME                          READY   STATUS    RESTARTS        AGE
load-generator                1/1     Running   1 (3m50s ago)   9m3s
php-apache-8669477df6-6qjsz   1/1     Running   0               5m55s
php-apache-8669477df6-gbj88   1/1     Running   0               14m
php-apache-8669477df6-h6428   1/1     Running   0               5m55s
php-apache-8669477df6-jzstj   1/1     Running   0               5m55s
php-apache-8669477df6-t879j   1/1     Running   0               5m40s
ubuntu@ip-172-31-9-199:~/metrics-server/deploy/1.8+$
ubuntu@ip-172-31-9-199:~/metrics-server/deploy/1.8+$
ubuntu@ip-172-31-9-199:~/metrics-server/deploy/1.8+$
ubuntu@ip-172-31-9-199:~/metrics-server/deploy/1.8+$ kubectl get pods
NAME                          READY   STATUS    RESTARTS        AGE
load-generator                1/1     Running   1 (7m43s ago)   12m
php-apache-8669477df6-gbj88   1/1     Running   0               17m


## User guide

You can find the user guide in
[the official Kubernetes documentation](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-metrics-pipeline/).

## Design

The detailed design of the project can be found in the following docs:

- [Metrics API](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/instrumentation/resource-metrics-api.md)
- [Metrics Server](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/instrumentation/metrics-server.md)

- [Metrics Server Git Hub](https://github.com/kubernetes-sigs/metrics-server.git)

For the broader view of monitoring in Kubernetes take a look into
[Monitoring architecture](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/instrumentation/monitoring_architecture.md)
