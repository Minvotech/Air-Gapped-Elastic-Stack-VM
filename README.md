# Air-Gapped-Elastic-Stack-VM  [ PODMAN ]
# Elastic Stack Deployment on Air-Gapped REHL-Server (v9.1.4)

This guide provides the full process to deploy Elasticsearch, Kibana, Fleet Server, and the Elastic Package Registry (EPR) on an **air-gapped Ubuntu server** using Docker Compose.

---

## 1. ✅ Air-Gap Preparation (Offline Setup)

### Requirements

- A machine with internet access to download packages and Docker images
- USB or secure method to transfer files to the air-gapped server

### Docker Images to Download (on online machine)

```
docker pull docker.elastic.co/elasticsearch/elasticsearch:9.1.4
docker pull docker.elastic.co/kibana/kibana:9.1.4
docker pull docker.elastic.co/elastic-agent/elastic-agent:9.1.4
docker pull docker.elastic.co/package-registry/distribution:9.1.4
```

### Save Docker Images

```
docker save -o elasticsearch.tar docker.elastic.co/elasticsearch/elasticsearch:9.1.4
docker save -o kibana.tar docker.elastic.co/kibana/kibana:9.1.4
docker save -o elastic-agent.tar docker.elastic.co/beats/elastic-agent:8.17.5
.. Fleet Server >> Working on new image with V9 > ::
> docker save -o fleet-v9.tar docker.elastic.co/elastic-agent/elastic-agent:9.1.4
docker save -o epr.tar docker.elastic.co/package-registry/distribution:9.1.4
```

### Download Packages & Tools

- Required .deb packages:
    - jq, libjq1, libonig5, adduser, debconf, libsystemd0, apparmor, git, pigz
    - ca-certificates, iptables, ubuntu-fan, containerd, libc6, xz-utils
    - containerd.io, docker.io, libslirp0, docker-buildx-plugin
    - docker-compose-plugin, runc
- Docker Compose:

```
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

- Elastic Agent Binaries:

```
# Linux
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.17.5-linux-x86_64.tar.gz

# Windows
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.17.5-windows-x86_64.zip -OutFile elastic-agent-8.17.5-windows-x86_64.zip
```

---

## 2. 🚛 Transfer All Files to Air-Gapped Server

### Open Required Ports

Ensure the following ports are open:

```
9200, 5601, 8220, 8200, 8080, 80, 443 , 9600 , 9300
```

### Install Transferred Packages

```
dpkg --install <package-name.deb>
or rpm "rehl" redhat 
```

### Docker Setup

```
groupadd podman
usermod -aG podman$USER
systemctl daemon-reload
systemctl reset-failed docker.service
systemctl start podman.socket
systemctl start podman.server
podman--version
podman ps
```

### Docker Compose Setup

```
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
cp docker-compose /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose version
- echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.bashrc
- echo 'export PATH="/usr/local/bin:$PATH"' >> /etc/bash.bashrc
- source ~/.bashrc
```

### Kernel & System Configurations

Edit `/etc/sysctl.conf` and append:

```
vm.max_map_count=262144
net.ipv4.ip_forward=1
net.ipv4.tcp_retries2=5
vm.swappiness=1
```

Apply changes:

```
sudo sysctl -p
```

### Limits Configuration

Edit `/etc/security/limits.conf`:

```
*                soft    nofile         1024000
*                hard    nofile         1024000
*                soft    memlock        unlimited
*                hard    memlock        unlimited
opc              soft    nproc          unlimited
opc              hard    nproc          unlimited
root             soft    nofile         1024000
root             hard    nofile         1024000
root             soft    memlock        unlimited
```

### Clear Cache

```
sudo sync && echo 3 | sudo tee /proc/sys/vm/drop_caches
```

---

## 3. ✅ Load Docker Images

```
podman load -i elasticsearch.tar
podman load -i kibana.tar
podman load -i elastic-agent.tar
podman load -i epr.tar
```

---

## 4. 🚀 Deploy Elastic Stack via Script

### Setup Directory and Download Script

```
mkdir -p /home/ubuntu/elastic-deployment
cd /home/ubuntu/elastic-deployment
curl -O https://raw.githubusercontent.com/jlim0930/scripts/master/deploy-elastic.sh
chmod +x deploy-elastic.sh
```

### Run Deployment Script

```
./deploy-elastic.sh stack 9.1.4
```

This creates `/home/ubuntu/elasticstack` with full configuration (elasticsearch.yml, kibana.yml, SSL certs).

### Verify Containers

```
podman ps
podman logs <container-name>
```

### Deploy Fleet Server

```
./deploy-elastic.sh fleet 9.1.4
```

---

## 5. 🌐 Deploy Elastic Package Registry (EPR)

### Update Docker Compose

Edit `/home/ubuntu/elasticstack/stack-compose.yml`:

```
epr:
  image: docker.elastic.co/package-registry/distribution:9.1.4
  ports:
    - 8080:80
