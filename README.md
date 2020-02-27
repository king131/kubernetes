### kubernetes 学习过程和脚本
***
![开发协作平台](https://github.com/king131/kubernetes/blob/master/kubeadm/images/2019-architecture.png)
两种搭建方式：  
[kubeadm](https://github.com/king131/kubernetes/tree/master/kubeadm)  
[组件式搭建](https://github.com/king131/kubernetes/blob/master/%E7%BB%84%E4%BB%B6%E5%BC%8F%E6%90%AD%E5%BB%BAk8s.md)（比较繁琐）  

#### CICD
---
Topology:
![topology](https://github.com/king131/kubernetes/blob/master/kubeadm/images/CICD.png)		

#### 写在最后
---
目前的情况，中小企业使用k8s的情况还是比较少的。最大的障碍来之于大家对k8s的不认知。当然也有现存业务服务之前调用的形式。同时，引入k8s的同时，也要引入很多的技术，这都为现在已经“运行良好“的系统带来未知数。和几个朋友沟通之后，他们对为了解决一些问题而引入新的变量表示不太愿意接受。  

现在k8s有一种技术隔离的态势，如果你要上k8s，那么可能需要开发团队，QA，运维人员拥有基本的k8s知识。这个在目前的阶段看来是比较难实现的。所以，就有了接下来的疑问：”如何让同事快速上手k8s？“  

目前，为了让同事更快的上手k8s，同时也为了让k8s变得透明。也兼顾当下的”制品库“的概念。”制品“将是我们的产出。之前业务image build至docker image 并push到repository已经不够了 ，我们需要的是deployment.yml这种类型的文件了，因此我们的“制品”将是helm的charts。
