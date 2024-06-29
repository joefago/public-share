## 4-1 Architecture
### Virtual Machines

```mermaid
graph LR
subgraph proxy ["Proxy / Load Balancer <br> Gateway / DMZ"]
  direction LR
  pxyExtIf[Ext]
  lb("nginx")
  pxyIntIf[Int]
end
subgraph db ["Database"]
  Oracle[("Oracle")]
end
subgraph mgmt ["Management"]
  k8cp2("rke2-server")
  cockpit("Cockpit")
end
subgraph log ["Log"]
  k8cp3("rke2-server")
end
subgraph k8m ["K8s"]
  k8cp1("rke2-server")
end
subgraph app ["Application"]
  jboss("App Server")
end

subgraph Users
 user(("App"))
 mgr(("Mgr"))
end

user----pxyExtIf
mgr:::red----pxyExtIf
pxyExtIf---lb---pxyIntIf
pxyIntIf----jboss:::green
pxyIntIf----k8cp1
pxyIntIf----k8cp2
pxyIntIf----k8cp3
pxyIntIf----cockpit:::red
jboss---Oracle:::green

linkStyle 0,4,9 stroke:green,color:green;
linkStyle 1,8 stroke:red,color:red;
linkStyle 5,6,7 stroke:blue,color:blue;

classDef txt stroke-width:0,fill:none
classDef blue stroke:blue
classDef red stroke:red
classDef green stroke:green
classDef dmz fill:#5e5e5e
classDef transparent fill:none,stroke-width:0

class proxy dmz
class Users transparent
class log,k8m internalNetwork
class pxy txt
class user green
class k8cp1,k8cp2,k8cp3 blue
class cockpit red
```

## Functional Components

### External Load Balancer
The external load balancer is how end users access internal applications and services. It is the sole entrypoint to the system and isolates the internal network from the external. It provides SSL termination, exposes UIs to end users as well as APIs to browsers and external integration services.

### Application Server
The application server runs web applications in servlet containers.

### Database
The database provides persistence for web applications and container services. 

### Kubernetes Cluster
The kubernetes cluster consists of three control-plane/worker nodes and provides access to container workloads

```mermaid
flowchart LR
user(("User"))
subgraph elb ["External Load Balancer"]
  elb-grafana(https://proxy-server/grafana)
end
subgraph k8s ["&nbsp;"]
  subgraph k81-node ["Node 1"]
    k81-grafana-np["NodePort<br>33000"]
    k81-grafana-pod("Grafana Pod 1")
    k81-postgres-pod("Postgres Pod")
  end
  subgraph k82-node ["Node 2"]
    k82-grafana-np["NodePort<br>33000"]
    k82-grafana-pod("Grafana Pod 2")
    k82-postgres-pod("Postgres Pod")
  end
  subgraph k83-node ["Node 3"]
    k83-grafana-np["NodePort<br>33000"]
    k83-grafana-pod("Grafana Pod 3")
    k83-postgres-pod("Postgres Pod")
  end
  subgraph grafana-svc ["Grafana Service"]
    grafana-svc-path["grafana:3000"]
  end
  subgraph postgres-svc ["Postgres Service"]
    postgres-svc-path["postgres:5432"]
  end
end

user---elb-grafana
elb-grafana---k81-grafana-np
elb-grafana-.-k82-grafana-np
elb-grafana-.-k83-grafana-np
k81-grafana-np---grafana-svc
k82-grafana-np-.-grafana-svc
k83-grafana-np-.-grafana-svc
grafana-svc---k83-grafana-pod
grafana-svc-.-k82-grafana-pod
grafana-svc-.-k81-grafana-pod
k83-grafana-pod---postgres-svc---k82-postgres-pod
k81-grafana-pod-.-postgres-svc
k82-grafana-pod-.-postgres-svc
postgres-svc-.-k81-postgres-pod
postgres-svc-.-k83-postgres-pod

linkStyle 0,1,4,7,10,11 stroke:blue
linkStyle 10,11 stroke:green

classDef dmz fill:#5e5e5e
classDef txt stroke-width:0,fill:none
classDef inactive fill:none,stroke-dasharray:3 3
class elb dmz
class grafana-svc-path,elb-grafana,postgres-svc-path txt
class k81-postgres-pod,k83-postgres-pod inactive
```
