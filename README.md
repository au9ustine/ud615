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
   $ kubectl get pods -l "app=monolith,secure=enabled"
   $
   $ kubectl label pods secure-monolith "secure=enabled"
   pod "secure-monolith" labeled
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

## Deployments

1. Create a deployment
   ```
   $ kubectl create -f deployments/auth.yaml
   deployment "auth" created
   ```

2. Describe a deployment
   ```
   $ kubectl describe deployments auth
   Name:			auth
   Namespace:		default
   CreationTimestamp:	Thu, 24 Nov 2016 15:22:02 +0800
   Labels:			app=auth
   			track=stable
   Selector:		app=auth,track=stable
   Replicas:		1 updated | 1 total | 1 available | 0 unavailable
   StrategyType:		RollingUpdate
   MinReadySeconds:	0
   RollingUpdateStrategy:	1 max unavailable, 1 max surge
   OldReplicaSets:		<none>
   NewReplicaSet:		auth-2330758036 (1/1 replicas created)
   Events:
     FirstSeen	LastSeen	Count	From				SubobjectPath	Type		Reason			Message
     ---------	--------	-----	----				-------------	--------	------			-------
     1m		1m		1	{deployment-controller }			Normal		ScalingReplicaSet	Scaled up replica set auth-2330758036 to 1
   ```

3. Create a responding service
   ```
   $ kubectl create -f services/auth.yaml
   service "auth" created
   ```
   and similarly applied to `hello`
   ```
   $ kubectl create -f deployments/hello.yaml
   deployment "hello" created
   $ kubectl create -f services/hello.yaml
   service "hello" created
   ```

4. Create frontend configuration
   ```
   $ kubectl create secret generic tls-certs --from-file=tls/
   secret "tls-certs" created

   $ kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
   configmap "nginx-frontend-conf" created

   $ kubectl create -f deployments/frontend.yaml
   deployment "frontend" created

   $ cat services/frontend.yaml | sed 's/LoadBalancer/NodePort/g' | kubectl create -f -
   You have exposed your service on an external port on all nodes in your
   cluster.  If you want to expose this service to the external internet, you may
   need to set up firewall rules for the service port(s) (tcp:32078) to serve traffic.

   See http://releases.k8s.io/release-1.3/docs/user-guide/services-firewalls.md for more details.
   service "frontend" created

   $ kubectl get services
   NAME             CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
   auth             10.0.0.3     <none>        80/TCP     46m
   frontend         10.0.0.249   <nodes>       443/TCP    2m
   hello            10.0.0.233   <none>        80/TCP     42m
   hello-minikube   10.0.0.189   <nodes>       8080/TCP   1d
   kubernetes       10.0.0.1     <none>        443/TCP    1d

   $ minikube service frontend --url
   http://192.168.42.17:32757
   ```
   Note:
   > For `minikube` it does not support `LoadBalancer` type service,
   > which would be `NodePort` indeed.

5. Test availability
   ```
   $ curl -k https://192.168.42.17:32757
   {"message":"Hello"}
   ```

## Scaling

1. List `replicas`
   ```
   $ kubectl get replicaset
   NAME                        DESIRED   CURRENT   AGE
   auth-2330758036             1         1         1h
   frontend-1629713508         1         1         45m
   hello-396922042             1         1         48m
   hello-minikube-3015430129   1         1         1d
   ```

2. Get a pod
   ```
   $ kubectl get pods -l "app=hello,track=stable"
   NAME                    READY     STATUS    RESTARTS   AGE
   hello-396922042-klpfa   1/1       Running   0          51m
   ```

3. Change `replicas` to scale out
   ```
   $ cat deployments/hello.yaml | sed 's/replicas: 1/replicas: 3/g' | kubectl apply -f -               
   deployment "hello" configured
   ```

