# Ansible and OpenShift Demo 

This is just a set of notes to get an OpenShift demo up and running with `oc cluster up`. They may not be complete, and also might be wrong. They're just notes. :) Sorry!

## Install

* Docs: https://github.com/openshift/origin/blob/master/docs/cluster_up_down.md

This is on a packet.net host that has some extra disks.

```
mkdir /var/lib/docker
mkfs.ext4 /dev/sda
mount /dev/sda /var/lib/docker
```

Output:

```
[root@oc ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdd3       108G  1.3G  101G   2% /
devtmpfs         32G     0   32G   0% /dev
tmpfs            32G     0   32G   0% /dev/shm
tmpfs            32G   17M   32G   1% /run
tmpfs            32G     0   32G   0% /sys/fs/cgroup
/dev/sdd1       511M  148K  511M   1% /boot/efi
tmpfs           6.3G     0  6.3G   0% /run/user/0
/dev/sda        440G   73M  418G   1% /var/lib/docker
```

### Install Docker

```
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

```
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

```
yum install docker-ce -y
```

*NOTE: This creates /etc/docker...*

```
systemctl start docker
```

```
docker run hello-world
```

Output:

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
d1725b59e92d: Pull complete 
Digest: sha256:0add3ace90ecb4adbf7777e9aacf18357296e799f81cabc9fde470971e499788
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

## OpenShift / OKD

Install `oc` command line.

```
wget https://github.com/openshift/origin/releases/download/v3.11.0/openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz
tar zxvf openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz 
mv openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit/oc /usr/local/bin
```

```
[root@oc ~]# oc version
oc v3.11.0+0cbc58b
kubernetes v1.11.0+d4cacc0
features: Basic-Auth GSSAPI Kerberos SPNEGO
```

Add full hostname to /etc/hosts.

```
[root@oc ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

<public IP> oc.example.com

[root@oc ~]# hostnamectl set-hostname oc.example.com
[root@oc ~]# hostname -f
oc.example.com
```

Setup insecure repository.

```
[root@oc ~]# vi /etc/docker/daemon.json
[root@oc ~]# cat /etc/docker/daemon.json 
{
   "insecure-registries": [
     "172.30.0.0/16"
   ]
}
```

Restart docker...

```
systemctl daemon-reload
systemctl restart docker
```

Check IP range.

```
[root@oc ~]# docker network inspect -f "{{range .IPAM.Config }}{{ .Subnet }}{{end}}" bridge
172.17.0.0/16
```

```
oc cluster up --routing-suffix=172.17.0.1.nip.io --public-hostname=172.17.0.1.nip.io --enable=service-catalog,router,registry,web-console,persistent-volumes,rhel-imagestreams,automation-service-broker
```

Example output:

