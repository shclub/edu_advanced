# K8S Network

<br/>

kubernetes network 구조를  이해한다.  

<br/>


1. CNI  

2. Cilium  

3. iptables  

4. CoreDNS  

5. AWS EKS networking

6. Network Policy  

7. Headless 서비스  

<br/>

## CNI

<br/>

참고  
- https://captcha.tistory.com/78    

<br/>

### CNI ( Container Network Interface ) 란?

<br/>


CNCF(Cloud Native Computing Foundation)의 프로젝트 중 하나인 CNI는 컨테이너 간의 네트워킹을 제어할 수 있는 플러그인을 만들기 위한 표준입니다.   

다양한 형태의 컨테이너 런타임과 오케스트레이터 사이의 네트워크 계층을 구현하는 방식이 다양하게 분리되어 각자만의 방식으로 발전하게 되는 것을 방지하고 공통된 인터페이스를 제공하기 만들어 졌습니다.  

쿠버네티스에서는 Pod 간의 통신을 위해서 CNI 를 사용합니다. 

 
<br/>

쿠버네티스 뿐만 아니라 Amazon ECS, Cloud Foundry 등 컨테이너 런타임을 포함하고 있는 다양한 플랫폼들은 CNI를 사용하고 있습니다.   

쿠버네티스는 기본적으로 'kubenet' 이라는 자체적인 CNI 플러그인을 제공하지만 네트워크 기능이 매우 제한적인 단점이 있습니다.
 
<br/>

그 단점을 보완하기 위해, 3rd-party 플러그인을 사용하는데 그 종류에는 Flannel, Calico, Weavenet, NSX 등 다양한 종류의 3rd-party CNI 플러그인들이 존재합니다.    

Openshift 는 OVN-Kubernetes 라는 별도의 CNI 를 사용합니다.

<br/>

### CNI 필요성 

<br/>


<img src="./assets/cni_1.png" style="width: 80%; height: auto;"/>    


<br/>

예를들어, 위 그림과 같이 컨테이너 기반으로 동작하는 애플리케이션에 Web UI 컨테이너, Login 컨테이너, 장바구니 Cart 컨테이너 이렇게 멀티 호스트로 구성되어 있습니다. Web UI, Login, Cart 컨테이너는 서로 간에 당연히 통신이 되어야 할 겁니다.   

UI 컨테이너(172.17.0.2) 에서 Login 컨테이너(172.17.0.2) 로 통신을 하기 위해 트래픽을 보낸다고 가정합니다.   

정상적인 통신 패턴이라면 UI 컨테이너는 veth0 인터페이스를 통해 docker0 라는 브릿지 인터페이스를 타고 NAT처리 되어 worker node #1의 물리 인터페이스인 ens160의 IP(10.200.155.22) 로 나갑니다.   

그 후,  worker node #2 의 물리 인터페이스인 ens160 (10.200.155.23) 으로 들어와 docker0 브릿지 인터페이스를 통해 Login 컨테이너의 veth0으로 들어옵니다.   


그러나 위 그림에서 보듯이, 두 컨테이너의 IP는 동일하기 때문에 UI 컨테이너에서 Login 컨테이너로 통신을 시도하면 자기 자신인 UI 컨테이너 로컬로 통신을 시도 할 것입니다.   

위와 같은 멀티 호스트로 구성되어 있는 컨테이너 끼리 통신을 하기 위해서는 CNI가 반드시 설치되어 있어야 합니다.  

<br/>

<img src="./assets/cni_2.png" style="width: 80%; height: auto;"/>    

<br/>

CNI는 Calico, Weavenet, AWS CNI 등 매우 다양한 종류가 있습니다.  

위 그림에서는 weave net이라고 하는 CNI가 브릿지 인터페이스를 만들고 컨테이너 네트워크 대역대를 나눠주며 라우팅 테이블까지 생성하여 UI 컨테이너가 Login, Cart 컨테이너로 통신하는데 전혀 문제 없도록 지원합니다.   

<br/>

