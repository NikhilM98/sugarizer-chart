# Sugarizer Chart
[Helm](https://helm.sh/) Chart for setting up [Sugarizer-Server](https://github.com/llaske/sugarizer-server) deployment on a [Kubernetes](https://kubernetes.io/) cluster.

## Provider Support
Sugarizer Chart supports two providers:
- [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine) (GKE) - [README](gke-enviornment/README.md)
- [Microk8s](https://microk8s.io) (It basically provides a bare-metal Kubernetes cluster) - [README](microk8s-enviornment/README.md)

Each provider has its folder in this repository. You can follow the steps in the README.md inside the folder to set-up a working environment for Sugarizer Chart.

## Usage
You can deploy multiple Sugarizer Server instances by editing the values of the YAML file and running simple `helm install` command. The Sugarizer Server instance can be accessed by the browser by opening the hostName URL. 

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

## License
This project is licensed under `Apache v2` License. See [LICENSE](LICENSE) for full license text.

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
