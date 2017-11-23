# 安装实践

- [Kubernetes 1.8.x 全手动安装教程](https://www.kubernetes.org.cn/3096.html) 

##### 特点：
  - 手动安装，熟悉了解 Kubernetes 创建流程和细节，加深理解；
  - 组件程序从 各项目github上发布的release 下载;
  - 配置文件在 github.com/kairen/kairen.github.io 博主项目下维护；

##### 注意点：
- 部署过程，容易让人眼乱的是证书部分，参考[Kubernetes安装之证书验证](https://www.kubernetes.org.cn/1861.html)；
-  出问题，通过journalctl -f -u XX 等查看服务日记；(eg:下载镜像可能要翻墙)
- 边安装边看架构图：  
  ![](./picture/k8s-arch.png)

##### 其他部署参考：
[和我一步步部署 kubernetes 集群](https://github.com/opsnull/follow-me-install-kubernetes-cluster)
   - 
 
# 原理架构
 主要参考[Kubernetes指南](https://kubernetes.feisky.xyz/)
 特点：
  -
 


# 源码分析
 