### CNI에서 사용되는 네트워크 모델  

<br/>

CNI Provider는 VXLAN(Virtual Extensible Lan), IP-in-IP 과 같은 캡슐화된 네트워크 모델 또는 BGP (Border Gateway Protocol)와 같은 캡슐화 되지 않은 네트워크 모델을 사용하여 네트워크 패브릭을 구현합니다.  

<br/>

### CNI 3rd-Party 플러그인 종류 및 지원하는 기능

<br/>

<img src="./assets/cni_3.png" style="width: 80%; height: auto;"/>    


<br/>

## Cilium

<br/>

참고   
- https://malwareanalysis.tistory.com/288

<br/>

### Cilium 등장 

<br/>

Cilium은 공식문서 소개 글처럼 linux eBPF를 이용한 고성능 네트워킹 솔루션입니다.  

쿠버네티스에서는 CNI로 동작합니다. 고성능 네트워킹 초점을 둔 이유는 iptables을 이용한 쿠버네티스 트래픽 라우팅의 단점을 보완하려는 목적이 있기 때문입니다.  


<br/>

### iptables 단정

<br/>

iptables의 특징때문에 파드와 서비스 갯수의 합이 몇천개, 몇만개 이상이라면 네트워크 성능이 낮아집니다.

<br/>

첫 번째 특징은, iptables는 일치한 iptables 규칙을 찾을 때까지 모든 규칙을 평가하는 특징이 있습니다. 파드와 서비스가 많아질수록 규칙을 찾는 시간이 지연되므로 네트워크 성능에 영향을 끼칩니다. 

<br/>


kube-proxy가 iptables모드를 사용하면, 파드 또는 서비스로 가는 트래픽은 iptables 규칙(rule)에 따라서 흘러갑니다. 파드/서비스가 생성될 때마다 iptables규칙이 여러개 생성됩니다. 예를 들어 파드 1개가 생성되면 iptables가 5개 이상 생성될 수 있습니다. 파드/서비스 갯수가 많아질수록 iptables는 기하급수적으로 증가합니다.   

결국, 일치하는 iptables를 찾을 때까지 수많은 iptables 규칙을 검사합니다. 

<br/>

두번째 특징은 iptaebls규칙 추가방법입니다. 새로운 규칙이 추가될 때마다 기존의 전체 규칙을 바꿔야 합니다. 데이터베이스 행(row)을 추가하는 방법이 아닙니다.   

이러한 결함을 "Incremental Update"기능 미지원이라고 부릅니다. 블로그 인용에 따르면 5000개의 서비스가 존재하는 상태에서 iptables 규칙을 추가하면 11분 정도가 소요된다고 합니다.  

<br/>

### route 경로  조회

<br/>

worker node 2번 서버에 접속을 한다.  

```bash
root@edu25:~# worker.sh
Worker Node OKD-2 connect.
core@okd-2.okd4.ktdemo.duckdns.org's password:
Fedora CoreOS 37.20230218.3.0
Tracker: https://github.com/coreos/fedora-coreos-tracker
Discuss: https://discussion.fedoraproject.org/tag/coreos

Last login: Sun Nov  5 18:55:37 2023 from 211.253.38.88
```  

<br/>

iptables 조회를 해봅니다. 

```bash
sudo iptables-save | grep 443
```  

