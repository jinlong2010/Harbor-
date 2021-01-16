# 一、部署环境
```bash
[root@localhost ~]# hostnamectl set-hostname harbor.k8s
[root@localhost ~]# more /etc/hostname             
harbor.k8s
```
# 二、安装docker 服务


安装 Docker 服务并配置阿里云加速器

## 1、安装基础软件包

```bash
[root@localhost ~]#yum -y install yum-utils device-mapper-persistent-data lvm2 
```

## 2、配置 YUM 镜像仓库
```bash
[root@localhost ~]#yum -y install wget
[root@localhost ~]#mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
[root@localhost ~]#wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
[root@localhost ~]#yum clean all
[root@localhost ~]#yum makecache
```

## 3、安装 Docker 服务
```bash
[root@localhost ~]#yum list docker-ce --showduplicates|sort -r   # 查询docker版本
[root@localhost ~]#yum -y install docker-ce-18.09.9   # 安装指定版本,根据生产环境自行选择
```

## 4、修改 deamon.json 配置阿里云镜像加速器
```bash
[root@localhost ~]#mkdir -p /etc/docker
[root@localhost ~]#tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://r5uzm3ad.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

[root@localhost ~]#systemctl daemon-reload
[root@localhost ~]#systemctl restart docker
```
修改cgroupdriver是为了消除告警： [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/


# 三、安装docker-compose服务
官方地址：https://github.com/docker/compose/releases

```bash
[root@localhost ~]#curl -L https://github.com/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
[root@localhost ~]#chmod +x /usr/local/bin/docker-compose
[root@localhost ~]#docker-compose -version
```

# 四、安装harbor服务

## 1、下载harbor软件包
```bash
[root@localhost ~]#wget https://github.com/goharbor/harbor/releases/download/v2.1.1/harbor-offline-installer-v2.1.1.tgz
```
## 2、解压
```bash
[root@localhost ~]#tar xvf harbor-offline-installer-v2.1.1.tgz  -C /home/ && cd /home/harbor/
```

## 3、修改harbor.yml配置文件

### 3.1 修改配置文件并创建对应目录
```bash
[root@localhost ~]#cd /home/harbor/
[root@localhost harbor]#cp harbor.yml.tmpl harbor.yml
[root@localhost harbor]#more harbor.yml|grep -v '#' 
hostname: harbor.k8s

https:
  port: 443
  certificate: /home/harbor/certs/harbor.crt
  private_key: /home/harbor/certs/harbor.key
  
harbor_admin_password: jinlong

database:
  password: jinlong
  max_idle_conns: 50
  max_open_conns: 1000
  
data_volume: /home/harbor/data

clair:
  updaters_interval: 12

trivy:
  ignore_unfixed: false
  skip_update: false
  insecure: false

jobservice:
  max_job_workers: 10

notification:
  webhook_job_max_retry: 10

chart:
  absolute_url: disabled

log:
  level: info
  local:
    rotate_count: 50
    rotate_size: 200M
    location: /var/log/harbor
	
_version: 2.0.0

proxy:
  http_proxy:
  https_proxy:
  no_proxy:
  components:
    - core
    - jobservice
    - clair
    - trivy
```
### 3.1 创建对应目录（根据配置文件修改）
```bash
[root@localhost ~]#mkdir -p /home/harbor/certs /home/harbor/data
```

