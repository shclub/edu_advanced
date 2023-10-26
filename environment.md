# 실습 환경 정보 

교육에 앞서 실습에 필요한 환경 정보입니다.   

<br/>


1. OKD 접속 정보

2. ArgoCD 접속 정보

3. Minio 접속 정보

4. Worker Node 접속 정보

5. Opensearch Dashboard 접속 정보

6. Kibana 접속 정보

7. Grafana 접속 정보


<br/>


## OKD 접속 정보
 
<br/>

```bash
oc login https://api.okd4.ktdemo.duckdns.org:6443 -u edu1 -p <비밀번호> --insecure-skip-tls-verify
```  

- 계정 : <edu + 순번>  ( 예: edu1 )
- 비밀번호 : 사전 공지    

<br/>

OKD WEB Console : https://console-openshift-console.apps.okd4.ktdemo.duckdns.org  


<br/>

## ArgoCD 접속 정보
 
<br/>


OKD : https://argocd.apps.okd4.ktdemo.duckdns.org   


- 계정 : edu + 순번  ( 예: edu1 )
- 비밀번호 : 사전 공지  

<br/>

## Minio 접속 정보
 
<br/>


Minio Web Console : https://minio.apps.okd4.ktdemo.duckdns.org   


- 계정 : edu + 순번  ( 예: edu1 )
- 비밀번호 : 사전 공지  

<br/>


## worker node 접속방법
 
<br/>

- Worker node okd-7 로 접속 : ssh core@ktdemo.duckdns.org -p 32222  


<br/>

## Opensearch Dashboard 접속 정보
 
<br/>


Opensearch Dashboard : https://opensearch.apps.okd4.ktdemo.duckdns.org   


- 계정 : edu + 순번  ( 예: edu1 )
- 비밀번호 : 사전 공지  

<br/>

## kibana ( elasticsearch ) 접속 정보
 
<br/>


kibana : https://kibana.apps.okd4.ktdemo.duckdns.org   


- 계정 : edu + 순번  ( 예: edu1 )
- 비밀번호 : 사전 공지  

<br/>

## Grafana  접속 정보
 
<br/>


grafana : https://grafana-route-openshift-user-workload-monitoring.apps.okd4.ktdemo.duckdns.org/  


- 계정 : edu + 순번  ( 예: edu1 )
- 비밀번호 : 사전 공지  

<br/>

## 환경 구성

<br/>

- OS : Fedora coreos 
- Master Node : 1개
- Worker Node : 6개


<br/>

