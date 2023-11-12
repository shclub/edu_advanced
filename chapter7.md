# K8S Operator ( CRD )

<br/>

Kubernetes Operator를 이해한다.  

<br/>



1. Operator  

2. CRD    

<br/>

## Operator

<br/>

참고  
- https://ikcoo.tistory.com/95
- https://ikcoo.tistory.com/96


### Operator 란

<br/>

참고  
- 악분일상 : https://youtu.be/sL2dVvDq32E?si=xO-QSl1LCGRqJFZz
- 11번가 :  https://youtu.be/1CLeRP_29Us?si=whoZqZScqgYu6uqt  
- Operator : https://kschoi728.tistory.com/m/4 , 
https://kschoi728.tistory.com/m/5


<br/>

Kubernetes Operator 개념은 2016년 CoreOS의 엔지니어들에 의해 개발되었습니다. 도메인별 지식이 필요한 Kubernetes 클러스터에서 모든 애플리케이션을 구축하고 구동하는 고급 네이티브 방식으로 개발되었습니다.  

쿠버네티스 API와의 긴밀한 협력을 통해 사람의 반응 없이 모든 애플리케이션 운영 프로세스를 자동으로 처리할 수 있는 일관된 접근 방식을 제공한다. 즉, Operator 는 Kubernetes 응용 프로그램을 패키징, 실행 및 관리하는 방법입니다.  

<br/>

 
쿠버네티스 오퍼레이터는 쿠버네티스 애플리케이션을 패키징, 배포 및 관리하는 방법입니다.   

쿠버네티스 오퍼레이터는 쿠버네티스 API의 기능을 확장하여 쿠버네티스 사용자를 대신해 복잡한 애플리케이션의 인스턴스를 생성, 설정 및 관리하는 애플리케이션별 컨트롤러입니다.  

쿠버네티스에서 컨트롤 플레인의 컨트롤러는 클러스터의 원하는 상태를 실제 상태와 반복적으로 비교하는 제어 루프를 구현합니다.  

클러스터의 실제 상태가 원하는 상태와 일치하지 않으면 컨트롤러는 이 문제를 해결하기 위한 작업을 수행합니다.   

오퍼레이터는 Custom Resources 를 사용하여 애플리케이션과 그 구성 요소를 관리하는 사용자 정의 쿠버네티스 컨트롤러입니다.   

Custom Resources 는 쿠버네티스 내의 API 확장 메커니즘입니다.  Custom Resource Definitions 는 Custom Resources 을 정의하고 오퍼레이터 사용자에게 제공되는 모든 설정 목록을 나열합니다.   

쿠버네티스 오퍼레이터는 Custom Resources 유형을 감시하고 애플리케이션별 작업을 수행하여 현재 상태를 해당 리소스에서 원하는 상태와 일치시킵니다.  

쿠버네티스 오퍼레이터는 애플리케이션이 실행되는 동안 이를 지속적으로 모니터링하고, 시간 경과에 따라 자동으로 데이터 백업, 장애 복구, 애플리케이션 업그레이드 작업을 할 수 있습니다.   

<br/>

### Operator 가 작업 자동화을 위해 Kubernetes를 확장합니다.  

<br/>

Operator 는 Custom Resources 를 사용하여 쿠버네티스 응용프로그램과 해당 구성요소를 관리하는 쿠버네티스의 확장입니다. 일반적으로 인간 운영자가 수행하는 소프트웨어 구성 및 유지 관리 작업을 자동화합니다.  

Kubernetes는 Stateless 응용 프로그램을 잘 관리하지만 데이터베이스와 같은 Stateful 응용 프로그램에 대해 더 복잡한 구성 세부 정보가 필요할 때 Operator 가 유용합니다. Stateful 워크로드은 Stateless 워크로드보다 관리하기가 더 어렵습니다. 워크로드의 상태에 따라 워크로드가 다음과 같이 변경됩니다.  

<br/>

- 인스톨 할 필요가 있습니다.  
- 새 버전으로 업그레이드합니다.  
- 장애로부터 복구합니다.  
- 모니터링이 필요합니다.  
- Scales out 했다가 다시 축소합니다.  
- Operator 는 Service 가 성공적으로 실행되는지 확인하는 데 필요한 모든 작업을 처리합니다. Operator 는 Service 를 더 자체 관리하도록 하므로, 애플리케이션 팀은 Service 를 관리하는 데 드는 노력을 줄이고 서비스 사용에 더 많은 노력을 기울일 수 있습니다.

