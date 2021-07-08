# Sugarizer Chart
[Helm](https://helm.sh/) Chart for setting up [Sugarizer-Server](https://github.com/llaske/sugarizer-server) deployment on a [Kubernetes](https://kubernetes.io/) cluster.

## Usage
You can deploy multiple Sugarizer Server instances by editing the values of the YAML file and running simple `helm install` command. The Sugarizer Server instance can be accessed by the browser by opening the `hostName` URL. 

## Provider Support
Sugarizer Chart supports four providers:
- [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/) (Amazon EKS)
- [Azure Kubernetes Service](https://azure.microsoft.com/en-in/services/kubernetes-service/) (AKS)
- [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine) (GKE)
- [Microk8s](https://microk8s.io) (It basically provides a bare-metal Kubernetes cluster)

**Note:** You can use Sugarizer Chart as an independent deployment or use it with [Sugarizer School Portal Chart](https://github.com/nikhilm98/sugarizer-school-portal-chart/) to issue dynamic deployments.
- If you want to use Sugarizer Chart with Sugarizer School Portal Chart then set `sspNamespace` as the namespace of Sugarizer School Portal Chart. In this case, same TLS Certificate for HTTPS will be used as in Sugarizer School Portal Deployment. You don't need to configure `cluster` in the `values.yaml` file in this case.
- If you set `https` to `false` in `values.yaml` then TLS Certificate will not be generated. You don't need to configure `cluster` in the `values.yaml` file in this case.
- If you want to use Sugarizer Chart independent of Sugarizer School Portal Chart and want HTTPS then you need to set up Cloud DNS on your cloud provider and enter its  configuration in the `values.yaml` file. Currently, you can not enable HTTPS for deployment running on [Microk8s](https://microk8s.io).


## Environment Setup
If you don't have a Kubernetes cluster set-up, you can follow these steps to set-up a working environment.

### Install Microk8s
**Note:** Not required if you have access to Amazon EKS, AKS or GKE.

You can follow MicroK8s [documentation](https://microk8s.io/docs/) to install MicroK8s in your system. MicroK8s can be installed by running these commands:
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

### Install MongoDB-Replicaset
**Note:** Do not install if already installed during [Sugarizer School Portal Chart](https://github.com/nikhilm98/sugarizer-school-portal-chart/) setup.

You can install MongoDB-Replicaset using [Bitnami/MongoDB](https://github.com/bitnami/charts/tree/master/bitnami/mongodb) Helm Chart.
You should use the values in the [mongodb](mongodb/) directory as the values file.  
MongoDB-Replicaset can be installed by following these commands:
```bash
# Add Chart Repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```
For AKS or GKE, run following command to install MongoDB-Replicaset:
```bash
# mongodb/default-values.yaml is the location of the YAML file containing configuration for Amazon EKS, GKE and AKS.
helm install ssp-mongodb -f mongodb/default-values.yaml bitnami/mongodb
```
For Microk8s, run following command to install MongoDB-Replicaset:
```bash
# mongodb/default-values.yaml is the location of the YAML file containing values for Microk8s.
helm install ssp-mongodb -f mongodb/microk8s-values.yaml bitnami/mongodb
```

### Install Kubernetes-Reflector
**Note-1:** Only required if Sugarizer Chart will be used along with [Sugarizer School Portal Chart](https://github.com/nikhilm98/sugarizer-school-portal-chart/) on HTTPS. Set `sspNamespace` as the namespace of Sugarizer School Portal Chart. Do not install if already installed during [Sugarizer School Portal Chart](https://github.com/nikhilm98/sugarizer-school-portal-chart/) setup.

**Note-2:** Not required if you want to use Sugarizer Chart independent of [Sugarizer School Portal Chart](https://github.com/nikhilm98/sugarizer-school-portal-chart/).

**Note-3:** Not required if you do not want HTTPS support. Currently, Sugarizer Chart for Microk8s does not supports HTTPS.

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
**Note:** Do not install if already installed during [Sugarizer School Portal Chart](https://github.com/nikhilm98/sugarizer-school-portal-chart/) setup.

The [NGINX Ingress Controller](https://github.com/nginxinc/kubernetes-ingress/) provides an implementation of an Ingress controller for NGINX and NGINX Plus.

The Ingress is a Kubernetes resource that lets you configure an HTTP load balancer for applications running on Kubernetes, represented by one or more Services. Such a load balancer is necessary to deliver those applications to clients outside of the Kubernetes cluster.

Clone the chart repository:
```bash
git clone https://github.com/nginxinc/kubernetes-ingress/
cd deployments/helm-chart/
```
Open the `values.yaml` file and add these `customPorts` under `controller.service.customPorts`:
```bash
# For HTTP only support (AKS/EKS/GKE/Microk8s)
customPorts:
  - port: 8039
    targetPort: http
    protocol: TCP
    name: presence
```
Or
```bash
# If you want to have HTTPS support (AKS/EKS/GKE + Cloud DNS)
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
After installing the controller, point the hosts (domain name) to the External IP of the NGINX Ingress Controller. You can find the External IP by running:
```bash
kubectl get service --all-namespaces
```
- For **AKS/EKS/GKE**, point the `A` record of domain to the External IP of the controller.
- For **Microk8s**, edit the `/etc/hosts` file by running
  ```bash
  sudo vi /etc/hosts
  ```
  Add `<External IP> <Host>` pairs at the end of the file.
  ```bash
  # 10.55.2.220 is the example External IP. It may be different in your case.
  10.55.2.220 sugarizer.example.com
  10.55.2.220 sugarizer2.example.com
  ```

### Install Cert-Manager
**Note-1:** Required if you want to use Sugarizer Chart independent of [Sugarizer School Portal Chart](https://github.com/nikhilm98/sugarizer-school-portal-chart/) and want HTTPS. You also need to set up Cloud DNS on your cloud provider and enter its  configuration in the `values.yaml` file.

**Note-2:** Do not install if already installed during [Sugarizer School Portal Chart](https://github.com/nikhilm98/sugarizer-school-portal-chart/) setup.

**Note-3:** Not required if you do not want HTTPS support. Currently, Sugarizer Chart for Microk8s does not supports HTTPS.

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
**Note-1:** Not required if you already have it configured for [Sugarizer School Portal Chart](https://github.com/nikhilm98/sugarizer-school-portal-chart/). Set `sspNamespace` as the namespace of Sugarizer School Portal Chart.

**Note-2:** Not required if you do not want HTTPS support. Currently, Sugarizer Chart for Microk8s does not supports HTTPS.

**Instructions:**
- **AKS:**
You need to create a Service Principal for Azure. You can follow these [instructions](https://cert-manager.io/docs/configuration/acme/dns01/azuredns/) to create a Service Principal for Azure.

- **EKS:**
You need to use Amazon Route53 as Cloud DNS to allow certificate verification. You can follow these [instructions](https://cert-manager.io/docs/configuration/acme/dns01/route53/) to configure Route53 for AWS. 

- **GKE:**
You can create a Service Account for GCP by following these [instructions](https://cert-manager.io/docs/configuration/acme/dns01/google/).
Save the service account key. It will be required in the values.yaml file while chart installation. It'll also be required if you set-up backup and restore using MGOB and intend to use gcloud bucket.

## Chart Installation
If you have Kubernetes set-up, then Sugarizer Chart can be installed by following these steps:

### Clone Sugarizer Chart
```bash
git clone https://github.com/NikhilM98/sugarizer-chart.git
```

### Edit Default Values
Open [values.yaml](values.yaml) and edit the default values.

**[schoolShortName] -**
Kubernetes [Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) for the chart.

**[hostName] -**
The host (domain/subdomain) name from which Sugarizer Server will be accessible. Must be a valid hostname as defined in [RFC 1123](https://tools.ietf.org/html/rfc1123).
- For **AKS**, **EKS** or **GKE**, the host should point towards the External IP of the NGINX Ingress controller.
- For **Microk8s**, the host should be defined in `/etc/hosts` if you want to access the Sugarizer Server through your browser.

**[deployment]**
- **sspNamespace:** Namespace of the Sugarizer School Portal Server (if it is installed). If SSP namespace is used, then instead of issuing new certificates, existing SSP TLS Certificate will be used for HTTPS. This process is faster and it removes the cert-manager dependence.    
Note that Kubernetes Reflecter need to be installed if you intend to use SSP TLS Certificate.    
Set to `none` if you intend use Sugarizer Chart independent of Sugarizer School Portal. In this case, new TLS Certificate will be generated for the Sugarizer-Server deployment.

- **https:** Boolean. Set it to `true` to enable HTTPS support.

- **production:** Boolean. Use to switch between letsencrypt Staging and Production server. Set it to `true` to switch to Production server.

**[database]**
- **databaseUrl:** The URL of the MongoDB database. If replicaset is used, it can be the name of your replicaset like `mymongodb` which maps to `mymongodb-mongodb-replicaset-0.mymongodb-mongodb-replicaset.default.svc.cluster.local:27017,mymongodb-mongodb-replicaset-1.mymongodb-mongodb-replicaset.default.svc.cluster.local:27017,mymongodb-mongodb-replicaset-2.mymongodb-mongodb-replicaset.default.svc.cluster.local:27017` in the .ini file or if a single database without replicaset is used, then it can be like `sugarizer-service-db-mymongodb.sugarizer-mymongodb.svc.cluster.local`.

- **replicaset:** Boolean. Defines if databaseUrl is the URL of a replicaset or a single database. Set it to `true` if MongoDB replicaset chart is used. `false` if database is used without replicasets. 

**[cluster]**: Not required if HTTPS is `false`. Required only if you plan to use Sugarizer Chart independent of Sugarizer School Portal (`sspNamespace` is set to `none`). 
- **provider:** The provider on which cluster is hosted on. Options: `gke`, `aws`, `azure`, `microk8s`.

*If the provider is `gke`:*
- **gcpProjectId:** The Project ID of the project on Google Cloud Platform.
- **gcpServiceAccount:** Your GCP Service Account key in base64 format.

*If the provider is `azure`:*
- **azureClientSecret:** Your Azure Service Principal Password in plain text format.
- **azureSPAppId:** Your Azure Service Principal App ID.
- **azureSubscriptionId:** The Subscription ID of your Azure Cloud Platform Account.
- **azureTenantId:** The Tenant ID for your Azure Service Principal.
- **azureDnsZoneResourceGroup:** The Resource Group that you have your DNZ Zone in.
- **azureDnsZone:** The name of your Azure DNS Zone.

*If the provider is `aws`:*
- **awsClientSecret:**  Your AWS Secret Access Key in plain text format.
- **awsRegion:**  The region on which your DNS Zone is hosted on.
- **awsAccessKeyId:** Your AWS Access Key ID.
- **awsDnsZone:** The name of your AWS DNS Zone.
- **awsRole:** (Optional Dependency) The Role attached to your account.

### Install Chart Using Helm
Go into chart directory and run:
```bash
helm install <chart-name> .
```
Where `<chart-name>` is the name you want to give to this chart.

After roughly 2 minutes, the Sugarizer-Server instance will be available on the `hostName` URL.

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

## License
This project is licensed under `Apache v2` License. See [LICENSE](LICENSE) for full license text.

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
