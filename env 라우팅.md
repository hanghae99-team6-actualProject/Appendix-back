[Readme 돌아가기](https://github.com/hanghae99-team6-actualProject/backend)  

# env 라우팅
최초에 dotenv 모듈로 환경변수를 관리하였다.  
그러나 점차 환경변수가 늘어가고, 로컬 환경과 배포 환경에서 차이점이 커지게 되었다. 또한 RDS와 ElastiCache 등 서버 endpoint 자체의 개수도 늘어나게 되어 환경변수를 관리해주는 시스템의 필요성을 느꼈다. 
따라서 현재 환경에 맞는 환경변수를 자동으로 세팅하도록 구현하였다.

```js
require('dotenv').config();

const nodeenv = process.env.NODE_ENV === '';
env = {
  NODE_ENV: process.env.NODE_ENV,

  EXPRESS_PORT: process.env.EXPRESS_PORT,development

  DB_USER: nodeenv ? process.env.DEV_DB_USER : process.env.PRODUCT_DB_USER,
  DB_PASSWORD: nodeenv ? process.env.DEV_DB_PASSWORD : process.env.PRODUCT_DB_PASSWORD,
  DB_NAME: process.env.DB_NAME,
  DB_END_POINT: nodeenv ? process.env.DEV_DB_END_POINT : process.env.PRODUCT_DB_END_POINT,

  DOMAIN: nodeenv ? process.env.LOCALHOST_END_POINT : process.env.PRODUCT_END_POINT,

  AWS_S3_KEY: process.env.AWS_S3_KEY,
  AWS_S3_PRIVATE_KEY: process.env.AWS_S3_PRIVATE_KEY,

  KAKAO_CLIENT_ID: process.env.KAKAO_CLIENT_ID,
  GOOGLE_CLIENT_ID: process.env.GOOGLE_CLIENT_ID,
  GOOGLE_CLIENT_SECRET: process.env.GOOGLE_CLIENT_SECRET,
  NAVER_CLIENT_ID: process.env.NAVER_CLIENT_ID,
  NAVER_CLIENT_SECRET: process.env.NAVER_CLIENT_SECRET,

  JWT_SECRET_KEY: process.env.JWT_SECRET_KEY,
};
module.exports = env;
```
