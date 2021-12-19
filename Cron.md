[Readme 돌아가기](https://github.com/hanghae99-team6-actualProject/backend)  

- [Cron](https://github.com/hanghae99-team6-actualProject/Appendix-back/blob/main/Cron.md)  
<img width="386" alt="스크린샷 2021-12-03 오전 8 06 15" src="https://user-images.githubusercontent.com/54808299/144517227-b8b23019-4669-4ea5-8c26-425c25fe2eea.png">

### 문제 인식

- 유저 데이터와 연결된 데이터의 범위가 넓고 서비스 특성 상 유저 개인의 활동 기록이 중요하므로 보수적으로 데이터를 처리할 필요성을 느꼈다.

### 구현

- User 테이블에 paranoid 속성을 적송하여 회원 탈퇴 시 soft-delete 하고 node-cron을 활용하여 한 달이상 soft-delete된 유저를 hard-delete 하도록 스케줄링 처리하였다.
    - 이를 통해 일시적인 변심 또는 실수로 탈퇴한 경우 데이터를 원복할 수 있도록 개선되었다.
