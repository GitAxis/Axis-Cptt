**题目：**使用OpenStack私有云平台创建两台云主机，分别作为Kubernetes集群的master节点和node节点，然后完成Kubernetes集群的部署，并完成Istio服务网格、KubeVirt虚拟化和Harbor镜像仓库的部署。

本地虚拟机环境：Centos7.9全新安装环境
网络模式：NAT，可联网
master：192.168.100.10 | 8g+100g | 1u2核
worker：192.168.100.20 | 6g+100g | 1u2核
#### 1.上传chinaskills_cloud_paas_v2.1.iso镜像到master节点并把文件拷贝到/opt目录
```bash
[root@localhost ~]# ls
anaconda-ks.cfg  chinaskills_cloud_paas_v2.1.iso  original-ks.cfg
[root@localhost ~]# mount chinaskills_cloud_paas_v2.1.iso /mnt/
mount: /dev/loop0 is write-protected, mounting read-only
[root@localhost ~]# cp -rf /mnt/* /opt/
[root@localhost ~]# umount /mnt/
[root@localhost ~]# cd /opt/ && ls
dependencies  kubeeasy.tar.gz  kubeeasy-v2.0
[root@localhost opt]#
```
#### 2.设置kubeeasy命令并查看帮助
```bash
[root@localhost opt]# mv kubeeasy-v2.0 /usr/local/bin/kubeeasy
[root@localhost opt]# kubeeasy -h
...
Example:
  [install dependencies package cluster]
  kubeeasy install dependencies \
  --host 10.24.2.10,10.24.2.20,10.24.2.30 \
  --user root \
  --password 000000 \
  --offline-file /opt/dependencies/packages.tar.gz

  [install k8s cluster offline]
  kubeeasy install kubernetes \
  --master 10.24.2.10 \
  --worker 10.24.2.20,10.24.2.30,10.24.2.40 \
  --user root \
  --password 000000 \
  --version 1.25.2 \
  --offline-file /opt/kubeeasy.tar.gz
```
#### 3.依据帮助指南部署平台（事先检查关闭防火墙和selinux）
由于本地虚拟机和比赛云实例环境有差别，本地虚拟机需要手动联网去安装依赖包
```bash
master节点
[root@localhost opt]# cd dependencies/
[root@localhost dependencies]# tar -xf packages.tar.gz
[root@localhost dependencies]# scp packages.tar.gz 172.23.21.20:/opt
[root@localhost dependencies]# cd packages/
[root@localhost packages]# yum install -y *
worker节点
[root@localhost ~]# cd /opt/
[root@localhost opt]# tar -xf packages.tar.gz
[root@localhost opt]# cd packages/
[root@localhost packages]# yum install -y *
```
配置ssh免认证密钥
```bash
[root@localhost opt]# ssh-keygen
[root@localhost opt]# ssh-copy-id 192.168.100.20
```
执行脚本部署
```
[root@localhost opt]# kubeeasy install depend --host 192.168.100.10,192.168.100.20 --user root \
>   --password 000000 \
>   --offline-file /opt/dependencies/packages.tar.gz
```
```
[root@localhost opt]# kubeeasy install k8s --master 192.168.100.10 --worker 192.168.100.20 --user root \
>   --password 000000 \
>   --version 1.25.2 \
>   --offline-file /opt/kubeeasy.tar.gz
```
部署完成：
![Clip_2024-05-13_10-27-09.png](https://cdn.nlark.com/yuque/0/2024/png/39268615/1715567236044-6806cd3a-53df-4b3d-9527-902bb8d1af1c.png#averageHue=%23292522&clientId=u9f0f5657-cfad-4&from=paste&height=668&id=ub593f656&originHeight=668&originWidth=834&originalType=binary&ratio=1&rotation=0&showTitle=false&size=491291&status=done&style=none&taskId=uc6b2ef03-3937-4729-8b2f-31c3fb88111&title=&width=834)
#### 4.访问dashbaord
http://192.168.100.10:30777![Clip_2024-05-13_10-28-08.png](https://cdn.nlark.com/yuque/0/2024/png/39268615/1715567295015-29a4e905-4d39-4f92-87af-936b7dd376b6.png#averageHue=%23f5f5f5&clientId=u9f0f5657-cfad-4&from=paste&height=1083&id=u5befa017&originHeight=1083&originWidth=1920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=170938&status=done&style=none&taskId=u8dd45c26-dba7-4438-accf-9de4581445a&title=&width=1920)
#### 5.访问harbor（用户名：admin密码：Harbor12345）
http://192.168.100.20
![Clip_2024-05-13_10-14-40.png](https://cdn.nlark.com/yuque/0/2024/png/39268615/1715566485275-70bee327-423e-49b6-9b7d-768e04e3f161.png#averageHue=%23192e39&clientId=u9f0f5657-cfad-4&from=paste&height=1083&id=ub54a6a90&originHeight=1083&originWidth=1920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=136384&status=done&style=none&taskId=uf8dbe130-fccc-46e8-a0a2-286f68ed6ca&title=&width=1920)
