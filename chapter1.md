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





<br/>


다른 namespace 에서 서비스 호출해 보기

참고 : https://anggeum.tistory.com/entry/Kubernetes-%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EC%84%9C%EB%B9%84%EC%8A%A4Service-Deep-Dive-Cluster-DNS  

<br/>

```bash
[root@bastion edu_ready]# cat netshoot.yaml
apiVersion: v1
kind: Pod
metadata:
  name: netshoot
spec:
  containers:
  - name: netshoot
    image: ghcr.io/shclub/netshoot
    command: ['sh', '-c', 'sleep 6000']
    imagePullPolicy: IfNotPresent
```  

<br/>

Network Policy : https://velog.io/@_zero_/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%A0%95%EC%B1%85NetworkPolicy-%EA%B0%9C%EB%85%90-%EB%B0%8F-%EC%84%A4%EC%A0%95  


<br/>


[root@bastion edu_ready]# kubectl get svc -n edu16
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
canary-service   NodePort    172.30.195.65    <none>        80:30564/TCP   5h
flask-edu4-app   ClusterIP   172.30.188.184   <none>        5000/TCP       7h2m
[root@bastion edu_ready]# kubectl exec -it netshoot sh -n edu18
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
~ # curl canary-service.edu16
{
    "color": "red",
    "status": "ok"
}
~ # curl flask-edu4-app.edu16:5000
 Container EDU | POD Working : flask-edu4-app-7876ccd5db-nmnrl | v=1
~ #