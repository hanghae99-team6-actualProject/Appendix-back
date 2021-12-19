[Readme 돌아가기](https://github.com/hanghae99-team6-actualProject/backend) 

## Redis

### 1. 개요
- 채팅기능 구현으로 인해 다수의 반복적인 처리가 필요한 작업이 생성
- 프로젝트에서 DB로 사용중인 mySQL에 데이터를 저장
- 보다 나은 성능을 제공할 수 있는 대안에 대한 모색
- Redis를 통한 성능 향상 기대
- Artillery를 이용한 부하테스트를 통해 검증
- 결과

### 2. 상세

- 채팅기능 구현으로 인해 다수의 반복적인 처리가 필요한 작업이 생성<br/>
  - Socket.io를 통해 실시간으로 유저들이 채팅을 주고 받는 기능을 구현.<br/>
    그리고 채팅을 통한 의사소통을 위해 채팅 내역을 지속해야할 필요를 느낌.<br/>
    또한 채팅의 특징으로 간단한 처리의 반복이 요구됨을 확인.

    <img width="790" alt="스크린샷 2021-12-20 오전 1 51 35" src="https://user-images.githubusercontent.com/59440469/146683571-f8c61160-d069-4668-849a-59b5309ee056.png"><br/><br/>
  
- 프로젝트에서 DB로 사용중인 mySQL에 데이터를 저장<br/>
  - 기능구현을 위해 프로젝트에서 DB로 사용중인 mySQL에 해당 데이터를 저장하기로 결정.

    <img width="30%" alt="스크린샷 2021-12-20 오전 2 02 34" src="https://user-images.githubusercontent.com/59440469/146683852-ffa2a4e5-8e86-42f5-8ab7-52defb8ff609.png"><br/><br/>

- 보다 나은 성능을 제공할 수 있는 대안에 대한 모색<br/>
  - 구글링을 통한 채팅 기능에 최적화된 DB를 조사.<br/>
    많은 경우 Redis를 통해 채팅 기능을 구현한다는 것을 확인.<br/><br/>

- Redis를 통한 성능 향상 기대<br/>
  - Redis를 통해 채팅 기능의 저장 및 불러오기 기능을 구현하기로 의사결정.<br/>
    Redis의 특성에 맞게 빠른 작업 처리를 기대.<br/>

    <img width="43%" alt="스크린샷 2021-12-20 오전 2 22 38" src="https://user-images.githubusercontent.com/59440469/146684574-635b4ccd-e960-4769-92e9-c07a2d77ca1a.png"><br/>
    <img width="70%" alt="스크린샷 2021-12-20 오전 2 23 51" src="https://user-images.githubusercontent.com/59440469/146684577-645f9602-4c9f-4b21-b0c8-c78b5b548149.png"><br/>
    <img width="85%" alt="스크린샷 2021-12-20 오전 2 24 36" src="https://user-images.githubusercontent.com/59440469/146684578-ed0438cc-87fe-44d1-aba3-6af56e9ffd49.png"><br/><br/>

- Artillery를 이용한 부하테스트를 통해 검증
  - mySQL과 Redis에 동일한 조건으로 테스트를 진행하여 얻은 결과를 통해 DB 변경시의 성능 확인.<br/><br/>
    동일한 시나리오 및 시간 설정<br/>
    <img width="50%" alt="스크린샷 2021-12-03 오전 5 46 29" src="https://user-images.githubusercontent.com/59440469/144500713-d0e2b321-8abf-4317-b5b0-917087c3f5bb.png"><br/>

    mySQL의 성능 테스트 결과<br/>
    <img width="50%" alt="스크린샷 2021-12-03 오전 5 48 22" src="https://user-images.githubusercontent.com/59440469/144501209-9ae555aa-8c8d-4056-adaf-cd6587f296e9.png"><br/>
    <img width="90%" alt="스크린샷 2021-12-03 오전 5 48 37" src="https://user-images.githubusercontent.com/59440469/144501306-8dcfcd5c-08d3-414d-b8ef-6f44b81ef086.png"><br/><br/>

    Redis의 성능 테스트 결과<br/>
    <img width="50%" alt="스크린샷 2021-12-03 오전 5 47 29" src="https://user-images.githubusercontent.com/59440469/144501427-b3624b91-4bf8-4f0b-9b35-015179ea3726.png"><br/>
    <img width="90%" alt="스크린샷 2021-12-03 오전 5 47 49" src="https://user-images.githubusercontent.com/59440469/144501431-6217c1d5-afea-4fe6-99df-5677af9d0399.png"><br/><br/>

- 결과
  - Artillery 테스트 결과로 Redis의 Latancy가 mySQL에 비해 약 1/4 가량으로 측정됨.<br/>
    이는 Reids를 프로젝트의 장기적인 측면에서 보다 안정적인 DB로 구성할 수 있는 이유가 됨.<br/>

<br/>
<br/>
백앤드 API는 Feature > redis-test 브랜치에 구현완료 상태.<br/>
서비스 적용 예정.