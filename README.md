# ud615 Notes

## Environments

* OS
  * Linux 4.4.0-47-generic
  * Ubuntu 16.04 LTS xenial
* hardware
  * Intel(R) Core(TM) i3-5010U @ 2.10GHz (`vmx` supported)
  * 8G memory
  * 4G ssd

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
   $ minikube start --vm-driver=kvm
   Starting local Kubernetes cluster...
   Downloading Minikube ISO
    36.00 MB / 36.00 MB [==============================================] 100.00% 0s
   Kubectl is now configured to use the cluster.
   ```
   and you'll see a host network would be created
   ```
   $ virsh net-list
    Name                 State      Autostart     Persistent
   ----------------------------------------------------------
    default              active     yes           yes
    docker-machines      active     yes           yes
   ```

## Test

1. Run a demo server to create a `deployment` and a `service`
   ```
   $ kubectl run hello-minikube --image=gcr.io/google_containers/echoserver:1.4 --port=8080
   deployment "hello-minikube" created
   $ kubectl expose deployment hello-minikube --type=NodePort
   service "hello-minikube" exposed
   ```

2. Check if pod is creating/running
   ```
   $ kubectl get pod
   NAME                              READY     STATUS    RESTARTS   AGE
   hello-minikube-3015430129-8tkkb   1/1       Running   1          14m
   ```

3. Test availability
   ```
   $ curl $(minikube service hello-minikube --url)
   CLIENT VALUES:
   client_address=172.17.0.1
   command=GET
   real path=/
   query=nil
   request_version=1.1
   request_uri=http://192.168.42.17:8080/

   SERVER VALUES:
   server_version=nginx: 1.10.0 - lua: 10001

   HEADERS RECEIVED:
   accept=*/*
   host=192.168.42.17:30166
   user-agent=curl/7.47.0
   BODY:
   -no body in request-
   ```                

4. Stop/Delete cluster to remove test data
   ```
   $ minikube stop
   ```
   or
   ```
   $ minikube delete
   ```
