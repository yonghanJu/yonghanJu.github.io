---
layout: post
title:  Devops Tutorial_0
date:   2021-07-08 21:58:46 +0300
categories:   Devops
---
# Devops Tutorial

<br>

### Devops Tutorial 일기장
---
Devops Tutorial을 진행하며 배운 점들과 문제 해결 과정을 정리하는 포스팅이다.

Docker 설치하기

<br>

![alt text](/public/img/devops_0_0.png)

<br>

위 모습은 성공적으로 Docker가 설치된 모습이다.

<br>

도커 설치부터 쉽진 않았다.

현재 나는 wsl 우분투를 사용하고 있었으며 wsl2로 업데이트가 되지 않은 상태였다.

Doker의 백엔드로 wsl2이 사용 되기 때문에 wsl2로 업데이트 하라는 경고 문구가 나오는 상태였다.

<br>
window powershell에서

``` wsl -l -v ```

명령어를 통해 wsl 버전을 확인 할 수 있다.

<br>

![alt text](/public/img/devops_0_1.png)

https://docs.microsoft.com/ko-kr/windows/wsl/install-win10

위 링크를 통해 wsl2 업데이트 패키지를 다운하고

``` wsl --set-version Ubuntu 2 ```

명령어를 통해 우분투를 wsl2로 설정하고 다시

``` wsl -l -v ```

명령어를 실행하면 우분투의 wsl 버전이 2로 바뀐걸 확인 할 수 있다.

그 후 Doker를 실행하니 문제가 없어졌다.

***
### Hello World

<br>

도커를 설치하고 가장 먼저 hello world를 pull 해보았다.

``` $ docker image pull hello-world:latest ```
를 커맨드 라인에 입력하면 hello-world 이미지를 pull 해 로컬 이미지로 가져온다.

그 후 ``` $ docker run hello-world  ```를 커맨드에 입력하면  

<br>
![alt text](/public/img/devops_0_2.png)

<br>

위 사진과 같은 실행 결과가 나온다면 성공한거다.