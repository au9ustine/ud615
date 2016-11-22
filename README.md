# ud615 Kubernetes

## Installation

1. Download and install `kubectl`
   ```
   $ cd ~/tmp
   $ wget -c -t 0 -O kubectl https://storage.googleapis.com/kubernetes-release/release/v1.3.0/bin/linux/amd64/kubectl
   $ chmod +x kubectl
   $ mv kubectl ~/bin/
   ```

2. Download and install `minikube`
   ```
   $ cd ~/tmp
   $ wget -c -t 0 -O minikube https://storage.googleapis.com/minikube/releases/v0.12.2/minikube-linux-amd64
   $ chmod +x minikube
   $ mv minikube ~/bin/
   ```

3. Install `kvm` and `docker-machine` with `kvm` support
   ```
   $ sudo apt-get update && sudo apt-get install -y kvm libvirt-bin qemu-kvm
   $ sudo usermod -aG libvirtd $(whoami)
   $ newgrp libvirtd
   ```
   And then check if `libvirt` network has been established
   ```
   $ virsh net-list
   Name                 State      Autostart     Persistent
   ----------------------------------------------------------
   default              active     yes           yes
   ```

4. Create Kubernetes cluster
   ```
   $ minikube start --vm-driver=kvm --kvm-network=default
   Starting local Kubernetes cluster...
   Kubectl is now configured to use the cluster.
   ```
