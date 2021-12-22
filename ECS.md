[Readme 돌아가기](https://github.com/hanghae99-team6-actualProject/backend)  

## 스케일업과 로드밸런싱의 필요성
젠킨스 CI/CD를 적용할 때 잦은 서버 다운으로 인해 서버가 다운되지 않게 하는 로드밸런싱과 스케일업의 필요성을 느꼈다.
비록 단일 서버로 돌아가던 젠킨스는 메모리 사용량으로 인해 AWS CodePipeline으로 교체하였고, MYSQL은 RDS 서버로 옮겨 일부는 마이크로서비스를 적용하였지만, 서비스 상용화 이후를 대비하기 위한 준비는 아무리 해도 부족하다는 생각으로 최대한 가용성을 확보하고 상용화 이후 조정하는 방식으로 결정하였다.

## nginx 도입
<img src="https://user-images.githubusercontent.com/42149645/146671943-292338e7-f11a-440d-a333-3cbc0c94b788.png" alt="nginx" width="200" height="auto"/>  

첫 로드밸런서로써 nginx를 선택하였다.  
우선 http를 https로 전환하는 것부터 마무리하고 이후 L7 포트 단계에서 부하분산을 할 예정이었고, https로 전환하는 것은 잘 마무리되었으나 이후 서버를 추가하여 로드밸런싱을 하는 단계에서 비용 문제로 도커 도입을 고민할 수 있었다.  
### 예상 아키텍쳐
<img src="https://user-images.githubusercontent.com/42149645/146909400-80c8bfac-ffc3-471b-a0dd-8246e815f6e0.png" alt="nginx_" width="800" height="auto"/>


## docker 도입
<img src="https://user-images.githubusercontent.com/42149645/146669653-61ad3773-de33-48f7-88a2-4d46dc6311ab.png" alt="docker" width="200" height="auto"/>  

우선 로드밸런싱을 수월하게 하기 위해서 동일한 기능을 하는 도커 서버를 여러 대 띄우는 것이 필요한 것을 확인하고 도커를 사용하게 되었다.  
dockerfile을 이용한 이미지화까지 성공하였지만 CI/CD에서 도커 이미지를 사용하는 방법을 몰라 해메게 되었다.  
### 예상 아키텍쳐
<img src="https://user-images.githubusercontent.com/42149645/146909392-c7ab06bb-83d7-461f-82a7-d6fc8c89d7ad.png" alt="docker_" width="800" height="auto"/>   


## AWS ECS 도입
<img src="https://user-images.githubusercontent.com/42149645/146671979-9d5de05e-b939-433e-93ad-3a05cfb464a8.png" alt="ecs" width="150" height="auto"/>   


애초 계획이었던 로드밸런싱과 오토스케일링, docker, CodePipeline CI/CD 연동까지 컨테이너 오케스트레이션을 통하여 관리하는 것이 편하다는 것을 알게 되어 컨테이너 오케스트레이션 툴을 고민하였다.   
우선적으로 쿠버네티스와 도커스웜, AWS ECS 중에서 선택할 예정이었고 간편한 오토스케일링 지원, 로그와 트래픽 시각화 지원, 그리고 최종적으로 모든 인프라를 AWS로 통일하는 편의성으로 인해 최종적으로 AWS ECS를 선택하게 되었다.  

## Elastic Load Balancing
AWS Elastic Load Balancer를 통해 로드 밸런싱을 시도하였다.  
HTTPS를 사용할 수 있도록 Application Load Balancer를 사용하였고 이 과정에서 nginx의 역할이 무의미해졌다고 생각하였기에 nginx를 제거하였다.  
부하 분산은 AWS ALB에서는 자동으로 교차 영역 로드밸런싱이 적용되었기 때문에 전체 클러스터에 자동으로 균일하게 부하를 분산시킬 수 있었다.  
추가로 무중단 배포를 위해 로드밸런싱을 도입하면서 필수적으로 블루/그린 배포를 목표로 두었는데 ECS와 ELB를 통해 블루/그린 배포 또한 성공하게 되었다.  
이전까지 100%가 넘어 터지기도 하였던 서버 환경에서 ECS와 로드밸런싱을 도입한 후 전체 ECS에서 20%의 메모리 사용량과 10%이하의 CPU 사용량을 확인할 수 있었다.   

## Auto Scaling
AWS ECS에서 제공하는 오토 스케일링 기능도 사용하였다.  
오토 스케일링은 ECS 서비스 생성 단계에서 매우 간편하게 설정 가능하였으며 사용시 최소, 일반, 최대 사용 작업을 선택 가능하여 1~3개로 설정하였다.  
자동 작업 조정 정책을 Cloud Watch와 연결하여 경보 시스템을 만들어 놓았고 Artillery를 사용하여 실제 경보 발생 부하테스트도 성공하였다.  
### 경보 발생 테스트  
<img src="https://user-images.githubusercontent.com/42149645/146909387-cdbfbb72-8f74-4724-84fb-2bdae28abcee.png" alt="cpu,메모리" width="800" height="auto"/>   

<img src="https://user-images.githubusercontent.com/42149645/146911115-ed5562dc-cb28-4e00-9954-e1cba8b24fb5.png" alt="경보" width="800" height="auto"/>   

<img src="https://user-images.githubusercontent.com/42149645/146911112-fde1dae2-3956-4eb9-bf81-0878e739fa7f.png" alt="autoscaling" width="800" height="auto"/>   

## 최종 아키텍쳐
<img src="https://user-images.githubusercontent.com/42149645/146909394-5ba8606e-9a87-4ed2-9118-3ef7e5df142c.png" alt="ecs_" width="800" height="auto"/>   

