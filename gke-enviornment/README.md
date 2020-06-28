# Sugarizer Chart for Google Kubernetes Engine (GKE) Cluster
[Helm](https://helm.sh/) Chart for setting up [Sugarizer-Server](https://github.com/llaske/sugarizer-server) deployment on a [GKE](https://cloud.google.com/kubernetes-engine) cluster.

## Environment Setup
If you don't have a [GKE](https://cloud.google.com/kubernetes-engine) cluster set-up, you can follow these steps to set-up a working environment.

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

### Install Kubernetes-Reflector (If Sugarizer-Chart will be used along with [Sugarizer School Portal Chart](https://github.com/nikhilm98/sugarizer-school-portal-chart/))
[Reflector](https://github.com/emberstack/kubernetes-reflector) is a Kubernetes addon designed to monitor changes to resources (secrets and configmaps) and reflect changes to mirror resources in the same or other namespaces. Reflector includes a cert-manager extension used to automatically annotate created secrets and allow reflection.    
You can install Reflector using its Helm Chart. It can be installed by following these commands:
```bash
# Add Chart Repository
helm repo add emberstack https://emberstack.github.io/helm-charts
helm repo update

# Install the chart with the release name reflector (You can change the release name)
helm upgrade --install reflector emberstack/reflector
```

### Install NGINX Ingress Controller
The [NGINX Ingress Controller](https://github.com/nginxinc/kubernetes-ingress/) provides an implementation of an Ingress controller for NGINX and NGINX Plus.

The Ingress is a Kubernetes resource that lets you configure an HTTP load balancer for applications running on Kubernetes, represented by one or more Services. Such a load balancer is necessary to deliver those applications to clients outside of the Kubernetes cluster.

Clone the chart repository:
```bash
git clone https://github.com/nginxinc/kubernetes-ingress/
cd deployments/helm-chart/
```
Open the `values.yaml` file and add these `customPorts` under `controller.service.customPorts`:
```bash
customPorts:
  - port: 8039
    targetPort: https
    protocol: TCP
    name: presence
```
Install the chart with the release name nginx-ingress (You can change the release name)
```bash
helm install nginx-ingress .
```

### Install Cert-Manager
[Cert-Manager](https://cert-manager.io/docs/) is a native Kubernetes certificate management controller. It can help with issuing certificates from a variety of sources, such as Let’s Encrypt, HashiCorp Vault, Venafi, a simple signing key pair, or self-signed.

We use Cert-Manager to issue HTTPS certificates to the Sugarizer School Portal Server and its Sugarizer-Server deployments.

You can refer to Cert-Manager [installation documentation](https://cert-manager.io/docs/installation/kubernetes/) or simply follow these commands to install Cert-Manager on the cluster:
```bash
# Create the namespace for cert-manager
kubectl create namespace cert-manager

# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io
helm repo update

# Cert-Manager requires a number of CRD resources to be installed into your cluster as part of installation.
# Install CRDs as part of the Helm release
helm install cert-manager jetstack/cert-manager --namespace cert-manager --version v0.15.1 --set installCRDs=true
```

### Create Service Account
You need to create a GCP service account key from the API & Services page. Save the service account key. It will be required in the values.yaml file while chart installation. It'll also be required if you set-up backup and restore using MGOB and intend to use gcloud bucket.

## Chart Installation
If you have Kubernetes set-up, then Sugarizer Chart can be installed locally by following these steps:

### Clone Sugarizer Chart
```bash
git clone https://github.com/NikhilM98/sugarizer-chart.git
```

### Edit Default Values
Open [values.yaml](sugarizer-chart/values.yaml) and edit the default values.

**schoolShortName:** Kubernetes [Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) and Subdomain for the chart. Must be a valid subdomain as defined in [RFC 1123](https://tools.ietf.org/html/rfc1123). Your chart will be available on `<schoolShortName>.yourdomain.com`.

**hostName:** The hostname from which Sugarizer Server will be accessible. Must be a valid domain/subdomain. Must be a valid hostname as defined in [RFC 1123](https://tools.ietf.org/html/rfc1123).

**databaseUrl:** The URL of the MongoDB database. If replicaset is used, it can be the name of your replicaset like `mymongodb` which maps to `mymongodb-mongodb-replicaset-0.mymongodb-mongodb-replicaset.default.svc.cluster.local:27017,mymongodb-mongodb-replicaset-1.mymongodb-mongodb-replicaset.default.svc.cluster.local:27017,mymongodb-mongodb-replicaset-2.mymongodb-mongodb-replicaset.default.svc.cluster.local:27017` in the .ini file or if a single database without replicaset is used, then it can be like `sugarizer-service-db-mymongodb.sugarizer-mymongodb.svc.cluster.local`.

**replicaset:** Boolean. Defines if databaseUrl is the URL of a replicaset or a single database. Set it to `true` if MongoDB replicaset chart is used. `false` if database is used without replicasets. 

**sspNamespace:** Namespace of the Sugarizer School Portal Server (if it is installed). If SSP namespace is used, then instead of issuing new certificates, existing SSP TLS Certificate will be used for HTTPS. This process is faster and it removes the cert-manager dependence.    
Note that Kubernetes Reflecter need to be installed if you intend to use SSP TLS Certificate.    
Set to none if you intend use Sugarizer Chart independent of Sugarizer School Portal. In this case, new TLS Certificate will be generated for the Sugarizer-Server deployment.

**gcpProjectId:** The Project ID of the project on Google Cloud Platform. Required only if sspNamespace is set to none.

**clouddns:** Your service account key in base64 format. Required only if sspNamespace is set to none.

### Install Chart Using Helm
Go into chart directory and run:
```bash
helm install <chart-name> gke-enviornment/sugarizer-chart/
```
Where `<chart-name>` is the name you want to give to this chart.

After roughly 2 minutes, the Sugarizer-Server instance will be available on `<schoolShortName>.yourdomain.com`.

## Usage
You can deploy multiple Sugarizer Server instances by editing the values of the YAML file and running simple `helm install` command. The Sugarizer Server instance can be accessed by the browser by opening the hostName URL.
