# 본 과정에 대해  ( Cloud Native 심화 과정 )
 
본 교육 과정은 Cloud Native 심화 과정으로 이론 및 실습을 수행한다. ( 총 24 시간 오프라인 교육 )
 

문의 :  이석환 ( seokhwan.lee@kt.com / shclub@gmail.com )
<br/>

2022년 1등 학습체 자료  : https://github.com/shclub/edu
- 각 솔루션 설치 가이드 (k3s , jenkins , ArgoCD 등 ) 
- Cloud Native 교육
- SpringBoot 초급  

<br/>

2022년 Cloud Native 전사 교육 자료  : https://github.com/shclub/newedu


<br/>

2023년 B2B 역량 강화 Cloud Native 교육
-  https://github.com/shclub/edu2023

<br/>

> 과제 :  [과제 문서보기](https://github.com/shclub/edu2023/blob/master/homework.md)   

<br/>

> 답안 :  [과제 답안보기](https://github.com/shclub/edu_homework/blob/master/homework.md)

<br/>

## 1주차


<br/>


0. Chapter 0 : 프로그램 설치  ( [가이드 문서보기](https://github.com/shclub/edu/blob/master/okd4_install.md) )  

     - OKD , Grafana , ArgoCD , Elastic Stack , Opensearch , Minio 설치


     - 환경 정보  ( [가이드 문서보기](./environment.md) )    
     - 실습 yaml 화일 ([manifest 보기](./manifest) )    

    <br/>

1. Chapter 1 : 1시간  ( [가이드 문서보기](./chapter1.md) )    

     - kubernetes Summary  
       - 환경 점검 : okd console   
       - k8s & OKD 구조   
       - k8s 주요 기능  
       - Observability  

     <br/>

2. Chapter 2 : 2시간  ( [가이드 문서보기](./chapter2.md) )  


     - metric 수집 방식
     - k8s metric 수집 구조
     - Prometheus 소개 / Grafana 사용 방법 
     - Metric 수집 실습 (과제) : Node Exporter ( 개인 Ubuntu VM )
     - Application Metric 수집 실습 :  Frontend / Backend
       - Grafana 로 Quarkus Application 수집 해보기  
     - Federation & Thanos 
       - Thanos Query 실습 
     - prometheus 내부 구조

     <br/>
    
3. Chapter 3 : 2시간  ( [가이드 문서보기](./chapter3.md) )   

     - Opensearch 소개
     - Opensearch 설정
       - data prepper 설치
       - collector 설치
     - OpenTelemetry 소개
     - OpenTelemetry 설정
       - cert-manager 설치
       - otel operator 설치
     - Otel 를 통한 데이터 수집 및 모니터링 실습

     <br/>

4. Chapter 4 : 3시간  ( [가이드 문서보기](./chapter4.md) )  

     - Elastic Stack 소개
     - Elastic 기본 사용법 및 Kibana Dev Tool 실습
     - Elastic 를 통합 데이터 수집 ( /w kubernetes integration )
       - K8S API 통합 수집
       - kubelet 를 통한 수집 ( Daemonset )
     - Application metric 수집 ( /w prometheus integration )
       - micrometer 를 활용한 metric 수집
     - Application log 보기 
       - container log 보기
     - Application trace 수집 ( /w APM )
       - SpringBoot 서비스 / trace 보기
     - Dashboard 만들기 ( Import/Export ) 
       - 외부 Dashboad Import 
     - snapshot 설정  
       - S3 에 저장 하고 복구 하기 
     - 실습   
       - Elastic Cloud 에 클러스터 구축 metric 수집  
     - Trouble Shooting

     <br/>

     - 과제   
       - Elastic Cloud 에 클러스터 구축하고 metric 수집
     

<br/>


## 2주차


<br/>

5. Chapter 5 : 2시간   ( [가이드 문서보기](./chapter5.md) )  

    - CNI  
    - Cilium  
    - iptables  
    - CoreDNS  
    - AWS EKS networking
    - Network Policy    
    - Headless 서비스    

6. Chapter 6 : 4시간   ( [가이드 문서보기](./chapter6.md) )    

    - 보안 component
    - K8S Security Overview  
    - Route 에 SSL 인증서 설정     
    - User Account vs Service Account  
    - Krew 설명 및 설치  
    - Kubernetes 보안 Components  
    - Security Context
    - Pod Security Policy ( PSP )  

7. Chapter 7 : 2시간   ( [가이드 문서보기](./chapter7.md) )  

    - Operater 
      - Operator vs Helm
      - CRD 
    - 실습
      - CRD 생성 하기  ( By python , Java )


<br/>

## 3주차

<br/>

8. Chapter 8 : 2시간   ( [가이드 문서보기](./chapter8.md) )  

     - 미정 
       - 미정
     - 실습


