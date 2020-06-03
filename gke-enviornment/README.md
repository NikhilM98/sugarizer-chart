# Sugarizer Chart for Google Kubernetes Engine (GKE) Cluster
[Helm](https://helm.sh/) Chart for setting up [Sugarizer-Server](https://github.com/llaske/sugarizer-server) deployment on a [GKE](https://cloud.google.com/kubernetes-engine) cluster.

## Environment Setup
If you don't have a [GKE](https://cloud.google.com/kubernetes-engine) cluster set-up, you can follow these steps to set-up a working environment.

### Set-up ExternalDNS
Sugarizer Chart for Google Kubernetes Engine depends on [External-DNS](https://github.com/kubernetes-sigs/external-dns) to configure external DNS servers for Kubernetes Ingresses and Services.
You can set up ExternalDNS by following this [documentation](https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/gke.md).


### Install MongoDB-Replicaset
You can install MongoDB-Replicaset using [MongoDB-Replicaset](https://github.com/helm/charts/tree/master/stable/mongodb-replicaset) Helm Chart.  
You should use the values in [gke-enviornment/mongo-chart/values.yaml](mongo-chart/values.yaml) file as the values file.  
MongoDB-Replicaset can be installed by following these commands:
```bash
# Add Chart Repository
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm repo update

# Install the chart with the release name mymongodb (You can change the release name)
# mongo-chart/values.yaml is the location of the YAML file containing values.
# You should use gke-enviornment/mongo-chart/values.yaml as the values file.
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

**schoolShortName:** Kubernetes [Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) and Subdomain for the chart. Must be a valid subdomain as defined in [RFC 1123](https://tools.ietf.org/html/rfc1123). Your chart will be available on `<schoolShortName>.yourdomain.com`.

**databaseUrl:** The URL of the MongoDB database. If replicaset is used, it can be the name of your replicaset like `mymongodb` which maps to `mymongodb-mongodb-replicaset-0.mymongodb-mongodb-replicaset.default.svc.cluster.local:27017,mymongodb-mongodb-replicaset-1.mymongodb-mongodb-replicaset.default.svc.cluster.local:27017,mymongodb-mongodb-replicaset-2.mymongodb-mongodb-replicaset.default.svc.cluster.local:27017` in the .ini file or if a single database without replicaset is used, then it can be like `sugarizer-service-db-mymongodb.sugarizer-mymongodb.svc.cluster.local`.

**replicaset:** Boolean. Defines if databaseUrl is the URL of a replicaset or a single database. Set it to `true` if MongoDB replicaset chart is used. `false` if database is used without replicasets. 

### Install Chart Using Helm
Go into chart directory and run:
```bash
helm install <chart-name> gke-enviornment/sugarizer-chart/
```
Where `<chart-name>` is the name you want to give to this chart.

After roughly 2 minutes, the Sugarizer-Server instance will be available on `<schoolShortName>.yourdomain.com`.

## Usage
You can deploy multiple Sugarizer Server instances by editing the values of the YAML file and running simple `helm install` command. The Sugarizer Server instance can be accessed by the browser by opening the hostName URL. 
