[Readme 돌아가기](https://github.com/hanghae99-team6-actualProject/backend)  

# RDS & ElastiCache

## 잦은 서버 다운
기존 Jenkins와 MYSQL이 EC2 서버에 전부 들어가 있었기 때문에 잦은 서버 다운이 발생하였으며 그로 인해 서버 다운을 해결할 때마다 전체 작업이 중단되는 문제가 생겼다.

## AWS RDS

<img src="https://t1.daumcdn.net/cfile/tistory/9961AB415C812AE938" alt="img" width="200" height="auto"/>  


Jenkins를 새로운 EC2 서버로 옮긴 이후 MYSQL도 새로운 서버로 옮기던 중 AWS RDS 서비스를 알게 되었다.  
RDS는 DB 전용서버로써 기본적으로 다양한 기능을 가지고 있는데  

1. DB 자동 설치 및 유지관리
2. 쉽고 간단한 스케일업
3. 백업 및 데이터베이스 스냅샷 자동화
4. 데이터베이스 최적화된 성능
5. 보안 서비스
6. 모니터링 서비스

이처럼 다양한 서비스를 제공받으며 비용도 거의 없었기 때문에 RDS를 도입하게 되었다.

## RDS 프록시
서버가 다운되는 일이 다시는 없도록 신경쓰는 일이 백엔드 최종 과제라고 생각하였고 이전 서버 다운의 원인이 당연하게도 트래픽의 급증이기 때문에 그 처리를 해 줄 방법으로 RDS 프록시를 사용하였다.  
RDS 프록시를 사용함으로써 DB연결 숫자를 제어하였고 그로써 과도한 트래픽의 급증을 처리할 수 있었다. 

## AWS ElastiCache

<img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTbJAZefP0uFhEfI5eM7j4ISqzuxblMK8m_UETPR6zi88oulhi8H3bcrroiaOOPE0kuMag&usqp=CAU" alt="img" width="200" height="auto"/>  

Redis 서버 또한 동일한 처리를 해 주었는데 Redis는 관계형 데이터베이스가 아닌 NoSQL이었기에 AWS ElastiCache를 사용하였다.  
RDS와 비슷한 장점이 있으며 서버가 항상 돌아가는 것을 유지하기 위해 복제 그룹을 생성하여 마스터-슬레이브 형식으로 마스터의 장애 발생 시 슬레이브(Read Only)를 마스터로 승격하는 방식을 취하고 있다.  
우선적으로 채팅 데이터 저장에 사용하고 순차적으로 Refresh Token, 캐싱 용도로 사용할 예정이었기에 선적용 후에 비용을 확인하여 최종 도입 결정을 하기로 하였다.  


