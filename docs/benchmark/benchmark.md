Benchmarking
============

- AWS
	- t2.micro
	- Ubuntu 20.04
	- Region: eu-central-1
	- Steps
		- Update: `apt update && apt upgrade -y && apt dist-upgrade -y && apt autoremove -y
		- Install docker (https://docs.docker.com/engine/install/ubuntu/):
			-  ```
				sudo apt-get install \
					apt-transport-https \
					ca-certificates \
					curl \
					gnupg-agent \
					software-properties-common
				```
			- `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`
			- ```
				sudo add-apt-repository \
				   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
				   $(lsb_release -cs) \
				   stable"
				```
			- ` apt update && apt install docker-ce docker-ce-cli containerd.io`
			- `docker version`
				- `20.10.3`
		- `reboot now`
		- run node-exporter: `docker run -d \
  --net="host" \
  --pid="host" \
  -v "/:/host:ro,rslave" \
  quay.io/prometheus/node-exporter:latest \
  --path.rootfs=/host`
	
- Local:
	- `docker run --rm -v "$(pwd)/prometheus.yml:/prometheus.yml" -p 9090:9090 -d prom/prometheus --config.file=/prometheus.yml`
	- config file:
```yaml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'k3d-benchmark'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['52.59.167.100:9100']
```

- install k3d: `wget -q -O - https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash`
	- ```
		root@ip-172-31-20-21:/home/ubuntu# k3d version
k3d version v4.1.1
k3s version v1.20.2-k3s1 (default)
	```

```
root@ip-172-31-20-21:/home/ubuntu# time k3d cluster create
INFO[0000] Prep: Network                                
INFO[0000] Created network 'k3d-k3s-default'            
INFO[0000] Created volume 'k3d-k3s-default-images'      
INFO[0001] Creating node 'k3d-k3s-default-server-0'     
INFO[0002] Pulling image 'docker.io/rancher/k3s:v1.20.2-k3s1' 
INFO[0006] Creating LoadBalancer 'k3d-k3s-default-serverlb' 
INFO[0008] Pulling image 'docker.io/rancher/k3d-proxy:v4.1.1' 
INFO[0011] Starting cluster 'k3s-default'               
INFO[0011] Starting servers...                          
INFO[0011] Starting Node 'k3d-k3s-default-server-0'     
INFO[0019] Starting agents...                           
INFO[0019] Starting helpers...                          
INFO[0019] Starting Node 'k3d-k3s-default-serverlb'     
INFO[0021] (Optional) Trying to get IP of the docker host and inject it into the cluster as 'host.k3d.internal' for easy access 
INFO[0025] Successfully added host record to /etc/hosts in 2/2 nodes and to the CoreDNS ConfigMap 
INFO[0025] Cluster 'k3s-default' created successfully!  
INFO[0025] --kubeconfig-update-default=false --> sets --kubeconfig-switch-context=false 
INFO[0026] You can now use it like this:                
kubectl config use-context k3d-k3s-default
kubectl cluster-info

