# kubernetes
# Node
- a server ( Physical or VM) on which k8s is installed.
- if this node failes, application will be down. so k8s cluster comes in picture.
- k8s cluster is a set of nodes grouped together.
- controlplane( master node) help to manages these working nodes.
- master node
  - api server
     - acts as front end for k8s.
  - etcd
  - controller-manager
  - kube-scheduler.
- worker node
  - kublet
  - kubeproxy
  - container runtime.
# Pod
- small unit in K8s.
- abstraction over a container.
- one container/application/IP per pod.
- new IP , every time a pod created.
- multiple Pods per node.
# service
- one service per Pod.
- each service will have one permanet IP.
- life cycle of service & pod are not connected.
- services are also inside the node.
- we need external service to access our application in the browser.
- ingress is an external service. Request go thought ingress service to internal service (application).
# config map & secrets
- my application wants to connect with mongo-db service.
- external configuration of our application.
- database URL is kept in this config map.
- if name of the service or endpoint changes, just need to update the config map.
- username & password is kept inside secrets(base 64 encoded), another type config map.
- both config map & secrets are configured with the application pod.
# data storage
- if the pod restarted, then data will be lost.
- attaching volumes (physical storage) to the pod. It can be local or remote(outside K8s cluster) w.r.t the node.

# Kubernetes https://www.youtube.com/watch?v=XuSQU5Grv1g
- With docker, we run one instance of an application.
- But with k8s, we can run thousands of instances in a single command.
  - "kubectl run --replicas=1000 my-web-server" =========> starting 1000 instances.
  - "kubectl scale --replicas=2000 my-web-server"========> scalling up to 2000 instances.
  - Scaling UP/DOWN the infrastrucrure/instances,can be done by configuring the k8s itself.
  - "kubectl rolling-update my-web-server --image=web-serer:2"
  - "kubectl rolling-update my-web-server --rollback"
- with K8s, we are able to define expected state of our application. This state is maintained even after any faliure to these instances.
  - webserver 2 instances.
  - payment service 2 instances.
  - redis service 3 instances.
  - database service 1 instance.

 