Output
```bash
-A KUBE-SERVICES -d 172.30.70.227/32 -p tcp -m comment --comment "openshift-machine-api/machine-api-operator-webhook:https has no endpoints" -m tcp --dport 443 -j REJECT --reject-with icmp-port-unreachable
-A KUBE-SERVICES -d 172.30.199.52/32 -p tcp -m comment --comment "opensearch/opentelemetry-opentelemetry-operator-webhook has no endpoints" -m tcp --dport 443 -j REJECT --reject-with icmp-port-unreachable
-A KUBE-SERVICES -d 172.30.17.250/32 -p tcp -m comment --comment "opensearch/opentelemetry-opentelemetry-operator:https has no endpoints" -m tcp --dport 8443 -j REJECT --reject-with icmp-port-unreachable
-A KUBE-SEP-223KLBGHH3GIBFEC -p tcp -m comment --comment "openshift-machine-api/cluster-baremetal-operator-service:https" -m tcp -j DNAT --to-destination 10.128.0.17:8443
-A KUBE-SEP-25KJPLCOM5CAULPE -s 10.128.0.10/32 -m comment --comment "openshift-operator-lifecycle-manager/packageserver-service:5443" -j KUBE-MARK-MASQ
-A KUBE-SEP-25KJPLCOM5CAULPE -p tcp -m comment --comment "openshift-operator-lifecycle-manager/packageserver-service:5443" -m tcp -j DNAT --to-destination 10.128.0.10:5443
-A KUBE-SEP-2LD5ANDIU5KY663M -p tcp -m comment --comment "openshift-machine-api/control-plane-machine-set-operator:https" -m tcp -j DNAT --to-destination 10.128.0.37:9443
-A KUBE-SEP-3V2YDWSK4J7LMC65 -p tcp -m comment --comment "openshift-user-workload-monitoring/grafana-operator-controller-manager-metrics-service:https" -m tcp -j DNAT --to-destination 10.130.0.162:8443
...
```

<br/>

### 라우팅 단순화를 위한 eBPF 활용

<br/>

많은 연구자와 실무자들이 좋은 방법을 찾기 위해 방법을 연구했었고, 지금 많이 주목받고 있는 주제가 eBPF입니다. eBPF는 BPF의 확장개념입니다.

<br/>

공식문서에서 소개된것 처럼 eBPF는 두가지 키워드가 있습니다. ①커널 소스코드 또는 모듈로드 없이 기능 확장과 ②실행될 때 샌드박스로 작업이 수행됩니다. 간단하게 리눅스에서 지원하는 필터기능인데 샌드박스로 실행된다라고 이해하면 됩니다.   

eBPF는 이벤트 hook기반으로 동작합니다.   

<br/>

<img src="./assets/cni_4.png" style="width: 80%; height: auto;"/>    


<br/>

eBPF는 다양한 분야에서 활용되고 연구중입니다. 네트워크 분야에서는 리눅스 네트워크 스택에 eBPF를 활용하여 사용자 정의 기능을 추가하거나 커널레벨 네트워크 레이어 흐름을 수정할 수 있습니다.  

<br/>


<img src="./assets/cni_5.png" style="width: 80%; height: auto;"/>    

<br/>

컨테이너 분야에서는 컨테이너로 트래픽이 라우팅 되는 과정을 eBPF를 이용합니다. 사용자가 직접 컨테이너에 eBPF를 적용하기 어려우니, Cilium이 쉽게 적용할 수 있도록 도와줍니다.

<br/>

<img src="./assets/cni_6.png" style="width: 80%; height: auto;"/>    

<br/>

쿠버네티스에 분야에서는 Cilium이 CNI로 도입되어서 쿠버네티스 클러스터 네트워킹 역할을 담당합니다.

<br/>

<img src="./assets/cni_7.png" style="width: 80%; height: auto;"/>    

<br/>

출처: https://cilium.io/blog/2018/04/17/why-is-the-kernel-community-replacing-iptables
출처: https://blog.naver.com/kangdorr/222593265958


<br/>

## iptables

<br/>

참고 : https://www.kangtaeho.com/69

<br/>

### 실습   

<br/>

nginx 서비스를 조회해 본다.  

```bash
root@edu25:~# kubectl get svc
NAME                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
backend                      ClusterIP   172.30.206.211   <none>        8080/TCP   6d1h
backend-springboot           ClusterIP   172.30.106.41    <none>        80/TCP     6d6h
elastic-agent-integrations   ClusterIP   172.30.65.7      <none>        8125/UDP   8d
external-node-exporter       ClusterIP   172.30.94.58     <none>        9100/TCP   12d
frontend                     ClusterIP   172.30.211.233   <none>        8080/TCP   6d1h
frontend-react               ClusterIP   172.30.240.142   <none>        80/TCP     7d23h
nginx                        ClusterIP   172.30.81.39     <none>        80/TCP     6d12h
```  

