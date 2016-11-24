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

## Pods Manipulation

1. Create
   ```
   $ kubectl create -f pods/monolith.yaml
   pod "monolith" created
   ```

2. Describe
   ```
   $ kubectl get pods
   NAME                              READY     STATUS    RESTARTS   AGE
   hello-minikube-3015430129-8tkkb   1/1       Running   2          23h
   monolith                          1/1       Running   0          22s
   ```
   ```
   $ kubectl describe pods monolith
   Name:		monolith
   Namespace:	default
   Node:		minikube/192.168.42.17
   Start Time:	Wed, 23 Nov 2016 21:18:49 +0800
   Labels:		app=monolith
   Status:		Running
   IP:		172.17.0.5
   Controllers:	<none>
   Containers:
     monolith:
       Container ID:	docker://ed6ccd4a3abf8008365bce752f94d96b787e31cba6e1931848457a68ae8bb048
       Image:		kelseyhightower/monolith:1.0.0
       Image ID:		docker://sha256:e7dd4c589e813abab743794bc7dd468ca9812c467b7daefdd3b25def491cf0ba
       Ports:		80/TCP, 81/TCP
       Args:
         -http=0.0.0.0:80
         -health=0.0.0.0:81
         -secret=secret
       Limits:
         cpu:	200m
         memory:	10Mi
       Requests:
         cpu:			200m
         memory:			10Mi
       State:			Running
         Started:			Wed, 23 Nov 2016 21:19:02 +0800
       Ready:			True
       Restart Count:		0
       Environment Variables:	<none>
   Conditions:
     Type		Status
     Initialized 	True
     Ready 	True
     PodScheduled 	True
   Volumes:
     default-token-t24dn:
       Type:	Secret (a volume populated by a Secret)
       SecretName:	default-token-t24dn
   QoS Tier:	Guaranteed
   Events:
     FirstSeen	LastSeen	Count	From			SubobjectPath			Type		Reason		Message
     ---------	--------	-----	----			-------------			--------	------		-------
     49s		49s		1	{default-scheduler }					Normal		Scheduled	Successfully assigned monolith to minikube
     48s		48s		1	{kubelet minikube}	spec.containers{monolith}	Normal		Pulling		pulling image "kelseyhightower/monolith:1.0.0"
     37s		37s		1	{kubelet minikube}	spec.containers{monolith}	Normal		Pulled		Successfully pulled image "kelseyhightower/monolith:1.0.0"
     37s		37s		1	{kubelet minikube}	spec.containers{monolith}	Normal		Created		Created container with docker id ed6ccd4a3abf; Security:[seccomp=unconfined]
     36s		36s		1	{kubelet minikube}	spec.containers{monolith}	Normal		Started		Started container with docker id ed6ccd4a3abf
   ```

3. Forward Ports
   ```
   $ kubectl port-forward monolith 10080:80
   Forwarding from 127.0.0.1:10080 -> 80
   Forwarding from [::1]:10080 -> 80
   ```

4. Test
   ```
   $ curl http://127.0.0.1:10080
   {"message":"Hello"}
   ```

5. Monitoring logs
   ```
   $ kubectl logs monolith
   2016/11/23 13:19:02 Starting server...
   2016/11/23 13:19:02 Health service listening on 0.0.0.0:81
   2016/11/23 13:19:02 HTTP service listening on 0.0.0.0:80
   127.0.0.1:42956 - - [Wed, 23 Nov 2016 13:21:35 UTC] "GET / HTTP/1.1" curl/7.47.0
   127.0.0.1:42958 - - [Wed, 23 Nov 2016 13:21:40 UTC] "GET /secure HTTP/1.1" curl/7.47.0
   127.0.0.1:42996 - - [Wed, 23 Nov 2016 13:21:55 UTC] "GET /login HTTP/1.1" curl/7.47.0
   127.0.0.1:43008 - - [Wed, 23 Nov 2016 13:22:10 UTC] "GET /login HTTP/1.1" curl/7.47.0
   127.0.0.1:43178 - - [Wed, 23 Nov 2016 13:23:02 UTC] "GET /secure HTTP/1.1" curl/7.47.0
   ```

