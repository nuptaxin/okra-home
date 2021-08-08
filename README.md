# 站点：home.okracode.com
# 技术：markdown + docsify + k8s
# 站点生成与部署说明
## markdown编辑站点
1. 参照docsify官方文档编写站点（https://docsify.js.org/#/quickstart）
	* 创建文件夹docs：mkdir docs
	* 进入docs：cd docs
	* 创建入口文件：[index.html](/docs/index.html)
	* 编写首页文件：[README.md](/docs/README.md)
2. 站点测试（已通过npm i docsify-cli -g指令本地安装过docsify）
	* 回到站点根目录：cd ..
	* 启动站点：docsify serve docs
	* 访问站点：http://localhost:3000
## 上传站点到github
1. 上传站点：git push
2. 如果仓库在github上，可在gitee上导入，防止云服务器拉取不到：https://gitee.com/projects/import/github/status
## 云服务器部署站点（腾讯云）
### 云服务器与域名绑定：购买云服务器与域名，并将域名绑定到服务器IP上
### k8s安装：云服务器安装k8s
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
6. 定义Ingress
	* yaml文件参照[okra-code-ing.yaml](/k8s/okra-code-ing.yaml)
7. 创建Ingress
	* kubectl create -f okra-code-ing.yaml
8. 定义nodeport接入外部流量
	* yaml文件参照[ingress-nginx.yaml](/k8s/ingerss-nginx.yaml)
9. 创建nodeport
	* kubectl create -f ingress-nginx.yaml 
