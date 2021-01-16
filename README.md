# 一、部署环境
```bash
[root@centos7 ~]# hostnamectl set-hostname harbor.k8s
[root@centos7 ~]# more /etc/hostname             
harbor.k8s
```
# 二、安装docker 服务


安装 Docker 服务并配置阿里云加速器

## 1、安装基础软件包

```bash
yum -y install yum-utils device-mapper-persistent-data lvm2 
```

## 2、配置 YUM 镜像仓库
```bash
yum -y install wget
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum clean all
yum makecache
```

## 3、安装 Docker 服务
```bash
yum list docker-ce --showduplicates|sort -r   # 查询docker版本
yum -y install docker-ce-18.09.9   # 安装指定版本,根据生产环境自行选择
```
