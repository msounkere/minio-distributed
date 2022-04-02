# Deploy MinIO in Distributed Mode

Un déploiement MinIO composé de 3 serveurs avec chacun 4 disques/volumes ou plus gérés par un ou plusieurs processus de serveur minio, où les processus gèrent la mise en commun des ressources de calcul et de stockage dans une seule ressource de stockage d'objets agrégée. Chaque serveur MinIO dispose d'une image complète de la topologie distribuée, de sorte qu'une application peut se connecter à n'importe quel nœud du déploiement et effectuer des opérations S3.
### Formatter les disques via XFS
```
mkfs.xfs /dev/sdb -L DISK1
mkfs.xfs /dev/sdc -L DISK2
mkfs.xfs /dev/sdd -L DISK3
mkfs.xfs /dev/sde -L DISK4
```
Edit Fstab to mount disk
```

nano /etc/fstab

  # <file system>  <mount point>  <type>  <options>         <dump>  <pass>
  LABEL=DISK1      /mnt/disk1     xfs     defaults,noatime  0       2
  LABEL=DISK2      /mnt/disk2     xfs     defaults,noatime  0       2
  LABEL=DISK3      /mnt/disk3     xfs     defaults,noatime  0       2
  LABEL=DISK4      /mnt/disk4     xfs     defaults,noatime  0       2
  
# Mount disk 
mkdir -p /mnt/disk{1,2,3,4}
```
### Mise en place des composants dockers
```
 # Install docker
 curl -fsSL https://get.docker.com -o get-docker.sh
 sudo sh get-docker.sh
 
 # Install docker-compose
 curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
 chmod +x /usr/local/bin/docker-compose
 ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
 
 # Test the installation.
 docker-compose --version
```
### Installer et configurer Minio with docker-compose
```
cd /opt/
git clone https://github.com/msounkere/minio-distributed.git

## Mettre à jour les valeurs dans le .env
```

### Sur chacun des noeuds faire
```
docker-compose pull
docker-compose up
```

## Ajout de prometheus à la configuration
Il es important de noté que dans une configuration de production vous devriez deployer votre noeud prometheus en dehors du cluster pour palier de problème de partage de donnée entre les differents noeuds

### docker-compose de déploiement de prometheus
```
version: '3.7'

services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - 9090:9090
    command:
      - --config.file=/etc/prometheus/prometheus.yml
      - --storage.tsdb.path=/prometheus
      - --storage.tsdb.retention.time=48h
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus_data:/prometheus
    restart: on-failure

## By default this config uses default local driver,
## For custom volumes replace with volume driver configuration.
volumes:
  prometheus_data:
```
#### Fichier de prometheus.yml
Le point de terminaison Prometheus dans MinIO nécessite une authentification par défaut. Prometheus prend en charge une approche de jeton porteur pour authentifier les demandes de grattage de Prometheus, remplacer la configuration par défaut de Prometheus par celle générée à l'aide de mc. Pour générer une configuration Prometheus pour un alias, utilisez mc comme suit 
`mc admin prometheus generate <alias>`

```
global:
  scrape_interval:     10s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 30s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  - job_name: minio-job
    bearer_token: secret***
    metrics_path: /minio/v2/metrics/cluster
    scheme: https
    tls_config:
      insecure_skip_verify: true
    static_configs:
    - targets: ['s3.toto.ci:443']
```
