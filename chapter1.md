# OKD  설치 방법
 
Redhat Openshift 의  `오픈소스 버전`인 OKD Cluster 를  생성하고 사용해본다.   ( OKD 4.12 버전 기준 : k8s 1.25 )     

<br/>

OKD 설명 참고 :  https://velog.io/@_gyullbb/OKD-%EA%B0%9C%EC%9A%94  

   
1. k8s & OKD 구조 

2. k8s 주요 기능

<br/>

## 1. k8s & OKD 구조  

<br>

간단 설명 : https://youtu.be/TlHvYWVUZyc?si=_74XT6vE4BC97w3J  

<br/>

https://www.duckdns.org/ 로 접속하여 가입을 하고 본인의 공유기의 WAN IP 와 도메인을 매칭 시킨다. WAN IP가 바뀌어도 주기적으로 update 된다.  

<br/>

ktdemo.duckdns.org 로 생성 을 한다. ip 를 변경하고 싶으면 ip를 수동으로 입력하고 update 한다.

<img src="./assets/duckdns_create.png" style="width: 80%; height: auto;"/>

<br/>

위의 도메인이 우리가 설치하는 base 도메인이 된다.  

<br/>

## 2. k8s 주요 기능

<br/>

참고자료 

- Deployment vs Statefulset vs Daemonset :    
https://youtu.be/30KAInyvY_o?si=GW9tbgiZeuG48bVv
- Node Selector vs Node Affinity : https://youtu.be/rX4v_L0k4Hc?si=iRGq9gjitRoFmy04  
- Ingress : https://youtu.be/1BksUVJ1f5M?si=01rbXov20XJdEPJh

<br/>

### route vs ingress

<br/>

route : haproxy ( L7 )
ingress : nginx 를 주로 사용

<br/>

### headless 서비스

<br/>

악분일상 : https://youtu.be/If03sN4isO4?si=sn43EXafiVFJ7JLH




Prometheus Journey : https://youtu.be/_bI_WcBc4ak?si=QoZMNBRKGDhTxLjn  
Prometheus helm 설치와 Operator : https://youtu.be/qHIgk547SVA?si=8_f0gBHVEQPxHFOr  
Prometheus Exporter 예제 : https://youtu.be/iJyC6A38qwY?si=d3HQ5PDU-pDUGYq1  