6. Log into container and debug
   ```
   $ kubectl exec monolith --stdin --tty -c monolith /bin/sh
   / # ping -c 3 google.com
   PING google.com (64.233.187.100): 56 data bytes
   64 bytes from 64.233.187.100: seq=0 ttl=44 time=54.926 ms
   64 bytes from 64.233.187.100: seq=1 ttl=44 time=55.377 ms
   64 bytes from 64.233.187.100: seq=2 ttl=44 time=54.701 ms

   --- google.com ping statistics ---
   3 packets transmitted, 3 packets received, 0% packet loss
   round-trip min/avg/max = 54.701/55.001/55.377 ms
   / # exit
   ```

## MHC (Monitoring and Health Checks)

## Security

1. Keys
   * `cert.pem` and `key.pem`: secure traffic on a monolith server
   * `ca-pem`: used by HTTP clients as a CA to trust

2. Create `secret`
   ```
   $ kubectl create secret generic tls-certs --from-file=tls/
   secret "tls-certs" created
   ```

3. Describe Keys
   ```
   $ kubectl describe secrets tls-certs
   Name:		tls-certs
   Namespace:	default
   Labels:		<none>
   Annotations:	<none>

   Type:	Opaque

   Data
   ====
   ca-key.pem:	1679 bytes
   ca.pem:		1180 bytes
   cert.pem:	1249 bytes
   key.pem:	1675 bytes
   ```

4. Create `configmap`
   ```
   $ kubectl create configmap nginx-proxy-conf --from-file nginx/proxy.conf
   configmap "nginx-proxy-conf" created
   ```
   and describe created `configmap`
   ```
   $ kubectl describe configmap nginx-proxy-conf
   Name:		nginx-proxy-conf
   Namespace:	default
   Labels:		<none>
   Annotations:	<none>

   Data
   ====
   proxy.conf:	176 bytes
   ```

5. Create `secure-monolith` pod
   ```
   $ kubectl create -f pods/secure-monolith.yaml
   pod "secure-monolith" created
   ```
   Note:
   > If the pod keeps `ContainerCreating` for very long time, it means
   > possibly the port conflict happens. You need to stop/delete those
   > pods that occupying `80` and `81`, e.g. deleting running pod
   > `healthy-monolith` and pod `monolith`

6. Forward ports
   ```
   $ kubectl port-forward secure-monolith 10443:443
   Forwarding from 127.0.0.1:10443 -> 443
   Forwarding from [::1]:10443 -> 443
   ```

7. Check created pod logs
   ```
   $ kubectl logs -c nginx secure-monolith
   ```

## Services

1. Create a service
   ```
   $ kubectl create -f services/monolith.yaml
   You have exposed your service on an external port on all nodes in your
   cluster.  If you want to expose this service to the external internet, you may
   need to set up firewall rules for the service port(s) (tcp:31000) to serve traffic.

   See http://releases.k8s.io/release-1.3/docs/user-guide/services-firewalls.md for more details.
   service "monolith" created
   ```

2. Get labeled pod
   ```
   $ kubectl get pods -l "app=monolith"
   NAME              READY     STATUS    RESTARTS   AGE
   secure-monolith   2/2       Running   0          1h
   $ kubectl describe pods secure-monolith | grep Labels
   Labels:		app=monolith
   ```

3. Label a pod
   ```
   $ kubectl label pods secure-monolith "secure=enabled"
   pod "secure-monolith" labeled
   $ kubectl get pods -l "app=monolith,secure=enabled"
   $
   ```
   
4. Get endpoint
   ```
   $ kubectl describe services monolith | grep Endpoints
   Endpoints:		172.17.0.7:443
   ```

5. Test availability
   ```
   $ minikube service monolith --url
   http://192.168.42.17:31000
   curl http://192.168.42.17:31000
   <html>
   <head><title>400 The plain HTTP request was sent to HTTPS port</title></head>
   <body bgcolor="white">
   <center><h1>400 Bad Request</h1></center>
   <center>The plain HTTP request was sent to HTTPS port</center>
   <hr><center>nginx/1.9.14</center>
   </body>
   </html>
   $ curl -k https://192.168.42.17:31000
   {"message":"Hello"}
   ```
   
