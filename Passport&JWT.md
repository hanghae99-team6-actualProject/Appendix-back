[Readme 돌아가기](https://github.com/hanghae99-team6-actualProject/backend)  

# Passport & JWT 토큰을 활용한 로그인 방식

## Cookie & Session vs JWT
| | Cookie & Session | JWT|
| --- | --- | --- |
| 장점 | 서버에서 Session을 통제 가능하므로 보안적으로 조금 더 우수| 인증을 위한 저장소가 필요 없으므로 서버쪽 부담이 적다 </br> 별도의 I/O 작업이 없어 인증 속도가 빠르다 </br> 확장성이 좋으며 모바일 환경에서도 사용 가능 |
| 단점 | 세션 저장소가 필요하기 때문에 서버에 부하 발생 | 특정 토큰을 강제로 만료시키기가 어려워져 보안 문제 |

세션 방식과 JWT 방식 중에서 선택하였다.  
passport의 기반 세팅이 session을 활용하는 방식이란 것을 이미 알고 있었기 때문에 session 방식을 먼저 생각하였으나 잦은 서버 과부하로 인해 서버에 최대한 부담을 주지 않는 방식으로 프로젝트를 진행하고 싶었으며 웹앱 환경을 만들 예정이었기에 jwt 토큰을 사용자에게 주고 인증하는 방식으로 선택하였다.  

## JWT의 보안 이슈
JWT토큰을 사용자의 로컬 스토리지에 보관하는 방식으로 자동 로그인을 제작하였다.   
하지만 세션 스토리지에 비해 길게 사용자에게 남아있기 때문에 탈취로 인한 보안 이슈가 예상되었다.  

## Access Token, Refresh Token
Access Token과 Refresh Token을 생성하여 사용자에게 보관하는 방식으로 생성하였다.   

```js
    // refresh token 발급 (2주)
    const refreshToken = jwt.sign({ providerId: user.providerId }, env.JWT_SECRET_KEY, {
      expiresIn: '14d',
      issuer: 'mingijuk',
    });

    // access token 발급 (24시간)
    const accessToken = jwt.sign({ providerId: user.providerId }, env.JWT_SECRET_KEY, {
      expiresIn: '1h',
      issuer: 'mingijuk',
    });
```
accessToken은 짧은 만료시간을 주어 탈취 피해를 최소화하였고 refreshToken을 통해 사용자 편의도 고려하였다.
또한 미들웨어를 통해 accessToken이 있지만 refreshToken이 없는 경우, refreshToken이 있지만 accessToken이 없는 경우 등 분기처리를 따로 하여 토큰을 재발급하였다.  
refreshToken은 검증을 위해 User DB에 저장하였다.  

## Passport
타겟층이 국내 사용자였으므로 kakao, google, naver 소셜 로그인을 사용하기 위해 Passport를 사용하였다.  

## 세션 방식 JWT로 수정
Passport의 샘플이 기본적으로 세션 스토리지를 지원하는 방식이기 때문에 JWT 형식으로 바꾸기 위해 세션을 비활성화하고 JWT 토큰을 발급하는 방법으로 수정하였다.   
하지만 이전까지 email을 고유값으로 사용하였기 때문에 소셜 로그인의 이메일이 동일한 경우를 처리할 수가 없었다.  
따라서 Passport에서 제공하는 providerId를 고유 아이디 값으로 사용하기 위하여 로컬 로그인의 email과 추가 값을 변경하는 형식으로 providerId로 지정하고 JWT 토큰을 providerId값으로 만들어서 해결하였다.    

## CORS 에러
passport를 적용한 이후 잦은 CORS 에러 문제가 발생하였다.  
보안을 위해 추가한 Helmet의 설정을 수정하고, cors 모듈도 추가해 주었지만 여전히 문제가 발생하였다.  
프론트엔드에서 발생하는 문제로 확인되었으며 passport의 소셜 로그인 요청 자체의 문제점이 있었다.  
프론트엔드에서의 요청을 a href로 보내고, 백엔드에서는 프론트의 빈 페이지를 redirect하는 것으로 토큰을 제공하였다. 프론트엔드에서는 redirect된 페이지에서 토큰을 받아와 다시 메인 페이지로 보내는 것으로 소셜 로그인 처리를 마무리하였다.  

