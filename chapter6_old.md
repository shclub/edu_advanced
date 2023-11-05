# K8S Security

Kubernetes Security 구조를  이해한다.  

<br/>



1. K8S Security Overview  

2. User Account vs Service Account  

3. Krew 설명 및 설치  

2. SSL 인증서     

<br/>


악분일상 : https://youtu.be/2SSecGVc7SA?si=Tx7IELrVXcPGkMI2


rba : https://blog.naver.com/PostView.naver?blogId=ijoos&logNo=222270771301  

psp  : https://ikcoo.tistory.com/68  
조대협 psp : https://bcho.tistory.com/m/1276 

rbac : https://happycloud-lee.tistory.com/259    


Network Polocy : https://waspro.tistory.com/768  

ssl 기본 : http://idchowto.com/%EC%9D%B8%EC%A6%9D-%EA%B8%B0%EA%B4%80certificate-authority-ca%EC%9D%B4%EB%9E%80/  

ca :  https://aws-hyoh.tistory.com/m/59  

cert-manager 그림 좋아  : https://velog.io/@wanny328/Kubernetes-Cert-Manager-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0


한번 보기 : https://findstar.pe.kr/2023/04/09/access-k8s-api-from-pod/

강추 : https://happycloud-lee.tistory.com/259

하나하나 세부 설명 영어 : https://snyk.io/blog/10-kubernetes-security-context-settings-you-should-understand/  

pod security : https://cloud.google.com/kubernetes-engine/docs/how-to/podsecurityadmission?hl=ko  

user account vs service account (좋아)) : https://kingofbackend.tistory.com/237  

온달 보안 관련 jwt token :https://happycloud-lee.tistory.com/259

eks netwoking : https://wolf-sheep.tistory.com/208  

온달 보안 : https://m.blog.naver.com/hiondal/221609932228  

polaris : https://velog.io/@imok-_/k8s-security  

API 서버 인증서 추가 : https://access.redhat.com/documentation/ko-kr/openshift_container_platform/4.7/html/security_and_compliance/api-server-certificates  



✅ Authorization: RBAC
✅ Authentication: SSO
✅ Secrets management
✅ Pod Security policy
✅ Network policy
✅ Observability: Auditing API server



crd  호출 flow 좋아 : https://ikcoo.tistory.com/95  

아키텍처 그림 : http://www.chlux.co.kr/bbs/board.php?bo_table=board02&wr_id=154&sca=Middleware  

<br/>

<br/>

## K8S Security Overview 

<br/>

그림 하나 그려 넣기 


<img src="./assets/k8s_security_overview.png" style="width: 80%; height: auto;"/>

<br/>

## User Account vs Service Account  

<br/>

쿠버네티스에는 쿠버네티스 내에 존재하는 자원에 대한 접근을 위한 2가지의 account 타입이 존재합니다.   

- User Account : 사용자 어카운트는 사람을 위한 것  
- Service Account :  서비스 어카운트는 파드에서 실행되는 프로세스를 위한 것

<br/>

<img src="./assets/k8s_sa_user_1.png" style="width: 60%; height: auto;"/>

<br/>

쿠버네티스에 할당된 유저가 User Account (유저 어카운트) 이고

Pod 가 다른 쿠버네티스 자원 (Pods, Services ..) 에 접근하기 위해 하는 증명 주체가 Service Account (서비스 어카운트) 입니다.  

<br/>

### User Account 계정 생성 

<br/>

서비스 Account를 하나 생성합니다.  

```bash
root@edu25:~# cat edu-sa.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: edu-sa
```  

<br/>

```bash
root@edu25:~# kubectl apply -f edu-sa.yaml
serviceaccount/edu-sa created
root@edu25:~# kubectl get sa
NAME            SECRETS   AGE
builder         1         16d
default         1         16d
deployer        1         16d
edu-sa          1         5s
elastic-agent   1         8d
```  

<br/>

secret 이 2개가 생성이되고 `edu-sa-token` 으로 시작 되는 secret 에 token과 인증서 정보가 들어 있다.  

```bash
root@edu25:~# kubectl get secret
NAME                            TYPE                                  DATA   AGE
edu-sa-dockercfg-k28h8          kubernetes.io/dockercfg               1      21s
edu-sa-token-sbf9n              kubernetes.io/service-account-token   4      21s
```  

<br/>

Pod를 하나 생성 하는데  `serviceAccountName` 은 앞에서 생성한 `edu-sa` 로 설정한다.  

```bash
root@edu25:~# cat nginx_sa.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sa-example
  labels:
    app: sa-example
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sa-example
  template:
    metadata:
      labels:
        app: sa-example
    spec:
      serviceAccountName: edu-sa # SA 를 지정해줍니다
      containers:
        - name: nginx
          image: ghcr.io/shclub/nginx:latest
          ports:
            - containerPort: 80
```

<br/>

nginx pod를 생성한다.  

```bash
kubectl apply -f nginx_sa.yaml
```  

<br/>

생성된 pod로 접속한다.

```bash
root@edu25:~# kubectl exec -it sa-example-7d494dd86c-ljf5d sh
```  

<br/>
`/var/run/secrets/kubernetes.io/serviceaccount` 폴더에 secret의 내용이 mount되어 있는 것을 알 수 있다.

```bash
# ls /var/run/secrets/kubernetes.io/serviceaccount
ca.crt	namespace  service-ca.crt  token
```

<br/>

폴더로 이동한후 TOKEN 값을 구하고 API Server로 pod를 조회 해본다.  

```bash
# cd /var/run/secrets/kubernetes.io/serviceaccount
# TOKEN=$(cat token)
# curl -X GET https://$KUBERNETES_SERVICE_HOST/api/v1/namespaces/edu25/pods --header "Authorization: Bearer $TOKEN" --insecure
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "pods is forbidden: User \"system:serviceaccount:edu25:edu-sa\" cannot list resource \"pods\" in API group \"\" in the namespace \"edu25\"",
  "reason": "Forbidden",
  "details": {
    "kind": "pods"
  },
  "code": 403
```  

<br/>

### 과제   

<br/>

edu-sa 서비스 어카운트는 권한이 없어서 호출이 불가능 하고 default sa의 TOKEN 값으로 pod를 조회해 본다.    

- edu_default_sa_role.yaml

<br/>


## Krew 설명 및 설치  

<br/>

### 쿠버네티스 플러그인 관리자 Krew  

<br/>

kubectl을 보다 편리하게 사용할 수 있도록 해주는 플러그인 관리 도구

apt, brew와 비슷하게 kubectl 플러그인을 검색하고 설치하는 도구로, 2023년 1월 기준 210개의 kubectl 플러그인이 배포되어 있습니다.  

macOS, Linux, Windows에서 사용할 수 있으며 kubectl v1.12 이상의 버전에서 사용할 수 있습니다.  

<img src="./assets/krew_logo.png" style="width: 60%; height: auto;"/>


<br/>

### 설치    

vm 에서 아래 명령어를 실행한다.  

<br/> 

```bash
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)
```  

Output
```bash
++ mktemp -d
+ cd /tmp/tmp.xNBOW8MuQ4
++ tr '[:upper:]' '[:lower:]'
++ uname
+ OS=linux
++ sed -e s/x86_64/amd64/ -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/'
++ uname -m
+ ARCH=amd64
+ KREW=krew-linux_amd64
+ curl -fsSLO https://github.com/kubernetes-sigs/krew/releases/latest/download/krew-linux_amd64.tar.gz
+ tar zxvf krew-linux_amd64.tar.gz
./LICENSE
./krew-linux_amd64
+ ./krew-linux_amd64 install krew
Adding "default" plugin index from https://github.com/kubernetes-sigs/krew-index.git.
Updated the local copy of plugin index.
Installing plugin: krew
Installed plugin: krew
\
 | Use this plugin:
 | 	kubectl krew
 | Documentation:
 | 	https://krew.sigs.k8s.io/
 | Caveats:
 | \
 |  | krew is now installed! To start using kubectl plugins, you need to add
 |  | krew's installation directory to your PATH:
 |  |
 |  |   * macOS/Linux:
 |  |     - Add the following to your ~/.bashrc or ~/.zshrc:
 |  |         export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
 |  |     - Restart your shell.
 |  |
 |  |   * Windows: Add %USERPROFILE%\.krew\bin to your PATH environment variable
 |  |
 |  | To list krew commands and to get help, run:
 |  |   $ kubectl krew
 |  | For a full list of available plugins, run:
 |  |   $ kubectl krew search
 |  |
 |  | You can find documentation at
 |  |   https://krew.sigs.k8s.io/docs/user-guide/quickstart/.
 | /
/
```  

Krew 실행파일의 위치를 PATH에 등록해줍니다.  

```bash
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
``` 

업데이트한 PATH를 Shell이 알 수 있도록 Shell을 재시작합니다.    

```bash  
source ~/.bashrc # zsh을 사용하는 경우, zshrc를 입력
```  

정상적으로 설치되었는지 확인하고 업데이트도 할 겸 아래 명령어를 수행합니다.  

```bash  
kubectl krew update
```

<br/>

### Plugin 설치    

<br/>

먼저 rolesum을 설치합니다.  

- rolesum : 사용자, Service Account별 RBAC 역할에 대해 간략하게 요약 정리  

```bash
kubectl krew install rolesum
```  

Output
```bash
Updated the local copy of plugin index.
Installing plugin: rolesum
Installed plugin: rolesum
\
 | Use this plugin:
 | 	kubectl rolesum
 | Documentation:
 | 	https://github.com/Ladicle/kubectl-rolesum
/
WARNING: You installed plugin "rolesum" from the krew-index plugin repository.
   These plugins are not audited for security by the Krew maintainers.
   Run them at your own risk.
```  

<br/>

현재 context 를 확인해 보고 edu 로 context 가 아니면 변경합니다.  

```bash
root@edu25:~# kubectl config get-contexts
CURRENT   NAME                                           CLUSTER                            AUTHINFO                                 NAMESPACE
*         dev25                                          api-okd4-ktdemo-duckdns-org:6443   dev25
          edu25/api-okd4-ktdemo-duckdns-org:6443/edu25   api-okd4-ktdemo-duckdns-org:6443   edu25/api-okd4-ktdemo-duckdns-org:6443   edu25
root@edu25:~# kubectl config use-context edu25/api-okd4-ktdemo-duckdns-org:6443/edu25
Switched to context "edu25/api-okd4-ktdemo-duckdns-org:6443/edu25".
```

<br/>

edu25 namespace의 default service account 의 권한을 조회해 봅니다.  

