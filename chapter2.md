# Metric 수집
 
Prometheus 를 통한 k8s metric 수집 아키텍처를 이해한다.  

<br/>


1. k8s metric 수집 구조 ( k8s api vs metric server )

2. Prometheus 소개 

3. Grafana 사용 방법 

4. Metric 수집 실습 : Node Exporter ( Ubuntu VM )

5. Federation & Thanos 


<br/>

## 1. k8s metric 수집 구조 ( k8s api vs metric server )

<br>



비교 : https://jmholly.tistory.com/m/entry/prometheus%EC%99%80-k8s-metric-server-%EB%B9%84%EA%B5%90 

<br/>

https://www.duckdns.org/ 로 접속하여 가입을 하고 본인의 공유기의 WAN IP 와 도메인을 매칭 시킨다. WAN IP가 바뀌어도 주기적으로 update 된다.  

<br/>

ktdemo.duckdns.org 로 생성 을 한다. ip 를 변경하고 싶으면 ip를 수동으로 입력하고 update 한다.

<img src="./assets/duckdns_create.png" style="width: 80%; height: auto;"/>

<br/>

위의 도메인이 우리가 설치하는 base 도메인이 된다.  

<br/>

## 2. Prometheus 소개 

<br/>

설치에 필요한 Node는 총 3 개이고 boostrap 서버는 master 노드 설치 이후 제거 가능하다.  


<br/>

## 3. Grafana 사용 방법

<br/>



## 4.Metric 수집 실습 : Node Exporter ( Ubuntu VM )

<br/>

metric 그림 좋음  ( 에전에 내가  썻던것 ) : https://hyuuny.tistory.com/220

Node Exporter : https://devocean.sk.com/blog/techBoardDetail.do?ID=163266  

https://ksr930.tistory.com/m/116

https://itnext.io/prometheus-kubernetes-endpoints-monitoring-with-blackbox-exporter-a027ae136b8d

서비스 모니터링

https://jerryljh.medium.com/prometheus-servicemonitor-98ccca35a13e


<br/>

서비스 모니터 vs pod 모니터링

https://nangman14.tistory.com/75
https://alexandre-vazquez.com/prometheus-concepts-servicemonitor-and-podmonitor/

<br/>


예제로 사용한것   
- https://www.justinpolidori.it/posts/20210829_monitor_external_services_with_promethues_outside_kubernetes/  
- manifest : https://gist.github.com/christophlehmann/b1bbf2821a876c7f91d8eec3b6788f24