<br/>
worker node에서  nginx 서비스의 ip로 iptables를 조회를 한다.  

```bash
[core@okd-2 ~]$ sudo iptables -t nat -S | grep 172.30.81.39
-A KUBE-SERVICES -d 172.30.81.39/32 -p tcp -m comment --comment "edu25/nginx:http cluster IP" -m tcp --dport 80 -j KUBE-SVC-5OS5BND5KMKOINHH
-A KUBE-SVC-5OS5BND5KMKOINHH -d 172.30.81.39/32 ! -i tun0 -p tcp -m comment --comment "edu25/nginx:http cluster IP" -m tcp --dport 80 -j KUBE-MARK-MASQ
```

<br/>

체인을 따라서  다시 KUBE-SVC 시작하는 값으로 추적을 한다.   
아래에 나오는 IP는 pod 의 IP 이다.  

```bash
[core@okd-2 ~]$ sudo  iptables -t nat -S | grep KUBE-SVC-5OS5BND5KMKOINHH
-N KUBE-SVC-5OS5BND5KMKOINHH
-A KUBE-SERVICES -d 172.30.81.39/32 -p tcp -m comment --comment "edu25/nginx:http cluster IP" -m tcp --dport 80 -j KUBE-SVC-5OS5BND5KMKOINHH
-A KUBE-SVC-5OS5BND5KMKOINHH -d 172.30.81.39/32 ! -i tun0 -p tcp -m comment --comment "edu25/nginx:http cluster IP" -m tcp --dport 80 -j KUBE-MARK-MASQ
-A KUBE-SVC-5OS5BND5KMKOINHH -m comment --comment "edu25/nginx:http -> 10.128.3.121:80" -m statistic --mode random --probability 0.20000000019 -j KUBE-SEP-BGX236REKJDO52A7
-A KUBE-SVC-5OS5BND5KMKOINHH -m comment --comment "edu25/nginx:http -> 10.129.1.208:80" -m statistic --mode random --probability 0.25000000000 -j KUBE-SEP-ND2BOW5Z7Q3X7BXE
-A KUBE-SVC-5OS5BND5KMKOINHH -m comment --comment "edu25/nginx:http -> 10.129.2.234:80" -m statistic --mode random --probability 0.33333333349 -j KUBE-SEP-477D6XEJNUO7OHCM
-A KUBE-SVC-5OS5BND5KMKOINHH -m comment --comment "edu25/nginx:http -> 10.130.1.86:80" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-ACO4VGHVUKOBOOEA
-A KUBE-SVC-5OS5BND5KMKOINHH -m comment --comment "edu25/nginx:http -> 10.131.0.17:80" -j KUBE-SEP-J2HVM7YGNXR4UJJO
```

<br/>

Pod 의 IP 를 조회해 본다.    
iptables 의 IP와 정확히 일치한다.  

```bash
root@edu25:~# kubectl get po -o wide | grep nginx-deployment
nginx-deployment-56569bbd7d-2rdxf                       1/1     Running   0              111m    10.129.2.234   okd-6.okd4.ktdemo.duckdns.org   <none>           <none>
nginx-deployment-56569bbd7d-8b5k8                       1/1     Running   0              27s     10.129.1.208   okd-3.okd4.ktdemo.duckdns.org   <none>           <none>
nginx-deployment-56569bbd7d-8rnp4                       1/1     Running   0              67s     10.130.1.86    okd-4.okd4.ktdemo.duckdns.org   <none>           <none>
nginx-deployment-56569bbd7d-cqzqx                       1/1     Running   0              67s     10.128.3.121   okd-5.okd4.ktdemo.duckdns.org   <none>           <none>
nginx-deployment-56569bbd7d-xrjsm                       1/1     Running   0              8s      10.131.0.17    okd-2.okd4.ktdemo.duckdns.org   <none>           <none>
```