```bash
kubectl rolesum default -n edu25
```  
```bash
ServiceAccount: edu25/default
Secrets:
• */default-dockercfg-tptzz

Policies:

• [CRB] */system:openshift:scc:anyuid ⟶  [CR] */system:openshift:scc:anyuid
  Resource                                            Name    Exclude  Verbs  G L W C U P D DC
  securitycontextconstraints.security.openshift.io  [anyuid]    [-]    [use]  ✖ ✖ ✖ ✖ ✖ ✖ ✖ ✖


• [CRB] */system:openshift:scc:hostnetwork ⟶  [CR] */system:openshift:scc:hostnetwork
  Resource                                              Name       Exclude  Verbs  G L W C U P D DC
  securitycontextconstraints.security.openshift.io  [hostnetwork]    [-]    [use]  ✖ ✖ ✖ ✖ ✖ ✖ ✖ ✖


• [CRB] */system:openshift:scc:privileged ⟶  [CR] */system:openshift:scc:privileged
  Resource                                              Name      Exclude  Verbs  G L W C U P D DC
  securitycontextconstraints.security.openshift.io  [privileged]    [-]    [use]  ✖ ✖ ✖ ✖ ✖ ✖ ✖ ✖
```  

<br/>

특정 계정의 권한을 조회해 봅니다.  

```bash
kubectl rolesum -k User edu25 -n edu25
```  

Output
```bash
User: edu25

Policies:
• [CRB] */cluster-admin-26 ⟶  [CR] */cluster-admin
  Resource  Name  Exclude  Verbs  G L W C U P D DC
  *.*       [*]     [-]     [-]   ✔ ✔ ✔ ✔ ✔ ✔ ✔ ✔


• [CRB] */edu-rolebinding ⟶  [CR] */edu-role
  Resource                                       Name  Exclude  Verbs  G L W C U P D DC
  clusterrolebindings.rbac.authorization.k8s.io  [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  clusterroles.rbac.authorization.k8s.io         [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  configmaps                                     [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  cronjobs.batch                                 [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  daemonsets.apps                                [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  deployments.apps                               [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  events                                         [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  instrumentations.opentelemetry.io              [*]     [-]     [-]   ✔ ✔ ✔ ✔ ✔ ✔ ✔ ✔
  jobs.batch                                     [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  namespaces                                     [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  nodes                                          [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  nodes/stats                                    [*]     [-]     [-]   ✔ ✖ ✖ ✖ ✖ ✖ ✖ ✖
  persistentvolumeclaims                         [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  persistentvolumes                              [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  pods                                           [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  replicasets.apps                               [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  replicasets.extensions                         [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  rolebindings.rbac.authorization.k8s.io         [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  roles.rbac.authorization.k8s.io                [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  serviceaccounts                                [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  servicemonitors.monitoring.coreos.com          [*]     [-]     [-]   ✔ ✔ ✔ ✔ ✔ ✔ ✔ ✔
  services                                       [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  statefulsets.apps                              [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  storageclasses.storage.k8s.io                  [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖


• [CRB] */node-view-rolebinding25 ⟶  [CR] */node-view-role
  Resource              Name  Exclude  Verbs  G L W C U P D DC
  nodes                 [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  nodes.metrics.k8s.io  [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
  pods.metrics.k8s.io   [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖
```  

<br/>

access-matrix : 서비스 리소스별 RBAC access 정보 확인  

```bash
root@edu25:~# kubectl krew install access-matrix
Updated the local copy of plugin index.
Installing plugin: access-matrix
Installed plugin: access-matrix
\
 | Use this plugin:
 | 	kubectl access-matrix
 | Documentation:
 | 	https://github.com/corneliusweig/rakkess
 | Caveats:
 | \
 |  | Usage:
 |  |   kubectl access-matrix
 |  |   kubectl access-matrix for pods
 | /
/
WARNING: You installed plugin "access-matrix" from the krew-index plugin repository.
   These plugins are not audited for security by the Krew maintainers.
   Run them at your own risk.
```

<br/>

```bash
root@edu25:~# kubectl access-matrix -n edu25
NAME                                                                      LIST  CREATE  UPDATE  DELETE
                                                                          ✖     ✖       ✖       ✖
                                                                          ✖     ✖       ✖       ✖
                                                                          ✖     ✖       ✖       ✖
                                                                          ✖     ✖       ✖       ✖
                                                                          ✖     ✖       ✖       ✖
                                                                          ✖     ✖       ✖       ✖
                                                                          ✖     ✖       ✖       ✖
                                                                          ✖     ✖       ✖       ✖
alertmanagerconfigs.monitoring.coreos.com                                 ✔     ✔       ✔       ✔
alertmanagers.monitoring.coreos.com                                       ✔     ✔       ✔       ✔
analysisruns.argoproj.io                                                  ✔     ✔       ✔       ✔
analysistemplates.argoproj.io                                             ✔     ✔       ✔       ✔
applicationactivities.spdx.softwarecomposition.kubescape.io               ✔     ✔       ✔       ✔
applicationprofiles.spdx.softwarecomposition.kubescape.io                 ✔     ✔       ✔       ✔
applicationprofilesummaries.spdx.softwarecomposition.kubescape.io         ✔     ✔       ✔       ✔
applications.argoproj.io                                                  ✔     ✔       ✔       ✔
applicationsets.argoproj.io                                               ✔     ✔       ✔       ✔
appliedclusterresourcequotas.quota.openshift.io                           ✔
appprojects.argoproj.io                                                   ✔     ✔       ✔       ✔
baremetalhosts.metal3.io                                                  ✔     ✔       ✔       ✔
bindings                                                                        ✔
bmceventsubscriptions.metal3.io                                           ✔     ✔       ✔       ✔
buildconfigs.build.openshift.io                                           ✔     ✔       ✔       ✔
builds.build.openshift.io                                                 ✔     ✔       ✔       ✔
catalogsources.operators.coreos.com                                       ✔     ✔       ✔       ✔
certificaterequests.cert-manager.io                                       ✔     ✔       ✔       ✔
certificates.cert-manager.io                                              ✔     ✔       ✔       ✔
challenges.acme.cert-manager.io                                           ✔     ✔       ✔       ✔
clickhouseinstallations.clickhouse.altinity.com                           ✔     ✔       ✔       ✔
clickhouseinstallationtemplates.clickhouse.altinity.com                   ✔     ✔       ✔       ✔
clickhouseoperatorconfigurations.clickhouse.altinity.com                  ✔     ✔       ✔       ✔
clusterserviceversions.operators.coreos.com                               ✔     ✔       ✔       ✔
configauditreports.aquasecurity.github.io                                 ✔     ✔       ✔       ✔
configmaps                                                                ✔     ✔       ✔       ✔
controllerrevisions.apps                                                  ✔     ✔       ✔       ✔
controlplanemachinesets.machine.openshift.io                              ✔     ✔       ✔       ✔
credentialsrequests.cloudcredential.openshift.io                          ✔     ✔       ✔       ✔
cronjobs.batch                                                            ✔     ✔       ✔       ✔
csistoragecapacities.storage.k8s.io                                       ✔     ✔       ✔       ✔
daemonsets.apps                                                           ✔     ✔       ✔       ✔
deploymentconfigs.apps.openshift.io                                       ✔     ✔       ✔       ✔
deployments.apps                                                          ✔     ✔       ✔       ✔
dnsrecords.ingress.operator.openshift.io                                  ✔     ✔       ✔       ✔
egressnetworkpolicies.network.openshift.io                                ✔     ✔       ✔       ✔
egressrouters.network.operator.openshift.io                               ✔     ✔       ✔       ✔
endpoints                                                                 ✔     ✔       ✔       ✔
endpointslices.discovery.k8s.io                                           ✔     ✔       ✔       ✔
events                                                                    ✔     ✔       ✔       ✔
events.events.k8s.io                                                      ✔     ✔       ✔       ✔
experiments.argoproj.io                                                   ✔     ✔       ✔       ✔
exposedsecretreports.aquasecurity.github.io                               ✔     ✔       ✔       ✔
firmwareschemas.metal3.io                                                 ✔     ✔       ✔       ✔
grafanadashboards.integreatly.org                                         ✔     ✔       ✔       ✔
grafanadatasources.integreatly.org                                        ✔     ✔       ✔       ✔
grafanafolders.integreatly.org                                            ✔     ✔       ✔       ✔
grafananotificationchannels.integreatly.org                               ✔     ✔       ✔       ✔
grafanas.integreatly.org                                                  ✔     ✔       ✔       ✔
hardwaredata.metal3.io                                                    ✔     ✔       ✔       ✔
horizontalpodautoscalers.autoscaling                                      ✔     ✔       ✔       ✔
hostfirmwaresettings.metal3.io                                            ✔     ✔       ✔       ✔
imagestreamimages.image.openshift.io
imagestreamimports.image.openshift.io                                           ✔
imagestreammappings.image.openshift.io                                          ✔
imagestreams.image.openshift.io                                           ✔     ✔       ✔       ✔
imagestreamtags.image.openshift.io                                        ✔     ✔       ✔       ✔
imagetags.image.openshift.io                                              ✔     ✔       ✔       ✔
infraassessmentreports.aquasecurity.github.io                             ✔     ✔       ✔       ✔
ingresscontrollers.operator.openshift.io                                  ✔     ✔       ✔       ✔
ingresses.networking.k8s.io                                               ✔     ✔       ✔       ✔
installplans.operators.coreos.com                                         ✔     ✔       ✔       ✔
instrumentations.opentelemetry.io                                         ✔     ✔       ✔       ✔
ippools.whereabouts.cni.cncf.io                                           ✔     ✔       ✔       ✔
issuers.cert-manager.io                                                   ✔     ✔       ✔       ✔
jobs.batch                                                                ✔     ✔       ✔       ✔
leases.coordination.k8s.io                                                ✔     ✔       ✔       ✔
limitranges                                                               ✔     ✔       ✔       ✔
localresourceaccessreviews.authorization.openshift.io                           ✔
localsubjectaccessreviews.authorization.k8s.io                                  ✔
localsubjectaccessreviews.authorization.openshift.io                            ✔
machineautoscalers.autoscaling.openshift.io                               ✔     ✔       ✔       ✔
machinehealthchecks.machine.openshift.io                                  ✔     ✔       ✔       ✔
machines.machine.openshift.io                                             ✔     ✔       ✔       ✔
machinesets.machine.openshift.io                                          ✔     ✔       ✔       ✔
network-attachment-definitions.k8s.cni.cncf.io                            ✔     ✔       ✔       ✔
networkpolicies.networking.k8s.io                                         ✔     ✔       ✔       ✔
opentelemetrycollectors.opentelemetry.io                                  ✔     ✔       ✔       ✔
operatorconditions.operators.coreos.com                                   ✔     ✔       ✔       ✔
operatorgroups.operators.coreos.com                                       ✔     ✔       ✔       ✔
operatorpkis.network.operator.openshift.io                                ✔     ✔       ✔       ✔
orders.acme.cert-manager.io                                               ✔     ✔       ✔       ✔
overlappingrangeipreservations.whereabouts.cni.cncf.io                    ✔     ✔       ✔       ✔
packagemanifests.packages.operators.coreos.com                            ✔
persistentvolumeclaims                                                    ✔     ✔       ✔       ✔
poddisruptionbudgets.policy                                               ✔     ✔       ✔       ✔
podmonitors.monitoring.coreos.com                                         ✔     ✔       ✔       ✔
podnetworkconnectivitychecks.controlplane.operator.openshift.io           ✔     ✔       ✔       ✔
pods                                                                      ✔     ✔       ✔       ✔
vulnerabilityreports.aquasecurity.github.io                               ✔     ✔       ✔       ✔
workloadconfigurationscans.spdx.softwarecomposition.kubescape.io          ✔     ✔       ✔       ✔
workloadconfigurationscansummaries.spdx.softwarecomposition.kubescape.io  ✔     ✔       ✔       ✔
```  


