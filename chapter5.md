# K8S Network

kubernetes network 구조를  이해한다.  

<br/>



1. K8S Network  

참고 자료   

exteranltrafficecluster 인가? 커피 고래
SNAT/DNAT : https://tech.kakao.com/2021/03/03/network-node-manager/  

cni란 : https://captcha.tistory.com/78  

pod networking : https://jonnung.dev/kubernetes/2020/02/24/kubernetes-pod-networking/  


kube-proxy + hidden network : https://medium.com/@seifeddinerajhi/kube-proxy-and-cni-the-hidden-components-of-kubernetes-networking-eb30000bf87a

k8s network 살펴보기  : https://www.kangtaeho.com/69  


vxaln   

https://ssup2.github.io/theory_analysis/Overlay_Network_VXLAN/ 
  

  network policy animation : https://ahmet.im/blog/kubernetes-network-policy/


network policy 

  https://kmaster.tistory.com/70


https://medium.com/finda-tech/kubernetes-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%A0%95%EB%A6%AC-fccd4fd0ae6


k8s 네트웍 3종류 설명 : https://medium.com/finda-tech/kubernetes-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%A0%95%EB%A6%AC-fccd4fd0ae6

headless : 
headless : https://malwareanalysis.tistory.com/339


eks netwoking : https://wolf-sheep.tistory.com/208  

```bash
[root@bastion elastic]# kubectl api-resources -o wide
NAME                                  SHORTNAMES                         APIVERSION                                    NAMESPACED   KIND                                 VERBS
bindings                                                                 v1                                            true         Binding                              [create]
componentstatuses                     cs                                 v1                                            false        ComponentStatus                      [get list]
configmaps                            cm                                 v1                                            true         ConfigMap                            [create delete deletecollection get list patch update watch]
endpoints                             ep                                 v1                                            true         Endpoints                            [create delete deletecollection get list patch update watch]
events                                ev                                 v1                                            true         Event                                [create delete deletecollection get list patch update watch]
limitranges                           limits                             v1                                            true         LimitRange                           [create delete deletecollection get list patch update watch]
namespaces                            ns                                 v1                                            false        Namespace                            [create delete get list patch update watch]
nodes                                 no                                 v1                                            false        Node                                 [create delete deletecollection get list patch update watch]
persistentvolumeclaims                pvc                                v1                                            true         PersistentVolumeClaim                [create delete deletecollection get list patch update watch]
persistentvolumes                     pv                                 v1                                            false        PersistentVolume                     [create delete deletecollection get list patch update watch]
pods                                  po                                 
...
```  

<br/>