<br/>

## CoreDNS

<br/>

참고   
- https://malwareanalysis.tistory.com/267

<br/>

### 향후 추가

<br/>


<br/>

## AWS EKS networking

<br/>

참고   
- https://wolf-sheep.tistory.com/208 

<br/>

### AWS VPC CNI

<br/>

Networking plugin for pod networking in Kubernetes using ENIs(Elastic Network Interfaces) on AWS  

<img src="./assets/aws_cni_1.png" style="width: 80%; height: auto;"/>    

<br/>

### VPC CNI 의 특징

<br/>


- 노드의 IP 대역과 파드의 IP 네트워크 대역이 같아서 직접 통신이 가능하다!!  
- VPC와 통합을 통해, VPC Flow logs, 라우팅 정책, 보안그룹 활용 가능  
- VPC 대역 내에서 각각의 파드에 IP 할당  
- VPC ENI에 미리 할당된 IP(Local-IPAM Warm IP Pool)를 파드에서 사용 가능   

<br/>

<img src="./assets/aws_cni_2.png" style="width: 80%; height: auto;"/>    

<br/>

VPC CNI는 Overlay Network를 통하지 않기 때문에 다른 CNI 대비 좀더 빠르다.

<img src="./assets/aws_cni_3.png" style="width: 80%; height: auto;"/>    

<br/>

Amazon VPC CNI, 이제 Kubernetes NetworkPolicy 시행 지원  
- https://aws.amazon.com/ko/about-aws/whats-new/2023/08/amazon-vpc-cni-kubernetes-networkpolicy-enforcement/


<br/>

## Network Policy  

<br/>

참고    
- https://lifeoncloud.kr/entry/Network-Policy   
- https://waspro.tistory.com/768  

<br/>

### Network Policy 란

<br/>

쿠버네티스의 네트워크를 레이블, IP주소, 포트 수준에서 제어할 수 있는 네임스페이스 단위의 리소스  

<br/>


- 쿠버네티스의 네트워크를 레이블, IP주소, 포트 수준에서 제어할 수 있다.  
- Network Policy를 적용할 Pod를 식별하는 방법으로 Pod, 네임스페이스, IP주소를 조합하여 만들 수 있다.  
- 특정 Network Policy를 적용한 Pod는 해당 Network Policy를 제외한 트래픽은 모두 deny한다.    
  - Network Policy가 없다면, 네임스페이스의 모든 트래픽이 열려있다.(default-allow)  
  - Network Policy가 있다면, 해당 Network Policy의 영향을 받는 Pod는 해당 Network Policy 를 제외하고 나머지 트래픽은 전부 막힌다.(default-deny)  

<br/>

특징  
- Network Policy는 Pod에만 적용된다.
- 쿠버네티스 클러스터에는 Network Policy가 설정되어 있지 않은 것이 기본값이다.

<br/>

### Network Policy의 spec 구성요소


<br/>

policyTypes
- Network Policy의 트래픽 종류를 지정한다.  

podSelector
- Network Policy가 적용될 Pod를 지정한다.

egress  
- 아웃바운드 트래픽 허용 정책을 정의한다.
- egress 항목이 {}과 같이 비어있으면 모두 허용이다.

ingress  
- 인바운드 트래픽 허용 정책을 정의한다.
- ingress 항목이 {}과 같이 비어있으면 모두 허용이다.

<br/>

### Network Policy 세가지 식별자(identifier) 종류

<br/>

podSelector
- 특정 레이블을 가진 Pod에서 들어오는/나가는 통신 허용

namespaceSelector
- 특정 네임스페이스에 있는 Pod에서 들어오는/나가는 통신 허용
- 특정 레이블을 가진 네임스페이스의 모든 Pod들을 대상으로 적용한다.

ipBlock
- 특정 CIDR에서 들어오는/나가는 통신 허용

<br/>

Network Policy Spec  

<img src="./assets/k8s_network_policy_1.png" style="width: 80%; height: auto;"/>

<br/>

Network Policy 세부 Spec  