<br/>

rbac-tool : rbac 관련 lookup, whoami policy-rules 등 여러가지 확인 기능 제공    

```bash
root@edu25:~# kubectl rbac-tool lookup system:nodes
  SUBJECT      | SUBJECT TYPE | SCOPE       | NAMESPACE      | ROLE
+--------------+--------------+-------------+----------------+----------------------------------------------------------------------+
  system:nodes | Group        | ClusterRole |                | system:certificates.k8s.io:certificatesigningrequests:selfnodeclient
  system:nodes | Group        | ClusterRole |                | system:node-proxier
  system:nodes | Group        | ClusterRole |                | system:sdn-reader
  system:nodes | Group        | Role        | openshift-node | system:node-config-reader
```


<br/>

rbac-view : 웹을 통해 Cluster Roles와 Roles를 확인 (  ARM 맥은 지원 안함 ) 

```bash
kubectl rbac-view
```  
```bash
INFO[0000] Getting K8s client
INFO[0000] serving RBAC View and http://localhost:8800
```  


<br/>

VM 서버 `edu25` 번에 설치 되어 있고 포트를 오픈 했기 때문에 브라우저로  `http://211.251.238.182:8800` 로 접속하면 아래 내용을 볼수 있다.    


<img src="./assets/krew_rbac_view_1.png" style="width: 100%; height: auto;"/>  


<br/>

### 사용법

<br/>

전체 리스트 보기  
```bash
kubectl krew search
```  
<br/>

특정 플러그인 검색하기  
```bash
kubectl krew search example-plugin
```  

<br/>

Krew로 플러그인 설치하기
```bash
kubectl krew install example-plugin
```  

<br/>

Krew로 설치한 플러그인 확인하기
```bash
kubectl krew list
```  

<br/>  

Krew로 설치한 플러그인 업데이트하기
```bash
kubectl krew upgrade
```  

<br/>

Krew로 설치한 플러그인 삭제하기  
```bash
kubectl krew uninstall example-plugin
```  

<br/>

```bash
curl -v https://frontend-react-edu25.apps.okd4.ktdemo.duckdns.org/
```  
<br/>

* SSL connection using TLSv1.3 / TLS_AES_128_GCM_SHA256

Output
```bash
*   Trying 119.207.184.6...
* TCP_NODELAY set
* Connected to frontend-react-edu25.apps.okd4.ktdemo.duckdns.org (119.207.184.6) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Unknown (8):
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Client hello (1):
* TLSv1.3 (OUT), TLS Unknown, Certificate Status (22):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_128_GCM_SHA256
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: CN=*.apps.okd4.ktdemo.duckdns.org
*  start date: Nov  2 02:37:52 2023 GMT
*  expire date: Jan 31 02:37:51 2024 GMT
*  subjectAltName: host "frontend-react-edu25.apps.okd4.ktdemo.duckdns.org" matched cert's "*.apps.okd4.ktdemo.duckdns.org"
*  issuer: C=US; O=Let's Encrypt; CN=R3
*  SSL certificate verify ok.
* TLSv1.3 (OUT), TLS Unknown, Unknown (23):
> GET / HTTP/1.1
> Host: frontend-react-edu25.apps.okd4.ktdemo.duckdns.org
> User-Agent: curl/7.58.0
> Accept: */*
>
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS Unknown, Unknown (23):
< HTTP/1.1 200 OK
< server: nginx
< date: Sun, 05 Nov 2023 03:22:42 GMT
< content-type: text/html
< content-length: 2296
< last-modified: Sat, 28 Oct 2023 23:51:01 GMT
< etag: "653d9e65-8f8"
< accept-ranges: bytes
< set-cookie: f50d6f2df2312230a72c4d9dc3d02096=c488fe09169ed72ba63c8a3dbcef07d5; path=/; HttpOnly; Secure; SameSite=None
< cache-control: private
<
<!doctype html><html lang="en"><head><meta charset="utf-8"/><link rel="icon" href="/favicon.ico"/><meta name="viewport" content="width=device-width,initial-scale=1"/><meta name="theme-color" content="#000000"/><meta name="description" content="Web site created using create-react-app"/><link rel="apple-touch-icon" href="/logo192.png"/><link rel="manifest" href="/manifest.json"/><title>React App</title><link href="/static/css/2.da84dedc.chunk.css" rel="stylesheet"><link href="/static/css/main.23f60385.chunk.css" rel="stylesheet"></head><body><noscript>You need to enable JavaScript to run this app.</noscript><div id="root"></div><script>!function(e){function t(t){for(var n,l,p=t[0],i=t[1],a=t[2],c=0,s=[];c<p.length;c++)l=p[c],Object.prototype.hasOwnProperty.call(o,l)&&o[l]&&s.push(o[l][0]),o[l]=0;for(n in i)Object.prototype.hasOwnProperty.call(i,n)&&(e[n]=i[n]);for(f&&f(t);s.length;)s.shift()();return u.push.apply(u,a||[]),r()}function r(){for(var e,t=0;t<u.length;t++){for(var r=u[t],n=!0,p=1;p<r.length;p++){var i=r[p];0!==o[i]&&(n=!1)}n&&(u.splice(t--,1),e=l(l.s=r[0]))}return e}var n={},o={1:0},u=[];function l(t){if(n[t])return n[t].exports;var r=n[t]={i:t,l:!1,exports:{}};return e[t].call(r.exports,r,r.exports,l),r.l=!0,r.exports}l.m=e,l.c=n,l.d=function(e,t,r){l.o(e,t)||Object.defineProperty(e,t,{enumerable:!0,get:r})},l.r=function(e){"undefined"!=typeof Symbol&&Symbol.toStringTag&&Object.defineProperty(e,Symbol.toStringTag,{value:"Module"}),Object.defineProperty(e,"__esModule",{value:!0})},l.t=function(e,t){if(1&t&&(e=l(e)),8&t)return e;if(4&t&&"object"==typeof e&&e&&e.__esModule)return e;var r=Object.create(null);if(l.r(r),Object.defineProperty(r,"default",{enumerable:!0,value:e}),2&t&&"string"!=typeof e)for(var n in e)l.d(r,n,function(t){return e[t]}.bind(null,n));return r},l.n=function(e){var t=e&&e.__esModule?function(){return e.default}:function(){return e};return l.d(t,"a",t),t},l.o=function(e,t){return Object.prototype.hasOwnProperty.call(e,t)},l.p="/";var p=this["webpackJsonpjwt-login-app"]=this["webpack* Connection #0 to host frontend-react-edu25.apps.okd4.ktdemo.duckdns.org left intact
Jsonpjwt-login-app"]||[],i=p.push.bind(p);p.push=t,p=p.slice();for(var a=0;a<p.length;a++)t(p[a]);var f=i;r()}([])</script><script src="/static/js/2.4858ba23.chunk.js"></script><script src="/static/js/main.57480ec1.chunk.js"></script></body></html>
```  

<br/>

SSL ciphers url을 참고해서 SSL Ciphers값을 찾는다

<br/>

route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         _gateway        0.0.0.0         UG    0      0        0 eth0
default         _gateway        0.0.0.0         UG    100    0        0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
172.27.0.0      0.0.0.0         255.255.0.0     U     0      0        0 eth0
_gateway        0.0.0.0         255.255.255.255 UH    100    0        0 eth0



<br/>

## Kubernetes Components

<br/>

참고 : https://cwal.tistory.com/18    

<br/>

## 

아래 그림과 같이 Kubernetes에 존재하는 모든 Component 간 통신은 HTTPS를 기반으로 이루어지며, 모든 트래픽이 암호화되므로 데이터의 신뢰성과 보안을 보장할 수 있다.  

<br/>

<img src="./assets/k8s_security_component_1.png" style="width: 80%; height: auto;"/>

<br/>

HTTPS 프로토콜을 사용하기 위해선 Server와 Client 양측 모두 SSL/TLS 인증서(Certificate)가 필요하며, 모든 인증서는 신뢰할 수 있는 Root CA(Certifiacte Authority)에 의해 서명되어야 한다.  

<br/>

Kubernetes 역시 마찬가지로 각각의 Component마다 Certificate을 요구하며, 아래와 같이 독자적인 PKI(Public Key Infrastructure)를 구성한다.      

CA는 'KUBERNETES-CA'라는 Common Name을 갖고 있으며, etcd를 제외한 모든 Component의 Certificate 서명에 사용된다.

<br/>

<img src="./assets/k8s_security_component_2.png" style="width: 80%; height: auto;"/>

<br/>

kube-apiserver는 Client Certificate 내용 중 Common Name을 통해 Client를 구분할 수 있으며, 특히 kubelet은 자신이 위치한 Node의 hostname으로 세분화된다.   

다시 말해서 각각의 kubelet은 자신만의 유니크한 인증서를 갖는다는 의미이다. 이를 통해 kubelet은 자신이 속한 Node에 배정된 Pod 이외의 정보를 읽거나 쓸 수 없도록 제한된다.  

<br/>

<img src="./assets/k8s_security_component_3.png" style="width: 80%; height: auto;"/>

