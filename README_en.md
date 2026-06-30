
## Must Read Before Installation
* Docker and docker-compose must be installed in advance.
* The service uses ports 80, 1883, and 8083. Please ensure these ports are not occupied by other programs.

## 1. If Docker and docker-compose are not installed, skip this step if already installed
### Install Docker on Ubuntu or Debian
```bash
sudo apt update
sudo apt install docker docker-compose -y
```

### Install Docker on CentOS
```bash
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install -y docker-ce docker-ce-cli containerd.io

sudo systemctl start docker
sudo systemctl enable docker
```

## 2. Project Installation
* Upload `ubibot-linux.tar.gz` to any directory on the server
> Execute:
```bash
# Extract the compressed file:
tar xvf ubibot-linux.tar.gz
# Enter the directory:
cd ubibot-linux
```
* Docker will download images from Docker Hub. For users who cannot access Docker Hub, exported images are provided; users need to import the images first:
```bash
docker load -i mysql.tar
docker load -i server.tar
```

## 3. Start the Service
```bash
# Start the service:
sudo docker-compose up -d
```

## Others

### HTTPS
> Only HTTP is enabled by default. If you want to configure HTTPS, a few simple steps are required:
1. Edit the `docker-compose.yaml` file
2. Modify the `services->server` section as follows:
```bash
services:
  server:
    container_name: ubibot-server
    #image: ubibot123/ubibot-server:v1.2.5 # <- here
    build: ./dockerbuild/https # <- here
    restart: always
    ports:
      - "80:80"
      - "443:443" # <- here
      - "1883:1883"
      - "8083:8083"
      - "8084:8084" # <- here
```
3. Rebuild and restart the service
```bash
docker compose down
docker compose build
# Restart the service
docker compose up -d
```

### Database
* If you need to connect to the database, you can uncomment the commented-out ports in the `db` section.
* This line can be uncommented during development and debugging. *** Comment it out in production deployment!!! *** to prevent the database from being exposed to the external network.

### File Permissions
> Before starting the service, ensure the following directories and files have correct read/write permissions:
```bash
chmod -R 777 storage
chmod -R 777 script
chmod -R 777 custom_logs
chmod -R 777 public/front/scripts
chmod -R 777 public/images
chmod 777 opp.env
chmod 777 .env
chmod -R 777 public/ota
```

### Disable System Auto Updates
> To prevent system auto-updates from affecting service operation, it is recommended to disable apt auto-updates:
```bash
# Stop services
sudo systemctl stop apt-daily.service apt-daily-upgrade.service

# Disable services
sudo systemctl disable apt-daily.service apt-daily-upgrade.service

# Stop timers
sudo systemctl stop apt-daily.timer apt-daily-upgrade.timer

# Disable timers
sudo systemctl disable apt-daily.timer apt-daily-upgrade.timer
sudo systemctl mask apt-daily.service apt-daily-upgrade.service
```

### Database Backup
> It is recommended to configure a cron job for automatic database backup:

1. Create a `.my.cnf` file in the `/root/` directory:
```bash
[client]
host = 127.0.0.1
user = root
password = root
```

2. Open the database port in `docker-compose.yaml` (only for backup purposes, recommended to close after backup):
```yaml
db:
  container_name: ubibot-db
  restart: always
  build: ./dockerfiles/mysql
  image: ubibot-db:v1.0.1
  ports:
    - "3306:3306"
```

3. Add a cron job (execute backup at 3:00 AM daily):
```bash
0 3 * * * /bin/bash /root/ubibot-linux/mysql_backup.sh >> /root/ubibot-linux/mysql_backup.log 2>&1
```

### Web Installation Page
> After starting the service for the first time, access the installation page for initialization:
1. Access the service address
2. Fill in the database information (Database name: `db`)
3. Complete system initialization
