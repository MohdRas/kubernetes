# kubernetes
# Node
- a server ( Physical or VM) on which k8s is installed.
- if this node failes, application will be down. so k8s cluster comes in picture.
- k8s cluster is a set of nodes grouped together.
- controlplane( master node) help to manages these working nodes.
- a cluster will have master & worker nodes together.
- master node
  - api server
     - acts as front end for k8s.
  - etcd
      - disributed key-value store.
  - controller-manager
      - brain for the orchestration of the nodes.
  - kube-scheduler
      - distributing across multiple nodes.
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
# kubectl
- kubectl get all
    - all objects
# Pod
- kubectl version
    - show version of "client" and "server" of kubectl
- kubectl --help
- kubectl get pods/nodes/replicatsets/deployments
- kubectl get pods/nodes/replicatsets/deployments -o wide
  - -o wide
    - to get more details.
- kubectl run my-pod --image=nginx
    - creating pod from nginx image
- pod-definition.yaml
    - apiVersion:v1---------------------------------------kubectl api-resources
    - kind:Pod--------------------------------------------kubectl api-resources
    - metadata:-------------------------------------------data about the object - pod
        - name: myapp-pod---------------------------------name of the pod
        - labels:
            - app: myapp----------------------------------group name like front-end, back-end, sales order service
            - type: front-end
   - spec:
      - containers:---------------------------------------List of containers
        - -name: nginx-container--------------------------first container
        - image: nginx
        - -name: nginx-container--------------------------second container
        - image: buxybox
- kubectl create -f pod-definition.yaml
- kubectl describe pods/nodes/replicasets/deployments NAME_OF_OBJECT
    - details about object.
- kubectl delete pods/nodes/replicasets/deployments NAME_OF_OBJECT
    - delete an object
# Replicaset
- group of 1 or more Pods
- spans across the cluster ( 1 or more worker nodes)
- Even if replicaset has 1 Pod and it fails. Replicaset automatically create another Pod in place of the failed one.
- Replicaset will always make sure that "desired" number of Pods are always up.
- replicaset-definition.yaml
    - apiVersion: apps/v1---------------------------------------kubectl api-resources
    - kind: ReplicaSet--------------------------------------------kubectl api-resources
    - metadata:-------------------------------------------data about the object - replicaset
        - name: myapp-replicaset---------------------------------name of the replicaset
        - labels:
            - app: myapp----------------------------------group name like front-end, back-end
            - type: front-end------------------------------label of Replicaset
   - spec:
      - template: --------------------------------------- Template of a Pod (metadata + spec) of a POd
          - metadata:----------------------------------------data about the object - pod
            - name: myapp-pod--------------------------------name of the pod
            - labels:
              - app: myapp-----------------------------------group name like front-end, back-end
              - type: front-end-------------------------------label of Pod
          - spec:
            - containers:---------------------------------------List of containers
              - -name: nginx-container--------------------------first container
              - image: nginx
              - -name: nginx-container--------------------------second container
              - image: buxybox
      - replicas: 3 ---------------------------------------3 PODS always ACTIVE all the time.
      - selectors:
          - matchLabels:
              - type: front-end---------Replicaset monitors only such Pods with same label as "front-end"
- kubectl create -f replicatset.yaml
- kubectl replace -f replicatset.yaml
    - To scal up as per the desired number of pods in file.
- kubectl scale --replicas=10 -f replicatset.yaml
    - To scale Pods upto 10.
- kubectl get replicasets
- kubectl describe replicasets myapp-replicaset
- kubectl delete replicasets myapp-replicaset
# deployments
- Pod(one instance) -> Replicaset (Multiple instances) -> deployments
- rolling(old version <-> new version) update of a production application.
- rolling updates, undo changes, pause & resume changes can be done by deployments.
- deployment-definition.yaml file is same as replicaset-definition.yaml except "kind:Deployment"
- commands are same as replicasets.


 
