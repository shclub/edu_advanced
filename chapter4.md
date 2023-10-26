# Elastic Stack 

Elastic Stack 을 통한 Observability 를 이해한다.  

<br/>


1. Elastic Stack 소개

2. Elastic 를 통합 데이터 수집 설명

3. Kibana Dev Tool 실습

4. 로그 수집 실습 및 Dashboard 만들기



<br/>

## 1. Elastic Stack 소개

<br>

elastic Overivew   

- ElasticSearch는 No-SQL DB의 일종이며 수집/저장/시각화 모듈을 제공함

<img src="./assets/elastic_overview_1.png" style="width: 100%; height: auto;"/>

<br/>

elastic Stack 주요 기능  
- x-pack 사용을 위해서는 상용 버전 구매 필요   

<img src="./assets/elastic_overview_2.png" style="width: 100%; height: auto;"/>

<br/>

elastic License 정책  

- 무료는  Basic  

<img src="./assets/elastic_overview_3.png" style="width: 100%; height: auto;"/>

<br/>

Opensearch vs Elastic 차이  

- Elastic Stack 7.10.2 버전을 AWS에서 Fork하여 OpenSearch 로 오픈소스화. 그 이후 버전 (7.11 부터) 은 SSPL (Server Side
Public License )  


<img src="./assets/opensearch_vs_elastic_1.png" style="width: 100%; height: auto;"/>

<br/>

elastic 8.0 소개 : https://youtu.be/GKIud5n7JeM?si=tg3vFJjtIQb-ldsu

<br/>

Elastic 개념 소개 :   
https://youtu.be/JqKDIg8fgd8?si=FGdfiZzAekuWO40O
- Datastream : 22분 42초 부터

<br/>

## 2. Elastic 를 통합 데이터 수집

<br/>

### fleet server 설치 ( Elastic Agent )

<br/>

Elastic Agent에 Fleet Server 역할을 부여하여 실행하면 Fleet Server 로
작동.

<br/>

<img src="./assets/fleet_0.png" style="width: 80%; height: auto;"/>

<br/>

