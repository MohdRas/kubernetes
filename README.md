# kubernetes

# Tech with Nana - https://www.youtube.com/watch?v=X48VuDVv0do

# Kubernetes
- Open source container orchestration tool, developed by Google, manages containers.
- Manages containerized applications in different envs - **Physical, Virtual, Cloud or even Hybrid**
- Changes from **Monolith to Microservies** ===implied====> **increased use of containers** =======demands====> **proper way of managing 100s of containers.**

- what is advantage
    - High Availability or no downtime, always available to serve requests.
    - Scalability or High performance -  high response rate from the application.
    - Disater recovery - backup and restore.
    - 
# POD
- abstraction over a container. It is smallest unit in K8s.
- one **container** per POD. 
- **Each POD get its own IP address whenever it is created or re-created.** so it is **not a permanent IP**
- 2 PODs can communicate using IP addresses within a node.
- **multiple PODs** per Node.
- 
# SERVICE & Ingress
- A SERVICE is associated to a POD => one SERVICE per POD. SERVICE are also inside the node.
- **Permanent IP** is assigned to each SERVICE.
- **life cycle of SERVICE and POD are not connected.**. Whenever POD die and re-created then a new IP assigned to POD. But IP of SERVICE will not change.
- We need external SERVICE to access our application via browser.
- Ingress is an external SERVICE. Request go though **(ingress SERVICE)** to **(internal SERVICE)** to **(POD)** to **(Container)** to **(application inside container)**
- 
# Config map & Secrets - External configuration to the POD ( Application )
- if our application wants to connect with mongo-db SERVICE.
- external configuration of our application. like database URL is kept in this config map. such config cannnot be part of the image built else every timme we need to rebuild image & deploy again.
- if name of the DB SERVICE or endpoint changes, just need to update the config map.
- **username & password** is kept inside **secrets(base 64 encoded)**, this is another type of config map.
- both config map & secrets are **configured with the POD**. and can be used as **environment variables** or even as **properties file**.
- 
# data storage - volumes
- **kubernetes does not manage this data storage.** We need to explicity manage it.
- If the POD restarted, then data will be lost.
- **So we are attaching volumes to the POD**. It can be **local ( inside the node)** or **remote(outside the node)**
- 
# Replication of node
- In both REPLICASET and DEPLOYMENT , we mention **how many replicas** we wanna create. **No of replicas is equal to no of PODs**. This given number of PODs will always be guaranteed alive.
- REPLICASET
    - for **stateless applications.**
    - **blue print for PODs of my application**
    - it is just a manager for desired running POD count.
    - It does not allow us to change the code without downtime.
    - APP UPDATE scenario below - 
    - If POD template changed from version V1 to V2 (image name update) then it won't create new REPLICATSET for new POD template.
    - If we want POD with updated version V2 then we need to delete old PODs and then REPLICASET will use new PODs.