<br/>


### Kubernetes Operators reconcile state  

<br/>

Kubernetes 에서 Control Plane 의 Controller 는 클러스터의 desired 상태를 current 상태와 반복적으로 비교하는 Control Loop 에서 실행된다. 
상태가 일치하지 않으면 Controller 가 current 상태를 desired 상태와 더 가깝게 조정하기 위한 작업을 수행합니다  

마찬가지로 Operator 의 Controller 는 특정 Custom Resource 유형을 감시하고 응용 프로그램별 작업을 수행하여 작업량의 현재 상태가 Custom Resource 에 표현된 대로 desired 상태와 일치하도록 합니다.  

다이어그램은 Control Plane 이 Controller 를 Loop 에서 어떻게 구동하는지 보여준다. 여기서 일부 Controller 는 Kubernetes 에 내장되어 있고 일부는 Operator 의 일부이다.  

<br/>

<img src="./assets/operator_1.png" style="width: 80%; height: auto;"/>  

<kubernetes 제어 루프 >

<br/>

Control Plane 의 Controller 는 `Stateless` 워크로드에 최적화되어 있으며, 모든 Stateless 워크로드에 대해 모두 유사하므로 하나의 Controller 집합이 작동합니다.   

Operator 의 Controller 는 하나의 특정 `Stateful` 워크로드에 맞게 사용자 지정됩니다. 각 Stateful 워크로드에는 해당 워크로드를 관리하는 방법을 아는 자체 Controller 있는 Operator 가 있습니다  

<br/>

### Operator에는 무엇이 있습니까?  

<br/>

Operator 는 특수 기능으로 쿠버네티스 Control Plane 을 확장하여 쿠버네티스 관리자를 대신하여 워크로드를 관리합니다.   

Operator 는 다음을 포함합니다.   

<br/>

- Custom Resource Definition (CRD) : 워크로드을 구성하는 데 사용할 수 있는 설정 스키마를 정의합니다. Operator 는 Custom Resource Definition 를 통해 새 개체 유형을 도입합니다. 
쿠버네티스 API는 쿠버네티스 클라이언트 도구를 통한 상호 작용과 역할 기반 액세스 제어 정책(RBAC)의 포함을 포함하여 네이티브 쿠버네티스 개체와 같은 객체를 처리합니다.   

- Custom Resource (CR) : Custom Resource Definition 에 의해 생성되고 Kubernetes API의 확장입니다. Operator 가 Custom Resource 내에서 구성 설정을 제공한 다음 Operator 가 사용자 지정 컨트롤러 로직을 사용하여 구성을 낮은 수준의 작업으로 변환하여 변환을 구현합니다.  

- Controller : 워크로드에 맞게 사용자 정의되고 Custom Resource 의 값으로 표시되는 desired 상태와 일치하도록 워크로드의 current 상태를 구성하는 컨트롤러입니다.

<br/>

### Operator 의 특징  ( vs Kubernetes workload)

<br/>

기본적으로 Kubernetes는 stateless 워크로드를 잘 관리합니다. Kubernetes가 동일한 논리를 사용하여 모두 관리할 정도로 유사합니다. Stateful 워크로드는 더 복잡하고 각각 다르기 때문에 맞춤형 관리가 필요합니다. 여기에서 Operator 가 들어옵니다.  

<br/>

- Operator 는 Worker Node 와 Control Plane의 구성요소만 사용합니다.  

- Operator 는 Worker Node 에서 실행됩니다. 그러나 Operator 는 Control Plan 에서 실행되는 컨트롤러를 구현합니다.  

- Operator 는 Worker Node 에서 컨트롤러를 실행하기 때문에 Control Plane 을 Worker Node 로 효과적으로 확장할 수 있습니다.

<br/>

### Operator 가 워크로드를 배포하는 방법  

<br/>

Operator 는 인간 관리자(또는 빌드 파이프라인)와 매우 유사한 방식으로 워크로드를 배포합니다.   
Operator 는 결국 IT 관리자가 수행하는 작업을 자동화해야 합니다.  

Kubernetes API는 클라이언트가 관리자인지 Operator 인지 알지 못하므로 클러스터는 동일한 방식으로 워크로드를 배포합니다. 컨트롤 플레인에서 일어나는 모든 일은 동일합니다.  

<br/>

