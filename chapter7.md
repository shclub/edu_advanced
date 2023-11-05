# K8S Operator ( CRD )

Kubernetes Security 구조를  이해한다.  

<br/>



1. K8S Security   
2. SSL 인증서     

<br/>

## Operator 란

<br/>

https://ikcoo.tistory.com/95



crd  호출 flow 좋아 : https://ikcoo.tistory.com/95  



### cert manager

<br/>

참고  
- 악분일상 : https://youtu.be/sL2dVvDq32E?si=xO-QSl1LCGRqJFZz
- 11번가 :  https://youtu.be/1CLeRP_29Us?si=whoZqZScqgYu6uqt  


<br/>

### 실습

<br/>

지난번에 만들었던 Frontend React UI 에 SSL 인증서를 추가하여 안전한 https 연동을 해본다.  

<br/>

순서는 아래와 같다.  

- 도메인에서 토큰 확인  
- 인증서 생성 ( 자동 ) : cert-manager를 통한 인증서 발급 및 관리
- 인중서 생성 ( 수동 ) : 멀티 인증서 발급 ( VM 에서 진행 )  
- OKD Route 에 추가
- 인증서 확인  
- Elastic Agent 설정 ( Uptime , TLS Certificate )  

<br/>
 

Duckdns 사이트 ( https://www.duckdns.org ) 에서 로그인 후 token 값을 구한다.  

<img src="./assets/okd_ssl_0.png" style="width: 100%; height: auto;"/>

<br/>


Duckdns 사이트에서 로그인 후 token 값을 구한다.  

<img src="./assets/okd_ssl_1.png" style="width: 100%; height: auto;"/>


<br/>

secret를 생성한다.  

duckdns_secret.yaml   
```bash
apiVersion: v1
kind: Secret
metadata:
  name: duckdns-api-key-secret
type: Opaque
stringData:
  api-key: 3f309d5c************99434d23d6
```  

<br/>

```bash
[root@bastion security]# kubectl apply -f  duckdns_secret.yaml
secret/duckdns-api-key-secret created
```  

<br/>

참고 : https://dev.to/javiermarasco/https-with-ingress-controller-cert-manager-and-duckdns-in-akskubernetes-2jd1  


```bash
[root@bastion security]# git clone https://github.com/ebrianne/cert-manager-webhook-duckdns.git
Cloning into 'cert-manager-webhook-duckdns'...
remote: Enumerating objects: 390, done.
remote: Counting objects: 100% (50/50), done.
remote: Compressing objects: 100% (19/19), done.
remote: Total 390 (delta 33), reused 31 (delta 31), pack-reused 340
Receiving objects: 100% (390/390), 143.35 KiB | 5.73 MiB/s, done.
Resolving deltas: 100% (203/203), done.
[root@bastion security]# cd cert-manager-webhook-duckdns
```  


<br/>

```bash
[root@bastion cert-manager-webhook-duckdns]# helm install cert-manager-webhook-duckdns -n cert-manager --set duckdns.token='3f309d5c-c04a-4d46-b22a-cb99434d23d6' --set clusterIssuer.production.create=true --set clusterIssuer.staging.create=true --set clusterIssuer.email='shclub@gmail.com' --set logLevel=2 ./deploy/cert-manager-webhook-duckdns
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/okd4/auth/kubeconfig
NAME: cert-manager-webhook-duckdns
LAST DEPLOYED: Wed Nov  1 21:01:15 2023
NAMESPACE: cert-manager
STATUS: deployed
REVISION: 1
TEST SUITE: None
[root@bastion cert-manager-webhook-duckdns]# kubectl get po -n cert-manager
NAME                                            READY   STATUS    RESTARTS          AGE
cert-manager-559b5d5b7d-xq5v9                   1/1     Running   64 (6h32m ago)    22d
cert-manager-cainjector-f5c6565d4-jw78p         1/1     Running   109 (6h32m ago)   22d
cert-manager-webhook-5f44bc85f4-dhr45           1/1     Running   1                 22d
cert-manager-webhook-duckdns-77b96b4b9b-bbwz9   1/1     Running   0                 13s
[root@bastion cert-manager-webhook-duckdns]# kubectl get sa -n cert-manager
NAME                           SECRETS   AGE
builder                        1         22d
cert-manager                   1         22d
cert-manager-cainjector        1         22d
cert-manager-webhook           1         22d
cert-manager-webhook-duckdns   1         23s
default                        1         22d
deployer                       1         22d
[root@bastion cert-manager-webhook-duckdns]# kubectl get certs -n cert-manager
NAME                                       READY   SECRET                                     AGE
cert-manager-webhook-duckdns-ca            True    cert-manager-webhook-duckdns-ca            33s
cert-manager-webhook-duckdns-webhook-tls   True    cert-manager-webhook-duckdns-webhook-tls   33s
```  

<br/>

참고 : https://velog.io/@_gyullbb/OKD-%EA%B0%9C%EC%9A%94-2

<br/>

Output
```bash
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/okd4/auth/kubeconfig
NAME: cert-manager-webhook-duckdns
LAST DEPLOYED: Wed Nov  1 18:33:11 2023
NAMESPACE: edu25
STATUS: deployed
REVISION: 1
TEST SUITE: None
```  

<br/>

```bash
[root@bastion elastic]# kubectl apply -f ingress.yaml
ingress.networking.k8s.io/frontend-react-ingress created
[root@bastion elastic]# kubectl get ing
NAME                     CLASS    HOSTS                                                ADDRESS   PORTS     AGE
frontend-react-ingress   <none>   frontend-react2-edu25.apps.okd4.ktdemo.duckdns.org             80, 443   4s
[root@bastion elastic]# kubectl get challenges
NAME                                                             STATE     DOMAIN                                               AGE
cert-manager-webhook-duckdns-staging-9k9sf-266131994-147567566   pending   frontend-react2-edu25.apps.okd4.ktdemo.duckdns.org   73s            
```  

<br/>


```bash
[root@bastion elastic]# kubectl get secret
NAME                                         TYPE                                  DATA   AGE
builder-dockercfg-wxv4x                      kubernetes.io/dockercfg               1      13d
builder-token-vwzlp                          kubernetes.io/service-account-token   4      13d
cert-manager-webhook-duckdns-staging-n942s   Opaque                                1      4m9s
default-dockercfg-tptzz                      kubernetes.io/dockercfg               1      13d
default-token-z5wjx                          kubernetes.io/service-account-token   4      13d
deployer-dockercfg-swrn7                     kubernetes.io/dockercfg               1      13d
deployer-token-xssq4                         kubernetes.io/service-account-token   4      13d
duckdns-api-key-secret                       Opaque                                1      3h57m
elastic-agent-dockercfg-krzmq                kubernetes.io/dockercfg               1      4d12h
elastic-agent-token-4t97p                    kubernetes.io/service-account-token   4      4d12h
[root@bastion elastic]# kubectl describe secret cert-manager-webhook-duckdns-staging-n942s
Name:         cert-manager-webhook-duckdns-staging-n942s
Namespace:    edu25
Labels:       cert-manager.io/next-private-key=true
              controller.cert-manager.io/fao=true
Annotations:  <none>

Type:  Opaque

Data
====
tls.key:  1704 bytes
```   

<br/>

```bash
[root@bastion elastic]# kubectl get challenges
NAME                                                              STATE     DOMAIN                                               AGE
cert-manager-webhook-duckdns-production-ndx4v-407049-2411669503   pending   frontend-react2-edu25.apps.okd4.ktdemo.duckdns.org   61s
[root@bastion elastic]# kubectl get certificate
NAME                                      READY   SECRET                                    AGE
cert-manager-webhook-duckdns-production   False   cert-manager-webhook-duckdns-production   109s
```  

<br/>

wildcard 도메인 인증서를 받기위해 수동 설치를 진행 합니다.  
아래 과정은 VM 에서 진행합니다. ( 강사가 사전 진행 완료. 교육생은 불필요 )  

<br/>

`letsencrypt_wildcard.sh` 스크립트에 값을 설정 한다.  
- docker 대신 podman 가 설치가 된 서버에서는 podman 으로 대체 한다.
- ./certs:/app/cert : 인증서가 저장될 폴더  
- DOMAIN : wildcard 도메인 사용을 하기 위해 설정   
- DuckDNS_Token : duckdns 본인의 token 값  

<br/>

```bash
[root@bastion security]# cat letsencrypt_wildcard.sh
docker run -d --rm -v ./certs:/app/cert -e DOMAIN="*.apps.okd4.ktdemo.duckdns.org" -e DuckDNS_Token="3f309d5c-c04a-4d46-b22a-cb99434d23d6" --name=letsencrypt wnsguddk1/wildcard-duckdns-acme:1.0
```  

<br/>

`certs` 라는 폴더를 만들고 `letsencrypt_wildcard.sh` 를 실행하면 certs 폴더에 3개의 pem 화일이 생성이된다.    
- letsencrypt 는 3시간에 10 까지 인증서를 만들수가 있어서 제한이 있음.
- 테스트는 ./manifest/security/certs 밑에 있는 화일을 사용   

<br/>

```bash
[root@bastion security]# mkdir certs
[root@bastion security]# sh letsencrypt_wildcard.sh
[root@bastion security]# ls certs
wildcard-cert.pem  wildcard-fullchain.pem  wildcard-key.pem
```  

<br/>

우리가 만들어 놓은 route 의 URL를 WEB 브라우저에서 조회해 보면 `Your connection is not private` 으로 나오고 `Advanced` 버튼을 클릭하여 https 인증을 무시하고 접속을 합니다.  

URL 옆에 열쇠 아이콘이 열려져 있습니다.  


<img src="./assets/okd_ssl_2.png" style="width: 80%; height: auto;"/>

<br/>

이제 브라우저에서 인증하는 공인된 인증서를 추가하여 https 연동은 해봅니다.  
OKD Console 에서 본인의 namespace 를 선택하고 Networking -> Route 로 이동하여  `frontend-react` 를 수정합니다.  

<img src="./assets/okd_ssl_3.png" style="width: 80%; height: auto;"/>

<br/>

아래와 같이 설정합니다.  

- TLS Termination : edge  
- Insecure traffic : 선택안함   
- Certificate : wildcard-cert.pem 

<img src="./assets/okd_ssl_4.png" style="width: 80%; height: auto;"/>

<br/>

- Private key : wildcard-key.pem   
- CA Certificate : wildcard-fullchain.pem   

<img src="./assets/okd_ssl_4-1.png" style="width: 80%; height: auto;"/>  

저장 버튼을 클릭해서 저장합니다.  

<br/>

웹 브라우저에서 route URL을 입력하면 URL 옆에 열쇠 아이콘이 잠겨져 있는 것을 볼 수 있고 열쇠를 클릭합니다.  

<img src="./assets/okd_ssl_5.png" style="width: 80%; height: auto;"/>  


<br/>

`Connection is secure` 문구 를 클릭하여 들어가면  `Certificate is valid` 볼수 있습니다.  

<img src="./assets/okd_ssl_6.png" style="width: 80%; height: auto;"/>  

<br/>

`Certificate is valid` 를 클릭하면 인증서 정보를 볼 수 있습니다.  

<img src="./assets/okd_ssl_7.png" style="width: 80%; height: auto;"/>  


<br/>

인증서는 정상적으로 적용이 된 것을 확인 했고 인증서는 대부분 유효기간이 1년이고 `Let's Encrypt` 는 3개월 입니다.  

아래는 Elastic 를 활용하여 시스템 모니터링과 인증서 관리 기능을 사용 해 봅니다. 

<br/>

Elastic 에서 해당 기능을 사용하기 위해서는 heartbeat 이라는 pod를 배포해야 하고 configmap에 host 정보를 입력해야 합니다. ( 강사가 사전 작업 완료. 교육생은 불필요 )    
- 향후 직접 설치시 elastic 계정과 비밀번호 변경  

```bash
[root@bastion security]# cat elastic_heartbeat.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: heartbeat-deployment-config
  namespace: elastic
  labels:
    heartbeat: "true"
data:
  heartbeat.yml: |-                                                                     ## configmap으로 관리됨
    heartbeat.monitors:
    - id: solutions                                                                     ## monitor의 고유ID 설정
      type: http                                                                        ## monitor 종류 설정, 솔루션 uptime을 확인하기 위한 것이므로 http 사용
      hosts: [                                                                          ## 여러 솔루션 URL을 입력하여 한꺼번에 check
        'https://frontend-react-edu25.apps.okd4.ktdemo.duckdns.org/',
        'https://frontend-edu25.apps.okd4.ktdemo.duckdns.org/'
      ]
      max_redirects: 5                                                                  ## 로그인 화면 등으로 redirect되는 경우가 있어 exception 처리하는 설정
      check.response.status: [200]                                                      ## status code 값 설정, 일반적으로 200 사용
      schedule: '@every 5m'                                                             ## check 주기 설정
    processors:
      - add_kubernetes_metadata:
    output.elasticsearch:
      hosts: ['${ELASTICSEARCH_HOST:elasticsearch-master}:${ELASITCSEARCH_PORT:9200}']  ## 데이터를 수신할 Elasticsearch service host와 port 설정
      username: ${ELASTICSEARCH_USERNAME}                                               ## Elasticsearch 계정 설정
      password: ${ELASTICSEARCH_PASSWORD}                                               ## 위 계정의 비밀번호 설정
      protocol: https
      ssl.verification_mode: none
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: heartbeat
  namespace: elastic
  labels:
    heartbeat: "true"
spec:
  replicas: 1
  selector:
    matchLabels:
      heartbeat: "true"
  template:
    metadata:
      labels:
        heartbeat: "true"
    spec:
      serviceAccountName: elastic-agent   ## 별도 serviceaccount 생성 필요없이 elastic-agent serviceaccount 사용하면 됨
      hostNetwork: true
      dnsPolicy: ClusterFirstWithHostNet
      containers:
      - name: heartbeat
        image: elastic/heartbeat:8.9.2
        imagePullPolicy: "IfNotPresent"
        args: [
          "-c", "/etc/heartbeat.yml",
          "--strict.perms=false",
          "-e",
          "-d", "*"
        ]
        env:
        - name: ELASTICSEARCH_HOST
          value: elasticsearch-master
        - name: ELASTICSEARCH_PORT
          value: "9200"
        - name: ELASTICSEARCH_USERNAME
          value: "elastic"
        - name: ELASTICSEARCH_PASSWORD
          value: "Shcl********"
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        securityContext:
          runAsUser: 0
          privileged: true
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 128Mi
        volumeMounts:
        - name: config
          mountPath: /etc/heartbeat.yml
          readOnly: true
          subPath: heartbeat.yml
        - name: data
          mountPath: /usr/share/heartbeat/data
      volumes:
      - name: config
        configMap:
          defaultMode: 0600
          name: heartbeat-deployment-config
      - name: data
        emptyDir: {}
      nodeSelector:                                                                     ## node selector 설정
        elastic: 'true'
        edu: 'true'
[root@bastion security]# kubectl apply -f elastic_heartbeat.yaml -n elastic
configmap/heartbeat-deployment-config created
deployment.apps/heartbeat created
[root@bastion security]# kubectl get po -n elastic
NAME                                                     READY   STATUS    RESTARTS   AGE
elastic-agent-integrations-deployment-5884d4464f-thm5f   1/1     Running   0          2d16h
elastic-agent-integrations-deployment-5884d4464f-w88b8   1/1     Running   0          5d21h
elasticsearch-master-0                                   1/1     Running   0          11d
heartbeat-84bd5f58bd-t2z9l                               1/1     Running   0          67s
```  

<br/>

Kibana 에서 아래 Path 로 이동합니다.    
- Elastic 8.9.1 (교육용 환경) : Observability -> Uptime -> Monitors  
- Elastic 8.10.1 (클라우드 환경) : Observability -> Synthetics -> Monitors   

<br/>

본인의 Cloud  Elastic 환경에서 Create Monitor 버튼을 클릭하고 아래와 같이 설정한다  
- Monitor Type : HTTP Ping  
- URL : 본인의 Frontend Route 설정 URL ( 예, https://frontend-react-edu25.apps.okd4.ktdemo.duckdns.org/ )  
- Monitor Name : 원하는 이름 ( 예, front_edu )  
- Locations : 아무거나 선택    

나머지는 변경하지 않고 Creat Monitor 클릭하여 저장   

<img src="./assets/kibana_tls_1.png" style="width: 80%; height: auto;"/>  

<br/>  

해당 URL의 status 를 확인 할 수 있다.   

<img src="./assets/kibana_tls_2.png" style="width: 80%; height: auto;"/>  

<br/>

<img src="./assets/kibana_tls_3.png" style="width: 80%; height: auto;"/>  

<br/>  

`TLS Certificates` 메뉴로 이동하면 설정된  인증서 정보를 가져오고 `Alerts and rules` 설정을 통해 만료일전에 알림을 받을 수 있다.  

<img src="./assets/kibana_tls_4.png" style="width: 80%; height: auto;"/>  


<br/>