- DEPLOYMENT ( Default )
    - for **stateless applications.**
    - **blue print for PODs of my application**
    - POD is abstraction on the container. **DEPLOYMENT is another layer of abstraction over the POD.**
    - it is just a manager for REPLICASETs.
    - It allow us to change the code without downtime.
    - If you create a DEPLOYMENT then it automatically create a REPLICASET behind the scene.
    - Your DEPLOYMENT -> REPLICASET A -> ( POD1, POD2 POD3)
    - APP UPDATE scenario below - 
    - Image version V1 to V2 then it create a brand new REPLICASET B with image V2.
    - Once new POD (V2) started then it tells REPLICASET A to kill old POD(V1)..it does same thing for all other PODs.
    - After this activity, REPLICASET A has zero PODs and REPLICASET B has 3 POD(V2)
    - Switching between these REPLICASETs happen when image version changes to V2.
    - BY DEFAULT, K8S uses **RollingUpdate** strategy to unsure zero downtime.
    - ROLLING UPDATE         
        - depends on **maxSurge (how many extra PODs K8s can spin up)** during the update. Default to 25%.
        - depends on **maxUnavailable (how many PODs can be completely offline)** during the update. Default to 25%.
        - Lets say DEPLOYMENT was configured for 4 Replicas. Image version changes from V1 to V2. So as per default settings, then maxSurge = 1 POD and maxUnavailable = 1 POD
        - kubectl apply with image version V2.
        - create another REPLICASET B, initially desired replica count = 0.
        - DEPLOYMENT look for **maxSurge** and tells it to create a new POD ( V2 )
        - So total 5 PODS ( 4 running + 1 starting)
        - verifying health of new POD ( v2) and shifting traffic to it. **Once new POD(V2) is marked HEALTHY**
        - DEPLOYMENT tells REPLICASET A to kill old POD (V1).
        - Now total = 4 ( 3 running + 1 new POD )
        - REPLICASET A (Old) -> 4 -> 3 -> 2 -> 1 ->0  ( Inactive) 
        - REPLICASET B (New) -> 1 -> 2 -> 3 -> 4 ->4  ( Fully Aactive) 
        - Once REPLICASET B becomes full healthy and REPLICASET A has zero replicas then **switch is completed but REPLICASET A is not deleted**
        - It is kept alive with zero replicas. so we can instantly roll backed.

- SATEFULSET
    - for **statefull applications** ( databases -mysql , postgresql and distributed system - kafka & Elasticsearch)
    - ideally we don't do this because we keep **databases outside of the K8s cluster.**
    - PODs are created sequencially liek mysql-0, mysql-1 and so on...
    - **If mysql-1 dies then a new POD with same name mysql-1 will be created & existing PersistentVolume will be attached to this new POD.**
    
# Development Tools - minicube

- minukube is a one node cluster ( acts as master as well as worker node too).
- As it as a master & worker node so **all the components of master & worker node already be installed.**
- **Minikube create virtual box on my machine.**
- **Basically minikube is a one node cluster inside a virtual box**.
- kubectl interacts with k8s cluster.
- follow https://aistudio.google.com/prompts/1sBCsV5KUDyObaOtrX7EWHL7mMlYqF7OI
- install & run docker desktop - https://www.docker.com/products/docker-desktop/
- download and run minikube exe - https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download
- minikube start --driver=docker or minikube start
- minikube delete --all -> in case of conflict with given driver with th existing driver.
- Check the cluster information using: kubectl cluster-info
- Verify the Minikube status with: minikube status
- After success, we get message  = Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
  
# Development Tools - kubeadm
 - settings -> kubernetes -> enable kubernetes
 - select **kubeadm (Single Node Cluster)**. Apply & Restart.
 - **One SERVICE & one node will be already installed**.
 - kubectl get node/SERVICEs
   - docker-desktop, control-plane (master node)
   - kubernetes ,  ClusterIP
 - **kubelet** runs on **each node**
 - **kubectl** is the command-line tool for **interacting with the Kubernetes cluster**. 

# Node
- a server ( Physical or VM) on which k8s is installed.
- if this node failes, application will be down. so k8s cluster comes in picture.
- k8s cluster is a set of node grouped together.
- controlplane( master node) help to manages these working node.
- a cluster will have master & worker node together.
- 
- master node
  - **api server** - acts as front end for k8s.
  - **etcd** - disributed key-value store. All the state of a POD is stored here.
  - **controller-manager** - detects state changes in cluster ( PODs destroyed )
  - **kube-scheduler** - distributing PODs to be created across multiple node.
- master node
  - **kublet**
  - container runtime ( **containerd or CRI-O**)
  - **kubeproxy**
 
# Master Node - Control Plane
- api-server
- etdc
- control manager
- scheduler
- api-server
  - gatway to the cluster for users & other components.

# Worker node
  - kublet
  - kubeproxy
  - container runtime ( **containerd or CRI-O**)
    - Now, **the kubelet only communicates with runtimes that directly implement the CRI standard** [ The CRI-Compliant Era (Kubernetes 1.24+) ]
    - The two most popular CRI-compliant runtimes today are:
    - **containerd:**
       - high-level container runtime that Docker itself uses under the hood! When you install Docker, you are also installing containerd.
       - Kubernetes can now bypass the Docker daemon and talk directly to containerd.
    - **CRI-O:**
       - Container Runtime Interface
       - lightweight container runtime built specifically for Kubernetes. It implements the CRI and nothing more.
       - 
 - **KUBELET** ( Lister Doer Monitor Reporter Provider ) LDMRP
    - is the main agent on worker node of a k8s cluster.
        - listens for instructions from the control plane - **The Lister**.
        - does the work of running and managing containers - **The Doer**.
        - monitors their health - **The Docker - The Monitor**.
        - reports the node's status back - **The Reporter**
        - provides required persistent volumes to the POD. - **The Provider or The Supplier**
    
    - **PODs life cycle management** - The "Doer"
       - manages entire life cycle of the PODs of the node.
       - **recieves PODSpecs ( specifications for PODs ) from api-server and instruct container runtime to work on it.**
       - instruct container runtime (containerd or CRI-O)
           - to pull image
           - to create container from the image.
           - to stop & remove container only if the corresponding POD is deleted or terminated.
    
    - **Executing health probes & monitoring of the containers of the POD** - The "Doctor"
       - continuously running **health checks** defined within POD's specification
       - Liveniness probe - check if container still running.
       - Readiness probe - check if container ready to accept traffic.
       - Startup probe - check if containerized application is started or not.
       - helps to achieve Kubernetes' **self-healing and high-availability** features.
    
    - **Node & POD Status reporting** - The "Reporter"
       - constantly communicates with api-server.
       - share node status ( available memory, disk space, cpu capacity ) to api-server.
       - share POD status ( pending, running, succeeded & failed PODs) to api server.
       - it helps scheduler to function effectively. **Without accurate node status, the scheduler might place PODs on unhealthy or overloaded node**.
    
    - **Managing Volumes and Secrets** (The "Supplier")
       - Mounting Volumes:
          - For volumes like ConfigMaps, Secrets, or emptyDir, it mounts them into the container.
       - Persistent Storage:
          - The kubelet now knows its mission: **"I must make the storage described by this PV available to the container(s) in this POD."**
          - "POD needs storage" request-------->into-------> providing that storage to a container on its specific node.
          - **PersistentVolume (PV):** storage that exists on (e.g., an Amazon EBS volume, a Google Persistent Disk, or an NFS share).
          - **PersistentVolumeClaim (PVC):** A POD's request for storage. **PODSpec** ->>>> **volume** section ->>>>> points to -->>>>>a **PersistentVolumeClaim**
          - **Container Storage Interface (CSI) Driver:**
             - **The CSI driver typically runs on every worker node.** .
             - Kublet call this CSI driver to talk to a PV to provide required PVC.
             - Kubelet-->>> call CSI driver- -->>> to talk to ---->> PV ----->>> to provide ---->>> required PVC for the POD.
             - The volume is now mounted to **a temporary, kubelet-managed path on the node (Node's file system)**.
             - The kubelet instructs the container runtime (*containerd or CRI-O*).
             - It says: "Start this container, and when you do, take the directory **/var/lib/kubelet/.../mount** from the host node and make it appear inside the container at the path **/data.**
             - 
- **CONATINER RUNTIME ( dockerd or containerd)**
    - kubelet decides what to do, the container runtime is the one that does it.
    - Running and Managing Containers
    - Managing Images
    - Managing Container Storage and Networking
    - POD Sandbox Management
       - A POD is a group of one or more containers that share a network and storage environment. The runtime is responsible for creating this shared environment         - 
 
 - **NETWORK PROXY**
    - runs on each node.
    - defined network rules on the node.
    - handles SERVICE discovery, load balancing of the node.
    - **Kube-Proxy:**
       - SERVICE Discovery and Internal Load Balancing
          - This proxy resolves **SERVICE name to IP** of one of the healthy PODs.
          - This proxy load balances the traffic across all available PODs for that SERVICE.
    - **Ingress Controller:**
       - A specialized proxy (like NGINX, HAProxy, or Traefik) that manages external access to the cluster.
       - This role acts as the **"front door" to the cluster**, managing how external users and systems access your applications.
       - Manages external HTTP/S traffic, **routing it to internal SERVICEs.**
    - **SERVICE Mesh Sidecar:**
       - A proxy (like Envoy or Linkerd-proxy) that runs alongside each POD to manage inter-SERVICE communication.
       - to secure the traffic between SERVICEs inside the cluster.
       - This proxy intercepts all incoming and outgoing network traffic for that POD.
       - **Secure:**
          - Automatically encrypt all traffic between SERVICEs using mutual TLS (mTLS), establishing a **"zero-trust" network** where identity is verified for every request.
       - **Observe:** '
          - Generate detailed metrics (request rates, error rates, latencies), distributed traces, and access logs for all traffic without any changes to the application code.

# kubeconfig - "C:\Users\mohdr\.kube\config"

- **kubectl config view** - to view kubeconfig file
- **kubectl get --raw /api/v1/namespaces/default/PODs/nginx** - value in etdc database
- https://aistudio.google.com/app/prompts/1UdrMUn0yGZZqa46FN3rq75MaZEP-070X
- current context binds "user" and "cluster"
- client = kubectl, server = api-server
- kubectl is a command-line interface (CLI) client. Its sole purpose is to take your commands, format them into standard REST API calls, and send them to the Kubernetes API server specified in your kubeconfig file. It is the active "messenger."
- Docker Desktop is not the client in this interaction. Instead, Docker Desktop is the provider or host of the Kubernetes server. It's the application that runs all the control plane components (like the API server) in a virtual machine on your local machine.
- **apiVersion:** v1
- **kind:** Config
- **clusters:**
  - name: docker-desktop
  - cluster:
    - certificate-authority-data: DATA+OMITTED - **authority**
    - server: https://kubernetes.docker.internal:6443 - **verifyer**
- **contexts:**
  - name: docker-desktop
  - context:
    - cluster: docker-desktop
    - user: docker-desktop
- **current-context:** docker-desktop
- preferences: {}
- **users:**
  - name: docker-desktop
  - user:
    - client-certificate-data: DATA+OMITTED - **public key**
    - client-key-data: DATA+OMITTED - **private key**
    - 

# Kubernetes Kodekloud https://www.youtube.com/watch?v=XuSQU5Grv1g 
- With docker, we run one instance of an application.
- But with k8s, we can run thousands of instances in a single command.
  - "kubectl **run** --replicas=1000 my-web-server" =========> starting 1000 instances.
  - "kubectl **scale** --replicas=2000 my-web-server"========> scalling up to 2000 instances.
  - Scaling UP/DOWN the infrastrucrure/instances,can be done by configuring the k8s itself.
  - "kubectl **rolling-update** my-web-server --image=web-serer:2"
  - "kubectl **rolling-update** my-web-server --rollback"
- with K8s, we are able to define expected state of our application. This state is maintained even after any faliure to these instances.
  - webserver 2 instances.
  - payment SERVICE 2 instances.
  - redis SERVICE 3 instances.
  - database SERVICE 1 instance.
# kubectl commands

**kubectl [VERB] [NOUN] [NAME] [FLAGS]**

           
		   

              [VERB] What do you want to do? (Action) get, describe, create, apply, delete, logs, exec

              [NOUN] What resource type are you targeting? (Object) pod (po), service (svc), deployment (deploy), replicaset (rs)

              [NAME] Which specific one are you targeting? (Identifier) web-pod, db-service, api-deploy

              [FLAGS] What extra rules are you applying? (Modifiers) -n kube-system, -o yaml, --watch



**Built in shortcodes for almost every [NOUN]**


              

              pod ➡️ po



              service ➡️ svc



              deployment ➡️ deploy



              replicaset ➡️ rs



              statefulset ➡️ sts



              namespace ➡️ ns


				[NOUN] mandatory
				- can be used with many kinds of k8s objects [ node, pod, service, deployment, replicaset, statefulset ]


					- get => kubectl get pod <object_name>                            =========> pod is mandatory
					- describe => kubectl describe deployment <object_name>           =========> deployment is mandatory
					- edit => kubectl edit service <object_name>                      =========> service is mandatory
					- scale => kubectl scale replicaset <object_name>                 =========> replicaset is mandatory
					- create => kubectl create deployment <object_name>               =========> deployment is mandatory
					- delete => kubectl delete deployment <object_name>               =========> deployment is mandatory


				[NOUN] optional
				- can be used with ONLY ONE kinds of k8s objects [ pod]
				- derived from YAML file.


					- logs ========> kubectl logs <POD_NAME>                           ========> kubectl logs my-pod                                   ===========> logs of a POD
					- run  ========> kubectl run <POD_NAME> --image=nginx              ========> kubectl run my-pod --image=nginx                       ===============> run a POD.
					- exec ========> kubectl exec -it <POD_NAME> -- sh                 ========>  kubectl exec -it my-pod -- sh                         ===============> inside a POD.
					- port-forward ========> kubectl port-forward <POD_NAME> 8080:80   ========>        kubectl port-forward <POD_NAME> 8080:80          ======> port-forward of a POD.
					- apply ========> kubectl apply -f resource.yaml                   ========>                       object is mentioned in YAML file.






**Bucket A: "The Observers" (Checking State) 👀**


    

              List all of a resource:

              kubectl get po (Get all pod)



              See deep, detailed specifications & events (troubleshooting):

              kubectl describe po web-pod (Describe the web-pod)



              Show live streaming logs of a container:

              kubectl logs -f web-pod (Follow logs of web-pod)





**Bucket B: "The Creators & Destroyers" (Managing Life) 🛠️**



             

              Create or update from a local file:

              kubectl apply -f deployment.yaml (Apply a file)



              Quickly create an interactive resource without a file (Imperative):

              kubectl run temp-redis --image=redis (Run a quick redis pod)



              Safely delete a resource:

              kubectl delete deploy web-deploy (Delete a deployment)





**Bucket C: "The Interveners" (Debugging & Fixing) 🔧**


     

              SSH / Jump inside a running container (Interactive Terminal):

              kubectl exec -it web-pod -- /bin/sh



              Forward a port from the cluster to your local machine:

              kubectl port-forward svc/web-service 8080:80 (Access cluster service on local port 8080)


**Bucket C: "The Exceptions"**


     

              create a k8s object with a file :

              kubectl apply -f deployment.yaml

			
			
			  Create POD from an image :

			  kubectl run my-pod --image=nginx



              Find logs :

			  kubectl logs -f <object_name>


# K8s OBJECTS

kubectl **explain** pod
kubectl **explain** service
kubectl **explain** deployment
kubectl **explain** replicaset
kubectl **explain** statefulset

                    
                    KIND:       Pod
                    VERSION:    v1
                    
                    DESCRIPTION:
                        Pod is a collection of containers that can run on a host. This resource is
                        created by clients and scheduled onto hosts.
                    
                    FIELDS:
                      apiVersion    <string>
                        APIVersion defines the versioned schema of this representation of an object.
                        Servers should convert recognized schemas to the latest internal value, and
                        may reject unrecognized values. 
                       
                    
                      kind  <string>
                        Kind is a string value representing the REST resource this object
                        represents.  Cannot be updated. In CamelCase. 
                       
                    
                      metadata      <ObjectMeta>
                        Standard object's metadata. 
                       
                    
                      spec  <PodSpec>
                        Specification of the desired behavior of the pod. 
                        
                    
                      status        <PodStatus>
                        Most recently observed status of the pod. This data may not be up to date.
                        Populated by the system. Read-only. 
                        








    
# POD
- **kubectl version** - show version of "client" and "server" of kubectl
- kubectl --help        
- kubectl **run** my-nginx-pod **--image nginx**
   - Create and run a particular image in a pod.

# GET OBEJECTS
- kubectl **get pod**/node/service/deployment/replicaset/statefulset  ==============> get ALL
- kubectl **get pod**/node/service/deployment/replicaset/statefulset **<OBJECT_NAME>** ==============> get ONLY ONE
- kubectl **get pod**/node/replicatsets/deployment/service **-o wide**   =======>        ADDITIONAL DETAILS



# DESCRIBE OBEJECTS
- kubectl **describe pod**/node/service/deployment/replicaset/statefulset  ==============> describe ALL
- kubectl **describe pod**/node/service/deployment/replicaset/statefulset **<OBJECT_NAME>** ==============> describe ONLY ONE

# DELETE OBEJECTS
- **we cannot delete all pod at once. Same applies to other type of objects.**
- kubectl **delete pod**/node/service/deployment/replicaset/statefulset **<OBJECT_NAME>** ==============> delete ONLY ONE

# CREATE DEPLOYMENTS
- kubectl **create deployment** mydeploy --image nginx
- kubectl **create** -f POD-definition.yaml
- 
- **POD-definition.yaml**
 - spec:containers:---------------------------------------**List of containers with name & image**

 - 
    - apiVersion:v1
    - kind:**POD**--------------------------------------------type
    - **metadata:**
        - name: myapp-pod---------------------------------name of the POD
        - labels:
            - app: myapp
            - type: front-end ----------------------------------group name like front-end, back-end or sales-order.
   - **spec:**
      - **containers**:---------------------------------------**List of containers with name & image**
        - -name: nginx-container
        - image: nginx
        - -name: nginx-container
        - image: buxybox







    
# REPLICASET
- group of 1 or more PODs
- spans across the cluster ( 1 or more worker node)
- Even if REPLICASET has 1 POD and it fails. REPLICASET automatically create another POD in place of the failed one. If any new POD created manually and no of PODs becomes more than desirable count then it will reduce automatically.

- **REPLICASET will always make sure that "desired" number of PODs are always up, in case of increase or decrease PODs**

- **spec.selectors.matchLabels** of replicaset must match with **spec.template.labels**


- **REPLICASET-definition.yaml**

    - apiVersion: **apps/v1**
    - kind: **REPLICASET**--------------------------------------------type
    - **metadata:**
        - name: myapp-replicaset---------------------------------name of the REPLICASET
        - labels:
            - app: myapp----------------------------------group name like front-end, back-end
            - type: front-end------------------------------label of REPLICASET
   - **spec:**
      - **template**: --------------------------------------- Template of a POD (metadata + spec)
          - metadata:----------------------------------------Meta Data - POD
            - name: myapp-POD--------------------------------name of the POD
            - labels:
              - app: myapp-----------------------------------group name like front-end, back-end
              - type: **front-end**-------------------------------**label of POD**
          - spec:
            - containers:---------------------------------------List of containers
              - -name: nginx-container--------------------------first container
              - image: nginx
              - -name: nginx-container--------------------------second container
              - image: buxybox
      - **replicas**: 3 ---------------------------------------**3 PODS always ACTIVE all the time.**
      - **selectors**:
          - matchLabels:
              - type: **front-end**-----------------------------------**label of POD**

                


# DEPLOYMENT -  Stateless Applications
- POD(one instance) -> REPLICASET (Multiple instances) -> DEPLOYMENTs
- rolling(old version <-> new version) update of a production application.
- rolling updates, undo changes, pause & resume changes can be done by DEPLOYMENTs.
- **DEPLOYMENT-definition.yaml file is same as REPLICASET-definition.yaml except "kind:DEPLOYMENT"**
- commands are same as REPLICASETs.
- change the DEPLOYMENT file and run "kubectl apply -f DEPLOYMENT-definition.yaml"

- **DEPLOYMENT-definition.yaml**

    - apiVersion: **apps/v1**
    - kind: **DEPLOYMENT**--------------------------------------------type
    - **metadata:**
        - name: myapp-replicaset---------------------------------name of the DEPLOYMENT
        - labels:
            - app: myapp----------------------------------group name like front-end, back-end
            - type: front-end------------------------------label of DEPLOYMENT
   - **spec:**
      - **template**: --------------------------------------- Template of a POD (metadata + spec)
          - metadata:----------------------------------------Meta Data - POD
            - name: myapp-POD--------------------------------name of the POD
            - labels:
              - app: myapp-----------------------------------group name like front-end, back-end
              - type: **front-end**-------------------------------**label of POD**
          - spec:
            - containers:---------------------------------------List of containers
              - -name: nginx-container--------------------------first container
              - image: nginx
              - -name: nginx-container--------------------------second container
              - image: buxybox
      - **replicas**: 3 ---------------------------------------**3 PODS always ACTIVE all the time.**
      - **selectors**:
          - matchLabels:
              - type: **front-end**-----------------------------------**label of POD**


# SERVICEs
- IP of PODs is lost if it is started again.
- **Pods are temporary & their IP change constantly. A service act as a router that sends traffic to your pod by matching labels**
- 
- **Cluster IP SERVICE**
    - communication wetween PODs within cluster.
    - webserver tries to connect with redis server.
 	- **spec.selectors** of service is matched with **metadata.lables** of pod.
 	- It must match all **key-value pairs**.
    - 
    - **cluster-ip-SERVICE.yaml**
        - apiVersion:**v1**---------------------------------------version
        - kind: **SERVICE**--------------------------------------------type
        - metadata:-------------------------------------------Meta Data - SERVICE
            - name: redis-SERVICE---------------------------------name of the SERVICE
       - spec:
            - type: **ClusterIP**-----------------------------------------SERVICE Type
            - ports:
                - -targetPort: 6379--------------------------**POD port**
                - port: 6379---------------------------------**SERVICE port** - mandatory
            - **selectors**
                - app: myapp-------------------------------------**selection of PODs**
                - mame: redis-pod---------------------------------**selection of PODs**


                
- **NodePort SERVICE**
    - exposed ternally on node IP and its port.
    - Layer 4 (TCP/UDP)
    - Based on NodeIP:Port'
    - It opens a specific static port (e.g., 30080) on the IP address of every node in the cluster. Any traffic hitting [Any-Node-IP]:30080 is forwarded to your SERVICE.
    - nodeport-ip-SERVICE.yaml
        - apiVersion:v1---------------------------------------version
        - kind: SERVICE--------------------------------------------type
        - metadata:-------------------------------------------Meta Data - SERVICE
            - name: redis-SERVICE---------------------------------name of the SERVICE
       - spec:
            - type: **NodePort**-----------------------------------------SERVICE Type
            - ports:
                - -targetPort: 6379--------------------------**POD port**
                - port: 6379---------------------------------**SERVICE port** - mandatory
                - **nodePort**: **30008**-------------------------------------**Node port** (30000-32767)
            - selector:
                - app: myapp-------------------------------------selection of PODs
                - mame: redis-POD---------------------------------selection of PODs
- load balancer
    - cloud provider's load balancer to our SERVICE.
- kubectl create -f cluster-ip-SERVICE.yaml
- kubectl get SERVICEs
- **curl http://192.168.1.2:30008**
- it uses **Random algorithm , sessioninfinity=yes** for the purpose of **selection of PODs to forward request to**.
- **istio & LinkerD** is also used for load balancing.
- **one POD per node, multiple PODs per node, multiple PODs across multiple node, same SERVICE definition will work, no additional setup needed**.

 