<br/>

인증서 발급 API  

<br/>
매번 K8s 관리자가 Client의 인증서를 수동으로 발급할 수는 없으므로, API 서버에 해당 작업을 위임할 수 있으며 아래와 같은 순서로 이루어진다.  


- Client 측에서 Private Key로부터 CSR(Certificate Signing Request)를 생성하고, 이를 API 서버에 전달한다.  

- 해당 요청은 K8s 상에서 CertificateSigningRequest라는 별도의 리소스(로 취급되며, 관리자의 승인이 있을 때까지 Pending 상태로 남는다.  

- 관리자 승인시, Client 인증서를 발급하며 해당 Certificate으로 K8s 접근이 가능하다.

<br/>
자세한 사용 예시는 Kubernetes 공식 문서를 참고하길 바란다.  
- https://kubernetes.io/ko/docs/tasks/tls/managing-tls-in-a-cluster/  


<br/>

### 실습  ( 사용자 생성 )

<br/>

K8S에서 프로그래밍 방식으로 접근하는 ServiceAccount와는 달리 사용자(User)라는 개념이 명시적으로 존재하지 않으며, 클러스터 외부의 독립적인 서비스로부터 사용자를 제공받는 것이 일반적이다.    

<br/>

다른 Component와 마찬가지로 kube-apiserver 접근시 Authentication은 Client Ceritificate의 'Common Name(CN)' 필드값을 통해 이루어진다. 다만 클러스터 CA가 서명한 인증서만으로는 kube-apiserver 접근만 가능할 뿐, 어떠한 리소스에도 사용권한이 없기 때문에 RBAC을 통한 Authorization이 필요하다.     

OKD는 PaaS 솔루션 이기 때문에 user 생성 기능이 포함되어 있어 쉽게 user를 생성한다.   

서비스 어카운트는 JWT Token 으로 인정하지만 일반 유저는 X509 인증서로 인증한다.  


<br/>

####  SSL Key 파일 및 CSR 파일 생성


<br/>

확인 사항 : 교육생은 본인의 순번에 맞게 dev 계정 생성 ( 예 : dev1, dev2 )

<br/>

SSL key 를 생성합니다.  

```bash
openssl genrsa -out dev25.key 2048
```  

Output
```bash
Generating RSA private key, 2048 bit long modulus (2 primes)
.................+++++
.....................+++++
e is 65537 (0x010001)
```  

<br/>

CSR 파일 생성시, Common Name에 새로 추가할 사용자의 이름을 입력하며 그 외 항목은 생략한다. 이 과정에서 dev25.key, dev25.csr 두 개의 파일이 생성된다.

```bash
openssl req -new -key dev25.key -out dev25.csr
```  

Output
```bash
139795455709632:error:2406F079:random number generator:RAND_load_file:Cannot open file:../crypto/rand/randfile.c:88:Filename=/root/.rnd
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:dev25
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

<br/>

#### K8s CSR ( CertificateSigningRequest ) 리소스 생성

<br/>

우선 다음 명령어로 CSR 파일의 내용을 base64로 인코딩할 필요가 있다.     

```bash
cat dev25.csr | base64 -w 0
```  

<br/>

Output
```bash
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ21qQ0NBWUlDQVFBd1ZURUxNQWtHQTFVRUJoTUNRVlV4RXpBUkJnTlZCQWdNQ2xOdmJXVXRVM1JoZEdVeApJVEFmQmdOVkJBb01HRWx1ZEdWeWJtVjBJRmRwWkdkcGRITWdVSFI1SUV4MFpERU9NQXdHQTFVRUF3d0ZaR1YyCk1qVXdnZ0VpTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCRHdBd2dnRUtBb0lCQVFEZW1WMS8rTHdKSGduZlZyZFEKQjFRdERiaUlTY0hTbXFpYlNEdUJ4cXBoTDlNM2JQTTYrRUZ0WW9HN3R1VExBZWxvcFlnd2Z6U0p3a0tlRGNuQQpMeE80b1hlZUV5cUFxY1JvNGpzZ1RmL3J2MzRXUFJMeXZ0WlZwRGZ6bVpsWXl1M3V0aStxb0xMWnIrL1RHWDRNCkNkSVlRVVd0dldsYWdoVWhMNGNNTUFuUTZJaFdET0xVQ2ViTy9ZZWJ4OThMQVcyS25abmJ6OWxHd0ZGaTNydnkKVGNKVThYL0QzUS9YTWFXYzNJQlhSSUVIS3V5ZW04YXMvckZSTHN0Q0wvTzhRTDM4MEsxUEtENmxOVnltY1IrVApkb1FBN1JXZnZJVFlPZ3cySmRoOThEVGNrVXI2REJtMVBzRlkxK29aMlJrZXZpZmpWTEJOWVZFWW8zTkdXWVRhCk8zcTlBZ01CQUFHZ0FEQU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUFRSi9CbmI3VmxBc1FLWEwvT3QyNlp4TVAKcEpTSFR1VDcwYU5rYXc0SzJuNmhqeVNwd3NLcmNyR1VpODVQeHNCVFhkSDg3MWFRckE1eHNvNGxlZUd5NkszVgorUGowRDBGZWlmclBoV2N4ZTMvdG9WTE1KZjZDYWVaWUE1aVlreS9wckRpS05QQ0l6S0I4ZWEyeUVib205aElRCm9pS1ZUTEd6NjBHTVVGWFp2SmUzVGI0aGpKZmFQbUMrMkNxYnY3T05NaEluQXBBSzhwSlNvOWtRQ3M4cWlpODQKWDNvRnpzcEUxRFVHMjR2SWZpMUxhUWllbW5pdVJWRXBiOFpycWMyemI4Sk0rK0hiR1BjMjhWUEFFL05obTVCVQpKTmpvWjloaERSZU0zOHZ5STAxaXpxMytGU0thZVZ4OVNMaEh5bENBZXJDVUQ4dkxtZ0dtT1pJWHh2c2ZvQT09Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
```

<br/>

아래와 같은 CSR Manifest 파일을 작성하는데 request 필드에 위에서 얻은 인코딩 텍스트를 복사 하여 붙여 넣기를 한다.  

<br/>

```bash
root@edu25:~/security# cat csr.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: dev25
spec:
  groups:
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ21qQ0NBWUlDQVFBd1ZURUxNQWtHQTFVRUJoTUNRVlV4RXpBUkJnTlZCQWdNQ2xOdmJXVXRVM1JoZEdVeApJVEFmQmdOVkJBb01HRWx1ZEdWeWJtVjBJRmRwWkdkcGRITWdVSFI1SUV4MFpERU9NQXdHQTFVRUF3d0ZaR1YyCk1qVXdnZ0VpTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCRHdBd2dnRUtBb0lCQVFEZW1WMS8rTHdKSGduZlZyZFEKQjFRdERiaUlTY0hTbXFpYlNEdUJ4cXBoTDlNM2JQTTYrRUZ0WW9HN3R1VExBZWxvcFlnd2Z6U0p3a0tlRGNuQQpMeE80b1hlZUV5cUFxY1JvNGpzZ1RmL3J2MzRXUFJMeXZ0WlZwRGZ6bVpsWXl1M3V0aStxb0xMWnIrL1RHWDRNCkNkSVlRVVd0dldsYWdoVWhMNGNNTUFuUTZJaFdET0xVQ2ViTy9ZZWJ4OThMQVcyS25abmJ6OWxHd0ZGaTNydnkKVGNKVThYL0QzUS9YTWFXYzNJQlhSSUVIS3V5ZW04YXMvckZSTHN0Q0wvTzhRTDM4MEsxUEtENmxOVnltY1IrVApkb1FBN1JXZnZJVFlPZ3cySmRoOThEVGNrVXI2REJtMVBzRlkxK29aMlJrZXZpZmpWTEJOWVZFWW8zTkdXWVRhCk8zcTlBZ01CQUFHZ0FEQU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUFRSi9CbmI3VmxBc1FLWEwvT3QyNlp4TVAKcEpTSFR1VDcwYU5rYXc0SzJuNmhqeVNwd3NLcmNyR1VpODVQeHNCVFhkSDg3MWFRckE1eHNvNGxlZUd5NkszVgorUGowRDBGZWlmclBoV2N4ZTMvdG9WTE1KZjZDYWVaWUE1aVlreS9wckRpS05QQ0l6S0I4ZWEyeUVib205aElRCm9pS1ZUTEd6NjBHTVVGWFp2SmUzVGI0aGpKZmFQbUMrMkNxYnY3T05NaEluQXBBSzhwSlNvOWtRQ3M4cWlpODQKWDNvRnpzcEUxRFVHMjR2SWZpMUxhUWllbW5pdVJWRXBiOFpycWMyemI4Sk0rK0hiR1BjMjhWUEFFL05obTVCVQpKTmpvWjloaERSZU0zOHZ5STAxaXpxMytGU0thZVZ4OVNMaEh5bENBZXJDVUQ4dkxtZ0dtT1pJWHh2c2ZvQT09Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
```  

<br/>

아래 명령어로 K8s CSR 리소스를 생성하여, 인증서 발급을 요청한다.


```bash
kubectl apply -f csr.yaml
``` 
Output
```bash
certificatesigningrequest.certificates.k8s.io/dev25 created
```  

<br/>


#### K8s CSR 확인 과 Approve

<br/>

CSR 을 확인해보면 현재 pending 되어있는 CSR를 확인 할 수 있다. 

```bash
root@edu25:~/security# kubectl get csr
NAME    AGE   SIGNERNAME                            REQUESTOR   REQUESTEDDURATION   CONDITION
dev25   9s    kubernetes.io/kube-apiserver-client   edu25       <none>              Pending
```  

<br/>  

다음 명령어로 해당 CSR을 승인하여, K8s CA가 서명한 인증서를 얻을 수 있다.

```bash
kubectl certificate approve dev25
```  

<br/>

```bash
certificatesigningrequest.certificates.k8s.io/dev25 approved
```  

<br/>


#### Client Certificate 파일 생성

<br/>

승인 완료시, 해당 CSR 리소스에 CA가 서명한 인증서 데이터가 추가되며, 다음 명령어로 이를 확인할 수 있다. 

<br/>

```bash
root@edu25:~/security# kubectl get csr dev25 -o yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"certificates.k8s.io/v1","kind":"CertificateSigningRequest","metadata":{"annotations":{},"name":"dev25"},"spec":{"groups":["system:authenticated"],"request":"LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ21qQ0NBWUlDQVFBd1ZURUxNQWtHQTFVRUJoTUNRVlV4RXpBUkJnTlZCQWdNQ2xOdmJXVXRVM1JoZEdVeApJVEFmQmdOVkJBb01HRWx1ZEdWeWJtVjBJRmRwWkdkcGRITWdVSFI1SUV4MFpERU9NQXdHQTFVRUF3d0ZaR1YyCk1qVXdnZ0VpTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCRHdBd2dnRUtBb0lCQVFEZW1WMS8rTHdKSGduZlZyZFEKQjFRdERiaUlTY0hTbXFpYlNEdUJ4cXBoTDlNM2JQTTYrRUZ0WW9HN3R1VExBZWxvcFlnd2Z6U0p3a0tlRGNuQQpMeE80b1hlZUV5cUFxY1JvNGpzZ1RmL3J2MzRXUFJMeXZ0WlZwRGZ6bVpsWXl1M3V0aStxb0xMWnIrL1RHWDRNCkNkSVlRVVd0dldsYWdoVWhMNGNNTUFuUTZJaFdET0xVQ2ViTy9ZZWJ4OThMQVcyS25abmJ6OWxHd0ZGaTNydnkKVGNKVThYL0QzUS9YTWFXYzNJQlhSSUVIS3V5ZW04YXMvckZSTHN0Q0wvTzhRTDM4MEsxUEtENmxOVnltY1IrVApkb1FBN1JXZnZJVFlPZ3cySmRoOThEVGNrVXI2REJtMVBzRlkxK29aMlJrZXZpZmpWTEJOWVZFWW8zTkdXWVRhCk8zcTlBZ01CQUFHZ0FEQU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUFRSi9CbmI3VmxBc1FLWEwvT3QyNlp4TVAKcEpTSFR1VDcwYU5rYXc0SzJuNmhqeVNwd3NLcmNyR1VpODVQeHNCVFhkSDg3MWFRckE1eHNvNGxlZUd5NkszVgorUGowRDBGZWlmclBoV2N4ZTMvdG9WTE1KZjZDYWVaWUE1aVlreS9wckRpS05QQ0l6S0I4ZWEyeUVib205aElRCm9pS1ZUTEd6NjBHTVVGWFp2SmUzVGI0aGpKZmFQbUMrMkNxYnY3T05NaEluQXBBSzhwSlNvOWtRQ3M4cWlpODQKWDNvRnpzcEUxRFVHMjR2SWZpMUxhUWllbW5pdVJWRXBiOFpycWMyemI4Sk0rK0hiR1BjMjhWUEFFL05obTVCVQpKTmpvWjloaERSZU0zOHZ5STAxaXpxMytGU0thZVZ4OVNMaEh5bENBZXJDVUQ4dkxtZ0dtT1pJWHh2c2ZvQT09Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=","signerName":"kubernetes.io/kube-apiserver-client","usages":["client auth"]}}
  creationTimestamp: "2023-11-05T00:08:19Z"
  managedFields:
  ...
  uid: 68705a17-1dd3-47f7-a662-f3a4cf235f61
spec:
  extra:
    scopes.authorization.openshift.io:
    - user:full
  groups:
  - system:authenticated:oauth
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ21qQ0NBWUlDQVFBd1ZURUxNQWtHQTFVRUJoTUNRVlV4RXpBUkJnTlZCQWdNQ2xOdmJXVXRVM1JoZEdVeApJVEFmQmdOVkJBb01HRWx1ZEdWeWJtVjBJRmRwWkdkcGRITWdVSFI1SUV4MFpERU9NQXdHQTFVRUF3d0ZaR1YyCk1qVXdnZ0VpTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCRHdBd2dnRUtBb0lCQVFEZW1WMS8rTHdKSGduZlZyZFEKQjFRdERiaUlTY0hTbXFpYlNEdUJ4cXBoTDlNM2JQTTYrRUZ0WW9HN3R1VExBZWxvcFlnd2Z6U0p3a0tlRGNuQQpMeE80b1hlZUV5cUFxY1JvNGpzZ1RmL3J2MzRXUFJMeXZ0WlZwRGZ6bVpsWXl1M3V0aStxb0xMWnIrL1RHWDRNCkNkSVlRVVd0dldsYWdoVWhMNGNNTUFuUTZJaFdET0xVQ2ViTy9ZZWJ4OThMQVcyS25abmJ6OWxHd0ZGaTNydnkKVGNKVThYL0QzUS9YTWFXYzNJQlhSSUVIS3V5ZW04YXMvckZSTHN0Q0wvTzhRTDM4MEsxUEtENmxOVnltY1IrVApkb1FBN1JXZnZJVFlPZ3cySmRoOThEVGNrVXI2REJtMVBzRlkxK29aMlJrZXZpZmpWTEJOWVZFWW8zTkdXWVRhCk8zcTlBZ01CQUFHZ0FEQU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUFRSi9CbmI3VmxBc1FLWEwvT3QyNlp4TVAKcEpTSFR1VDcwYU5rYXc0SzJuNmhqeVNwd3NLcmNyR1VpODVQeHNCVFhkSDg3MWFRckE1eHNvNGxlZUd5NkszVgorUGowRDBGZWlmclBoV2N4ZTMvdG9WTE1KZjZDYWVaWUE1aVlreS9wckRpS05QQ0l6S0I4ZWEyeUVib205aElRCm9pS1ZUTEd6NjBHTVVGWFp2SmUzVGI0aGpKZmFQbUMrMkNxYnY3T05NaEluQXBBSzhwSlNvOWtRQ3M4cWlpODQKWDNvRnpzcEUxRFVHMjR2SWZpMUxhUWllbW5pdVJWRXBiOFpycWMyemI4Sk0rK0hiR1BjMjhWUEFFL05obTVCVQpKTmpvWjloaERSZU0zOHZ5STAxaXpxMytGU0thZVZ4OVNMaEh5bENBZXJDVUQ4dkxtZ0dtT1pJWHh2c2ZvQT09Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  uid: e43391ac-b12c-4abe-8594-8d5d0d7c3bc2
  usages:
  - client auth
  username: edu25
status:
  certificate: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURTekNDQWpPZ0F3SUJBZ0lRUkttelViS1pkODc3d2N1c2o3eGtKekFOQmdrcWhraUc5dzBCQVFzRkFEQW0KTVNRd0lnWURWUVFEREJ0cmRXSmxMV056Y2kxemFXZHVaWEpmUURFMk9Ua3dNakV4TkRnd0hoY05Nak14TVRBMQpNREF3TkRVNFdoY05Nak14TWpBek1UUXhPVEE0V2pCVk1Rc3dDUVlEVlFRR0V3SkJWVEVUTUJFR0ExVUVDQk1LClUyOXRaUzFUZEdGMFpURWhNQjhHQTFVRUNoTVlTVzUwWlhKdVpYUWdWMmxrWjJsMGN5QlFkSGtnVEhSa01RNHcKREFZRFZRUURFd1ZrWlhZeU5UQ0NBU0l3RFFZSktvWklodmNOQVFFQkJRQURnZ0VQQURDQ0FRb0NnZ0VCQU42WgpYWC80dkFrZUNkOVd0MUFIVkMwTnVJaEp3ZEthcUp0SU80SEdxbUV2MHpkczh6cjRRVzFpZ2J1MjVNc0I2V2lsCmlEQi9OSW5DUXA0TnljQXZFN2loZDU0VEtvQ3B4R2ppT3lCTi8rdS9maFk5RXZLKzFsV2tOL09abVZqSzdlNjIKTDZxZ3N0bXY3OU1aZmd3SjBoaEJSYTI5YVZxQ0ZTRXZod3d3Q2REb2lGWU00dFFKNXM3OWg1dkgzd3NCYllxZAptZHZQMlViQVVXTGV1L0pOd2xUeGY4UGREOWN4cFp6Y2dGZEVnUWNxN0o2YnhxeitzVkV1eTBJdjg3eEF2ZnpRCnJVOG9QcVUxWEtaeEg1TjJoQUR0RlorOGhOZzZERFlsMkgzd05OeVJTdm9NR2JVK3dWalg2aG5aR1I2K0orTlUKc0UxaFVSaWpjMFpaaE5vN2VyMENBd0VBQWFOR01FUXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUhBd0l3REFZRApWUjBUQVFIL0JBSXdBREFmQmdOVkhTTUVHREFXZ0JSdUtvTlJnaER1Sm02TWx1d3FmUkc3VTR2VUJUQU5CZ2txCmhraUc5dzBCQVFzRkFBT0NBUUVBZG55cWszWlJFcE1RU0t2VklBcWJCTnRQaDlNRDdrVCtZQkJzWHpGYVQvTWwKK3FwaWNrMFRscEVxVDRudVVlbkhxRm1VbG9TSy9lQ0lPMHdtaVp0Mjd4Vkw5U1BlbXFSSnNQZ2taNkt2QkFQYwpFT0ZvZ1A3c3ZvRkJUY2JiN0Qxa0RSQVdWR0dKdHNqR3lzbVdMdHIvR3ZuekVvSVB2WFdaZHljdGwxUDhib2FSCmlXVkJPWGh4NlVNRXczOFJxMUJZdmkwTGZHNHlOOWVvcytSM2tFeURhRElxaEtlS0xFbDJMNjlrYlM2T0UzQjEKTHBZbjYxYThVYU0xSlpLS0tYeDZTQnVqT3gzVzNwMExRaEFDYk0vR1d0R1FjY0dpSHJqaXYvSUpBMGpVVFU1egpLU0d1d1NLOXNOaEZJUlBVMmFKeERPQUUyL205VUVTUnd4R2RGelU3eXc9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  conditions:
  - lastTransitionTime: "2023-11-05T00:09:58Z"
    lastUpdateTime: "2023-11-05T00:09:58Z"
    message: This CSR was approved by kubectl certificate approve.
    reason: KubectlApprove
    status: "True"
    type: Approved
```

<br/> 

.status.certificate 항목에 base64로 인코딩된 인증서가 위치하며, 아래 명령어로 원본 데이터인 CRT 화일을 얻을 수 있다.


```bash
echo "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURTekNDQWpPZ0F3SUJBZ0lRUkttelViS1pkODc3d2N1c2o3eGtKekFOQmdrcWhraUc5dzBCQVFzRkFEQW0KTVNRd0lnWURWUVFEREJ0cmRXSmxMV056Y2kxemFXZHVaWEpmUURFMk9Ua3dNakV4TkRnd0hoY05Nak14TVRBMQpNREF3TkRVNFdoY05Nak14TWpBek1UUXhPVEE0V2pCVk1Rc3dDUVlEVlFRR0V3SkJWVEVUTUJFR0ExVUVDQk1LClUyOXRaUzFUZEdGMFpURWhNQjhHQTFVRUNoTVlTVzUwWlhKdVpYUWdWMmxrWjJsMGN5QlFkSGtnVEhSa01RNHcKREFZRFZRUURFd1ZrWlhZeU5UQ0NBU0l3RFFZSktvWklodmNOQVFFQkJRQURnZ0VQQURDQ0FRb0NnZ0VCQU42WgpYWC80dkFrZUNkOVd0MUFIVkMwTnVJaEp3ZEthcUp0SU80SEdxbUV2MHpkczh6cjRRVzFpZ2J1MjVNc0I2V2lsCmlEQi9OSW5DUXA0TnljQXZFN2loZDU0VEtvQ3B4R2ppT3lCTi8rdS9maFk5RXZLKzFsV2tOL09abVZqSzdlNjIKTDZxZ3N0bXY3OU1aZmd3SjBoaEJSYTI5YVZxQ0ZTRXZod3d3Q2REb2lGWU00dFFKNXM3OWg1dkgzd3NCYllxZAptZHZQMlViQVVXTGV1L0pOd2xUeGY4UGREOWN4cFp6Y2dGZEVnUWNxN0o2YnhxeitzVkV1eTBJdjg3eEF2ZnpRCnJVOG9QcVUxWEtaeEg1TjJoQUR0RlorOGhOZzZERFlsMkgzd05OeVJTdm9NR2JVK3dWalg2aG5aR1I2K0orTlUKc0UxaFVSaWpjMFpaaE5vN2VyMENBd0VBQWFOR01FUXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUhBd0l3REFZRApWUjBUQVFIL0JBSXdBREFmQmdOVkhTTUVHREFXZ0JSdUtvTlJnaER1Sm02TWx1d3FmUkc3VTR2VUJUQU5CZ2txCmhraUc5dzBCQVFzRkFBT0NBUUVBZG55cWszWlJFcE1RU0t2VklBcWJCTnRQaDlNRDdrVCtZQkJzWHpGYVQvTWwKK3FwaWNrMFRscEVxVDRudVVlbkhxRm1VbG9TSy9lQ0lPMHdtaVp0Mjd4Vkw5U1BlbXFSSnNQZ2taNkt2QkFQYwpFT0ZvZ1A3c3ZvRkJUY2JiN0Qxa0RSQVdWR0dKdHNqR3lzbVdMdHIvR3ZuekVvSVB2WFdaZHljdGwxUDhib2FSCmlXVkJPWGh4NlVNRXczOFJxMUJZdmkwTGZHNHlOOWVvcytSM2tFeURhRElxaEtlS0xFbDJMNjlrYlM2T0UzQjEKTHBZbjYxYThVYU0xSlpLS0tYeDZTQnVqT3gzVzNwMExRaEFDYk0vR1d0R1FjY0dpSHJqaXYvSUpBMGpVVFU1egpLU0d1d1NLOXNOaEZJUlBVMmFKeERPQUUyL205VUVTUnd4R2RGelU3eXc9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==" | base64 -d > dev25.crt
```

<br/>   

Output  
```bash
-----BEGIN CERTIFICATE-----
MIIDSzCCAjOgAwIBAgIQRKmzUbKZd877wcusj7xkJzANBgkqhkiG9w0BAQsFADAm
MSQwIgYDVQQDDBtrdWJlLWNzci1zaWduZXJfQDE2OTkwMjExNDgwHhcNMjMxMTA1
MDAwNDU4WhcNMjMxMjAzMTQxOTA4WjBVMQswCQYDVQQGEwJBVTETMBEGA1UECBMK
U29tZS1TdGF0ZTEhMB8GA1UEChMYSW50ZXJuZXQgV2lkZ2l0cyBQdHkgTHRkMQ4w
DAYDVQQDEwVkZXYyNTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAN6Z
XX/4vAkeCd9Wt1AHVC0NuIhJwdKaqJtIO4HGqmEv0zds8zr4QW1igbu25MsB6Wil
iDB/NInCQp4NycAvE7ihd54TKoCpxGjiOyBN/+u/fhY9EvK+1lWkN/OZmVjK7e62
L6qgstmv79MZfgwJ0hhBRa29aVqCFSEvhwwwCdDoiFYM4tQJ5s79h5vH3wsBbYqd
mdvP2UbAUWLeu/JNwlTxf8PdD9cxpZzcgFdEgQcq7J6bxqz+sVEuy0Iv87xAvfzQ
rU8oPqU1XKZxH5N2hADtFZ+8hNg6DDYl2H3wNNyRSvoMGbU+wVjX6hnZGR6+J+NU
sE1hURijc0ZZhNo7er0CAwEAAaNGMEQwEwYDVR0lBAwwCgYIKwYBBQUHAwIwDAYD
VR0TAQH/BAIwADAfBgNVHSMEGDAWgBRuKoNRghDuJm6MluwqfRG7U4vUBTANBgkq
hkiG9w0BAQsFAAOCAQEAdnyqk3ZREpMQSKvVIAqbBNtPh9MD7kT+YBBsXzFaT/Ml
+qpick0TlpEqT4nuUenHqFmUloSK/eCIO0wmiZt27xVL9SPemqRJsPgkZ6KvBAPc
EOFogP7svoFBTcbb7D1kDRAWVGGJtsjGysmWLtr/GvnzEoIPvXWZdyctl1P8boaR
iWVBOXhx6UMEw38Rq1BYvi0LfG4yN9eos+R3kEyDaDIqhKeKLEl2L69kbS6OE3B1
LpYn61a8UaM1JZKKKXx6SBujOx3W3p0LQhACbM/GWtGQccGiHrjiv/IJA0jUTU5z
KSGuwSK9sNhFIRPU2aJxDOAE2/m9UESRwxGdFzU7yw==
-----END CERTIFICATE-----
```  

<br/>

이제 dev25.key, dev25.crt 두 개의 파일을 사용하여 kube-apiserver 접근이 가능하다.  

지금 까지 생성된 화일입니다.  

```bash
root@edu25:~/security# ls -al
total 24
drwxr-xr-x  2 root root 4096 Nov  5 00:13 .
drwx------ 10 root root 4096 Nov  5 00:07 ..
-rw-r--r--  1 root root 1528 Nov  5 00:07 csr.yaml
-rw-r--r--  1 root root 1204 Nov  5 00:13 dev25.crt
-rw-r--r--  1 root root  980 Nov  5 00:05 dev25.csr
-rw-------  1 root root 1679 Nov  5 00:04 dev25.key
```  

<br/>

인증서 정보를 확인해 본다.    

```bash
openssl x509 -noout  -text -in dev25.crt
```      

Output
```bash  
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            44:a9:b3:51:b2:99:77:ce:fb:c1:cb:ac:8f:bc:64:27
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = kube-csr-signer_@1699021148
        Validity
            Not Before: Nov  5 00:04:58 2023 GMT
            Not After : Dec  3 14:19:08 2023 GMT
        Subject: C = AU, ST = Some-State, O = Internet Widgits Pty Ltd, CN = dev25
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:de:99:5d:7f:f8:bc:09:1e:09:df:56:b7:50:07:
                    54:2d:0d:b8:88:49:c1:d2:9a:a8:9b:48:3b:81:c6:
                    aa:61:2f:d3:37:6c:f3:3a:f8:41:6d:62:81:bb:b6:
                    e4:cb:01:e9:68:a5:88:30:7f:34:89:c2:42:9e:0d:
                    c9:c0:2f:13:b8:a1:77:9e:13:2a:80:a9:c4:68:e2:
                    3b:20:4d:ff:eb:bf:7e:16:3d:12:f2:be:d6:55:a4:
                    37:f3:99:99:58:ca:ed:ee:b6:2f:aa:a0:b2:d9:af:
                    ef:d3:19:7e:0c:09:d2:18:41:45:ad:bd:69:5a:82:
                    15:21:2f:87:0c:30:09:d0:e8:88:56:0c:e2:d4:09:
                    e6:ce:fd:87:9b:c7:df:0b:01:6d:8a:9d:99:db:cf:
                    d9:46:c0:51:62:de:bb:f2:4d:c2:54:f1:7f:c3:dd:
                    0f:d7:31:a5:9c:dc:80:57:44:81:07:2a:ec:9e:9b:
                    c6:ac:fe:b1:51:2e:cb:42:2f:f3:bc:40:bd:fc:d0:
                    ad:4f:28:3e:a5:35:5c:a6:71:1f:93:76:84:00:ed:
                    15:9f:bc:84:d8:3a:0c:36:25:d8:7d:f0:34:dc:91:
                    4a:fa:0c:19:b5:3e:c1:58:d7:ea:19:d9:19:1e:be:
                    27:e3:54:b0:4d:61:51:18:a3:73:46:59:84:da:3b:
                    7a:bd
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Extended Key Usage:
                TLS Web Client Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Authority Key Identifier:
                keyid:6E:2A:83:51:82:10:EE:26:6E:8C:96:EC:2A:7D:11:BB:53:8B:D4:05

    Signature Algorithm: sha256WithRSAEncryption
         76:7c:aa:93:76:51:12:93:10:48:ab:d5:20:0a:9b:04:db:4f:
         87:d3:03:ee:44:fe:60:10:6c:5f:31:5a:4f:f3:25:fa:aa:62:
         72:4d:13:96:91:2a:4f:89:ee:51:e9:c7:a8:59:94:96:84:8a:
         fd:e0:88:3b:4c:26:89:9b:76:ef:15:4b:f5:23:de:9a:a4:49:
         b0:f8:24:67:a2:af:04:03:dc:10:e1:68:80:fe:ec:be:81:41:
         4d:c6:db:ec:3d:64:0d:10:16:54:61:89:b6:c8:c6:ca:c9:96:
         2e:da:ff:1a:f9:f3:12:82:0f:bd:75:99:77:27:2d:97:53:fc:
         6e:86:91:89:65:41:39:78:71:e9:43:04:c3:7f:11:ab:50:58:
         be:2d:0b:7c:6e:32:37:d7:a8:b3:e4:77:90:4c:83:68:32:2a:
         84:a7:8a:2c:49:76:2f:af:64:6d:2e:8e:13:70:75:2e:96:27:
         eb:56:bc:51:a3:35:25:92:8a:29:7c:7a:48:1b:a3:3b:1d:d6:
         de:9d:0b:42:10:02:6c:cf:c6:5a:d1:90:71:c1:a2:1e:b8:e2:
         bf:f2:09:03:48:d4:4d:4e:73:29:21:ae:c1:22:bd:b0:d8:45:
         21:13:d4:d9:a2:71:0c:e0:04:db:f9:bd:50:44:91:c3:11:9d:
         17:35:3b:cb
```  

<br/>

인증서 유효 기간 확인  

```bash  
openssl x509 -in dev25.crt -noout -dates
```  

<br/>

유효기간은 12월 3일 까지 인것을 확인 할 수 있다.  

```bash
notBefore=Nov  5 00:04:58 2023 GMT
notAfter=Dec  3 14:19:08 2023 GMT
```  

유효기간은 위에서 생성시에 `openssl req -new -key dev25.key -out dev25.csr` 위의 명령어에 `-days 3650` 를 추가하면 10년짜리 인증서 발급 가능하다.  


<br/>


이제 dev25.key, dev25.crt 두 개의 파일을 사용하여 kube-apiserver 접근이 가능하다. 물론 어떠한 RBAC 설정도 되지 않은 계정이기 때문에 아직 할 수 있는 동작은 없다.   

<br/>

#### kubeconfig 설정  

<br/>

새로 추가한 사용자 계정으로 kubectl을 사용하기 위해선 kubeconfig에 해당 정보가 있어야 한다.


<br/>

현재 context를 조회해 봅니다.    

```bash
kubectl config get-contexts
```  

Output
```bash
CURRENT   NAME                                           CLUSTER                            AUTHINFO                                 NAMESPACE
*         edu25/api-okd4-ktdemo-duckdns-org:6443/edu25   api-okd4-ktdemo-duckdns-org:6443   edu25/api-okd4-ktdemo-duckdns-org:6443   edu25
```   

<br/>  

새로 추가한 사용자 계정으로 kubectl을 사용하기 위해선 kubeconfig에 해당 정보가 있어야 한다. 아래 명령어로 추가해보자.  

```bash
kubectl config set-credentials dev25 --client-key=dev25.key --client-certificate=dev25.crt --embed-certs
```     

<br/>

기존이 edu25 계정과 함께 dev25 계정이 추가 된것을 확인 할 수 있습니다.  

```bash
kubectl config view
```  

Output
```bash
apiVersion: v1
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: https://api.okd4.ktdemo.duckdns.org:6443
  name: api-okd4-ktdemo-duckdns-org:6443
contexts:
- context:
    cluster: api-okd4-ktdemo-duckdns-org:6443
    namespace: edu25
    user: edu25/api-okd4-ktdemo-duckdns-org:6443
  name: edu25/api-okd4-ktdemo-duckdns-org:6443/edu25
current-context: edu25/api-okd4-ktdemo-duckdns-org:6443/edu25
kind: Config
preferences: {}
users:
- name: dev25
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
- name: edu25/api-okd4-ktdemo-duckdns-org:6443
  user:
    token: REDACTED
```

<br/>

이제 K8s 클러스터와 kubectl 사용자를 매칭하는 context를 정의할 차례다.

<br/>

```bash
kubectl config set-context dev25 --user=dev25 --cluster=api-okd4-ktdemo-duckdns-org:6443
```  

Output  
```bash
Context "dev25" created.
```  
<br/>

context를 다시 조회해 본다. 신규로 추가된 것을 알 수 있다.   

```bash
kubectl config get-contexts
```  

Output
```bash
CURRENT   NAME                                           CLUSTER                            AUTHINFO                                 NAMESPACE
          dev25                                          api-okd4-ktdemo-duckdns-org:6443   dev25
*         edu25/api-okd4-ktdemo-duckdns-org:6443/edu25   api-okd4-ktdemo-duckdns-org:6443   edu25/api-okd4-ktdemo-duckdns-org:6443   edu25
```

<br/>

context 를 바꿔봅니다.    

```bash
kubectl config use-context dev25
```  

Output
```bash
Switched to context "dev25".
```  

<br/>

새로 추가한 사용자 `dev25` 으로  OKD 클러스터 에 접근할 수 있다. 다만 RBAC 설정을 하지 않은 상태이므로, 대부분의 명령어는 실패한다.  

pod 를 조회해 보면 권한이 없다고 나옵니다.  

```bash
kubectl get po -n edu25
```  
Output
```bash
Error from server (Forbidden): pods is forbidden: User "dev25" cannot list resource "pods" in API group "" in the namespace "edu25"
```  

<br/>

#### PEM , OpenSSH , CRT , 개인키 차이

<br/>

참고 : https://www.lesstif.com/software-architect/pem-cer-der-crt-csr-113345004.html  

<br/>

> PEM  

PEM (Privacy Enhanced Mail)은 Base64 로 인코딩한 텍스트 형식의 파일입니다.

Binary 형식의 파일을 전송할 때 손상될 수 있으므로 TEXT 로 변환하며 소스 파일은 모든 바이너리가 가능하지만 주로 인증서나 개인키가 됩니다.

AWS 에서 EC2 Instance 를 만들때 접속용으로 생성하는 개인키도 PEM 형식입니다.

어떤 바이너리 파일을 PEM 으로 변환했는지 구분하기 위해 파일의 맨 앞에 dash(-) 를 5 개 넣고 BEGIN 파일 유형을 넣고 다시 dash(-) 를 5개 뒤에 END 파일유형 구문을 사용합니다.      

<br/>

> OpenSSH Private Key  

즉 아래는 OPENSSH Private Key 를 PEM 으로 변환한 예시로 BEGIN OPENSSH PRIVATE KEY 로 시작하는 것을 확인할 수 있습니다.    

ssh-keygen 으로 생성하며 id_rsa private key이 아래와 같이 만들어 집니다.

```bash
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAsXOlJGsuzdxK6fC9DuQ473b6bwCz85Zi0AcpG9HZg1YhAyJNpl6S
2CQCd3gxi3OtdEofXbtRr9xyr7GfD5Cl8vw9d0gQ1q21mrVC+b1/lAiI5XRI9qvi4ORRSf
SwOviCse3cqAZwMlbOUhWKzynLeYF11JdTQH/uAhSSROa0wgKGlPfCdgRYo7piU7UDXHnz
t17w+CpofslmihF2gPEzRicbAmL9hkUDifwFnY/6fVuc0DSQDqgGGRLaKG32/FFX0iP4zW
yMRrCkdEo39E9wyLS3nx1xjQdYIEkjVYBxSiktWKEiYoVlmVUmBejmzNXOh/XQPs2tzUqM
Ji67bGMl5niAH9W2V5MxH7HiqZceR9ovOZuu/BajFrGP3H6EKMyNC9t9gwe9RV2bUMmrPW
+ygHoF8fIR8ZGoSv2GvoVtMBu/6QOkucp+DH+8bdHqRZNWK0muk/BEy8NPnDp2bpd/EDMV
2fJVvi1iZYPxO0vM73PZGQpoYfVEbh5fDNiBH5wbAAAFmAduPlUHbj5VAAAAB3NzaC1yc2
EAAAGBALFzpSRrLs3cSunwvQ7kOO92+m8As/OWYtAHKRvR2YNWIQMiTaZektgkAnd4MYtz
rXRKH127Ua/ccq+xnw+QpfL8PXdIENattZq1Qvm9f5QIiOV0SPar4uDkUUn0sDr4grHt3K
gGcDJWzlIVis8py3mBddSXU0B/7gIUkkTmtMIChpT3wnYEWKO6YlO1A1x587de8PgqaH7J
ZooRdoDxM0YnGwJi/YZFA4n8BZ2P+n1bnNA0kA6oBhkS2iht9vxRV9Ij+M1sjEawpHRKN/
RPcMi0t58dcY0HWCBJI1WAcUopLVihImKFZZlVJgXo5szVzof10D7Nrc1KjCYuu2xjJeZ4
gB/VtleTMR+x4qmXHkfaLzmbrvwWoxaxj9x+hCjMjQvbfYMHvUVdm1DJqz1vsoB6BfHyEf
...
1naaYgzRtVzMcQn7cKt8JBAQu44x15ocvGTKFPLM4O04nhmuobGBCzU2KuLjFjivbRU7bE
z6SuQgNn+YAHFX2BuPhkfD6TQKBzdIbLCkBZSi2ANOAyN/Vli+siDnf58cGD6K0WfDNF/1
1E/yW2sXbmiRsAAAAdamFrZWxlZUBqYWtlLU1hY0Jvb2tBaXIubG9jYWwBAgMEBQY=
-----END OPENSSH PRIVATE KEY-----
```  

<br/>

> CRT  : PKI 인증서(Certificate)는 BEGIN CERTIFICATE 구문으로 시작합니다.  

인증서를 의미하는 CERT 의 약자로 보통 PEM 형식의 인증서를 의미하며 Linux 나 Unix 계열에서 .crt 확장자를 많이 사용합니다. 

에디터로 파일을 열어서 BEGIN CERTIFICATE 구문이 있는지 확인하면 됩니다.  

```bash
cat dev25.crt
```  

Output
```bash
-----BEGIN CERTIFICATE-----
MIIDSzCCAjOgAwIBAgIQRKmzUbKZd877wcusj7xkJzANBgkqhkiG9w0BAQsFADAm
MSQwIgYDVQQDDBtrdWJlLWNzci1zaWduZXJfQDE2OTkwMjExNDgwHhcNMjMxMTA1
MDAwNDU4WhcNMjMxMjAzMTQxOTA4WjBVMQswCQYDVQQGEwJBVTETMBEGA1UECBMK
U29tZS1TdGF0ZTEhMB8GA1UEChMYSW50ZXJuZXQgV2lkZ2l0cyBQdHkgTHRkMQ4w
DAYDVQQDEwVkZXYyNTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAN6Z
XX/4vAkeCd9Wt1AHVC0NuIhJwdKaqJtIO4HGqmEv0zds8zr4QW1igbu25MsB6Wil
iDB/NInCQp4NycAvE7ihd54TKoCpxGjiOyBN/+u/fhY9EvK+1lWkN/OZmVjK7e62
L6qgstmv79MZfgwJ0hhBRa29aVqCFSEvhwwwCdDoiFYM4tQJ5s79h5vH3wsBbYqd
mdvP2UbAUWLeu/JNwlTxf8PdD9cxpZzcgFdEgQcq7J6bxqz+sVEuy0Iv87xAvfzQ
rU8oPqU1XKZxH5N2hADtFZ+8hNg6DDYl2H3wNNyRSvoMGbU+wVjX6hnZGR6+J+NU
sE1hURijc0ZZhNo7er0CAwEAAaNGMEQwEwYDVR0lBAwwCgYIKwYBBQUHAwIwDAYD
VR0TAQH/BAIwADAfBgNVHSMEGDAWgBRuKoNRghDuJm6MluwqfRG7U4vUBTANBgkq
hkiG9w0BAQsFAAOCAQEAdnyqk3ZREpMQSKvVIAqbBNtPh9MD7kT+YBBsXzFaT/Ml
+qpick0TlpEqT4nuUenHqFmUloSK/eCIO0wmiZt27xVL9SPemqRJsPgkZ6KvBAPc
EOFogP7svoFBTcbb7D1kDRAWVGGJtsjGysmWLtr/GvnzEoIPvXWZdyctl1P8boaR
iWVBOXhx6UMEw38Rq1BYvi0LfG4yN9eos+R3kEyDaDIqhKeKLEl2L69kbS6OE3B1
LpYn61a8UaM1JZKKKXx6SBujOx3W3p0LQhACbM/GWtGQccGiHrjiv/IJA0jUTU5z
KSGuwSK9sNhFIRPU2aJxDOAE2/m9UESRwxGdFzU7yw==
-----END CERTIFICATE-----
```  

<br/>

> 개인키  ( PEM 형식 )

개인키는 BEGIN RSA PRIVATE KEY 로 시작합니다.  

```bash
cat dev25.key
```  

```bash
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA3pldf/i8CR4J31a3UAdULQ24iEnB0pqom0g7gcaqYS/TN2zz
OvhBbWKBu7bkywHpaKWIMH80icJCng3JwC8TuKF3nhMqgKnEaOI7IE3/679+Fj0S
8r7WVaQ385mZWMrt7rYvqqCy2a/v0xl+DAnSGEFFrb1pWoIVIS+HDDAJ0OiIVgzi
1Anmzv2Hm8ffCwFtip2Z28/ZRsBRYt678k3CVPF/w90P1zGlnNyAV0SBByrsnpvG
rP6xUS7LQi/zvEC9/NCtTyg+pTVcpnEfk3aEAO0Vn7yE2DoMNiXYffA03JFK+gwZ
tT7BWNfqGdkZHr4n41SwTWFRGKNzRlmE2jt6vQIDAQABAoIBAQDEnnMYNnzhEMdn
nxEMf2y63wPAXmX1wOZtQsBNQU39ymCm9HVkASTJmdk+Fa7CIk4pQQ2qyLF/fTea
pFMwjmS9EOK3nfZM76etfSb8wejsM5kLy6aRBEAOJZ/GbEYnSBgiYop4DLntzpnn
vPy5ZXNOOVlyvXvxljVTusdu3H/PJfAON0MPjqScbGwx8g5A5IeVBpwUQZPAPZVV
TgWN1cBF8VYHhRVmDfH9h72CkH37FWbHB/2L3Z76vGcYap1SrSJhj/BeggbHDi2c
u60xS2oV1dN6cyCy3oxE+AKKahoZ6ri/t/ooFqIhbPl8hMr6yyekBikg9uftcmRp
TXLnFX+BAoGBAPqy3faB805A+FrkiHENRxtORDwoPY8hVrvu9XBVlk5jr/bYfCPL
GesTpJjNVRvNyLEBNL06qOvPaFHdsTVPUZ2Jwlhdcff8HhXmxkl6hY9vp7yFFVdv
0DmFpFZOrGaHhZvuayP9PZ3ZsqJmzxDLmBD/kzUrnyy+pYjugqh09VnzAoGBAONO
YjdvTbKLZcqDCV7D6CJuqmDWZPgzfLwAtBvMbYUVsFvO0HMAwbkR8wI9UFSdocPE
iWebiYSV5ArrR71yaCuQaPahrMVq35HYP/Lot6mPJgg6LnyhKgDy/7TWDgKsw23C
0PXcKG89+2oLoo1Zs7wclWlpOR3SlmOj5Suxy9SPAoGAJD2rPLF4fL2DqZAT8VPc
DaR41MF0dLZ7FVvr+ztEKTzb+TE+cOYxbvw99SDpxsUu1/e2qgxK0xv+lqcXsP8w
aze48pE/onu91aiwzXp6yEt50hTjCurNDSO2qAtjfMbml64VqvQ27hTEcBmwoVrt
NrfbjfoqXouI3oysMrIFreUCgYA4aNNm/nBBxuZUA4Dny6ZoJR6TOaGFFwH1hhcs
bucfB+rkXcbNQ3rP+uxbueudlCD4/GU9GRRfmvMk4o7DLQk9BnGGA0llFMi24Pu9
xJMPuT6u/AFdXIGYCrX6osSHVWiKbLZ+zUwbjz49avXELma0YEOUDVDnXcOEpr/Q
wCbdcQKBgQDGNhJUJz4tmjP/+Fh13sIwA6fwX+PSLdUT1mrL61vBnggQqvLCan9Y
+jrgg2IhoO6LFqGfW62QzM4l4FK/G4CW5qkCFVxC6L1pXXY3qlyP1VbrHn/0uIPn
suUrjOj+/IYpbbjTbDxfS71lQdV1ekl3C3SCkK7GABFA+EWiV7EVNw==
-----END RSA PRIVATE KEY-----
```  

<br/>

#### openshift ca 보기

<br/>

elastic 에서 kube-state-metric를 조회하기 위하여  

`openshift-monitoring`  namespace 의 secret 에서 `kube-state-metrics-token` 으로 시작하는 값을 선택 했었습니다.


<img src="./assets/elastic_metric_15.png" style="width: 80%; height: auto;"/>

<br/>

`service-ca.crt` 는 서비스 SSL CA 이고 `ca.crt` 는 API 서버의  SSL CA 이다.    

<img src="./assets/elastic_metric_16.png" style="width: 80%; height: auto;"/>  

reveal values를 해서 `ca.crt` 값을 복사해서 저장합니다.  

<br/>

```bash
openssl x509 -noout  -text -in ca.crt
```  

인증서에 대한 내용을 볼수 있고 유효기간 ( 10년 ) 과 CN 등을 확인 합니다.

```bash
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 3825283850589108238 (0x351623651766f80e)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: OU = openshift, CN = kube-apiserver-lb-signer
        Validity
            Not Before: Sep  1 00:36:54 2023 GMT
            Not After : Aug 29 00:36:54 2033 GMT
        Subject: OU = openshift, CN = kube-apiserver-lb-signer
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:e7:07:f8:8a:48:37:2b:0b:ec:a4:9f:3e:63:21:
                    49:e4:6f:d6:4f:ae:68:b5:87:5f:f0:3b:ce:70:89:
                    5f:7b:9f:6e:27:56:69:16:d1:e5:39:6d:52:a4:c2:
                    40:7b:e3:14:26:6b:78:e9:2a:eb:d4:dd:6f:6f:d2:
                    95:63:7e:0b:00:5c:51:82:a0:c1:77:61:13:3e:b6:
                    16:ad:02:2e:b3:87:bc:e5:af:2b:29:bc:d1:c8:22:
                    60:68:22:74:63:cd:b8:fa:26:19:12:ee:2d:e9:bc:
                    42:dd:9d:80:de:b9:f1:65:d0:41:b4:57:2a:5f:a0:
                    f5:4f:e1:66:96:8a:e8:58:da:7a:4f:fd:9a:d6:01:
                    30:0d:cd:d9:4c:a8:3c:59:30:00:74:35:f1:5d:c7:
                    3c:9e:a0:f2:8c:88:12:f7:0d:d2:9a:e5:06:5c:a4:
                    60:26:0b:54:8a:40:0e:84:40:6b:a0:fc:fe:01:c1:
                    34:e7:0e:6c:ec:54:a0:45:7c:4c:37:86:67:1c:14:
                    90:31:c7:b7:85:b6:c6:31:ea:fd:5e:82:aa:d7:ce:
                    35:7f:e9:46:c2:5c:7a:de:8a:2c:e7:f6:bf:e7:94:
                    a6:a6:16:1d:2f:09:97:80:43:a8:dc:9a:51:4e:d6:
                    c3:8d:79:99:17:a6:bb:7b:c5:5a:d2:71:05:1e:1c:
                    df:3d
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment, Certificate Sign
            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Subject Key Identifier:
                C8:13:AA:B0:A6:6E:CD:8C:9F:CD:3E:B1:D8:7F:4F:57:2D:63:24:BC
    Signature Algorithm: sha256WithRSAEncryption
         a0:1f:09:d0:ba:c3:0c:b8:ac:64:fa:5b:a4:01:d2:be:45:c6:
         0a:a5:53:9f:99:eb:b9:ef:29:2c:7e:f0:9f:a1:37:ad:b8:7f:
         10:7b:69:c0:cc:19:80:f0:a9:69:4e:fa:42:f9:9c:13:56:76:
         11:3f:d2:b6:66:dc:07:d3:5c:ec:b0:89:f7:49:50:be:5f:f0:
         54:35:f4:85:e1:5a:f5:14:59:ff:6f:fa:35:eb:5f:32:5a:6a:
         af:96:04:cd:5b:b1:e1:a2:88:7d:9a:44:f2:65:3f:e0:e5:a6:
         6c:84:71:50:b4:9c:f3:f7:1d:46:4d:4f:48:db:c4:85:d7:3f:
         f4:9f:e6:d7:b6:82:68:d2:0a:48:40:e1:37:c9:ad:21:39:18:
         bd:45:8b:9e:3c:2d:3d:6b:ea:39:fc:e5:6c:87:ed:b9:73:5c:
         cc:e7:a0:8d:d4:39:40:da:7e:97:e6:b4:c0:0a:23:a7:c6:fd:
         0f:63:4d:a3:7b:c0:de:c6:81:d9:04:b7:79:0f:b8:26:65:b4:
         84:72:50:d9:30:0a:0d:9f:8a:a1:41:b7:ac:b7:91:18:90:62:
         ab:f5:42:47:27:fc:e2:77:ec:40:32:a0:4b:6c:26:ad:89:37:
         44:36:c9:3a:74:15:9d:c4:2d:8e:82:ce:c8:49:bc:09:91:6e:
         e4:a6:a0:a7
```  

<br/>

#### 과제

<br/>

dev 계정에 pod를 조회할수 있는 role 을 생성하고 rolebining을 합니다.  
- 권한 할당을 위해서는 context switching 을 해서 edu 유저의 권한으로 주어야 함.  

<br/>


<br/>