4. List `replicas` (Notice `DESIRED` and `CURRENT` have been turned to 3)
   ```
   $ kubectl get replicasets
   NAME                        DESIRED   CURRENT   AGE
   auth-2330758036             1         1         1h
   frontend-1629713508         1         1         54m
   hello-396922042             3         3         57m
   hello-minikube-3015430129   1         1         1d
   ```

5. List pods (Notice the count of `hello-*` pods has been turned from 1 to 3)
   ```
   $ kubectl get pods
   NAME                              READY     STATUS    RESTARTS   AGE
   auth-2330758036-n9a67             1/1       Running   0          1h
   frontend-1629713508-rdfui         1/1       Running   0          1h
   hello-396922042-33kva             1/1       Running   0          9m
   hello-396922042-3vaw6             1/1       Running   0          9m
   hello-396922042-klpfa             1/1       Running   0          1h
   hello-minikube-3015430129-8tkkb   1/1       Running   2          1d
   ```

6. Check pod info
   ```
   $ kubectl describe deployment hello | grep 'Replicas'
   Replicas:		3 updated | 3 total | 3 available | 0 unavailable
   ```

7. Check availability
   ```
   $ curl -k https://192.168.42.17:32757
   {"message":"Hello"}
   ```

## Updating

1. Change `auth` from version 1 to version2
   ```
   $ cat deployments/auth.yaml | sed 's/udacity\/example-auth:1.0.0/kelseyhightower\/auth:2.0.0/g' | kubectl apply -f -
   deployment "auth" configured
   ```
   
   Note: 
   > [The issue](https://github.com/udacity/ud615/issues/1) happened
   > still as per today 2016-11-24T18:29:51+08:00
   
2. If completed, it would be seen in pod info
   ```
   $ kubectl describe pod auth
   Name:		auth-3377368039-0k5lx
   Namespace:	default
   Node:		minikube/192.168.42.17
   Start Time:	Thu, 24 Nov 2016 18:31:46 +0800
   Labels:		app=auth
   		pod-template-hash=3377368039
   		track=stable
   Status:		Running
   IP:		172.17.0.5
   Controllers:	ReplicaSet/auth-3377368039
   Containers:
     auth:
       Container ID:	docker://732b870a7f2f6721d3acdfc440506d5dad36d89e2b2a3b72776a74ce7c47a034
       Image:		kelseyhightower/auth:2.0.0
       Image ID:		docker://sha256:d411413c856b5621a2aaef93f0511bb0cf267d02b7fb061b858cb9a5a0faa3c0
       Ports:		80/TCP, 81/TCP
       Limits:
         cpu:	200m
         memory:	10Mi
       Requests:
         cpu:			200m
         memory:			10Mi
       State:			Running
         Started:			Thu, 24 Nov 2016 18:31:59 +0800
       Ready:			True
       Restart Count:		0
       Liveness:			http-get http://:81/healthz delay=5s timeout=5s period=15s #success=1 #failure=3
       Readiness:			http-get http://:81/readiness delay=5s timeout=1s period=10s #success=1 #failure=3
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
     FirstSeen	LastSeen	Count	From			SubobjectPath		Type		Reason		Message
     ---------	--------	-----	----			-------------		--------	------		-------
     9m		9m		1	{default-scheduler }				Normal		Scheduled	Successfully assigned auth-3377368039-0k5lx to minikube
     9m		9m		1	{kubelet minikube}	spec.containers{auth}	Normal		Pulling		pulling image "kelseyhightower/auth:2.0.0"
     9m		9m		1	{kubelet minikube}	spec.containers{auth}	Normal		Pulled		Successfully pulled image "kelseyhightower/auth:2.0.0"
     9m		9m		1	{kubelet minikube}	spec.containers{auth}	Normal		Created		Created container with docker id 732b870a7f2f; Security:[seccomp=unconfined]
     9m		9m		1	{kubelet minikube}	spec.containers{auth}	Normal		Started		Started container with docker id 732b870a7f2f
   ```