```
[root@oc ~]# oc cluster up --routing-suffix=172.17.0.1.nip.io --public-hostname=172.17.0.1.nip.io --enable=service-catalog,router,registry,web-console,persistent-volumes,rhel-imagestreams,au
tomation-service-broker
Getting a Docker client ...
Checking if image openshift/origin-control-plane:v3.11 is available ...
Pulling image openshift/origin-control-plane:v3.11
E1103 12:39:37.714864    3266 helper.go:179] Reading docker config from /root/.docker/config.json failed: open /root/.docker/config.json: no such file or directory, will attempt to pull imag
e docker.io/openshift/origin-control-plane:v3.11 anonymously
Pulled 2/5 layers, 40% complete
Pulled 3/5 layers, 69% complete
Pulled 4/5 layers, 99% complete
Pulled 5/5 layers, 100% complete
Extracting   
Image pull complete
Pulling image openshift/origin-cli:v3.11
E1103 12:39:54.814785    3266 helper.go:179] Reading docker config from /root/.docker/config.json failed: open /root/.docker/config.json: no such file or directory, will attempt to pull imag
e docker.io/openshift/origin-cli:v3.11 anonymously
Image pull complete
Pulling image openshift/origin-node:v3.11
E1103 12:39:55.148269    3266 helper.go:179] Reading docker config from /root/.docker/config.json failed: open /root/.docker/config.json: no such file or directory, will attempt to pull imag
e docker.io/openshift/origin-node:v3.11 anonymously
Pulled 6/6 layers, 100% complete
Extracting   
Image pull complete
Checking type of volume mount ...
Determining server IP ...
Checking if OpenShift is already running ...
Checking for supported Docker version (=>1.22) ...
Checking if insecured registry is configured properly in Docker ...
Checking if required ports are available ...
Checking if OpenShift client is configured properly ...
Checking if image openshift/origin-control-plane:v3.11 is available ...
Starting OpenShift using openshift/origin-control-plane:v3.11 ...
I1103 12:40:02.795456    3266 config.go:40] Running "create-master-config"
I1103 12:40:05.167564    3266 config.go:46] Running "create-node-config"
I1103 12:40:06.228846    3266 flags.go:30] Running "create-kubelet-flags"
I1103 12:40:06.939983    3266 run_kubelet.go:49] Running "start-kubelet"
I1103 12:40:07.150994    3266 run_self_hosted.go:181] Waiting for the kube-apiserver to be ready ...
I1103 12:40:33.164430    3266 interface.go:26] Installing "kube-proxy" ...
I1103 12:40:33.164462    3266 interface.go:26] Installing "kube-dns" ...
I1103 12:40:33.164479    3266 interface.go:26] Installing "openshift-service-cert-signer-operator" ...
I1103 12:40:34.966409    3266 interface.go:41] Finished installing "kube-proxy" "kube-dns" "openshift-service-cert-signer-operator" "openshift-apiserver"
I1103 12:42:19.990658    3266 run_self_hosted.go:242] openshift-apiserver available
I1103 12:42:19.990704    3266 interface.go:26] Installing "openshift-controller-manager" ...
I1103 12:42:19.990729    3266 apply_template.go:81] Installing "openshift-controller-manager"
I1103 12:42:22.245544    3266 interface.go:41] Finished installing "openshift-controller-manager"
Adding default OAuthClient redirect URIs ...
Adding registry ...
Adding router ...
Adding persistent-volumes ...
Adding automation-service-broker ...
Adding rhel-imagestreams ...
Adding service-catalog ...
Adding web-console ...
I1103 12:42:22.260776    3266 interface.go:26] Installing "openshift-image-registry" ...
I1103 12:42:22.260800    3266 interface.go:26] Installing "openshift-router" ...
I1103 12:42:22.260815    3266 interface.go:26] Installing "persistent-volumes" ...
I1103 12:42:22.260829    3266 interface.go:26] Installing "automation-service-broker" ...
I1103 12:42:22.260844    3266 interface.go:26] Installing "rhel-imagestreams" ...
I1103 12:42:22.260859    3266 interface.go:26] Installing "openshift-service-catalog" ...
I1103 12:42:22.260875    3266 interface.go:26] Installing "openshift-web-console-operator" ...
W1103 12:42:22.260974    3266 create_secret.go:78] Error reading $HOME/.docker/config.json: open /root/.docker/config.json: no such file or directory, imagestream import credentials will not be setup
I1103 12:42:22.261067    3266 apply_list.go:67] Installing "rhel-imagestreams"
I1103 12:42:22.261372    3266 apply_template.go:81] Installing "openshift-web-console-operator"
I1103 12:42:22.263898    3266 apply_template.go:81] Installing "automation-service-broker"
I1103 12:42:22.285347    3266 apply_template.go:81] Installing "service-catalog"
I1103 12:43:32.099049    3266 interface.go:41] Finished installing "openshift-image-registry" "openshift-router" "persistent-volumes" "automation-service-broker" "rhel-imagestreams" "openshift-service-catalog" "openshift-web-console-operator"
Login to server ...
Creating initial project "myproject" ...
Server Information ...
OpenShift server started.

The server is accessible via web console at:
    https://172.17.0.1.nip.io:8443

You are logged in as:
    User:     developer
    Password: <any value>

To login as administrator:
    oc login -u system:admin
```

With hostname...

```
oc cluster up --routing-suffix=oc.example.com --public-hostname=oc.example.com --enable=service-catalog,router,registry,web-console,persistent-volumes,rhel-imagestreams,automation-service-broker
```

*NOTE: `oc cluster up` configures with an allow all password, so anyone can login with developer or admin with NO PASSWORD!*

## Using

### APB

There's an example APB in the webhook-apb folder. It is as basic as it gets.

### svcat 

* https://docs.openshift.com/container-platform/3.10/architecture/service_catalog/service_catalog_cli.html


```
wget https://download.svcat.sh/cli/v0.1.37/linux/amd64/svcat
mv svcat /usr/local/bin
chmod +x /usr/local/bin/svcat
```

