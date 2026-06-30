
## 安装前必读
* 需要提前安装好docker和docker-compose。
* 服务占用了80, 1883, 8083端口，请确保端口不被其他程序占用。

## 1. 如果未安装docker和docker-compose，如已安装，跳过此步
### unbuntu或debain环境安装docker
```bash
sudo apt update
sudo apt install docker docker-compose -y
```

### centos安装docker
```bash
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install -y docker-ce docker-ce-cli containerd.io

sudo systemctl start docker
sudo systemctl enable docker
```

## 2. 安装工程：
* 将 ubibot-linux.tar.gz上传至服务器任意目录
> 执行：
```bash
# 解压缩文件：
tar xvf ubibot-linux.tar.gz
# 进入目录：
cd ubibot-linux
```
* docker会从docker hub上下载镜像，对于无法访问docker hub的用户，提供导出的镜像，用户需要先导入镜像，解压后执行docker load，按实际解压文件名称执行：
```bash
docker load -i mysql.tar
docker load -i server.tar
```

## 3. 启动服务
```bash
# 启动工程：
sudo docker-compose up -d
```

## 其他

### https
> 默认只启用了http，如果要配置https，需要简单的几步即可：
1. 编辑docker-compose.yaml文件
2. 将services->server这部分按照如下进行修改：
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
3. 重新构建并重启服务
```bash
docker compose down
docker compose build
# 重新启动服务
docker compose up -d
```

### 数据库
* 如果需要连接数据库，可以放开db中注释掉的端口。
* 平时开发调试时可以放开这一行，*** 正式部署时一定要注释掉！！！！！*** 避免数据库暴露在外部网络

### 文件权限
> 启动服务前，请确保以下目录和文件有正确的读写权限：
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

### 禁用系统自动更新
> 为避免系统自动更新影响服务运行，建议禁用apt自动更新：
```bash
# 停止服务
sudo systemctl stop apt-daily.service apt-daily-upgrade.service

# 禁用服务
sudo systemctl disable apt-daily.service apt-daily-upgrade.service

# 停止定时器
sudo systemctl stop apt-daily.timer apt-daily-upgrade.timer

# 禁用定时器
sudo systemctl disable apt-daily.timer apt-daily-upgrade.timer
sudo systemctl mask apt-daily.service apt-daily-upgrade.service
```

### 数据库备份
> 建议配置定时任务自动备份数据库：

1. 在 `/root/` 目录下创建 `.my.cnf` 文件：
```bash
[client]
host = 127.0.0.1
user = root
password = root
```

2. 在 `docker-compose.yaml` 中开放数据库端口（仅备份时使用，备份完成后建议关闭）：
```yaml
db:
  container_name: ubibot-db
  restart: always
  build: ./dockerfiles/mysql
  image: ubibot-db:v1.0.1
  ports:
    - "3306:3306"
```

3. 添加定时任务（每天凌晨3点执行备份）：
```bash
0 3 * * * /bin/bash /root/ubibot-linux/mysql_backup.sh >> /root/ubibot-linux/mysql_backup.log 2>&1
```

### Web 安装页面
> 首次启动服务后，访问安装页面进行初始化：
1. 访问服务地址
2. 填写数据库信息（数据库名：`db`）
3. 完成系统初始化