<img src="./assets/k8s_network_policy_2.png" style="width: 80%; height: auto;"/>

<br/>

#### Network Policy 기본 정책  

<br/>

> (기본값) 모든 Ingress와 모든 Egress 트래픽 허용    

`podSelector: {}` 라는 것은 모든 Pod에 적용

<br/>

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-allow-all
spec:
  policyTypes:
  - Ingress
  - Egress
  podSelector: {}
  ingress:
  - {}
  egress:
  - {}
```  

<br/>

> 모든 Ingress와 모든 Egress 트래픽 거부

<br/>

policyTypes은 Ingress, Egress지만,
허용하는 조건을 명시하는 ingress와 egress 항목 모두 없으므로, 허용되는 트래픽이 인바운드, 아웃바운드 모두 없다.  

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  policyTypes:
  - Ingress
  - Egress
  podSelector: {}
```  

<br/>

#### 실습 ( Network Policy )  

<br/>

각 namespace 에  nginx pod와 service를 생성한다.  

```bash
root@edu25:~# cat nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: ghcr.io/shclub/nginx:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    targetPort: 80
    name: http
  selector:
    app: nginx
  type: ClusterIP
root@edu25:~# kubectl apply -f nginx.yaml
```

<br/>

오픈 해줄 namespace 의 label을 확인 합니다.   

```bash
root@edu25:~# kubectl edit namespace edu24
```  

<br/>  

`kubernetes.io/metadata.name: edu24` 가 `key : value` 형태로 구성 되어 있습니다.  

```bash
 labels:
    kubernetes.io/metadata.name: edu24
    pod-security.kubernetes.io/audit: privileged
    pod-security.kubernetes.io/audit-version: v1.24
    pod-security.kubernetes.io/warn: privileged
    pod-security.kubernetes.io/warn-version: v1.24
```  

<br/>

network_policy 를 생성합니다.  

```bash
root@edu25:~# cat network_policy_nginx.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: nginx-policy
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: edu24
        - podSelector:
            matchLabels:
              app: nginx
      ports:
        - protocol: TCP
          port: 80
root@edu25:~# kubectl apply -f network_policy_nginx.yaml
```  

<br/>

다른 namespace 에서 pod에 들어가서 curl 로 서비스를 확인 해봅니다.  

```bash
root@edu25:~# kubectl exec -it nginx-deployment-56569bbd7d-bxl5k sh -n edu24
```  

<br/>

정상적으로 접속이 됩니다.  

```bash
# curl nginx.edu25
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

<br/>


## Headless 서비스  

<br/>

참고    
- https://malwareanalysis.tistory.com/493 

<br/>

### Headless서비스란?

<br/>


Headless서비 스는 `ClusterIP가 없는 서비스` 입니다. kubectl로 서비스를 조회하면 ClusterIP가 None으로 표시됩니다.    

```bash
[root@bastion ~]# kubectl get svc -n elastic
NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
apm-server                      ClusterIP   172.30.49.195    <none>        8200/TCP            25d
elastic-agent-integrations      ClusterIP   172.30.191.250   <none>        8125/UDP            9d
elasticsearch-master            ClusterIP   172.30.169.170   <none>        9200/TCP,9300/TCP   18d
elasticsearch-master-headless   ClusterIP   None             <none>        9200/TCP,9300/TCP   18d
fleet-server                    ClusterIP   172.30.162.24    <none>        8220/TCP            25d
kibana-kibana                   ClusterIP   172.30.70.14     <none>        5601/TCP            23d
```  

<br/>

### Headless서비스와 일반 서비스 비교  

<br/>

Headless서비스는 kube-proxy가 처리하지 않아서 `서비스` 를 이용한 로드 밸런싱 기능은 지원하지 않습니다.    


일반 서비스는 kube-proxy가 처리하므로 서비스가 트래픽을 로드밸런싱 합니다. 하지만 headless서비스는 서비스를 거치지 않고 pod에 직접 접근합니다 .

<img src="./assets/headless_1.png" style="width: 80%; height: auto;"/>    


먼저 nginx pod를 하나 생성 합니다.  

```bash
kubectl run my-nginx --image=nginx --port=80 --labels="app=nginx-for-svc"
```  

<br/>

headless service를 생성합니다.      

`clusterIP: None` 입니다.

```bash
[root@bastion security]# cat headless-manifest.yaml
# headless.yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  type: ClusterIP
  clusterIP: None
  selector:
    app: nginx-for-svc
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