```
[root@oc ~]# svcat marketplace
           CLASS                 PLANS                  DESCRIPTION            
+-------------------------+-----------------+---------------------------------+
  dh-rhpam-apb              managed           Red Hat Process Automation       
                                              Manager (APB)                    
                            immutable-mon                                      
                            immutable-kie                                      
                            authoring                                          
                            trial                                              
  dh-gluster-s3-apb         default           GlusterFS S3 object service      
  dh-pyzip-demo-apb         default           Python Zip Demo APB              
                                              Implementation                   
  dh-nginx-apb              default           An open source reverse proxy     
                                              web server                       
  dh-proxy-config-apb       passthrough       Proxy Configuration              
                            custom                                             
  dh-eclipse-che-apb        prod              Eclipse Che                      
  dh-galera-apb             persistent        Deploy Galera with a             
                                              Replication Operator             
                            ephemeral                                          
  dh-vnc-desktop-apb        f28               A Desktop environment            
                                              accessible via noVNC.            
                            f27                                                
                            f29                                                
  dh-postgresql-apb         dev               SCL PostgreSQL apb               
                                              implementation                   
                            prod                                               
  dh-mssql-remote-apb       default           Microsoft SQL Server Remote APB  
  dh-kibana-apb             persistent        APB to deploy Kibana OSS         
                            ephemeral                                          
  dh-import-vm-apb          url               Import Virtual Machine           
                            vmware                                             
                            url-template                                       
                            vmware-template                                    
  dh-mariadb-apb            dev               Mariadb apb implementation       
                            prod                                               
  dh-pyzip-demo-db-apb      default           Python Zip Demo Database APB     
                                              Implementation                   
  dh-manageiq-apb           default           ManageIQ                         
  dh-awx-apb                default           AWX APB Implementation           
  dh-unifi-controller-apb   Ephemeral         Ubiquiti UniFi Controller        
                            Persistent                                         
  dh-homeassistant-apb      Persistent        Open source home automation      
                                              that puts local control and      
                                              privacy first                    
                            Ephemeral                                          
  dh-hello-world-apb        default           deploys hello-world web          
                                              application                      
  dh-wordpress-ha-apb       default           High Availability Wordpress APB  
  dh-hastebin-apb           default           This is a sample application     
                                              generated by apb init            
  dh-es-apb                 ephemeral         APB to deploy cluster-ready      
                                              Elasticsearch                    
                            persistent                                         
  dh-hello-world-db-apb     default           A sample APB which deploys       
                                              Hello World Database             
  dh-keycloak-apb           persistent        Keycloak - Open Source Identity  
                                              and Access Management            
                            external                                           
                            ephemeral                                          
  dh-mssql-apb              ephemeral         Deployment of Microsoft SQL      
                                              Server on Linux                  
                            persistent                                         
  dh-rds-postgres-apb       default           Managed relational database      
                                              service with a choice of six     
                                              popular database engines.        
                                              Set up, operate, and scale       
                                              a relational database in the     
                                              cloud with just a few clicks.    
  dh-etherpad-apb           default           Note taking web application      
  dh-mysql-apb              prod              Software Collections MySQL APB   
                            dev                                                
  dh-mongodb-apb            cluster           Deploy MongoDB app on your       
                                              Openshift Project                
                            persistent                                         
                            ephemeral                                          
  dh-thelounge-apb          default           This is a sample application     
                                              generated by apb init            
  dh-jenkins-apb            default           Jenkins service with optional    
                                              persistent storage and S2I       
                                              build                            
  dh-mediawiki-apb          default           Mediawiki apb implementation     
  dh-prometheus-apb         ephemeral         Deploy Prometheus on your        
                                              Project                          
                            persistent                                         
  dh-tiller-apb             default           Installs tiller for              
                                              single-namespace use 
```

* docs: https://docs.openshift.com/container-platform/3.10/architecture/service_catalog/service_catalog_cli.html

```
[root@oc ~]# svcat get brokers
                 NAME                   NAMESPACE                                 URL                                  STATUS  
+-------------------------------------+-----------+------------------------------------------------------------------+--------+
  openshift-automation-service-broker               https://broker.openshift-automation-service-broker.svc:1338/osb/   Ready  
```

## Issues 

Many...

* https://github.com/openshift/origin-web-console/issues/3001