```

### Start EPR

```
docker-compose -f /home/ubuntu/elasticstack/stack-compose.yml up -d epr
```

### Verify EPR Health

```
curl http://localhost:8080/health
```

### Update Kibana Configuration

Edit `/home/ubuntu/elasticstack/kibana.yml`:

```
xpack.fleet.registryUrl: "http://es-epr-1:8080"
```

Restart Kibana:

```
podman restart kibana
```

### Final Check

Verify all containers are up and healthy:

```
podman ps
```

# .......................... # ........................... # ......................... # ...................... # ....................... # .................... # .................... #

#  Air-Gapped-Elastic-Stack-VM  [ DOCKER ] 

# Elastic Stack Deployment on Air-Gapped Ubuntu Server (v8.17.5)

This guide provides the full process to deploy Elasticsearch, Kibana, Fleet Server, and the Elastic Package Registry (EPR) on an **air-gapped Ubuntu server** using Docker Compose.

---

## 1. ✅ Air-Gap Preparation (Offline Setup)

### Requirements

- A machine with internet access to download packages and Docker images
- USB or secure method to transfer files to the air-gapped server

### Docker Images to Download (on online machine)

```
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.17.5
docker pull docker.elastic.co/kibana/kibana:8.17.5
docker pull docker.elastic.co/beats/elastic-agent:8.17.5
docker pull docker.elastic.co/package-registry/distribution:8.17.5
```

### Save Docker Images

```
docker save -o elasticsearch.tar docker.elastic.co/elasticsearch/elasticsearch:8.17.5
docker save -o kibana.tar docker.elastic.co/kibana/kibana:8.17.5
docker save -o elastic-agent.tar docker.elastic.co/beats/elastic-agent:8.17.5
docker save -o epr.tar docker.elastic.co/package-registry/distribution:8.17.5
```

### Download Packages & Tools

- Required .deb packages:
    - jq, libjq1, libonig5, adduser, debconf, libsystemd0, apparmor, git, pigz
    - ca-certificates, iptables, ubuntu-fan, containerd, libc6, xz-utils
    - containerd.io, docker.io, libslirp0, docker-buildx-plugin
    - docker-compose-plugin, runc
- Docker Compose:

```
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

- Elastic Agent Binaries:

```
# Linux
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.17.5-linux-x86_64.tar.gz

# Windows
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.17.5-windows-x86_64.zip -OutFile elastic-agent-8.17.5-windows-x86_64.zip
```

---

## 2. 🚛 Transfer All Files to Air-Gapped Server

### Open Required Ports

Ensure the following ports are open:

```
9200, 5601, 8220, 8200, 8080, 80, 443
```

### Install Transferred Packages

```
dpkg --install <package-name.deb>
```

### Docker Setup

```
groupadd docker
usermod -aG docker $USER
systemctl daemon-reload
systemctl reset-failed docker.service
systemctl start docker.socket
systemctl start docker
docker --version
docker ps
```

### Docker Compose Setup

```
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
cp docker-compose /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose version
```

### Kernel & System Configurations

Edit `/etc/sysctl.conf` and append:

```
vm.max_map_count=262144
net.ipv4.ip_forward=1
net.ipv4.tcp_retries2=5
vm.swappiness=1
```

Apply changes:

```
sudo sysctl -p
```

### Limits Configuration

Edit `/etc/security/limits.conf`:

```
*                soft    nofile         1024000
*                hard    nofile         1024000
*                soft    memlock        unlimited
*                hard    memlock        unlimited
ubuntu           soft    nproc          unlimited
ubuntu           hard    nproc          unlimited
root             soft    nofile         1024000
root             hard    nofile         1024000
root             soft    memlock        unlimited
```

### Clear Cache

