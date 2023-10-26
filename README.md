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

<br/>

> 과제 :  [과제 문서보기](./homework.md)   

<br/>

> 답안 :  [과제 답안보기](https://github.com/shclub/edu_homework/blob/master/homework.md)

<br/>

## 1주차


<br/>


0. Chapter 0 : 프로그램 설치  ( [가이드 문서보기](https://github.com/shclub/edu/blob/master/okd4_install.md) )  

     - OKD , ArgoCD , Elastic Stack , Opensearch , Minio 설치


     - 환경 정보  ( [가이드 문서보기](./environment.md) )  

    <br/>

1. Chapter 1 : 1시간  ( [가이드 문서보기](./chapter1.md) )  

     - kubernetes Summary
       - 환경 점검 : okd console 
       - k8s 아키텍처 및 리소스 
       - Observability

     <br/>

2. Chapter 2 : 2시간  ( [가이드 문서보기](./chapter2.md) )  

     - k8s metric 수집 구조 ( k8s api vs metric server )
     - Prometheus 설명 및 실습
       - Prometheus 구조 , OKD 에서의 Prometheus
       - Grafana 사용 방법
       - Metric 수집 실습 : Node Exporter ( Ubuntu VM )
     - Federation & Thanos  
          
     <br/>
    
3. Chapter 3 : 2시간  ( [가이드 문서보기](./chapter3.md) )  

     - Opensearch 소개
     - OpenTelemetry 설명
     - Otel 를 통한 데이터 수집 및 모니터링 실습

     <br/>

4. Chapter 4 : 3시간  ( [가이드 문서보기](./chapter4.md) )  

     - Elastic Stack 소개
     - Elastic 를 통합 데이터 수집 설명 ( Metric )
     - Kibana Dev Tool 실습
       - Index 생성 및 조회, snapshot 생성하기 ( S3 )
     - 로그 수집 실습 및 Dashboard 만들기
     
     <br/>

     - 과제   
          - NGINX 로그 수집 해보기

<br/>


## 2주차


<br/>

3. Chapter 5 : 2시간   ( [가이드 문서보기](./chapter5.md) )  

