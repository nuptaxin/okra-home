# 站点
## home.okracode.com
# 技术
## markdown
## docsify
## docker
## k8s
# 环境
## 云服务器
1. 操作系统镜像：CentOS 7.5 64位
2. IP地址
	* 公网：49.\*.\*.155
	* 内网：172.17.0.17
3. 安全组
	* 开放80端口入站（用于站点正常访问）
	* 开放8080端口入站（用于调试）
	* 开放22端口（用于远程）
4. 远程到服务器
	* ssh root@49.\*.\*.155
## 域名
1. A记录添加
	* www：www.okracode.com记录添加（非本次必须）
	* @：okracode.com记录添加（非本次必须）
	* \*：\*.okracode.com记录添加（必须，否则子域名无法解析到主机）
# 站点生成与部署说明
## docker安装
1. 添加软件源
	* yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
2. 更新源（可能需要几分钟）
	* yum -y update
3. 查询可用安装包
	* yum list docker-ce --showduplicates | sort -r
4. 安装docker（此处选择3:20.10.5-3.el7版本; 注：指定版本号的格式为docker-ce-版本号.x86_64；此处安装需要几分钟）
	* yum -y install docker-ce-3:20.10.5-3.el7.x86_64
5. 启动docker并设置开机启动
	* systemctl start docker.service && systemctl enable docker.service
6. 查看docker状态（Active: active (running)）
	* systemctl status docker.service
## k8s安装
1. 配置源
	* cat <<EOF > /etc/yum.repos.d/kubernetes.repo
```
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```
2. 安装kubectl/kubelet/kubeadm
	* yum install -y kubelet-1.19.8 kubectl-1.19.8 kubeadm-1.19.8
	* 启动kubelet：systemctl enable kubelet && systemctl start kubelet
3. kubeadm初始化（参照文档：https://docs.projectcalico.org/getting-started/kubernetes/quickstart）
	* 初始化kubeadm：kubeadm init --ignore-preflight-errors=NumCPU --image-repository registry.aliyuncs.com/google_containers --kubernetes-version=v1.19.8 --pod-network-cidr=192.168.0.0/16
	* 根据控制台输出内容配置kubectl
		* mkdir -p $HOME/.kube
		* sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
		* sudo chown $(id -u):$(id -g) $HOME/.kube/config
	* 验证kubectl可用(此时node状态为NotReady)
		* kubectl get no
4. 安装网络插件calico（参照文档同上）
	* kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
	* kubectl create -f https://docs.projectcalico.org/manifests/custom-resources.yaml
	* 监控是否安装完成(该ns下有pod且状态为Running时即可)：kubectl get pods -n calico-system --watch
	* 确认node已经Ready：kubectl get no
5. 允许master节点部署pod：kubectl taint nodes --all node-role.kubernetes.io/master-
## 创建docker镜像
1. 定义docsify的docker镜像
	* vim Dockerfile
```
FROM node:latest
LABEL description="A demo Dockerfile for build Docsify."
WORKDIR /docs
RUN npm install -g docsify-cli@latest
EXPOSE 3000/tcp
ENTRYPOINT docsify serve .
```
2. 创建镜像
	* docker build -f Dockerfile -t nuptaxin/docsify-docker:v1.0.0 .
## markdown编辑站点
1. 参照docsify官方文档编写站点（https://docsify.js.org/#/quickstart）
	* 创建入口文件：[index.html](/docs/index.html)
	* 编写首页文件：[README.md](/docs/README.md)
2. 本地站点测试（已通过npm i docsify-cli -g指令本地安装过docsify）
	* 启动站点：docsify serve docs
	* 访问站点：http://localhost:3000
## 上传站点到github
1. 上传站点：git push
2. 如果仓库在github上，可在gitee上导入，防止云服务器拉取不到：https://gitee.com/projects/import/github/status
## 云服务器部署站点
### git-sync同步站点&&docsify解析站点
1. 定义replicaSet
	* pod上有两个容器
		* 容器1：使用git-sync同步git仓库
		* 容器2：部署docsify解析站点
	* yaml文件参照[okra-home-rs.yaml](/k8s/okra-home-rs.yaml)
2. 创建 replicaSet
	* kubectl create -f okra-home-rs.yaml
3. 测试访问
	* 端口映射临时访问（需要开放对应targetPort的防火墙）：kubectl port-forward rs/okra-home-rs 8080:3000 --address 0.0.0.0
	* 访问站点：http://okracode.com:8080
4. 定义服务将docsify转发到云服务器的80端口
	* yaml文件参照[okra-home-svc.yaml](/k8s/okra-home-svc.yaml)
5. 创建Service
	* kubectl create -f okra-home-svc.yaml
### 安装Ingress(参考文档：https://kubernetes.github.io/ingress-nginx/deploy)
1. docker下载镜像（google镜像仓库上不去）：docker pull willdockerhub/ingress-nginx-controller:v0.48.1
2. 下载安装脚本：wget http://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.48.1/deploy/static/provider/baremetal/deploy.yaml
3. 修改安装脚本：将image: k8s.gcr.io/ingress-nginx/controller:v0.48.1xxx改为willdockerhub/ingress-nginx-controller:v0.48.1
4. 安装Ingress：kubectl apply -f deploy.yaml
5. 确认ingress-nginx-controller-xxx的pod已经running：kubectl get pods -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx --watch
6. 定义Ingress
	* yaml文件参照[okra-code-ing.yaml](/k8s/okra-code-ing.yaml)
7. 创建Ingress
	* kubectl create -f okra-code-ing.yaml
8. 定义nodeport接入外部流量
	* yaml文件参照[ingress-nginx.yaml](/k8s/ingerss-nginx.yaml)
9. 创建nodeport
	* kubectl create -f ingress-nginx.yaml 