<img src="./assets/operator_2.png" style="width: 80%; height: auto;"/>  

<br/>

## CRD

<br/>

참고 : https://heumsi.github.io/blog/posts/crd-and-kopf-tutorial/

<br/>

### CRD 란?  

<br/>

CRD는 Custom Resource Definition의 약자로, 이름 그대로 사용자가 정의하는 Kubernetes 리소스다. 우리는 CRD를 먼저 정의하고 배포한 후 CR(Custom Resource)를 배포할 수 있다.  

Kubernetes에는 Pod, Deployment, Service 등 기본적인 리소스 유형들이 존재한다. 그런데 이렇게 기본적인 리소스 유형말고 개발자가 직접 리소스 유형을 정의해서 사용할 수 있는데, 이렇게 직접 정의된 리소스들을 CRD라고 부르는 것이다.  

사실 CRD는 아예 새로운 것을 만들어내는 것은 아니고, Kubernetes의 기본 리소스 유형을 조합하여 사용에 맞게 추상화한 리소스라고 보면 된다.  

<br/>

> Kubernetes의 리소스 유형 확장 방법으로 CRD가 유일한 것은 아니다. API Aggregation 이라는 방법도 있는데, 이에 대해서는 Alice ( https://blog.naver.com/alice_k106/221579974362 ) 블로그 글 참고  

<br/>

### CRD 작성하기  

<br/>


보통 Kubernetes에서 앱을 배포하기 위해서는 Deployment와 Service를 배포해야 한다. 그런데 매번 Deployment와 Service의 yaml 파일을 일일이 다 작성하는게 귀찮다. 개발자는 배포에 다음 설정 값들만 작성해도 충분하다고 해보자.  

- Deployment에 어떤 컨테이너 이미지를 쓸지 (image)  
- Deployment에 Pod 수는 몇 개로 할지 (replicas)  
- Service에 포트는 몇번으로 할지 (port)  

<br/>

위 문제 상황을 해결하기 위해 Application 이라는 CR을 만드려고 한다. Application 리소스를 작성할 때 위 3개의 설정 값만 주고, Application 을 배포하면 Deployment와 Service가 자동으로 만들어지게끔 할 것이다.  

이제 Application 의 CRD를 작성을 시작해보자. crd.yaml 을 만들어 다음처럼 작성한다.  

<br/>

```bash
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # name must match the spec fields below, and be in the form: <plural>.<group>
  name: applications.shclub.github.io
```  

- CRD를 작성할 때는 위와 같은 apiVersion 과 kind 를 사용한다.
- metadata.name 값은 CRD의 Full Name이다.
  - 이 값은 <plural>.<group> 형태로 작성되어야 하는데, 아래에서 설명하겠다.
  - 리소스의 Full Name은 API Group(위의 경우 shclub.github.io)을 포함하는데, API Group은 리소스 구분을 위한 일종의 네임스페이스라 보면 된다.     

<br/>

```bash
# crd.yaml

apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # name must match the spec fields below, and be in the form: <plural>.<group>
  name: applications.shclub.github.io
spec:
  # group name to use for REST API: /apis/<group>/<version>
  group: shclub.github.io
  # either Namespaced or Cluster
  scope: Namespaced
  names:
    # plural name to be used in the URL: /apis/<group>/<version>/<plural>
    plural: applications
    # singular name to be used as an alias on the CLI and for display
    singular: application
    # kind is normally the CamelCased singular type. Your resource manifests use this.
    kind: Application
    # shortNames allow shorter string to match your resource on the CLI
    shortNames:
      - app
```  

<br/>

- spec.group 의 값은 API Group이다. API Group은 보통 자신의 도메인을 사용한다.
- spec.scope 의 값은 이 커스텀 리소스가 배포되었을 때의 유효 범위이다.
  - Namespaced 로 주면 커스텀 리소스는 네임스페이스 범위로 배포된다.
  - Cluster 로 주면 커스텀 리소스는 클러스터 범위로 배포된다.
  - CRD 자체는 Cluster 범위다. 여기서 말하는 scoped 값은 CR(여기선 Applicaton)의 범위를 말하는 것이다.
- spec.names 에는 이 리소스를 부르는 여러 이름을 담는다.
  - plural 의 값은 이 리소스의 복수 형태의 이름이다.
  - singular 의 값은 이 리소스의 단수 형태의 이름이다.
  - kind 의 값은 이 리소스를 생성할 때 작성하는 단수 형태의 이름이다.
  - shortNames 의 값은 이 리소스를 부르는 단축 형태의 이름으로, 일종의 Alias다.  

  <br/>

  이제 실제로 Application 은 어떤 스펙을 담고있어야 하는지 작성해보자.

 <br/>

 ```bash
# crd.yaml

apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # name must match the spec fields below, and be in the form: <plural>.<group>
  name: applications.shclub.github.io
spec:
  # group name to use for REST API: /apis/<group>/<version>
  group: shclub.github.io
  # either Namespaced or Cluster
  scope: Namespaced
  names:
    # plural name to be used in the URL: /apis/<group>/<version>/<plural>
    plural: applications
    # singular name to be used as an alias on the CLI and for display
    singular: application
    # kind is normally the CamelCased singular type. Your resource manifests use this.
    kind: Application
    # shortNames allow shorter string to match your resource on the CLI
    shortNames:
      - app
  # list of versions supported by this CustomResourceDefinition
  versions:
    - name: v1
      # Each version can be enabled/disabled by Served flag.
      served: true
      # One and only one version must be marked as the storage version.
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              x-kubernetes-preserve-unknown-fields: true
              properties:
                image:
                  type: string
                replicas:
                  type: integer
                port:
                  type: integer
              required:
                - image
                - replicas
                - port
            status:
              type: object
              x-kubernetes-preserve-unknown-fields: true
          required:
            - spec
 ```  

- spec.versions 내에 버전 단위로 스펙을 작성한다.
  - served 와 storage는 실제 운영에 제공할 버전에 대한 표시이며, 제공할 버전은 true 로 둔다.
  - schema 항목에 실제 스펙을 작성하며, 스펙 포맷은 openAPIV3Schema 를 사용한다.
    - 우리가 json 스키마를 정의할 때 흔히 쓰는 그 OpenAPI 스키마가 맞다.
- x-kubernetes-preserve-unknown-fields 를 true 로 주면 추후에 동적으로 해당 field 값을 수정할 수 있다. 왠만하면 true 로 주자. 

<br/>

여기서는 위에서 필요하다고 이야기한 설정 값에 맞게 image, replicas, port 이 3개의 속성 값을 spec 안에 설정하였다.

<br/>

### CRD 배포하기 

<br/>

이제 위에서 작성한 CRD를 다음처럼 배포하자.

```bash
root@edu25:~/operator# kubectl apply -f crd.yaml
customresourcedefinition.apiextensions.k8s.io/applications.shclub.github.io created
```

<br/>

아래처럼  조회도 되고 이렇게 CRD 자체는 클러스터 전역에서 배포된다.  

```bash
root@edu25:~/operator# kubectl get crd | grep shclub
applications.shclub.github.io
```

<br/>

### CR 작성하기

<br/>

이제 위에서 CRD로 정의한 CR을 작성해보자. application.yaml 을 만들고 다음처럼 작성한다.

```bash
# application.yaml

apiVersion: shclub.github.io/v1
kind: Application
metadata:
  name: nginx
spec:
  image: ghcr.io/shclub/nginx
  replicas: 1
  port: 80
```  

<br/>

위 crd.yaml 에서 작성한 spec 에 맞춰 application.yaml 을 작성하였다. 이제 우리는 이 application.yaml 을 배포하면 다음의 일들이 일어나길 기대한다.  

- Deployment 가 배포된다.  
- Container는 하나이며 image 는 ghcr.io/shclub/nginx 이다.  
- replicas 는 1 이다.  
- Service 가 배포된다.  
- Deployment 로 만들어진 Pod 들에 연결된다.  
- 포트는 80:80 으로 설정된다.  

<br/>

### CR 배포하기 

<br/>

위에서 작성한 application.yaml 을 배포하자.  

```bash
root@edu25:~/operator# kubectl apply -f application.yaml
application.shclub.github.io/nginx created
```  

<br/>

```bash
root@edu25:~/operator# kubectl get applications
NAME    AGE
nginx   51s
```  

<br/>

shortNames 으로 조회해 본다.  

```bash
root@edu25:~/operator# kubectl get app
NAME    AGE
nginx   51s
```  

<br/>  

CR을 배포했으나 아직 Deployment나 Service는 만들어지지 않았다. CR을 모니터링하며 어떤 추가적인 작업을 실행하려면 Operator를 만들어야 한다.  

<br/>

Operator는 위에서 설명한대로, 특정 리소스를 계속 모니터링하며 무언가의 작업을 수행하는 역할을 담당한다.    

예를 들어 어떤 CR이 생성되면 추가적인 특정 파드를 띄운다던가, CR에 변경이 일어나면 이러한 내용을 어딘가에 저장하게 할 수 있다. 이렇게 리소스를 모니터링하며 발생하는 이벤트에 따라 사용자가 추가적인 로직을 부여하고 싶을 때 Operator를 사용한다.


### Kopf 란?  

<br/>

Kopf는 Kubernetes OPerator Framework의 약자로, Python으로 Operator를 만들 수 있게 도와주는 프레임워크다.  

말 그대로 프레임워크다 보니, 많은 것들이 추상화 되어있고 우리는 주어진 프레임 안에서 적당히 잘 코딩을 하면 된다. 비슷한 도구로 CoreOS Operator SDK나 CoreOS Operator Framework들도 있지만, Kopf 개발자들은 Kopf가 이들보다 사용하기 훨씬 쉬운 프레임워크라고 말한다.  

<br/>

### Kopf 설치하기

<br/>

먼저 다음처럼 kopf 를 설치한다.

python 이 없으면 먼저 설치한다.  

```bash
root@edu25:~/operator# apt update
Get:1 https://download.docker.com/linux/ubuntu bionic InRelease [64.4 kB]
Hit:2 http://kr.archive.ubuntu.com/ubuntu bionic InRelease
Get:3 http://kr.archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]
Get:4 http://kr.archive.ubuntu.com/ubuntu bionic-backports InRelease [83.3 kB]
Get:5 http://kr.archive.ubuntu.com/ubuntu bionic-proposed InRelease [242 kB]
Get:6 http://kr.archive.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]
Fetched 567 kB in 3s (206 kB/s)
Reading package lists... Done
Building dependency tree
Reading state information... Done
59 packages can be upgraded. Run 'apt list --upgradable' to see them.
root@edu25:~/operator# apt install python3 -y
Reading package lists... Done
...
python3 is already the newest version (3.6.7-1~18.04).
0 upgraded, 0 newly installed, 0 to remove and 59 not upgraded.
root@edu25:~/operator# apt-get -y install python3.8-pip
root@edu25:~/operator# apt install python-pip
```  

<br/>

```bash
pip install kopf
```

```bash
kopf --version
kopf, version 1.35.5
```  

<br/>

### yaml 파일 작성하기  

<br/>

먼저 Deployment와 Service로 사용할 yaml 파일을 다음처럼 작성하자. service.yaml 을 만든 뒤 다음처럼 작성한다.  


```bash
# service.yaml

apiVersion: v1
kind: Service
metadata:
  name: "{name}"
spec:
  selector:
    app: "{name}"
  ports:
    - name: http
      protocol: TCP
      port: {port}
      targetPort: {port}
```  

<br/>

deployment.yaml 을 만든 뒤 다음처럼 작성한다.  

```bash
# deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: "{name}"
  labels:
    app: "{name}"
spec:
  replicas: {replicas}
  selector:
    matchLabels:
      app: "{name}"
  template:
    metadata:
      labels:
        app: "{name}"
    spec:
      containers:
      - name: "{name}"
        image: "{image}"
        ports:
        - containerPort: {port}
```  

yaml 파일 내 {} 로 표현된 변수들은 Python 코드에서 주입해줄 예정이다.

<br/>

### CR 생성 이벤트 핸들러 작성하기  

<br/>

이제 진짜로 Operator를 작성해보자. main.py 를 만든 뒤 다음처럼 작성한다.

```bash
# main.py

import kopf


@kopf.on.create('shclub.github.io', 'v1', 'applications')
def create(name, logger, **kwargs):
    logger.info(f"Application '{name}' is created.")
```  

<br/>

`applications.v1.shclub.github.io` 리소스가 생성될 때 이를 감지하여 create 함수가 실행되는 코드다. 즉 create 라고 하는 이벤트의 핸들러를 kopf 에서는 위처럼 작성하게 된다.

위 코드를 다음처럼 실행한다.    


```bash
kopf run main.py --verbose
```   

Output
```bash  
[2022-07-13 22:18:08,392] kopf._core.reactor.r [DEBUG   ] Starting Kopf 1.35.5.
[2022-07-13 22:18:08,392] kopf._core.engines.a [INFO    ] Initial authentication has been initiated.
[2022-07-13 22:18:08,392] kopf.activities.auth [DEBUG   ] Activity 'login_via_client' is invoked.
[2022-07-13 22:18:08,417] kopf.activities.auth [DEBUG   ] Client is configured via kubeconfig file.
[2022-07-13 22:18:08,418] kopf.activities.auth [INFO    ] Activity 'login_via_client' succeeded.
[2022-07-13 22:18:08,418] kopf._core.engines.a [INFO    ] Initial authentication has finished.
[2022-07-13 22:18:08,464] kopf._cogs.clients.w [DEBUG   ] Starting the watch-stream for customresourcedefinitions.v1.apiextensions.k8s.io cluster-wide.
[2022-07-13 22:18:08,465] kopf._cogs.clients.w [DEBUG   ] Starting the watch-stream for applications.v1.heumsi.github.io cluster-wide.
[2022-07-13 22:18:08,573] kopf.objects         [DEBUG   ] [default/nginx] Creation is in progress: {'apiVersion': 'heumsi.github.io/v1', 'kind': 'Application', 'metadata': {'annotations': {'kubectl.kubernetes.io/last-applied-configuration': '{"apiVersion":"heumsi.github.io/v1","kind":"Application","metadata":{"annotations":{},"name":"nginx","namespace":"default"},"spec":{"image":"nginx:latest","port":80,"replicas":1}}\n'}, 'creationTimestamp': '2022-07-13T13:18:02Z', 'generation': 1, 'managedFields': [{'apiVersion': 'heumsi.github.io/v1', 'fieldsType': 'FieldsV1', 'fieldsV1': {'f:metadata': {'f:annotations': {'.': {}, 'f:kubectl.kubernetes.io/last-applied-configuration': {}}}, 'f:spec': {'.': {}, 'f:image': {}, 'f:port': {}, 'f:replicas': {}}}, 'manager': 'kubectl-client-side-apply', 'operation': 'Update', 'time': '2022-07-13T13:18:02Z'}], 'name': 'nginx', 'namespace': 'default', 'resourceVersion': '79214', 'uid': 'f6f0f256-aaa8-4a42-83f2-3df27755ab06'}, 'spec': {'image': 'nginx:latest', 'port': 80, 'replicas': 1}}
[2022-07-13 22:18:08,574] kopf.objects         [DEBUG   ] [default/nginx] Handler 'create' is invoked.
[2022-07-13 22:18:08,574] kopf.objects         [INFO    ] [default/nginx] Application 'nginx' is created.
[2022-07-13 22:18:08,576] kopf.objects         [INFO    ] [default/nginx] Handler 'create' succeeded.
[2022-07-13 22:18:08,576] kopf.objects         [INFO    ] [default/nginx] Creation is processed: 1 succeeded; 0 failed.
[2022-07-13 22:18:08,577] kopf.objects         [DEBUG   ] [default/nginx] Patching with: {'metadata': {'annotations': {'kopf.zalando.org/last-handled-configuration': '{"spec":{"image":"nginx:latest","port":80,"replicas":1}}\n'}}}
[2022-07-13 22:18:08,690] kopf.objects         [DEBUG   ] [default/nginx] Something has changed, but we are not interested (the essence is the same).
[2022-07-13 22:18:08,690] kopf.objects         [DEBUG   ] [default/nginx] Handling cycle is finished, waiting for new changes.
```    


로그를 보면 핸들러에서 applications.shclub.github.io 리소스가 생성된 것을 감지하고 있다.  

또한 생성된 CR의 Events에도 위 Operator가 작업한 이벤트가 기록된다.  

<br/>
그러면 이제 main.py 내 create 함수 내에 Deployment와 Service를 배포하는 코드를 작성해보자.

코드에서 Kubernetes API를 Python SDK 형태로 쓰기 위해 다음 패키지를 설치한다.

```bash
pip install kubernetes
pip list | grep kubernetes
```    


```bash
kubernetes         24.2.0
```  

<br/>

main.py 를 다음처럼 수정한다.  

```bash
import os

import kopf

import kubernetes
import yaml


@kopf.on.create('shclub.github.io', 'v1', 'applications')
def create(name, spec, namespace, logger, **kwargs):
    logger.info(f"Application '{name}' is created.")
    kapps_v1 = kubernetes.client.AppsV1Api()
    kcore_v1 = kubernetes.client.CoreV1Api()

    # Create Deployment
    path = os.path.join(os.path.dirname(__file__), 'deployment.yaml')
    tmpl = open(path, 'rt').read()
    text = tmpl.format(name=name, **spec)
    data = yaml.safe_load(text)
    kopf.adopt(data)  # Add as child
    deployment = kapps_v1.create_namespaced_deployment(
        namespace=namespace,
        body=data
    )
    logger.info(f"Deployment '{name}' is created.")
    logger.debug(deployment)

    # Create Service
    path = os.path.join(os.path.dirname(__file__), 'service.yaml')
    tmpl = open(path, 'rt').read()
    text = tmpl.format(name=name, **spec)
    data = yaml.safe_load(text)
    kopf.adopt(data)  # Add as child
    service = kcore_v1.create_namespaced_service(
        namespace=namespace,
        body=data
    )
    logger.info(f"Service '{name}' is created.")
    logger.debug(service)

    return {
        "deployment": deployment.metadata.name,
        "service": service.metadata.name,
    }
```  

<br/>

별 다른 내용은 없고, kubernetes 패키지로 Deployment와 Service를 생성하는 코드다. tmpl.format(name=name, **spec) 코드를 통해 이전에 작성한 deployment.yaml 와 service.yaml 에 name 및 spec 의 데이터를 주입하여 실제로 배포할 yaml 파일을 렌더링한다. 이후 kubernetes.client 를 통해 이 yaml 파일을 배포한다.  

 <br/>

 > kubernetes.client 는 리소스의 apiVersion 에 따라 나뉜다. 예를 들어 apiVersion 이 v1 인 리소스는 kubernetes.client.CoreV1Api 를 사용하면 되고, apiVersion 이 apps/v1 인 리소스는 kubernetes.client.AppsV1Api 를 사용하면 된다.

 <br/>

그리고 kopf.adopt(data) 코드를 넣어주게 되면, 배포되는 yaml 파일은 이 CR의 자식 리소스가 된다. 즉, 이 CR이 지워질 때 자식 리소스(여기서는 Deployment와 Service)가 같이 지워지게 된다.  

마지막에는 Dict 를 반환하는데, Key, Value들은 CR의 status 항목에 핸들러 함수 이름과 함께 저장된다. 이는 아래에서 CR 배포 후 확인해보자.  

<br/>

이제 Operator도 다시 실행하고, CR도 지웠다가 다시 생성해보자.  

```bash
kopf run main.py --verbose
kubectl delete -f application.yaml
        
application.heumsi.github.io "nginx" deleted
```  

<br/>

```bash  
kubectl apply -f application.yaml

application.shclub.github.io/nginx created
```  

이제 정상적으로 CR(Application)와 Deployment, Service가 잘 배포되었는지 확인해보자.

<br/>

```bash  
kubectl get app
```  

```bash  
NAME    AGE
nginx   20s
```  

<br/>

```bash  
kubectl get deployment
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           24s
```  

```bash  
kubectl get service                   
```  

```bash  
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   3d8h
nginx        ClusterIP   10.96.249.187   <none>        80/TCP    27s
```  

잘 배포된 것을 확인했다.

배포된 Deployment를 확인해보면 다음처럼 ownerReferences 에 CR이 설정된 것을 확인할 수 있다. 이는 Deployment의 상위 리소스가 CR이라는 표현으로, kopf.adopt(data) 코드에 의해 생성된 것이다.  

<br/>

```bash  
kubectl get deploy nginx -o yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  ...
  ownerReferences:
  - apiVersion: heumsi.github.io/v1
    blockOwnerDeletion: true
    controller: true
    kind: Application
    name: nginx
    uid: 963a4fd2-effe-4c5e-bdcf-c089a3ea9480
  ...
spec:
  ...
```  

<br/>

## java 로 Operator 만들기

<br/>

```bash
root@edu25:~/operator/java-operator-sample# kubectl apply -f crd.yaml
customresourcedefinition.apiextensions.k8s.io/elevenstapps.11st.example.com created
root@edu25:~/operator/java-operator-sample# kubectl api-resources --api-group=11st.example.com
NAME           SHORTNAMES   APIVERSION            NAMESPACED   KIND
elevenstapps                11st.example.com/v1   true         ElevenStApp

root@edu25:~/operator/java-operator-sample# kubectl apply -f cr.yaml
elevenstapp.11st.example.com/elevenst-service created
```