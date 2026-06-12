
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
* docker会从docker hub上下载镜像，对于无法访问docker hub的用户，提供导出的镜像，用户需要先导入镜像：
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
