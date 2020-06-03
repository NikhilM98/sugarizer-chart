# Sugarizer Chart for MicroK8s Kubernetes Cluster
[Helm](https://helm.sh/) Chart for setting up [Sugarizer-Server](https://github.com/llaske/sugarizer-server) deployment on a local [MicroK8s](https://microk8s.io/) cluster.

## Environment Setup
If you don't have a local [MicroK8s](https://microk8s.io/) cluster set-up, you can follow these steps to set-up a working development environment.

### Install MicroK8s
You can follow MicroK8s [documentation](https://microk8s.io/docs/) to install MicroK8s in your system.

MicroK8s can be installed by running these commands:
```bash
# Install MicroK8s
sudo snap install microk8s --classic --channel=1.18/stable

# Join the Group
# MicroK8s creates a group to enable seamless usage of commands which require admin privilege.
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
# You will also need to re-enter the session for the group update to take place.
su - $USER

# Check if MicroK8s is up and running (Optional)
microk8s status --wait-ready

# Enable add-ons
microk8s enable dns helm3 storage

# Create Aliases (Optional)
alias kubectl='microk8s kubectl'
alias helm='microk8s helm3'
```
You need to enable [metallb](https://metallb.universe.tf/) add-on which is an implementation of network load-balancers for bare metal clusters.
Before enabling metallb you need to find the Internal IP of your node. It can be obtained by running:
```
kubectl get nodes -o wide 
```
Then to enable metallb, run:
```
microk8s enable metallb
```
Metallb will ask for an IP address range. The IPs should be on the same network as the node Internal IP. If the Internal IP of your node is `10.55.2.114` then the IP address range can be set to `10.55.2.220-10.55.2.250`.


### Install NGINX Ingress Controller
NGINX Ingress Controller can be installed by running these commands:
```bash
# Add NGINX Helm repository.
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo update

# Install the chart with the release name my-release (my-release is the name that you choose).
helm install my-release nginx-stable/nginx-ingress
```
After installing the controller, edit the `/etc/hosts` to point the hosts (domain name) to the External IP of the NGINX Ingress Controller. You can find the External IP by running:
```bash
kubectl get service --all-namespaces
```
Edit the `/etc/hosts` file by running
```bash
sudo vi /etc/hosts
```
Add `<External IP> <Host>` pairs at the end of the file.
```bash
# 10.55.2.220 is the example External IP. It may be different in your case.
10.55.2.220 sugarizer.example.com
10.55.2.220 sugarizer2.example.com
```

### Install MongoDB-Replicaset
You can install MongoDB-Replicaset using [MongoDB-Replicaset](https://github.com/helm/charts/tree/master/stable/mongodb-replicaset) Helm Chart.  
You should use the values in [microk8s-enviornment/mongo-chart/values.yaml](mongo-chart/values.yaml) file as the values file.  
MongoDB-Replicaset can be installed by following these commands:
```bash
# Add Chart Repository
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm repo update

# Install the chart with the release name mymongodb (You can change the release name)
# mongo-chart/values.yaml is the location of the YAML file containing values.
# You should use microk8s-enviornment/mongo-chart/values.yaml as the values file.
helm install mymongodb -f mongo-chart/values.yaml stable/mongodb-replicaset
``` 

## Chart Installation
If you have Kubernetes set-up, then Sugarizer Chart can be installed locally by following these steps:

### Clone Sugarizer Chart
```bash
git clone https://github.com/NikhilM98/sugarizer-chart.git
```

### Edit Default Values
Open [values.yaml](sugarizer-chart/values.yaml) and edit the default values.

**schoolShortName:** Kubernetes [Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) for the Chart.

**hostName:** The host (domain name) of the server. Must be a valid subdomain as defined in [RFC 1123](https://tools.ietf.org/html/rfc1123). The host should be defined in `/etc/hosts` if you want to access the Sugarizer Server through your browser.

**databaseUrl:** The URL of the MongoDB database. If replicaset is used, it can be the name of your replicaset like `mymongodb` which maps to `mymongodb-mongodb-replicaset-0.mymongodb-mongodb-replicaset.default.svc.cluster.local:27017,mymongodb-mongodb-replicaset-1.mymongodb-mongodb-replicaset.default.svc.cluster.local:27017,mymongodb-mongodb-replicaset-2.mymongodb-mongodb-replicaset.default.svc.cluster.local:27017` in the .ini file or if a single database without replicaset is used, then it can be like `sugarizer-service-db-mymongodb.sugarizer-mymongodb.svc.cluster.local`.

**replicaset:** Boolean. Defines if databaseUrl is the URL of a replicaset or a single database. Set it to `true` if MongoDB replicaset chart is used. `false` if database is used without replicasets. 

### Install Chart Using Helm
Go into chart directory and run:
```bash
helm install <chart-name> microk8s-enviornment/sugarizer-chart/
```
Where `<chart-name>` is the name you want to give to this chart.

## Usage
You can deploy multiple Sugarizer Server instances by editing the values of the YAML file and running simple `helm install` command. The Sugarizer Server instance can be accessed by the browser by opening the hostName URL. 
