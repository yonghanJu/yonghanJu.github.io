---
layout: post
title:  Devops Tutorial_1 - Build Images
date:   2021-08-07 13:17:46 +0300
categories:   Devops
---
# Devops Tutorial

<br>

### Devops Tutorial 일기장
---
Devops Tutorial을 진행하며 배운 점들과 문제 해결 과정을 정리하는 포스팅이다. 

<br>

#### Docker 이미지 빌드

도커 이미지는 컨테이너를 실행하기 위한 파일이다. 

컨테이너는 하드웨어에서 분리된 형태로 어플리케이션을 배포한다.

<br>

도커 이미지 빌드 공식 문서
https://docs.docker.com/language/python/build-images/

<br>

``` $ docker build --tag <images name> .  ``` 명령어를 통해 현재 위치를 빌드 하기 위해선 작업 디렉토리에 requirments.txt, Dockerfile이 필요하다.

<br>

Dockerfile은 이미지 빌드를 위한 커맨드 파일이며 빌드 명령어를 실행하면 해당 디렉토리에서 Dockerfile을 찾아 명령어를 실행하고 이미지를 빌드한다.

Dockerfile 이름을 강제하진 않지만 관용적으로 통일해 사용하는걸 권장한다.

<br>

---

<br>

directory structure

```
python-docker
|____ app.py
|____ requirements.txt
|____ Dockerfile
```

ex : https://github.com/yonghanJu/pyprojects

<br>

Dockfile
```
FROM python:3.8-slim-buster

WORKDIR /app

COPY requirements.txt requirements.txt

RUN pip3 install -r requirements.txt

COPY . .

CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0"]
```

<br>

requirments.txt (dependencies)

``` Flask==1.1.1 ```

<br>

``` $ docker build --tag <images name> .  ```

빌드 성공후엔 ```$ docker images ``` 명령어를 통해 이미지를 확인할 수 있고 docker desktop을 통해서도 이미지를 확인할 수 있다.

<br>

![alt text](/public/img/devops_1_1.png)

<br>

![alt text](/public/img/devops_1.png)

<br>

``` $ docker tag <images name>:latest <images name>:v1.0.0 ```

위 명령어를 통해서 태그를 바꿀 수 있다. 기본 태그는 ```latest```이며 images name:\<tag name> 형태이다.

<br>

이미지를 지울 땐 ``` $ docker rmi <images name> ``` 명령어를 사용하면 되고 또 docker desktop의 그래픽 인터페이스를 통해 지울 수 있다.