host 설정이 안되어 있을 경우 name, URL(ex. https://fleet-server.elastic.svc:8220 ) 입력한 후 "Generate Fleet Server Policy" 로 policy 생성  

<img src="./assets/fleet_1.png" style="width: 80%; height: auto;"/>

<br/>

설치 전 Kibana Fleet UI에서 Fleet Server의 service token 발급 필수이어서 아래의 `fleet-server-service-token` 값 복사 저장 필요.  


<img src="./assets/fleet_2.png" style="width: 80%; height: auto;"/>

<br/>

```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.5.1-linux-x86_64.tar.gz
tar xzvf elastic-agent-8.5.1-linux-x86_64.tar.gz
cd elastic-agent-8.5.1-linux-x86_64
sudo ./elastic-agent install \
  --fleet-server-es=http://localhost:9200 \
  --fleet-server-service-token=AAEAAWVsYXN0aWMvZmxlZXQtc2VydmVyL3Rva2VuLTE2OTcwMDQ3MT**c4LUY5a0MwZ0JJZw \
  --fleet-server-policy=fleet-server-policy
```  

<br/>

인증서 검증 절차 스킵 설정


Elasticsearch는 default로 자체 발급 사설인증서를 사용하기 때문에 별도로 인증서 발급 관련 설정하지 않을 경우 인증
서 검증 절차로 인해 Elasticsearch와 기타 요소 간의 통신이 안될 수 있음.    

<br/>

대표적으로 "x509: cetificate signed by unknown authority" error가 발생.
따라서 fleet server manifest에서 "FLEET_SERVER_ELASTICSEARCH_INSECURE"을 "1"(True)로 설정하여 인증서 검증 절차를 스킵  

<br/>

그리고 같은 클러스터에서 실행되는 daemonset Elastic Agent 또한 같은 이유로 아래와 같이 설정하여 인증서 검증 절차 를 스킵할 수 있음  

<br/> 

#### trouble shooting 

<br/>

fleet server 가 unhealth 가 보이거나 정상적이지 않은 경우에는
elasic-master pod 에 terminal 로 접속하여 elastic-agent restart 를 한다.

<br/>

Kibana > 우상단 (三) 메뉴 > Fleet > Settings Outputs > default(elasticsearch type) 편집  

<img src="./assets/fleet_3.png" style="width: 80%; height: auto;"/>

<br/>

fleetserver 용 service accout 를 생성한다.  

```bash
[root@bastion elastic]# kubectl apply -f fleetserver_sa.yaml -n elastic
clusterrolebinding.rbac.authorization.k8s.io/elastic-agent-clusterrolebinding created
rolebinding.rbac.authorization.k8s.io/elastic-agent-rolebinding created
rolebinding.rbac.authorization.k8s.io/elastic-agent-kubeadm-config-rolebinding created
clusterrole.rbac.authorization.k8s.io/elastic-agent-clusterrole created
role.rbac.authorization.k8s.io/elastic-agent-role created
role.rbac.authorization.k8s.io/elastic-agent-kubeadm-config-role created
serviceaccount/elastic-agent created
[root@bastion elastic]# kubectl get sa -n elastic
NAME            SECRETS   AGE
builder         1         5d6h
default         1         5d6h
deployer        1         5d6h
elastic-agent   1         5s
```  

<br/>

fleet server 를 배포한다.  

```bash
[root@bastion elastic]# cat fleetserver_deploy.yaml
[root@bastion elastic]# kubectl apply -f fleetserver_deploy.yaml -n elastic
deployment.apps/integration-server-deployment created
service/fleet-server created
service/apm-server created
[root@bastion elastic]# kubectl get po -n elastic
NAME                                            READY   STATUS    RESTARTS   AGE
elasticsearch-master-0                          1/1     Running   0          56m
integration-server-deployment-dccb495dc-zpp7m   1/1     Running   0          54s
kibana-kibana-d8dcc5f6-4stg7                    1/1     Running   0          49m
```

<br/>

### APM 서버 설치

<br/>

APM Server는 8버전부터 APM integration으로 대체되어 바이너리 설 치 대신 APM integration을 Fleet server 혹은 기 설치된 Elastic Agent에 추가해서 사용한다.  

<br/>

이번 예제는  Fleet server에 APM integration을 추가함

<img src="./assets/elastic_apm_1.png" style="width: 80%; height: auto;"/>


APM integration 추가 및 설정  
- Kibana > 우상단 (三) 메뉴 > Integrations 으로 가서 APM 클릭

<img src="./assets/elastic_apm_2.png" style="width: 80%; height: auto;"/>


<br/>

Manage APM integration in Fleet 클릭  
- Add Elastic APM 클릭  

<br/>

서버 설정을 한다.  
- Host: 0.0.0.0:8200
- URL: {APM server route 주소} (ex. https://apm.apps.okd4.ktdemo.duckdns.org )  

<img src="./assets/elastic_apm_3.png" style="width: 80%; height: auto;"/>


<br/>

적용할 Agent policy 선택   
- Fleet server의 Agent policy 선택한 다음 save and continue

<img src="./assets/elastic_apm_4.png" style="width: 80%; height: auto;"/>

<br/>

위와 같이 추가된 것을 확인 가능  

<img src="./assets/elastic_apm_5.png" style="width: 80%; height: auto;"/>

<br/>

웹 브라우저에서 https://apm.apps.okd4.ktdemo.duckdns.org/ 로 연결하여 아래 처럼 `publish_ready` 가 `true` 로 되어 있으면 정상  

```bash
{
  "build_date": "2023-08-09T15:40:16Z",
  "build_sha": "c4ebfa692b24e576d0f1960f981deb461d99fae8",
  "publish_ready": true,
  "version": "8.9.1"
}
```


<br/>



```bash
[root@bastion elastic]# kubectl get svc -n elastic
NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
apm-server                      ClusterIP   172.30.179.236   <none>        8200/TCP            104s
elasticsearch-master            ClusterIP   172.30.217.171   <none>        9200/TCP,9300/TCP   56m
elasticsearch-master-headless   ClusterIP   None             <none>        9200/TCP,9300/TCP   56m
fleet-server                    ClusterIP   172.30.115.208   <none>        8220/TCP            105s
kibana-kibana                   ClusterIP   172.30.17.252    <none>        5601/TCP            50m
```  

<br/>


metric 수집   


```bash
[root@bastion elastic]# kubectl get svc -n openshift-monitoring
NAME                                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                               AGE
alertmanager-main                       ClusterIP   172.30.147.68    <none>        9094/TCP,9092/TCP,9097/TCP            40d
alertmanager-operated                   ClusterIP   None             <none>        9093/TCP,9094/TCP,9094/UDP            40d
cluster-monitoring-operator             ClusterIP   None             <none>        8443/TCP                              40d
kube-state-metrics                      ClusterIP   None             <none>        8443/TCP,9443/TCP                     40d
node-exporter                           ClusterIP   None             <none>        9100/TCP                              40d
openshift-state-metrics                 ClusterIP   None             <none>        8443/TCP,9443/TCP                     40d
prometheus-adapter                      ClusterIP   172.30.153.100   <none>        443/TCP                               40d
prometheus-k8s                          ClusterIP   172.30.8.59      <none>        9091/TCP,9092/TCP                     40d
prometheus-k8s-thanos-sidecar           ClusterIP   None             <none>        10902/TCP                             40d
prometheus-operated                     ClusterIP   None             <none>        9090/TCP,10901/TCP                    40d
prometheus-operator                     ClusterIP   None             <none>        8443/TCP                              40d
prometheus-operator-admission-webhook   ClusterIP   172.30.65.57     <none>        8443/TCP                              40d
telemeter-client                        ClusterIP   None             <none>        8443/TCP                              40d
thanos-querier                          ClusterIP   172.30.131.225   <none>        9091/TCP,9092/TCP,9093/TCP,9094/TCP   40d
```  

<br/>

###  Elastic Agent 설치 ( 2가지 )

<br/>


Elastic Stack은  크게 2가지 방식으로 Elastic Agent를 사용  

- `elastic` namespace에 Deployment로 Elastic Agent를 배포하여 여러 solution metric 을 수집   
- `elastic-daemonset` namespace에 Daemonset으로 Elastic Agent를 각 OKD 노드에 배포하여 노드 metric과 container log를 수집    


<br/>

#### kube metric 수집 ( 내부 API )

<br/>


Agent policy에 Integration 추가한다. 

<br/>

<img src="./assets/elastic_apm_6.png" style="width: 80%; height: auto;"/>

<br/>

Add Metrics Integration을 클릭한다.  

<img src="./assets/elastic_metric_1.png" style="width: 80%; height: auto;"/>

<br/>

Kubernetes 선택 한다.


<img src="./assets/elastic_metric_2.png" style="width: 80%; height: auto;"/>


Kubernetes metric 수집 항목은 아래와 같고 Add Kubernetes 버튼을 클릭한다.  

<img src="./assets/elastic_metric_3.png" style="width: 80%; height: auto;"/>

<br/>


<img src="./assets/elastic_metric_4.png" style="width: 80%; height: auto;"/>

내부 API를 호출 하는경우는 아래와 같이 체크를 한다.  ( kubelet 은 나중에 설정한다.   )

<img src="./assets/elastic_metric_5.png" style="width: 80%; height: auto;"/>

<br/>

host 에는 `https://kube-state-metric.openshift-monitoring.svc:8443` 를 입력한다.  

Leader Election 은 체크 하지 않는다.  

<br/>

<img src="./assets/elastic_metric_6.png" style="width: 80%; height: auto;"/>

<br/>

Adavanced Options 를 펼치고 아래와 같이 입력한다.  
- Bearer Token file : `/var/run/secrets/kubernetes.io/serviceaccount/token`
- SSL CA : `/var/run/secrets/kubernetes.io/serviceaccount/service-sa`

<br/>

<img src="./assets/elastic_metric_7.png" style="width: 80%; height: auto;"/>


<br/>

Host에는 클러스터에 설치되어 있는 kube-state-metrics의 service host와 port를 입력 (`https://kube-state-metrics.openshift-monitoring.svc:8443`)  

<br/>

Fleet Server는 Deployment로 배포되었기 때문에 Leader Election은 해제 (여러 Daemonset Elastic Agent 중 leader option을 가진 agent만 수집하기 위한 설정이므로 사용하지 않는다.)  

<br/>

processor는 전처리 모듈로 Elastic Agent가 Elasticsearch로 데이터를 전송하기 전에 데이터를 가공할 수 있다.

<br/>

API 서버 설정을 한다.    

기존 값은 변경하지 말고 processor에 아래 값을 추가한다.  

- Bearer Token file : `/var/run/secrets/kubernetes.io/serviceaccount/token`
- SSL CA : `/var/run/secrets/kubernetes.io/serviceaccount/ca`
- Hosts : `https://${env.KUBERNETES_SERVICE_HOST}:${env.KUBERNETES_SERVICE_PORT}`
- processor
    ```bash
    - add_fields:
        target: orchestrator.cluster
        fields:
          name: "ktdemo"
          url: "https://api.okd4.ktdemo.duckdns.org:6443"        
    ```  

<br/>

<img src="./assets/elastic_metric_8.png" style="width: 80%; height: auto;"/>

<br/>

Events도 마찬가지로 Leader Election 해제 processor는 위 설명과 동일
설정 완료하면 Existing Host에서 fleet 서버 선택후 save integration


<img src="./assets/elastic_metric_8_1.png" style="width: 80%; height: auto;"/>

<br/>

<img src="./assets/elastic_metric_9.png" style="width: 80%; height: auto;"/>

<br/>

아래와 같이 생성 된 것을 확인 할 수 있다.  

<img src="./assets/elastic_metric_10.png" style="width: 80%; height: auto;"/>

<br/>

Fleet Server Policy를 확인한다.   

<img src="./assets/elastic_metric_11.png" style="width: 80%; height: auto;"/>

<br/>

kubernetes Integration 을 생성하면 관련된 dashboard가 자동으로 생성이 된다.  


<img src="./assets/elastic_metric_12.png" style="width: 80%; height: auto;"/>

<br/>

Elastic Agent를 클릭하면 Agent의 metric 정보를 볼 수 있다.  

<br/>

<img src="./assets/elastic_metric_14.png" style="width: 80%; height: auto;"/>


<br/> 

##### toekn 값 확인

<br/>

`openshift-monitoring`  namespace 의 secret 에서 `kube-state-metrics-token` 으로 시작하는 값을 선택한다.  


<img src="./assets/elastic_metric_15.png" style="width: 80%; height: auto;"/>

<br/>

`service-ca.crt` 는 서비스 SSL CA 이고 `ca.crt` 는 API 서버의  SSL CA 이다.    

`token`은 Bearer token 이다.

<img src="./assets/elastic_metric_16.png" style="width: 80%; height: auto;"/>


```bash
[root@bastion elastic]# kubectl exec -it kube-state-metrics-7fc57d8785-r52m9 sh -n openshift-monitoring
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
sh-4.4$ cd /var/run/secrets/kubernetes.io/serviceaccount
sh-4.4$ pwd
/var/run/secrets/kubernetes.io/serviceaccount
sh-4.4$ ls -al
total 0
drwxrwsrwt. 3 root 1000430000 160 Oct 11 13:02 .
drwxr-xr-x. 3 root root        60 Oct 10 12:21 ..
drwxr-sr-x. 2 root 1000430000 120 Oct 11 13:02 ..2023_10_11_13_02_47.1047291665
lrwxrwxrwx. 1 root 1000430000  32 Oct 11 13:02 ..data -> ..2023_10_11_13_02_47.1047291665
lrwxrwxrwx. 1 root 1000430000  13 Oct 10 11:55 ca.crt -> ..data/ca.crt
lrwxrwxrwx. 1 root 1000430000  16 Oct 10 11:55 namespace -> ..data/namespace
lrwxrwxrwx. 1 root 1000430000  21 Oct 10 11:55 service-ca.crt -> ..data/service-ca.crt
lrwxrwxrwx. 1 root 1000430000  12 Oct 10 11:55 token -> ..data/token
sh-4.4$ ls
ca.crt	namespace  service-ca.crt  token
sh-4.4$ pwd
/var/run/secrets/kubernetes.io/serviceaccount
sh-4.4$ cat /var/run/secrets/kubernetes.io/serviceaccount/service-ca.crt
-----BEGIN CERTIFICATE-----
MIIDUTCCAjmgAwIBAgIIQ88lQjt5bBQwDQYJKoZIhvcNAQELBQAwNjE0MDIGA1UE
Awwrb3BlbnNoaWZ0LXNlcnZpY2Utc2VydmluZy1zaWduZXJAMTY5MzUzODM5MDAe
Fw0yMzA5MDEwMzE5NDlaFw0yNTEwMzAwMzE5NTBaMDYxNDAyBgNVBAMMK29wZW5z
aGlmdC1zZXJ2aWNlLXNlcnZpbmctc2lnbmVyQDE2OTM1MzgzOTAwggEiMA0GCSqG
******
bfGbmjwo3/A2DBLczYfJPmmFdlg5lRcFb3gh4MO/UFhbJw7jRxo+PynAXp9COZWA
f3G1/i1d5IMbRSMulaFqpyQBNboF4GL/OMRqbaXM3PT0KWIIOyXPd3DziC+lA9Yx
VSmOIAv12CRxennyDRCsFgqLvX09m0MUw37IsK0uk1lfFDDeVQ==
-----END CERTIFICATE-----
```  


<br/>

데이터가 수집이 되는지 확인 하기 위해서 아래처럼 Index Management -> Data Streams 에 보면 `metrics-kuber` 로 시작되는 stream이 보인다.  

<img src="./assets/elastic_metric_17.png" style="width: 80%; height: auto;"/>

<br/>

Discover 에서  `dataset` 을 `metrics-kubernetes` 로 생성한다.

<img src="./assets/elastic_metric_18.png" style="width: 80%; height: auto;"/>


<br/>

`Selected fields` 에   `data_stream.dataset` 을 선택하면 dataset 값만 보여진다.

<img src="./assets/elastic_metric_19.png" style="width: 80%; height: auto;"/>

<br/>

kuubernetes overview dashboard 를 보면 데이터가 안나오는 것들이 있는데 이 값은 `kube_state_metric` 이 아닌 `kubelet` 에서 수집할 수 있다.  

<img src="./assets/elastic_metric_20.png" style="width: 80%; height: auto;"/>

<br/>

<img src="./assets/elastic_metric_21.png" style="width: 80%; height: auto;"/>


<br/>

### kubelet 으로  metric 과 container 로그 수집

<br/>

kubelet 과 container 로그를 수집 하기 위해서는 worker node에서 데이터를 수집해야 하며 Daemonset 형태로 Elastic Agent 가 배포가 되어야 한다.  

<br/>

default 라는 이름으로 agent policy 를 생성한다.  

<br/>

<img src="./assets/elastic_kubelet_metric_1.png" style="width: 80%; height: auto;"/>

<br/>

Add Agent를 클릭한다.

<br/>

<img src="./assets/elastic_daemonset_agent_1.png" style="width: 80%; height: auto;"/>

<br/>

TOKEN 값만 확인 하고 복사한다.

<img src="./assets/elastic_daemonset_agent_2.png" style="width: 80%; height: auto;"/>


Agent policy에 Integration 추가
- Add Kubernetes를 클릭한다.  

<br/>

<img src="./assets/elastic_kubelet_metric_2.png" style="width: 80%; height: auto;"/>


<br/>

`kubernetes-common` 이라는 이름으로 integration을 만든다.  

<img src="./assets/elastic_kubelet_metric_3.png" style="width: 80%; height: auto;"/>

<br/>

OKD Console 에서 kubelet 에서 metric 서비스 포트를 확인한다.  

<img src="./assets/elastic_kubelet_metric_4.png" style="width: 80%; height: auto;"/>


<br/>


`kubernetes-common` 이라는 이름의 integration을 수정한다.  

<img src="./assets/elastic_daemonset_agent_5.png" style="width: 80%; height: auto;"/>

<br/>

아래와 같이 수집할 항목을 체크한다.  

<img src="./assets/elastic_daemonset_agent_6.png" style="width: 80%; height: auto;"/>

<br/>

먼저 Kubelet API 를 설정한다.   
- Bearer Token File 은 기존 값 유지 : `/var/run/secrets/kubernetes.io/serviceaccount/token`
- hosts의 포트는 워커노드 hostname 을 일력하는데 Daemonset 이 hostnetwork 로 설정이 되기때문에 localhost로  설정
- hosts의 포트는 워커노드의 kubelet 노출하는 포트는 10250 으로 설정  
- Hosts : `https://localhost:10250`
- Period : `15s`
- SSL Verification Mode : `none`  

<br/>

Advanced Options 을 열고 processor 에는 아래와 같이 설정한다.    

<br/>

```bash
- add_fields:
    target: orchestrator.cluster
    fields:
      name: "ktdemo"
      url: "https://api.okd4.ktdemo.duckdns.org:6443"
```  

<br/>

OKD의 Observe -> Targets 메뉴를 보면 kublet이 10250 으로 포트를 노출하고 있는 것을 확인 할 수 있다.  

<img src="./assets/elastic_kubelet_metric_4.png" style="width: 80%; height: auto;"/>

<br/>

`elastic-daemonset` namespace를 생성하고 `node selector`를 설정 하지 않는다.    
- Daemonset 이 모든 Node에서 작동해야 함

<br/>

```bash
[root@bastion elastic]# oc new-project elastic-daemonset
Now using project "elastic-daemonset" on server "https://api.okd4.ktdemo.duckdns.org:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=k8s.gcr.io/e2e-test-images/agnhost:2.33 -- /agnhost serve-hostname
```  

<br/>

권한은 cluster role 로 생성한다.  

```bash
[root@bastion elastic]# kubectl apply -f elastic_daemonset_sa.yaml -n elastic-daemonset
clusterrolebinding.rbac.authorization.k8s.io/elastic-agent-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/elastic-agent-kubeadm-config-rolebinding created
clusterrole.rbac.authorization.k8s.io/elastic-agent-role created
clusterrole.rbac.authorization.k8s.io/elastic-agent-kubeadm-config-role created
serviceaccount/elastic-agent created
```

`elastic-agent` sa 에 권한을 할당한다.  

```bash
[root@bastion elastic]# oc adm policy add-scc-to-user anyuid -z elastic-agent -n elastic-daemonset
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:anyuid added: "elastic-agent"
[root@bastion elastic]# oc adm policy add-scc-to-user privileged -z elastic-agent -n elastic-daemonset
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:privileged added: "elastic-agent"
```

<br/>

node-exporter 권한을 할당한다. ( 불필요 할듯 )  

```bash
[root@bastion elastic]# oc adm policy add-scc-to-user node-exportor -z elastic-agent -n elastic-daemonset
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:node-exportor added: "elastic-agent"
```  

<br/>

elastic_daemonset_agent.yaml 에서 token 값을 replace 한다.  

```bash
[root@bastion elastic]# vi elastic_daemonset_agent.yaml
[root@bastion elastic]# kubectl apply -f elastic_daemonset_agent.yaml -n elastic-daemonset
daemonset.apps/elastic-agent created
[root@bastion elastic]# kubectl get po -n elastic-daemonset -o wide
NAME                  READY   STATUS    RESTARTS   AGE     IP              NODE                            NOMINATED NODE   READINESS GATES
elastic-agent-8fx95   1/1     Running   0          7m46s   192.168.1.149   okd-3.okd4.ktdemo.duckdns.org   <none>           <none>
elastic-agent-8qr6d   1/1     Running   0          7m46s   192.168.1.146   okd-1.okd4.ktdemo.duckdns.org   <none>           <none>
elastic-agent-qvfzg   1/1     Running   0          7m46s   192.168.1.154   okd-5.okd4.ktdemo.duckdns.org   <none>           <none>
elastic-agent-tdx82   1/1     Running   0          7m46s   192.168.1.150   okd-4.okd4.ktdemo.duckdns.org   <none>           <none>
elastic-agent-zhmzg   1/1     Running   0          7m47s   192.168.1.148   okd-2.okd4.ktdemo.duckdns.org   <none>           <none>
``` 

<br/>

Fleet agent 가 노드 마다 설정 이 된 것을 확인 할 수 있다.  

<img src="./assets/elastic_daemonset_agent_3.png" style="width: 80%; height: auto;"/>

<br/>

정상적으로 Agent POD 가 생성이 되면 데이터를 수집하게 되고 대쉬보드를 보면 기본에 빠졌던 필드가 채워진 것을 볼 수 있다.   

<img src="./assets/elastic_daemonset_agent_4.png" style="width: 80%; height: auto;"/>

<br/>



metric 이 정상적으로 수집이 되는 것을 확인 후에 다음 설정을 진행한다. 

<br/>

Kubernetes Scheduler 를 설정한다.   
- Bearer Token File 은 기존 값 유지 : `/var/run/secrets/kubernetes.io/serviceaccount/token`
- Hosts : `https://0.0.0.0:10259`
- Period : `15s`
- SSL Verification Mode : `none`  

<br/>

Advanced Options 을 열고 processor 에는 아래와 같이 설정한다.      
- Kubernetes Scheduler Label key : `app`
- Kubernetes Scheduler Label value : `openshift-kube-scheduler`
- processor

  ```bash
  - add_fields:
      target: orchestrator.cluster
      fields:
        name: "ktdemo"
        url: "https://api.okd4.ktdemo.duckdns.org:6443"
  ```  
<br/>

<img src="./assets/elastic_kubelet_metric_5.png" style="width: 80%; height: auto;"/>

<br/>

Openshift/OKD는 Kubernetes와 다르게 scheduler label이 `app: openshift-kube-scheduler` 이고 k8s는 `app:kube-scheduler`

<br/>

Kubernetes Controller Manger 를 설정한다.   
- Bearer Token File 은 기존 값 유지 : `/var/run/secrets/kubernetes.io/serviceaccount/token`
- Hosts : `https://0.0.0.0:10257`
- Period : `15s`
- SSL Verification Mode : `none`  

<br/>

Advanced Options 을 열고 processor 에는 아래와 같이 설정한다.      
- Kubernetes Scheduler Label key : `app`
- Kubernetes Scheduler Label value : `kube-controller-manager`
- processor

  ```bash
  - add_fields:
      target: orchestrator.cluster
      fields:
        name: "ktdemo"
        url: "https://api.okd4.ktdemo.duckdns.org:6443"
  ```  
<br/>

<img src="./assets/elastic_kubelet_metric_6.png" style="width: 80%; height: auto;"/>

<br/>

로그를 수집하기 위한 설정을 한다.  

- condition : `${kubernetes.container.name} != 'istio-proxy' and ${kubernetes.container.name} != 'istio-init' and ${kubernetes.container.name} != 'elastic-agent' and ${kubernetes.container.name} != 'elastic-agent-integrations' and ${kubernetes.container.name} != 'elasticsearch' and ${kubernetes.container.name} != 'kibana'`
- Kubernetes container log path : `/var/log/containers/*${kubernetes.container.id}.log`
- Container parser's stream configuration : `all`
- Container parser's format configuration : `auto`
- processor

  ```bash
  - add_fields:
      target: orchestrator.cluster
      fields:
        name: "ktdemo"
        url: "https://api.okd4.ktdemo.duckdns.org:6443"
  ```    


<br/>

로그 수집 설정을 한다.    

<img src="./assets/elastic_log_1.png" style="width: 80%; height: auto;"/>

<br/>



<br/><br/><br/>

elastic 7.x 부터는 repository-s3 plugin 은 기본으로 설치되어있어 snapshot을 s3에 저장 할수 있다.  

<br/>

Dev Tool 에러 아래과 같은 등록을 하면 에러가 발생한다.

```bash
PUT _snapshot/my_elastic_s3_repository
  {
     "type": "s3",
      "settings": {
         "region" : "ap-northeast-2",
         "bucket": "elastic-log-snapshot",
         "access_key": "ErolFCoc111SI6xC",
         "secret_key": "aGjzlkinxC111VCTqNF5EY0n97eoNikunDYim",
         "endpoint": "http://my-minio.minio.svc.cluster.local:9000",
         "path_style_access": "true",
         "protocol": "http"
      }
  }
``` 

<br/>

```bash
#! Using s3 access/secret key from repository settings. Instead store these in named clients and the elasticsearch keystore for secure settings.
{
  "error": {
    "root_cause": [
      {
        "type": "repository_verification_exception",
        "reason": "[my_elastic_s3_repository] path  is not accessible on master node"
      }
    ],
    "type": "repository_verification_exception",
    "reason": "[my_elastic_s3_repository] path  is not accessible on master node",
    "caused_by": {
      "type": "illegal_argument_exception",
      "reason": "Setting [access_key] is insecure, but property [allow_insecure_settings] is not set"
    }
  },
  "status": 500
}
```  

<br/>

저장하고 나서 elastic-master statefulset 을 재기동한다.  

```bash
[root@bastion elastic]# kubectl rollout restart statefulset elasticsearch-master -n elastic
```  

<br/>

pod가 재기동시 정상적으로 올라오지 않는 에러를 보면 Probe 에러인 `wait_for_status=green&timeout=1s` 라는 메시지가 나오는데 single node로 elastic을 사용하는 경우 이런 현상이 발생한다.      

index 의 경우에는 yellow 상태를 볼수 있는데 이것도 single node로 사용하는 경우에 볼 수 있다.  

<br/>

<img src="./assets/elastic_index_management_1.png" style="width: 80%; height: auto;"/>

<br/>

기본적으로 elastic 은 master/slave 로 사용이 되어야 하며  single node 로 사용시 helm 의 values.yaml을 아래와 같이 변경한다.     

- 이전값 : `clusterHealthCheckParams: "wait_for_status=green&timeout=1s"`
- 변경값 : `clusterHealthCheckParams: "level=cluster"`

<br/>

```bash
[root@bastion elastic]# helm upgrade elasticsearch elastic/elasticsearch -f values.yaml -n elastic
```

<br/>

참고  
- https://www.elastic.co/guide/en/elasticsearch/reference/current/repository-s3.html 
- minio S3 : https://ahmetcan.org/elasticsearch-snapshots-to-minio/
- Public Cloud Storage : https://opster.com/guides/opensearch/opensearch-operations/how-to-set-up-snapshot-repositories/

<br/>

최신버전은 key 값 같은 중요한 정보는 keystore 에 저장한다.      

가운데 `okd`는 repository 생성시 client 구분자로 사용하기 위한 값이다.  

```bash
bin/elasticsearch-keystore add s3.client.okd.access_key
bin/elasticsearch-keystore add s3.client.okd.secret_key
```

<br/>

<img src="./assets/elastic_s3_keystore_1.png" style="width: 80%; height: auto;"/>

<br/>

값을 확인 하는 방법은 아래와 같다.  

```bash
sh-5.0$ bin/elasticsearch-keystore list                         
bootstrap.password
keystore.seed
s3.client.okd.access_key
s3.client.okd.secret_key
sh-5.0$ bin/elasticsearch-keystore show s3.client.okd.access_key
ErolFCocGsP12SgSI6xC
```  

<br/>

Dev Tool에서 적용을 한다.  

```bash
POST _nodes/reload_secure_settings
```  


```bash
{
  "_nodes": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "cluster_name": "elasticsearch",
  "nodes": {
    "XtzT52GYTg6OvAPhjN8hRQ": {
      "name": "elasticsearch-master-0"
    }
  }
}
```  

<br/>
repository 를 생성한다.  

```bash
PUT _snapshot/my_elastic_s3_repository
  {
     "type": "s3",
      "settings": {
         "region" : "ap-northeast-2",
         "bucket": "elastic-log-snapshot",
         "client": "okd",
         "endpoint": "http://my-minio.minio.svc.cluster.local:9000",
         "path_style_access": "true",
         "protocol": "http"
      }
  }
```
<br/>

정상적으로 생성이 되면 아래와 같은 응답이 온다.  

```bash
{
  "acknowledged": true
}
```  

<br/>

Snapshot repostory 화면에서 생성을 확인한다.  

<img src="./assets/elastic_s3_repository_1.png" style="width: 80%; height: auto;"/>

<br/>

repository 이름으로 들어가서 verify repository 버튼을 클릭하여 확인을 한다.  

<img src="./assets/elastic_s3_repository_2.png" style="width: 80%; height: auto;"/>

<br/>

수동으로 전체 index의 snapshot을 생성해본다.  

```bash
PUT /_snapshot/my_elastic_s3_repository/my-first-snapshot?wait_for_completion=true
```  

<br/>

minio 에 가면 생성된 snapshot 을 확인 할 수 있다.  

<img src="./assets/elastic_s3_snapshot_save_1.png" style="width: 80%; height: auto;"/>

<br/>

앞에 생성한 snapshot 을 삭제 한다.    

```bash
DELETE _snapshot/my_elastic_s3_repository/my-first-snapshot
```   
<br/>

응답 메시지  

```bash
{
  "acknowledged": true
}
```
<br/>

특정 index 를 snapshot 으로 저장해 봅니다. 
- index name : movies  

<br/>

```bash
PUT /_snapshot/my_elastic_s3_repository/snapshot_movies_1
{
         "indices": "movies",
         "ignore_unavailable": "true",
         "include_global_state": false
}
```

<br/>

응답메시지  

```bash
{
  "accepted": true
}
```  

<br/>

스냅샷 상태 확인

```bash
GET _snapshot/my_elastic_s3_repository/snapshot_movies_1
```  

<br/>


```bash
{
  "snapshots": [
    {
      "snapshot": "snapshot_movies_1",
      "uuid": "UzYzn-70TOuocoP99sFGHg",
      "repository": "my_elastic_s3_repository",
      "version_id": 8090199,
      "version": "8.9.1",
      "indices": [
        "movies"
      ],
      "data_streams": [],
      "include_global_state": false,
      "state": "SUCCESS",
      "start_time": "2023-10-18T09:00:30.418Z",
      "start_time_in_millis": 1697619630418,
      "end_time": "2023-10-18T09:00:30.819Z",
      "end_time_in_millis": 1697619630819,
      "duration_in_millis": 401,
      "failures": [],
      "shards": {
        "total": 1,
        "failed": 0,
        "successful": 1
      },
      "feature_states": []
    }
  ],
  "total": 1,
  "remaining": 0
}
```  
<br/>

minio 다시 가서 생성된 snapshot 을 확인 할 수 있다.  

<img src="./assets/elastic_s3_snapshot_save_1.png" style="width: 80%; height: auto;"/>

<br/>

snapshot 을 복구하려면 아래 명령어 처럼 사용하고 movies 라는 index 가 존재하면 삭제를 하거나
snapshot 복구시 다른 이름으로 index 를 만들어야 한다.   

```bash
POST _snapshot/my_elastic_s3_repository/snapshot_movies_1/_restore
```  

<br/>