```
sudo sync && echo 3 | sudo tee /proc/sys/vm/drop_caches
```

---

## 3. ✅ Load Docker Images

```
docker load -i elasticsearch.tar
docker load -i kibana.tar
docker load -i elastic-agent.tar
docker load -i epr.tar
```

---

## 4. 🚀 Deploy Elastic Stack via Script

### Setup Directory and Download Script

```
mkdir -p /home/ubuntu/elastic-deployment
cd /home/ubuntu/elastic-deployment
curl -O https://raw.githubusercontent.com/jlim0930/scripts/master/deploy-elastic.sh
chmod +x deploy-elastic.sh
```

### Run Deployment Script

```
./deploy-elastic.sh stack 8.17.5
```

This creates `/home/ubuntu/elasticstack` with full configuration (elasticsearch.yml, kibana.yml, SSL certs).

### Verify Containers

```
docker ps
docker logs <container-name>
```
#......... N O T E .........#

- Deploy elastic-stack from curl -O <https://raw.githubusercontent.com/jlim0930/scripts/master/deploy-elastic.sh>
>> Final Update Script-v9 :: 
- download deployment script :
mkdir /home/ubuntu/elastic-deployment
cd /home/ubuntu/elastic-deployment
- ./deploy-elastic.sh stack 9.1.5   " Run script with select Version "	
( after script is deployed will new dire is created in /home/user/elasticstack contains all configration  kibna.yml - elasticsearch  ssl certificate  )
- Verifiy containers >> docker ps & check logs to veifiy all container are healthy and working fine
- deploy Fleet Server >> ./deploy-elastic.sh fleet 8.17.5
- - >> create token manual from this commadn :
	 curl -k -X POST "https://localhost:9200/_security/service/elastic/fleet-server/credential/token/token1" -H 'Content-Type: application/json' -u elastic:N5CTb3e444GSNaRRwTuUoKncs
	 - update fllet-compose.yml to add token in FLEET_SERVER_SERVICE_TOKEN=
	 & update docker-compose > > 
	 docker-compose -f /root/elasticstack/fleet-compose.yml up  -d fleet
- to deploy epr in docker compose and update kibana configration ::
   * Update docker compose in /home/user/elasticstack/docker-comose.yml
   * add it ::
   epr:
    container_name: epr-1
    image: docker.elastic.co/package-registry/distribution:9.1.5
    ports:
      - 8080:80
   * Deploy new Update From docker-comose >>
	   docker-compose -f /root/elasticstack/stack-compose.yml up -d epr
	 * Verifiy Container epr working and Healthy
	 * Update Kibana.yml in /root/elasticstack/kibana.yml to add pkgs-registry URL :
	   xpack.fleet.registryUrl: "http://epr-1:8080"
	   restart kibana >  docker restart kibana
	 * Check  Contianers to verifiy everything is working Fine 
	 = Notes ::
	 = when  add anew agent from new policy :
	 url-fleet : <https://ip:8220>
     = Use this command :
	 > ./elastic-agent install  --fleet-server-es=https://10.200.3.235:9200 
	  --fleet-server-service-token=AAEAAWVsYXN0aWMvZmxlZXQtc2VydmVyL3Rva2VuLTE3NTk5NTEwOTU1NzQ6UU52eVRob2RRWHFmRGs5bzdnMlVFQQ
	    --fleet-server-policy=fleet-server-policy  --fleet-server-es-insecure  --fleet-server-port=8220  --insecure


#......... N O T E .........#


### Deploy Fleet Server

```
./deploy-elastic.sh fleet 8.17.5
```

---

## 5. 🌐 Deploy Elastic Package Registry (EPR)

### Update Docker Compose

Edit `/home/ubuntu/elasticstack/stack-compose.yml`:

```
epr:
  image: docker.elastic.co/package-registry/distribution:8.17.5
  ports:
    - 8080:80
```

### Start EPR

```
docker-compose -f /home/ubuntu/elasticstack/stack-compose.yml up -d epr
```

### Verify EPR Health

```
curl http://localhost:8080/health
```

### Update Kibana Configuration

Edit `/home/ubuntu/elasticstack/kibana.yml`:

```
xpack.fleet.registryUrl: "http://es-epr-1:8080"
```

Restart Kibana:

```
docker restart kibana
```

### Final Check

Verify all containers are up and healthy:


