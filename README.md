# kubernetes
# Node
- a server ( Physical or VM)
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
 
