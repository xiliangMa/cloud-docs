# helm 简介
Helm 是 Deis 开发的一个用于 Kubernetes 应用的包管理工具，主要用来管理 Charts。有点类似于 Ubuntu 中的 APT 或 CentOS 中的 YUM。

对于应用发布者而言:
    可以通过Helm打包应用，管理应用依赖关系，管理应用版本并发布应用到软件仓库。

对于使用者而言:
    使用Helm后不用需要了解Kubernetes的Yaml语法并编写应用部署文件，可以通过Helm下载并在kubernetes上安装需要的应用。

除此以外，Helm还提供了kubernetes上的软件部署，删除，升级，回滚应用的强大功能。

---

# 组件
- helm
 > Helm 是一个命令行下的客户端工具。主要用于 Kubernetes 应用程序 Chart 的创建、打包、发布以及创建和管理本地和远程的 Chart 仓库。

- tiller
 > Tiller 是 Helm 的服务端，部署在 Kubernetes 集群中。Tiller 用于接收 Helm 的请求，并根据 Chart 生成 Kubernetes 的部署文件（ Helm 称为 Release ），然后提交给 Kubernetes 创建应用。Tiller 还提供了 Release 的升级、删除、回滚等一系列功能。

- chart
> Helm 的软件包，采用 TAR 格式。类似于 APT 的 DEB 包或者 YUM 的 RPM 包，其包含了一组定义 Kubernetes 资源相关的 YAML 文件。

- Repoistory
> Helm 的软件仓库，Repository 本质上是一个 Web 服务器，该服务器保存了一系列的 Chart 软件包以供用户下载，并且提供了一个该 Repository 的 Chart 包的清单文件以供查询。Helm 可以同时管理多个不同的 Repository。

- Release
> 使用 helm install 命令在 Kubernetes 集群中部署的 Chart 称为 Release。

---

# 安装环境：
  1. mac os
  2. kubernetes v1.14.3


---

# 安装
假设你已经拥有kubernetes环境，[centos7 kubernetes 环境搭建](https://blog.csdn.net/weixin_41806245/article/details/89381752)
- 安装helm
官方链接: https://helm.sh/docs/using_helm/#installing-helm

```
brew install kubernetes-helm
```


- 安装tiller
> 由于 Helm 默认会去 storage.googleapis.com 拉取镜像，如果你当前执行的机器不能访问合理上网的话可以使用阿里的源来安装：

```
helm init --upgrade --tiller-image registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.14.0 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts

```

**查看版本：**

```
helm version
Client: &version.Version{SemVer:"v2.14.0", GitCommit:"05811b84a3f93603dd6c2fcfe57944dfa7ab7fd0", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.14.0", GitCommit:"05811b84a3f93603dd6c2fcfe57944dfa7ab7fd0", GitTreeState:"clean"}
```

**查看kubernetest 集群中的tiller pod:**

```
kubectl get pod -n kube-system | grep tiller
tiller-deploy-765dcb8745-fj52j           1/1     Running   0          4h23m
```

- rbac 授权

**创建tiller用户、cluster-admin 角色 并用ClusterRoleBinding 将其绑定。**

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```


---

# 使用

- 查看 repo 列表：

```
helm repo list
NAME  	URL
stable	https://kubernetes-charts.storage.googleapis.com
local 	http://127.0.0.1:8879/charts
aliyun	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
```

- 搜索mysql:


```
helm search  mysql
NAME                            	CHART VERSION	APP VERSION	DESCRIPTION
aliyun/mysql                    	0.3.5        	           	Fast, reliable, scalable, and easy to use open-source rel...
stable/mysql                    	1.3.0        	5.7.14     	Fast, reliable, scalable, and easy to use open-source rel...
stable/mysqldump                	2.5.1        	2.4.1      	A Helm chart to help backup MySQL databases using mysqldump
stable/prometheus-mysql-exporter	0.5.1        	v0.11.0    	A Helm chart for prometheus mysql exporter with cloudsqlp...
aliyun/percona                  	0.3.0        	           	free, fully compatible, enhanced, open source drop-in rep...
aliyun/percona-xtradb-cluster   	0.0.2        	5.7.19     	free, fully compatible, enhanced, open source drop-in rep...
stable/percona                  	1.1.0        	5.7.17     	free, fully compatible, enhanced, open source drop-in rep...
stable/percona-xtradb-cluster   	1.0.0        	5.7.19     	free, fully compatible, enhanced, open source drop-in rep...
stable/phpmyadmin               	3.0.0        	4.9.0-1    	phpMyAdmin is an mysql administration frontend
aliyun/gcloud-sqlproxy          	0.2.3        	           	Google Cloud SQL Proxy
aliyun/mariadb                  	2.1.6        	10.1.31    	Fast, reliable, scalable, and easy to use open-source rel...
stable/gcloud-sqlproxy          	0.6.1        	1.11       	DEPRECATED Google Cloud SQL Proxy
stable/mariadb                  	6.7.4        	10.3.17    	Fast, reliable, scalable, and easy to use open-source rel...
```

- 部署 aliyun/mysql
 默认安装在default 命名空间


```
helm install aliyun/mysql
```
- 查看 mysql pod


```
kubectl get pod
NAME                                           READY   STATUS     RESTARTS   AGE
solitary-bird-mysql-755bdb7d6c-rn9cf           0/1     Pending    0          12s
```

- helm 查看 mysql

```
helm list
NAME         	REVISION	UPDATED                 	STATUS  	CHART              	APP VERSION	NAMESPACE
solitary-bird	1       	Tue Aug  6 15:38:26 2019	DEPLOYED	mysql-0.3.5        	           	default
```

- 删除 mysql


```
helm delete solitary-bird
```


查看集群中的mysql pod，已经被删除。

```
kubectl get pod
NAME                                           READY   STATUS     RESTARTS   AGE
```



