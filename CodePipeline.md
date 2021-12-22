[Readme 돌아가기](https://github.com/hanghae99-team6-actualProject/backend)  

# CodePipeline

## CI/CD
프로젝트가 정상 궤도로 들어서고 CI/CD에 대한 고민을 하게 되었다.  
가장 먼저 알아본 결과 젠킨스, Circle CI, Travis CI, Github Actions 등으로 좁혔으며, 비용도 없고, 현업에서 가장 오래 사용하였다는 젠킨스를 최종 CI/CD 툴로 결정하게 되었다.  

## 잦은 서버 다운
젠킨스 적용 이후 잦은 서버 다운이 있었다.  
이유는 서버 환경이 EC2 프리티어 단일 서버로 젠킨스와 DB까지 동일 서버에서 사용 중이었기 때문이었다.   
엔드포인트로 도메인 대신 퍼블릭 DNS를 사용했었기에 서버 다운을 해결할 때마다 EC2 서버의 주소가 바뀌어 불편함을 크게 느꼈었고 서버 다운 해결중에는 프론트의 작업이 중단된다는 큰 문제가 발생했다.  
이때 처음으로 젠킨스가 생각보다 메모리를 많이 잡아먹는다는 것을 알았고 EC2 프리티어 서버도 생각 이상으로 작다는 것을 알게 되었으며 마이크로서비스의 필요성에 대해 느꼈다.  

## Swap Memory
Swap Memory를 사용하여 하드디스크의 디스크를 메모리로 사용할 수 있게 설정하였다.  
하지만 이후 도커를 적용하여 로드밸런싱을 실행하기로 결정하였었는데 결국 도커가 실행되는 물리적 서버는 하나이기에 작은 ec2 프리티어 서버로 다수의 도커 이미지 실행시에도 버틸 수 있을지 의문이 들었고, 또한 하드디스크를 메모리로써 사용하므로 속도가 느려 사용자 경험 측면에서 좋지 않다고 판단하였다.

## 젠킨스 전용 EC2서버
젠킨스 전용 EC2 서버를 따로 생성하여 CI/CD 기능만 따로 그 서버에서 이용하는 방식을 시도했다.  
당시 로드밸런싱에 대한 이해도가 적었기에 남는 서버의 가용 공간을 사용할 방법을 찾지 못하여 비용 낭비가 크다고 판단하였다.

## Travis CI 

<img src="https://user-images.githubusercontent.com/42149645/146668486-52843d32-0d5c-46e2-8c44-b267b7b75151.jpg" alt="travis-ci" width="200" height="auto"/>  

Travis CI를 사용하여 CI/CD를 구축하였다. 하지만 개인용 레포지토리 이상의 팀 프로젝트에서는 비용이 꽤 발생함을 확인 후 변경하였다.  

## Github Actions

<img src="https://user-images.githubusercontent.com/42149645/146668561-a36bac5b-a9c9-404b-8562-850466e26c99.png" alt="img" width="200" height="auto"/>  

이후 Github Actions를 사용하였다. 그러나 젠킨스를 실사용하다가 사용하게 되니 서비스가 약간 부족해보이는 느낌도 들었으며 파이프라인의 Build 단계에서 S3를 사용하는 것을 깨닫고 아예 아키텍쳐를 AWS로 통일하자는 생각이 들게 되었다.

## AWS CodePipeline

<img src="https://user-images.githubusercontent.com/42149645/146668655-1a1d592d-2c12-4c0f-8050-8a69c24dc5d2.png" alt="img" width="200" height="auto"/>  


최종 CI/CD 툴로써 AWS CodePipeline을 사용하게 되었다.  
우선 활성 파이프라인 1개는 무료로 제공되기 때문에 비용 측면에서 낫고 AWS이기 때문에 서비스도 나쁘지 않을 것이라고 판단하였다.   
실제로 AWS CodePipeline은 CodeBuild에서의 비용을 제외하고 무료로 사용할 수 있었으며 이후의 아키텍쳐간의 연결과 Cloud Watch를 통한 로그 확인 등에서 젠킨스만큼의 훌륭한 서비스도 제공해주어 간편한 사용이 가능하였다.

## AWS ECS적용 CI/CD설정
이전 EC2 단일 서버를 사용할 때 잦은 서버 다운으로 인해 컨테이너 오케스트레이션을 도입하였다. 그로 인해 CI/CD의 설정 파일도 EC2 단일 서버 배포용 yml파일에서 도커와 AWS ECR을 사용한 배포 설정으로 변경하였다.

### ec2 단일 서버, pm2로 실행
- appspce.yaml
```
version: 0.0
os: linux
files:
  - source: /
    destination: /home/ubuntu/mingijuk
    file_exists_behavior: OVERWRITE
hooks:
  ApplicationStop:
    - location: /code_deploy/application_stop.sh
      timeout: 500
  BeforeInstall:
    - location: /code_deploy/before_install.sh
      timeout: 500
  ApplicationStart:
    - location: /code_deploy/application_start.sh
      timeout: 500
```

- /code_deploy/application_stop.sh
```
npm uninstall -g pm2

if pgrep node; 
then pkill node; 
fi
```
- /code_deploy/before_install.sh
```
#!/bin/bash

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
. ~/.nvm/nvm.sh
nvm install node

DIR="/home/ubuntu/mingijuk"

if [ -d "$DIR" ]; then
  echo folder exists
else 
  mkdir ${DIR}
fi

sudo rm -rf /home/ubuntu/mingijuk/node_modules
sudo rm -rf /etc/localtime
sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
```
- /code_deploy/application_start.sh 
```
#!/bin/bash

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
. ~/.nvm/nvm.sh
nvm install node

sudo chmod -R 777 /home/ubuntu/mingijuk
cd /home/ubuntu/mingijuk

export NVM_DIR="$HOME/.nvm"	
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # loads nvm	
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # loads nvm bash_completion (node is in path now)

npm install
echo "npm installed"

npm install -g pm2
echo "pm2 installed"

pm2 start NODE_ENV=production node server.js
pm2 save
```

---

### ECS 블루/그린 배포, ecr으로 빌드 후 이미지 가져와서 도커로 실행
- appspce.yaml
```
version: 0.0
Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: <TASK_DEFINITION>
        LoadBalancerInfo:
          ContainerName: "mingijuk-ecr"
          ContainerPort: 8080
```
- buildspec.yaml
```
version: 0.2

phases:
  pre_build:
    commands:
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $REPOSITORY_URI
  build:
    commands:
      - docker build --tag $REPOSITORY_URI .
      - docker tag $IMAGE_TAG:latest $REPOSITORY_URI/$IMAGE_TAG:latest
  post_build:
    commands:
      - echo Build started on `date`
      - echo Pushing docker image
      - docker push $REPOSITORY_URI/$IMAGE_TAG:latest
      - printf '{"name":"%s","imageUri":"%s"}' $IMAGE_TAG $REPOSITORY_URI/$IMAGE_TAG:latest > imageDetail.json
artifacts:
  files:
    - imageDetail.json
    - appspec.yaml
    - taskdef.json
```