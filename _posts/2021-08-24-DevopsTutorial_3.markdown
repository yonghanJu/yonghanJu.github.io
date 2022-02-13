---
layout: post
title:  Devops Tutorial_3 - AWS CodeBuild
date:   2021-08-24 16:05:46 +0300
categories:   Devops
---
# Devops Tutorial

<br>

#### 개요

이전 [도커 이미지 빌드][1], [컨테이너 실행][2]에 이어서 __AWS CodeBuild__ 를 이용해 도커 이미지 빌드 자동화 적용

[1]: https://yonghanju.github.io/2021/08/07/DevopsTutorial_1/

[2]: https://yonghanju.github.io/2021/08/08/DevopsTutorial_2/

<br>

AWS CodeBuild 공식 사이트 https://aws.amazon.com/ko/codebuild/

---

<br>

## AWS CodeBuild 프로젝트 생성

#### buildsepc.yml 생성

도커 이미지 빌드 자동화 AWS CodeBuild를 사용하기 위해선 최상위 디렉토리에 CodeBuild의 작업을 정의한 ```buildspec.yml```를 생성해야한다.

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
     
```

<br>

https://github.com/yonghanJu/DevopsTutorial/blob/master/buildspec.yml

<br>

---

<br>

#### Set up AWS CodeBuild

[AWS CodeBuild][ab]에 들어가 아래 내용 진행

[ab]: https://ap-northeast-2.console.aws.amazon.com/codesuite/codebuild/projects

1. 소스: Github, 내 GitHub 계정의 리포지토리

2. [GitHub Personal access token][3] 생성 필요

[3]: https://docs.github.com/en/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token

3. 권한은 [repo, admin:repo_hook][4]

[4]: https://docs.aws.amazon.com/codebuild/latest/userguide/access-tokens.html#access-tokens-github

4. Webhook: 코드 변경이 이 리포지토리에 푸시될 때마다 다시 빌드

5. 이벤트 유형: ```PULL_REQUEST_CREATED```, ```PULL_REQUEST_UPDATED```, ```PULL_REQUEST_REOPENED```

6. 특정 Branch 이름이나 Tag로 이벤트를 감지 하고 싶다면 ```Start a build under these condition```에 필터 추가

    https://docs.aws.amazon.com/codebuild/latest/userguide/github-webhook.html 
    
    ```One or more optional filters``` 부분 참고
7. 환경: 관리형 이미지, Ubuntu, Standard, aws/codebuild/standard:4.0, 권한 승격 활성화

8. 서비스 역할: 새 서비스 역할 (Name: default e.g., codebuild-[project_name]-service-role)

<br>

---

<br>

#### 프로젝트 생성 후

1. Additional configuration 에 환경 변수 설정:

2. TAG_VERSION(일반 텍스트): latest

    이미지 자동 빌드시 latest 태그 사용

3. DOCKERHUB_USER(Secrets Manager): dockerhub:username

4. DOCKERHUB_PW(Secrets Manager): dockerhub:password

<br>

---

__겪었던 오류 수정__

[수정 내역][수정] 

Additional configuration 에 환경 변수 설정 과정에서 변수명을 __TAG_VERSION__ 으로 설정 했지만 ```buildspec.yml```에는 변수명을 __TAG__ 로 작성해 태그가 적용이 되지 않았음.

수정 후 태그가 잘 적용됨.

추가: AWS 콘솔에서 변수명 자체를 바꾸는 것 보단 ```buildspec.yml``` 을 수정하는게 더 편리하다.

SecretManager 정책상 변수명을 변경하기 힘들어서 관련 정책을 모두 지우고 다시 설정해줘야 하기 때문.

---

[수정]: https://github.com/yonghanJu/DevopsTutorial/commit/f9f598639d6e3181eef9f7d8c625c43687a373bf#diff-69387ac97f1b775f19989fc28150f89e785406cbff04643a969f5e396573dd5c

<br>

username, password 보안을 위해 Secrets Manager를 활용 [참고 문서][smm]

[smm]: https://aws.amazon.com/ko/premiumsupport/knowledge-center/codebuild-docker-pull-image-error/?nc1=h_ls#Store_your_DockerHub_credentials_with_AWS_Secrets_Manager


---

<br>

1. BuildSpec: default(빈칸, default: buildspec.yml)

2. 입력값 필요 없음

3. Artifact: 없음 

4. CloudWatch: Default(CloudWatch 로그 선택)

---

<br>

## Configure Secrets Manager

[Secret Managers][sm] 이동

[sm]: https://ap-northeast-2.console.aws.amazon.com/secretsmanager/home?region=ap-northeast-2#!/listSecrets

[DockerHub 자격증명 저장][d]

[d]: https://aws.amazon.com/ko/premiumsupport/knowledge-center/codebuild-docker-pull-image-error/?nc1=h_ls#Store_your_DockerHub_credentials_with_AWS_Secrets_Manager

Store a new secret - Type: Other type of Secrets - Secret key/vale에 username: <dockerhub 계정>, password: <dockerhub 패스워드> 입력 - Secret Name: ```dockerhub```

---

<br>

## Configure IAM policy

secret manager을 통해 관리하는 dockerhub 관련 정보들을 읽어올 권한을 부여하기 위해 

```CodeBuildSecretsManagerPolicy-[codebuild project name]-ap-northeast-2```의 Resource에 secretsmanager dockerhub ARN 추가 필요

[Secertmanager dockerhub ARN][sm]

![alt text](/public/img/devops_3_1.png)

---

<br>

#### IAM 정책에 dockerhub secrets resource 추가

[IAM policy][iam]

[iam]: https://console.aws.amazon.com/iam/home?region=ap-northeast-2#/policies

```CodeBuildSecretsManagerPolicy-[codebuild project name]-ap-northeast-2``` 검색후 수정하기, Json resource 추가

<br>

![alt text](/public/img/devops_3_2.png)

<br>

__AWS CodeBuild 설정 완료__

---

<br>

### AWS CodeBuild 작동 유무 확인하기

<br>

https://ap-northeast-2.console.aws.amazon.com/codesuite/codebuild build > project build 에 빌드 내역 확인 가능

https://hub.docker.com 에서 빌드된 이미지 확인 가능

#### Pull Request 테스트

branch 생성 후 Pull Request, master branch로 merge 되면 자동으로 AWS CodeBuild를 통해 이미지 빌드 확인

<br>

![alt text](/public/img/devops_3_3.png)

(pull request check 과정에서 조금 기다려야 함)

<br>

__위 사진 처럼 나오면 문제 없음!__
