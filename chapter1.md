# k8s summary 와 Observability 개념

<br/>

Redhat Openshift 의  `오픈소스 버전`인 OKD Cluster 를  생성하고 사용해본다.   ( OKD 4.12 버전 기준 : k8s 1.25 )      

또한 Observability 에 대한 개념을 이해한다.  

<br/>

OKD 설명 참고 :  https://velog.io/@_gyullbb/OKD-%EA%B0%9C%EC%9A%94  

   
1. k8s & OKD 구조 

2. k8s 주요 기능

3. Observability 


<br/>

## 1. k8s & OKD 구조  

<br>
 

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

### Network Policy


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




## 3. Observability 

<br/>

참고 : youtube link

<br/>

Observability는 크게 Elastic Stack 과 Grafana Labs 의 Loki Stack 으로 구분이 되며 최근에는 SaaS 형태로도 지원 

<br/>

<img src="./assets/observability_1.png" style="width: 100%; height: auto;"/>

<br/>

Loki Stack은 Grafana Labs 의 Observability  솔루션으로 오픈소스로 사용이 가능하며 OKD 4.11 버전부터는 EFK (Elastic-Fluntd-Kibana) 대신 Loki Stack으로 기본 모니터링이 변경 되었다.

<img src="./assets/observability_2.png" style="width: 100%; height: auto;"/>

<br/>

Open Tracing 비교 : Distributed tracing은 복잡한 마이크로 서비스 모니터링 환경에서 아주 중요하고 아래는 3종류의 오픈소스 트레이싱 솔루션을 비교한다   


<img src="./assets/observability_3.png" style="width: 100%; height: auto;"/>

<br/>

최근에는 OpenTelemetry 를 통하여 데이터를 수집 하는 방식을 많이 사용한다.  

<br/>

OpenTelemetry (축약해서 Otel)은 trace, metric, log와 같은 telemetry 데이터를 instrumenting, generating, collecting, exporting 하기 위한 말 그대로 모든 것이 열려있는 개방적인 모니터링 도구   

- 2019년 2개 프로젝트 병합  :  OpenTracing + OpenCensus (Google Open Source community project)  

<br/>

개별 모니터링 제품 벤더가 내부적으로 개발하는 폐쇄적인 방식이 아닌 오픈 소스로 개방되어 수많은 개발자들이 만들어가는 모니터링 도구입니다.  
Otel 의 목표는 벤더에 종속되지 않는 SDK, API, tool을 통해 telemetry 데이터를 측정하고 Observability backend로 전송하는 것.  

- Dynatrace 는 Otel 의 Top Contributor  

<br/>

<img src="./assets/observability_4.png" style="width: 100%; height: auto;"/>

<br/>  

Elastic은 OpenTelemetry에 적극적으로 참여하고 있으며 Elastic APM Agent / Elastic Agent에서 변환하여 Elasticsearch에 저장할 수 있다.    

- Jenkins 같은 경우 OpenTelemetry plugin 을 통해 Otel을 지원 하며 Elastic 과 연동이 가능 하다.  
  
<img src="./assets/observability_5.png" style="width: 100%; height: auto;"/>

<br/>  

BCI (Byte Code Instrumentation) 란 Java 에서 가장 원초적이고 강력한 프로그래밍 기법이다.
BCI 는 Java 의 Byte Code에 대해 직접적으로 수정을 가해서, 소스 파일의 수정 없이 원하는 기능을 부여할수 있고
이러한 특징 때문에 모니터링(APM) 툴들이 대부분 BCI 기능을 이용하고 있으며, BCI 를 통해 애플리케이션의 수정 없이 성능 측정에 필요한 요소들을 삽입할 수 있다.   

<br/>

AOP 를 구현하는 핵심 기술이 바로 BCI 이다.
AOP 컴포넌트들이 컴파일시간이나 런타임 시간에 Aspect 와 Business Logic 을 Weaving 할수 있는 이유가 바로 BCI 를 사용해서 Java 바이트 코드를 직접 수정할 수 있는 기술을 사용하기 때문이다.  

- 참고 : https://pinpoint-apm.github.io/pinpoint/techdetail.html

<br/>

<img src="./assets/observability_6.png" style="width: 100%; height: auto;"/>

<br/>