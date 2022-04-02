# Deploy MinIO in Distributed Mode

Un déploiement MinIO distribué se compose de 4 disques/volumes ou plus gérés par un ou plusieurs processus de serveur minio, où les processus gèrent la mise en commun des ressources de calcul et de stockage dans une seule ressource de stockage d'objets agrégée. Chaque serveur MinIO dispose d'une image complète de la topologie distribuée, de sorte qu'une application peut se connecter à n'importe quel nœud du déploiement et effectuer des opérations S3.

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

## Sur chacun des noeuds faire
```
docker-compose pull
docker-compose up
```