<br/>

서비스를 조회합니다.   

```bash
[root@bastion security]# kubectl get svc
NAME                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
headless-service             ClusterIP   None             <none>        80/TCP     8m42s
nginx                        ClusterIP   172.30.81.39     <none>        80/TCP     6d22h
```  

<br/>

POD의 IP 를 확인합니다.  

```bash
[root@bastion security]# kubectl get po -o wide | grep my-nginx
my-nginx                                                1/1     Running   0              10m     10.129.1.214   okd-3.okd4.ktdemo.duckdns.org   <none>           <none>
```  

<br/>

netshoot 라는 네트웍 진단을 위한 POD를 하나 생성합니다.   

```bash
[root@bastion security]# cat netshoot_pod.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netshoot-pod
spec:
  replicas: 1
  selector:
    matchLabels:
      app: netshoot-pod
  template:
    metadata:
      labels:
        app: netshoot-pod
    spec:
      containers:
      - name: netshoot-pod
        image: ghcr.io/shclub/netshoot
        command: ["tail"]
        args: ["-f", "/dev/null"]
      terminationGracePeriodSeconds: 0
```  

<br/>

netshoot pod에 접속하여 nslookup을 해봅니다.    

```bash
kubectl exec -it netshoot-pod-85b5dfb564-fxmmt sh
```

<br/>

먼저 headless 서비스를 호출해 봅니다.  
pod의 ip 가 나오는것을 확인 할수 있습니다.  

```bash
~ # nslookup headless-service.edu25.svc
Server:		172.30.0.10
Address:	172.30.0.10#53

Name:	headless-service.edu25.svc.cluster.local
Address: 10.129.1.214
```  
<br/>

일반 서비스를 호출 합니다.  
service의 ip가 나오고 `kube-proxy` 가 로드밸런싱을 합니다.  

```bash
~ # nslookup nginx.edu25.svc
Server:		172.30.0.10
Address:	172.30.0.10#53

Name:	nginx.edu25.svc.cluster.local
Address: 172.30.81.39
```

<br/>

###  headless는 언제 사용될까?

<br/>

headless서비스는 `pod`를 그룹으로 관리하고 POD IP목록을 직접 조회가 필요한 아키텍처에서 사용합니다. master/slave, 클러스터 등 아키텍처를 가지고 있는 데이터베이스에서 많이 사용합니다.  

예로 Elsaticsearch helm chart에서 headless서비스를 사용하여 master목록을 관리합니다.  

<br/>

<img src="./assets/headless_2.png" style="width: 80%; height: auto;"/>    

<br/>


## 참고 자료  

<br/>

- SNAT/DNAT : https://tech.kakao.com/2021/03/03/network-node-manager/   

- pod networking : https://jonnung.dev/kubernetes/2020/02/24/kubernetes-pod-networking/  

- kube-proxy + hidden network : https://medium.com/@seifeddinerajhi/kube-proxy-and-cni-the-hidden-components-of-kubernetes-networking-eb30000bf87a  

- vxlan : https://ssup2.github.io/theory_analysis/Overlay_Network_VXLAN/   
  

- Network Policy animation : https://ahmet.im/blog/kubernetes-network-policy/  
- Network Policy : https://kmaster.tistory.com/70  

- 쿠버네티스(Kubernetes) 네트워크 정리 ( k8s 네트웍 4가지 ) : https://medium.com/finda-tech/kubernetes-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%A0%95%EB%A6%AC-fccd4fd0ae6

- headless : https://malwareanalysis.tistory.com/339  

 
<br/>