real    0m26.402s
user    0m0.116s
sys     0m0.008s
root@ip-172-31-20-21:/home/ubuntu# date
Mon 08 Feb 2021 08:30:12 AM UTC
```

```
root@ip-172-31-20-21:/home/ubuntu# k3d cluster rm -a
INFO[0013] Deleting all clusters...                     
INFO[0039] Deleting cluster 'k3s-default'               
INFO[0052] Deleted k3d-k3s-default-serverlb             
INFO[0057] Deleted k3d-k3s-default-server-0             
INFO[0057] Deleting cluster network 'k3d-k3s-default'   
INFO[0057] Deleting image volume 'k3d-k3s-default-images' 
INFO[0057] Removing cluster details from default kubeconfig... 
INFO[0057] Removing standalone kubeconfig file (if there is one)... 
INFO[0057] Successfully deleted cluster k3s-default!    
root@ip-172-31-20-21:/home/ubuntu# time k3d --timestamps cluster create test
INFO[2021-02-08T08:33:46Z] Prep: Network                                
INFO[2021-02-08T08:33:46Z] Created network 'k3d-test'                   
INFO[2021-02-08T08:33:46Z] Created volume 'k3d-test-images'             
INFO[2021-02-08T08:33:47Z] Creating node 'k3d-test-server-0'            
INFO[2021-02-08T08:33:47Z] Creating LoadBalancer 'k3d-test-serverlb'    
INFO[2021-02-08T08:33:47Z] Starting cluster 'test'                      
INFO[2021-02-08T08:33:47Z] Starting servers...                          
INFO[2021-02-08T08:33:47Z] Starting Node 'k3d-test-server-0'            
INFO[2021-02-08T08:33:56Z] Starting agents...                           
INFO[2021-02-08T08:33:56Z] Starting helpers...                          
INFO[2021-02-08T08:33:56Z] Starting Node 'k3d-test-serverlb'            
INFO[2021-02-08T08:33:58Z] (Optional) Trying to get IP of the docker host and inject it into the cluster as 'host.k3d.internal' for easy access 
INFO[2021-02-08T08:34:03Z] Successfully added host record to /etc/hosts in 2/2 nodes and to the CoreDNS ConfigMap 
INFO[2021-02-08T08:34:03Z] Cluster 'test' created successfully!         
INFO[2021-02-08T08:34:03Z] --kubeconfig-update-default=false --> sets --kubeconfig-switch-context=false 
INFO[2021-02-08T08:34:04Z] You can now use it like this:                
kubectl config use-context k3d-test
kubectl cluster-info

real    0m17.756s
user    0m0.084s
sys     0m0.042s

root@ip-172-31-20-21:/home/ubuntu# snap install kubectl --classic
kubectl 1.20.2 from Canonical✓ installed
root@ip-172-31-20-21:/home/ubuntu# kubectl get nodes
NAME                STATUS   ROLES                  AGE   VERSION
k3d-test-server-0   Ready    control-plane,master   28m   v1.20.2+k3s1
root@ip-172-31-20-21:/home/ubuntu# kubectl get all --all-namespaces
NAMESPACE     NAME                                          READY   STATUS      RESTARTS   AGE
kube-system   pod/local-path-provisioner-7c458769fb-nz9w7   1/1     Running     0          28m
kube-system   pod/metrics-server-86cbb8457f-pvttl           1/1     Running     0          28m
kube-system   pod/coredns-854c77959c-w5k97                  1/1     Running     0          28m
kube-system   pod/helm-install-traefik-s2jwz                0/1     Completed   0          28m
kube-system   pod/svclb-traefik-vz67p                       2/2     Running     0          28m
kube-system   pod/traefik-6f9cbd9bd4-7rzsj                  1/1     Running     0          28m

NAMESPACE     NAME                         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
default       service/kubernetes           ClusterIP      10.43.0.1       <none>        443/TCP                      29m
kube-system   service/kube-dns             ClusterIP      10.43.0.10      <none>        53/UDP,53/TCP,9153/TCP       29m
kube-system   service/metrics-server       ClusterIP      10.43.229.255   <none>        443/TCP                      28m
kube-system   service/traefik-prometheus   ClusterIP      10.43.186.114   <none>        9100/TCP                     28m
kube-system   service/traefik              LoadBalancer   10.43.255.173   172.19.0.2    80:31942/TCP,443:31039/TCP   28m

NAMESPACE     NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
kube-system   daemonset.apps/svclb-traefik   1         1         1       1            1           <none>          28m

NAMESPACE     NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/local-path-provisioner   1/1     1            1           29m
kube-system   deployment.apps/metrics-server           1/1     1            1           28m
kube-system   deployment.apps/coredns                  1/1     1            1           29m
kube-system   deployment.apps/traefik                  1/1     1            1           28m

NAMESPACE     NAME                                                DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/local-path-provisioner-7c458769fb   1         1         1       28m
kube-system   replicaset.apps/metrics-server-86cbb8457f           1         1         1       28m
kube-system   replicaset.apps/coredns-854c77959c                  1         1         1       28m
kube-system   replicaset.apps/traefik-6f9cbd9bd4                  1         1         1       28m

NAMESPACE     NAME                             COMPLETIONS   DURATION   AGE
kube-system   job.batch/helm-install-traefik   1/1           36s        28m
root@ip-172-31-20-21:/home/ubuntu# date
Mon 08 Feb 2021 09:03:05 AM UTC
```