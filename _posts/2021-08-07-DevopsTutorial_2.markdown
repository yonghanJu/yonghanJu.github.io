---
layout: post
title:  Devops Tutorial_2 - Run Containers
date:   2021-08-08 16:34:46 +0300
categories:   Devops
---
# Devops Tutorial

<br>

### Devops Tutorial 일기장
---
Devops Tutorial을 진행하며 배운 점들과 문제 해결 과정을 정리하는 포스팅이다. 

<br>

#### 컨테이너의 필요성

기존 vm을 통해 여러 어플리케이션을 실행시키는 방식에서는 여러 가상화 머신 위에 각 OS가 설치어야했지만 컨테이너는 격리된 어플리케이션이 하나의 host os 위에서 동작하므로 훨씬 가볍고 효율적입니다.

![alt text](/public/img/devops_2.png)

https://www.docker.com/resources/what-container

<br>

---

<br>

#### Run Containers

이전 포스팅에서 도커 이미지를 성공적으로 빌드 했다면 이미지를 이용해 컨테이너는 실행할 차례이다.

<br>

``` $ docker run --publish 5000:5000 <images name> ``` 명령어를 입력하면 이미지를 통해 컨테이너를 실행 시켜 localhost:5000에 배포한다. 컨테이너의 id가 output으로 나온다.

--publish 대신 --p 사용가능, --p 의 파라미터는 [host port]:[container port] 형태

<br>

![alt text](/public/img/devops_2_1.png)

<br>

---

<br>

#### List Containers

``` $ docker ps ``` 

명령어를 입력해 __현재 실행 중__ 인 컨테이너를 확인 할 수 있고 

``` --all ``` 또는 ``` --a ``` 를 붙이면 실행 중지된 컨테이너를 포함해 모든 컨테이너 정보를 반환한다. (docker desktop을 통해서도 확인 가능)

<br>

![alt text](/public/img/devops_2_2.png)

<br>

---

아래와 같이 Docker Desktop을 통해서 컨테이너를 실행시킬 수 있다.

<br>

![alt text](/public/img/devops_2_3.png)

<br>

``` $ docker stop <container name> ``` 
명령어를 입력해 컨테이너를 중지 시킬 수 있다.

이때 컨테이너는 사라지지 않고 실행 중지 상태의 status를 갖게되며
``` $ docker restart <container name> ``` 명령어를 입력해 다시 실행 시킬 수 있다.

<br>

---

현재까지 도커 이미지 빌드, 컨테이너 실행까지 진행 완료 

다음 단계는 AWS codebuild를 이용해 도커 이미지 빌드 자동화

---

Container run docker 공식 문서 https://docs.docker.com/language/python/run-containers/