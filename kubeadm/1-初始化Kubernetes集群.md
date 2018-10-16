环境：  
  master，etcd：ip   
  node1: ip    
  node2: ip    

前提：（所有节点）    
  1、基于主机名通信： /etc/hosts  
  2、时间同步  
  3、关闭firewalld和iptables.service   
  systemctl stop firewalld  
  systemctl disable firewalld  
  4、关闭swap：swapoff -a   
  5、关闭selinux  
  vim /etc/sysconfig/selinux  
  将SELINUX修改为disabled  
  setenforce 0  
  6、打开桥接  
  cat <<EOF >  /etc/sysctl.d/k8s.conf  
  net.bridge.bridge-nf-call-ip6tables = 1  
  net.bridge.bridge-nf-call-iptables = 1  
  EOF  



  OS： Centos 7.x，extras仓库  

安装配置步骤：  
  1、etcd cluster，仅master节点  
  2、flannel，集群的所有节点  
  3、配置k8s的master：仅master节点；  
    kubernetes-master  
      启动的服务：  
        kube-apiserver，kube-scheduler，kube-controller-manager  
  4、配置k8s的个Node节点：  
      kubeneters-node  
      先设定启动docker服务；  
      启动k8s的服务：  
        kuber-proxy，kubelet  

kubeadm：  
  1、master，nodes：安装kubelet、kubeadm、docker  
  2、master：kubeadm init  
  3、nodes：kubeadn join  
reference：  
https://github.com/kubernetes/kubeadm/blob/master/docs/design/design_v1.10.md  
·


master:  
  1, 禁止firewalld  


  2,添加yum仓库  
  docker仓库：  
  cd /etc/yum.repos.d/  
  wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo  

  kubernetes仓库：  
  cat >> kubernetes.repo <<EOF  
  [kubernetes]  
  name=Kubernetes repo  
  baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/  
  gpgcheck=1  
  gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg  
  enable=1  
  EOF  

  yum repolist  
  检查仓库添加是否成功  

  把仓库复制到其他两个节点：  
  scp docker-ce.repo kubernetes.repo root@node1:/etc/yum.repos.d/    
  scp docker-ce.repo kubernetes.repo root@node2:/etc/yum.repos.d/  

  master:  
  yum install -y docker-ce kubelet kubeadm kubectl  

  node:  
  yum install -y docker-ce kubelet kubeadm  


  ~~因为某些原因，我们无法获取到google提供的镜像：  
  vim /usr/lib/systemd/system/docker.service  
  在[service]里添加：  
  Environment="HTTPS_PROXY=http://www.ik8s.io:10080"  
  重新加载docker daemon    
  systemctl daemon-reload~~  


  算了 ，你们想办法看看外面的世界吧：  
  我这里有两个教程，拿去：  
  https://blog.csdn.net/u012375924/article/details/78706910  


  启动docker：  
  systemctl start docker  
  systemctl enable docker  

  vim /etc/sysconfig/kubelet  
  KUBELET_EXTRA_ARGS="--fail-swap-on=false"  

  sysctl --system

##  这里一个大坑，systemctl start kubelet现在是无法启动的，执行下面的命令之后，kubelet服务正常，但是会卡在 Unable to update cni config: No networks found in /etc/cni/net.d   这种情况考虑是开了系统代理导致，由于可能是全局代理，导致 IP 访问API Server 也是不通的，通常建议此时关闭系统代理，仅仅开启 Docker 代理就足够应付接下来的安装。
  kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12 --ignore-preflight-errors=Swap

  启动kubelet
  systemctl enable kubectl

##  coredns服务无法running：
  plugin/loop: Seen "HINFO IN 2098247157494701651.7065398508885882649." more tha n twi

  解决方案：
  http://johng.cn/minikube-fatal-pluginloop-seen/  
