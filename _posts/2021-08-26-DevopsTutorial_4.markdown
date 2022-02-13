---
layout: post
title:  Devops Tutorial_4 - AWS CodeBuild
date:   2021-08-26 05:56:46 +0300
categories:   Devops
---
# Devops Tutorial

<br>

#### 개요

이전 [도커 이미지 빌드][1], [컨테이너 실행][2], [AWS codebuild 도커 빌드 자동화][3]에 이어서 __AWS CodePipeline__, __AWS ElasticBeanstalk__ 를 이용해 웹 어플리케이션 배포하기

[1]: https://yonghanju.github.io/2021/08/07/DevopsTutorial_1/

[2]: https://yonghanju.github.io/2021/08/08/DevopsTutorial_2/

[3]: https://yonghanju.github.io/2021/08/24/DevopsTutorial_3/

<br>

AWS CodePipeline 공식 사이트 https://ap-northeast-2.console.aws.amazon.com/codepipeline/home?region=ap-northeast-2#/welcome

<br>

AWS ElasticBeanstalk 공식 사이트 https://ap-northeast-2.console.aws.amazon.com/elasticbeanstalk/home?region=ap-northeast-2#/welcome


---

<br>

## AWS ElasticBeanstalk 생성

#### Create Application

<br>

1. Application Name

2. Platform: Docker, Platform Branch: Docker running...Amazon Linux 2, Platform version: Recommended

3. Application Code: Sample application

<br>

## buildspec.yml 내용 추가

<br>

기존 ```buildspec.yml``` 밑에 

```
    - printf '{"AWSEBDockerrunVersion":"1","Image":{"Name":"%s"},"Ports":[{"ContainerPort":"5000"}]}' $IMAGE_REPO_NAME:$TAG_VERSION > Dockerrun.aws.json
artifacts:
    files: Dockerrun.aws.json
```
추가

<br>

<br>

buildspec.yml 
```
version: 0.2

phases:
  pre_build:
    commands:
      - echo Logging in to Docker Hub...
      - docker login -u $DOCKERHUB_USER -p $DOCKERHUB_PW
      - TAG="$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | head -c 8)"
  build:
    commands:
      - echo Build started on `date`
      - echo Building the Docker image...
      - docker build -t $IMAGE_REPO_NAME:$TAG_VERSION .
      - docker tag $IMAGE_REPO_NAME:$TAG_VERSION $IMAGE_REPO_NAME:$TAG
  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker image...
      - docker push $IMAGE_REPO_NAME:$TAG_VERSION
      - echo Writing image definitions file...
      - printf '{"AWSEBDockerrunVersion":"1","Image":{"Name":"%s"},"Ports":[{"ContainerPort":"5000"}]}' $IMAGE_REPO_NAME:$TAG_VERSION > Dockerrun.aws.json
artifacts:
    files: Dockerrun.aws.json
```
<br>

https://github.com/yonghanJu/DevopsTutorial/blob/master/buildspec.yml

<br>

---
### 겪었던 오류(포트 수정)

<br>

[수정 내역][수정]

[수정]:https://github.com/yonghanJu/DevopsTutorial/commit/38e36915b36006832041845873d93047acaf5615

[Dockerfile][d]을 보면 포트를 따로 지정해 주지 않고 기본 포트를 사용했기 때문에 포트를 Buildspec.yml에 추가한 포트를 8000에서 5000으로 수정해 주었다. __각자 자신에 맞는 포트 설정 필요.__

[d]:https://github.com/yonghanJu/pyprojects/blob/master/Dockerfile#L11 

<br>

---

<br>

#### Set up AWS CodePipeline

[AWS CodePipeline][ab]에 들어가 아래 내용 진행

[ab]: https://ap-northeast-2.console.aws.amazon.com/codesuite/codepipeline/pipelines

<br>

__파이프라인 생성__

1. 파이프라인 이름 작성

2. Service Role: New Service Role

3. Role Name: ```AWSCodePipelineServiceRole-ap-northeast-2-[Pipeline Name]```

<br>

__Source Stage__

1. 소스: Github(Version 1), 내 GitHub 계정의 리포지토리

2. Github v1

3. Repository, Branch: 본인의 Repo의 원하는 branch

4. Detection option: GitHub Webhook(recommended)

__Build Stage__

[이전 포스팅][3]에서 만들었던 codebuild 선택

<br>

__Deploy Stage__

1. Provider: AWS Elastic Beanstalk

2. Application Name, Environment Name: 위에서 buildspec.yml에 추가한 EB 정보

<br>

---

### Pipeline 작동 확인

별도의 Branch를 만들어 코드 변경 후 main branch로 PR 수행.



https://ap-northeast-2.console.aws.amazon.com/codesuite/codepipeline/pipelines

Pipeline 도구가 변경 사항을 인지하여 자동으로 빌드/배포가 수행 되는지 확인

<br>

![alt text](/public/img/devops_4_2.png)

code pipeline에서 __CI_test__  branch로 부터 Pull Request를 감지

<br>

source(github)에 변화 감지
(webhook) -> build(AWS codebuild, Docker)

![alt text](/public/img/devops_4_3.png)

<br>

build(AWS codebuild, Docker) -> Deploy(AWS ElasticBeanstalk)

![alt text](/public/img/devops_4_4.png)

<br>


<br>

__코드 변경 사항이 잘 적용, 배포 되었다면 성공적!__

<br>
