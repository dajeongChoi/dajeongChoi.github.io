---
title: docker 기본 명령어들
date: 2018-10-10 14:30:00
categories:
- Server
- Docker
tags:
- docker
- infra
- devOps
metaAlignment: center
thumbnailImagePosition: left
thumbnailImage: http://drive.google.com/uc?export=view&id=1vYFhABydiUz68_awyIMluz3QBcHLowlj
---
도커 설치부터 도커이미지 pull, 컨테이너 실행 등 기본 명령어들에 대해 설명합니다.
<!--more-->
<!-- toc -->
## Install
앞선 포스트에서 설명했듯이 도커는 리눅스 컨테이너를 이용하고 있으므로 리눅스 환경에서 동작한다.
그래서 맥이나 윈도우에서 도커를 실행하기 위해서는 가상머신을 설치하고 그 위에 도커를 돌려야한다.
귀찮게 가상머신까지 설치해야 하나 싶지만 그냥 도커에서 제공하는
[Docker for mac](https://docs.docker.com/docker-for-mac/)(Mac용),[Docker for Window](https://docs.docker.com/docker-for-windows/)(Window용)를 깔고 실행해주면 된다.

## Basic Command
### 설치 확인
위 링크를 통해 도커를 깔았다면 아래 커맨드를 통해
{% codeblock lang:bash %}
	docker --version
	docker-compose --version
	docker-machine --version
{% endcodeblock %}
Error없이 정보가 제대로 출력되는 것이 확인되면 정상적으로 설치가 된 것이다.
`docker`외에도 도커 프로젝트인 `docker-compose`,`docker-machine`도 함께 설치된다.

### 이미지 받아오기
이미지는 `DockerFile`로 생성해서 써도 되고 [DockerHub](https://hub.docker.com/)에서 `pull`해서 쓸 수 있다.
일단 이미지를 `pull`해 실행시켜보자.
{% codeblock lang:bash %}
	docker search centos
	docker pull centos:latest
{% endcodeblock %}
search를 통해 원하는 이미지를 탐색할 수 있다.
centos이미지를 검색하면 검색결과가 주욱 뜨는데 맨위에 뜬 official이미지를 받는다 `:latest`태그를 붙이면 최신버전을 받을 수 있고`:6.8`이렇게 원하는 버전을 지정해 받을 수도 있다.

### 이미지 조회와 삭제
다운받은 이미지를 조회하는 커맨드이다.
{% codeblock lang:bash %}
	# 조회
	docker images
	# 삭제
	docker image rm [imageID]
	# 또는
	docker rmi [imageID]
{% endcodeblock %}

### 컨테이너 생성과 실행
컨테이너 실행은 run 명령어를 사용한다.
{% codeblock lang:bash %}
	docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
	# example
	docker run -it -p 8080:80 -name centosTest centos:latest /bin/bash
{% endcodeblock %}
아래는 `run`과 함께 쓰이는 옵션들이다.
	
| 옵션 | 설명 |
| :-: | :-: |
| -d | detached mode, 백그라운드 모드 |
| -p | 호스트와 컨테이너의 포트를 연결 |
| -v | 호스트와 컨테이너의 디렉토리 마운트 |
| -e | 컨테이너 내에서 사용할 환경변수 설정 |
| –name | 컨테이너 이름 설정 |
| –rm | 프로세스 종료시 컨테이너 자동 제거 |
| -it | 터미널 입력을 위한 옵션 |
| –link | 컨테이너 연결 |
위에서 예시로 한 커맨드를 한번 살펴보면 centos를 `/bin/bash`쉘을 통해서 실행시킨다.
`bin/bash`가 없으면 컨테이너는 생성되자마자 종료되어지므로 Command를 넘겨 주도록 하자.
-p는 포트번호로 Host의 8080포트와 컨테이너의 80포트를 연결한다.
-name은 컨테이너의 이름을 지정하는 옵션으로 생략하면 랜덤으로 지정된다.
### 컨테이너 목록 확인
실행 중인 컨테이너의 목록을 확인
{% codeblock lang:bash %}
	docker container ls
	# 또는
	docker ps
{% endcodeblock %}
모든 컨테이너의 목록을 확인
{% codeblock lang:bash %}
	docker container ls -a
	# 또는
	docker ps -a
{% endcodeblock %}
### 컨테이너 중지와 삭제
{% codeblock lang:bash %}
	# 중지
	docker stop [containerID or NAME]
	# 삭제
	docker rm [containerID or NAME]
{% endcodeblock %}
컨테이너를 삭제할 때 주의해야 할 것이 있다. 
컨테이너를 제거하는 것은 컨테이너에서 생성한 파일을 모두 삭제한다는 것을 의미한다.
컨테이너 삭제 시 꼭 유지시켜야 하는 데이터의 경우 데이터 볼륨을 컨테이너에 추가해 별도로 관리할 수 있는데
`run` 옵션에서 소개한 -v를 사용하여 호스트 디렉토리를 컨테이너에 마운트해 사용하는 방법이 있다.
{% codeblock lang:bash %}
	docker run -it -d -v [host directory]:[container directory]
{% endcodeblock %}


### 실행중인 컨테이너 접속
컨테이너를 `-d`옵션으로 백그라운드 실행시켰을 때
컨테이너에 접속할 수 있는 명령어로 `exec`를 쓸 수 있다.
`run`과 유사하지만 `run`은 컨테이너를 새로 만들어 실행하고 `exec`는 이미 실행 중인 컨테이너에 명령을 내린다는 차이점이 있다.
{% codeblock lang:bash %}
	docker exec [options] [containerID] [command] [arg...]
	# example
	docker exec -it [containerID] /bin/bash
{% endcodeblock %}
bash쉘을 실행하여 완전한 권한을 얻는 방법 말고도 command를 바로실행할 수도 있음
{% codeblock lang:bash %}
	docker exec -it [containerID] ls
{% endcodeblock %}

## 정리
지금까지 사용해 본 커맨드에 대해 정리해 보았다.
포스팅한 내용 외에도 수 많은 도커 명령어들이 있으므로 `docker help`를 통해 알아보고 써먹어 보자.