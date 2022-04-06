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
```
## Avec Docker-compose

NB : Mttre à jour les valeurs dans le .env
### Sur chacun des noeuds faire
```
docker-compose pull
docker-compose up
```

## Avec Docker-swarm
### Monter le cluster swarm et joindre les différents noeuds
```
docker swarm init --advertise-addr "first_node_ip"
>> Swarm initialized: current node (tj1k4kiz39x8s4jm3gu5rm8ek) is now a manager.
>> To add a worker to this swarm, run the following command:
>> docker swarm join --token SWMTKN-1-wwwwwwwwxxxxxxxxxxxxxx first_node_ip:2377
```
Sur les autres noeuds faire :
```
docker swarm join --token SWMTKN-1-wwwwwwwwxxxxxxxxxxxxxx first_node_ip:2377
```
### Déclarer les secrets de minio
```
echo "minioadmin" | docker secret create access_key -
echo "minioadmin" | docker secret create secret_key -
```
### Taguer les différents noeuds pour contraindre les services 
Ceci permettra aux services de demarrer sur des noeuds specifiques et ainsi eviter toutes corruptions des données
```
docker node update --label-add minio-node01=true MINIO01
docker node update --label-add minio-node02=true MINIO02
docker node update --label-add minio-node03=true MINIO03

```
### Lancer le stack

```
docker stack deploy --compose-file=docker-swarm.yml minio

```
