# 站点：home.okracode.com
# 技术：markdown + docsify + k8s
# 站点生成与部署说明
## markdown编辑站点
1. 参照docsify官方文档编写站点（https://docsify.js.org/#/quickstart）
* 创建文件夹docs：mkdir docs
* 创建入口文件：[index.html](/docs/index.html)
* 编写首页文件：[README.md](/docs/README.md)
2. 站点测试（已通过npm i docsify-cli -g指令本地安装过docsify）
* docsify serve docs
## 上传站点到github
1. 上传站点：git push
2. 如果仓库在github上，可在gitee上导入，防止云服务器拉取不到：https://gitee.com/projects/import/github/status
## git-sync同步站点
## docsify解析站点
