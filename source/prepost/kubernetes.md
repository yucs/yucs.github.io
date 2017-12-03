

# kubeadm安装实践
- kubernetes架构图：  
  ![](./picture/k8s-arch.png)
- [Kubernetes指南 之 kubeadm部署](https://github.com/feiskyer/kubernetes-handbook/blob/master/deploy/kubeadm.md)

## 原理
- [Kubernetes指南 之 kubeadm工作原理](https://github.com/feiskyer/kubernetes-handbook/blob/master/components/kubeadm.md)
- [kubeadm工作机制分析](http://blog.csdn.net/waltonwang/article/details/70162993)
- [源码分析之kubeadm](http://blog.csdn.net/u010278923/article/details/70225173)

# 视屏教程
[ibm Kubernentes系列](https://www.ibm.com/developerworks/community/wikis/home?lang=zh#!/wiki/W30b0c771924e_49d2_b3b7_88a2a2bc2e43/page/IBM%E5%BC%80%E6%BA%90%E6%8A%80%E6%9C%AF%E5%BE%AE%E8%AE%B2%E5%A0%82)
 
# 原理架构
  [Kubernetes指南](https://github.com/feiskyer/kubernetes-handbook)
 
# 组件核心之etcd 

 api接口
 
- [Kubernetes API 分析 ( Kube-apiserver )](https://www.kubernetes.org.cn/3119.html)
- [Kubernetes调度详解](http://dockone.io/article/2885)
 

# 编写CNI
背景：
  
知其然所以然：

[kubernetes 简介：kubelet 和 pod](http://cizixs.com/2016/10/25/kubernetes-intro-kubelet)



原理分析：
kubelet 
 - 它的核心工作是监听 apiserver，一旦发现当前节点的 pod 配置发生变化，就根据最新的配置执行响应的动作，保证运行的 pod 状态和期望的一致。
 - 定时汇报当前节点的状态给 apiserver，以供调度的时候使用。
 - 镜像和容器的清理工作，保证节点上镜像不会占满磁盘空间，退出的容器不会占用太多资源
- 运行 HTTP Server，对外提供节点和 pod 信息，如果在 debug 模式下，还包括调试信息

# kubelet源码分析：
[kubelet 源码分析：启动流程](http://cizixs.com/2017/06/06/kubelet-source-code-analysis-part-1) （v1.5.0版本)

  - 解析参数配置信息等初始化准备后， 创建kubeDeps这个对象：
    - 其实它内部保存了 kubelet 各个重要组件的对象，之所以要把它作为参数传递，是为了实现 dependency injection。简单地说，就是把 kubelet 依赖的组件对象作为参数传进来，这样可以控制 kubelet 的行为。比如在测试的时候，只要构建 fake 的组件实现，就能很轻松进行测试。KubeDeps 包含的组件很多，下面列出一些：
   - CAdvisorInterface：提供 cAdvisor 接口功能的组件，负责收集容器的监控信息
   - DockerClient：docker 客户端，用来和 docker 交互
	- KubeClient：apiserver 客户端，用来和 api server 通信
	- Mounter：执行 mount 相关操作
	- NetworkPlugins：网络插件，执行网络设置工作
	- VolumePlugins：volume 插件，执行 volume 设置工作
  - RunKubelet 函数:
    - 初始化各个对象，比如 eventBroadcaster，这样就能给 apiserver 发送 kubelet 的事件
    - 通过 builder 创建出来 Kubelet对象（pkg/kubelet/kubelet.go#NewMainKubelet）
    - 根据运行模式，运行 Kubelet对象，各种组件以 goroutine运行启动
       - 异步事件驱动：syncLoop 是 kubelet 的主循环方法，它从不同的管道（文件、URL 和 apiserver）监听变化，并把它们汇聚起来。当有新的变化发生时，它会调用对应的处理函数，保证 pod 处于期望的状态。
  -  kubelet对象 中包含的重要对象：
    - podConfig：这个对象里面会从文件、网络和 apiserver 三个来源中汇聚节点要运行的 pod 信息，并通过管道发送出来，读取这个管道就能获取实时的 pod 最新配置
    - runtime：容器运行时，对容器引擎（docker 或者 rkt）的一层封装，负责调用容器引擎接口管理容器的状态，比如启动、暂停、杀死容器等
    - probeManager：如果 pod 配置了状态监测，那么 probeManager 会定时检查 pod 是否正常工作，并通过 statusManager 向 apiserver 更新 pod 的状态
    - volumeManager：负责容器需要的 volume 管理。检测某个 volume 是否已经 mount、获取 pod 使用的 volume 等
    - podWorkers：具体的执行者，每次有 pod 需要更新的时候都会发送给它
    - podManager：缓存了 pod 的信息，是所有需要该信息都会去访问的地方
    - nodeLister：能够读取 apiserver 中节点的信息
    
[kubelet 源码分析：pod 新建流程](http://cizixs.com/2017/06/07/kubelet-source-code-analysis-part-2)（v1.5.0版本)：

 - pod的创建顺序留意点：
   - 创建 pod 的数据目录，存放 volume 和 plugin 信息
   - 如果定义了 PV，等待所有的 volume mount 完成（volumeManager 会在后台做这些事情）
   - 如果有 image secrets，去 apiserver 获取对应的 secrets 数据
   - 调用 container runtime 的 SyncPod 方法，去实现真正的容器创建逻辑
      -  通过 docker 创建出来一个运行的 pause 容器。
	  -  网络配置：如果 pod 是主机模式，容器也是；其他情况下，**容器会使用 None 网络模式，让 kubelet 的网络插件自己进行网络配置**。
	  - pod-infra-container

 
 其他参考：
 [kubelet源码分析－启动运行与信息处理](http://m.blog.csdn.net/zhonglinzhang/article/details/75005851)
 
 
 [Kubernetes的网络接口CNI及灵雀云的实践](http://dockone.io/article/2901)
 
   - 运维人员视角，在传统运维工作强调对IP要有很强的管控（银行等），POD需要固定IP:
     - 于运维来说，网络方面是很重要的资源，要对IP进行强管控，服务来回飘，会让他的安全感下降很多。
     - 运维服务有很多基于IP的东西，有流量和突发的监控，如果你服务的IP一直变化，通过这个IP它很难用到这个服务，相当于IP的监控就没有意义，因为根本不知道IP流量上去了是哪个服务的，很难对应到这个事。
     -  还有是对于IP安全策略没有办法做。
  - kubelet 与 CNI plugin调用逻辑图：
    ![cni](/picture/CNI.png)
  - K8S 不管你的IP,需要单独的管理.
  - 定制化网络管理，需要自己实现IPAM，plugin.


[CNI：容器网络接口](http://cizixs.com/2017/05/23/container-network-cni) 

- 网络插件是独立的可执行文件，被上层的容器管理平台调用。网络插件只有两件事情要做：把容器加入到网络以及把容器从网络中删除。
- 调用插件的数据通过两种方式传递：环境变量和标准输入。

- kubernetes 使用了 CNI 网络插件之后 工作流程：
	- kubernetes 先创建 pause 容器生成对应的 network namespace
	- 调用网络 driver（因为配置的是 CNI，所以会调用 CNI 相关代码）
	- CNI driver 根据配置调用具体的 cni 插件
	cni 插件给 pause 容器配置正确的网络
	- pod 中其他的容器都是用 pause 的网络
  
- https://github.com/containernetworking/cni/blob/master/SPEC.md
http://cizixs.com/ 
http://blog.csdn.net/WaltonWang/article/list/1

[从源码看kubernetes与CNI Plugin的集成](http://blog.csdn.net/waltonwang/article/details/72669826)
# 网络基本常识