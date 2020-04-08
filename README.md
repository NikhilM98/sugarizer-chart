# Local Sugarizer Chart

[Helm](https://helm.sh/) Chart for setting up [Sugarizer-Server](https://github.com/llaske/sugarizer-server) deployment on a local single node [Kubernetes](https://kubernetes.io/) cluster.

## Environment Setup
If you don't have a Local Kubernetes set-up, you can follow these steps to set-up a working development environment.

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
microk8s enable dns helm3 storage metallb

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


## Chart Installation

If you have Kubernetes already set-up, then Sugarizer Chart can be installed locally by following these steps:

### Clone Sugarizer Chart


    git clone https://github.com/NikhilM98/local-sugarizer-chart.git

### Edit Default Values

Open [values.yaml](https://github.com/NikhilM98/local-sugarizer-chart/blob/master/values.yaml) and edit the default values.

**schoolShortName:** Kubernetes [Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) for the Chart.

**hostName:** The host (domain name) of the server. Must be a valid subdomain as defined in [RFC 1123](https://tools.ietf.org/html/rfc1123). The host should be defined in `/etc/hosts` if you want to access the Sugarizer Server through your browser.

**storagePath:** The storage path for the database data. This path should exist before installing the chart.

**nodeName:** Name of the cluster node. Run `kubectl get nodes` to get the node name.

### Install Chart Using Helm
Go into chart directory and run:

    helm install <chart-name> .

Where `<chart-name>` is the name you want to give to this chart.

## Usage
You can deploy multiple Sugarizer Server instances by editing the values of the YAML file and running simple `helm install` command. The Sugarizer Server instance can be accessed by the browser by opening the hostName URL. 

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

## License
This project is licensed under `Apache v2` License. See [LICENSE](LICENSE) for full license text.

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
