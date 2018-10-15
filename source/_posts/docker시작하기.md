---
title: docker시작하기
date: 2018-08-20 18:30:05
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
docker, 컨테이너, 도커 이미지의 기본 개념에 대한 설명입니다.
<!--more-->
<!-- docker image -->
{% image center clear docker.png 70% "도커로고는 귀엽다!" %}
<!-- toc -->
## 서론
일전에 회사에서 IDC이전 프로젝트를 참여하여 서버, 인프라를 다룰 수 있는 기회가 있었다.
규모가 크지 않아 그나마 다행이었지만 직접 서버를 셋팅하는 것은 시간도 꽤 오래 걸렸고 정말 쉽지 않았다.
서비스 앞단에 로드밸런서를 두고 웹서버를 여러대 돌리는 구조로 서버들을 똑같은 환경으로 셋팅해야 했는데 설정 미스 등으로 여러번 보수 작업을 진행해야 했다.
이 후 서버 자동화, 개발, 서비스 서버의 효율적 관리에 대한 필요성으로 개발부에서 일부 서비스에 대한 컨테이너화가 진행되었고 도커를 접하게 되었다.

## Docker?!
**docker concepts [(도커 공식문서)](https://docs.docker.com/get-started/)**
> Docker is a platform for developers and sysadmins to develop, deploy, and run applications with containers.

도커는 컨테이너 기반의 오픈소스 가상화 플랫폼이다.
서버를 컨테이너화한다는 것은 서버에서 실행되는 환경, 설정, 프로그램등을 추상화해서 동일한 인터페이스로 제공, 배포, 관리하는 것이고 
이것을 보다 쉽게 가능하게 해주는 것이 오픈소스 컨테이너 프로젝트인 도커이다.

### 가상화
흔히 사용하는 VM, VirtualBox 같은 가상 머신은 HostOS위에 GuestOS를 설치해야 하므로 느리다.
또한 가상머신 이미지 안에 OS가 포함되어 용량도 크다. 이미지 용량이 커지면 이미지의 배포, 관리도 어려워진다.

그래서 리눅스 컨테이너라는 개념이 등장하게 된다.
가상머신과 달리 GuestOS가 없이 HostOS 위에서 자원을 공유하여 실행되며 단순히 프로세스만 격리시키기 때문에 빠르다.
도커는 이 리눅스 컨테이너의 개념을 채용하여 개발되었으므로 역시 빠르고 이미지 용량이 크지않아 이미지 생성과 배포가 편하다.
특히 [GitHub](https://github.com/)에서 소스를 push,pull하는 것처럼 도커도 [DockerHub](https://hub.docker.com/)라는 서비스를 제공하고 있어 이미지의 배포, 공유를 아주 쉽게 할 수있다!
{% image center clear docker_container.png 90% 80% "가상머신과 도커 컨테이너 비교" %}

## 도커에서 컨테이너와 이미지란?
통상 컨테이너는 화물을 운송할 때 쓰이는 규격화 된 용기로 무엇이든 담아 운반과 관리를 용이하게 할 수 있는 것을 뜻하는 것처럼 서버에서 컨테이너도 OS, 서버 운영에 필요한 프로그램, 라이브러리 등(JAVA,Mysql,Redis etc...)을 담아 실행하고 관리할 수 있다. 
도커에서 컨테이너는 {% hl_text blue %}이미지를 실행한 상태{% endhl_text %}이며
이미지는 {% hl_text blue %}서비스 운영에 필요한 프로그램, 라이브러리(JAVA,Mysql,Redis etc...)등의 설정값을 포함{% endhl_text %}하고있다.

CentOS 서버를 실행시키고 싶다면 CentOS이미지를 다운([DockerHub에서!](https://hub.docker.com/))받아서 실행시키면 되고, 더불어 Mysql이 필요하다면 CentOS이미지에 Mysql이미지를 올려 컨테이너화 시키면 되는 것.
다시 말해 이미지는 컨테이너를 실행하기 위한 모든 정보를 갖고 있어 원하는 이미지를 다운로드받아 설치, 설정 다 필요없이 실행만 시키면 된다! ~~개꿀?~~

### 베이스 이미지, 도커 이미지
도커 이미지는 베이스 이미지에 추가로 설치, 설정, 저장한 이미지이다.
>베이스 이미지란, 리눅스 배포판의 커널 영역을 제외한 사용자 공간/영역(유저랜드)만을 이미지에 포함한 파일을 의미한다.
 출처: http://dev.youngkyu.kr/32 [조영규의 블로그]

베이스 이미지의 정의는 이렇다고 하는데, 위에서 예를 든 CentOS이미지를 베이스 이미지라고 이해하면 되겠다.
그리고 이 이미지를 추가, 변형하여 변형된 이미지를 만든다고 한다면 이것을 도커 이미지라고 할 수 있다. 

도커 이미지의 특징은 유니온 파일 시스템이라는 처리 방식을 사용하고 있다.
베이스 이미지에 무언가 추가되거나 변형이 이루어지면 그 바뀐 부분만 이미지로 생성하는 방식으로 컨테이너로 실행할 때는 {% hl_text red %} 베이스 이미지 + 추가된 이미지 {% endhl_text %} 를 합쳐서 실행한다.
이 방식을 이용하면 이미지를 공유할 때 변형된 이미지만 주고 받으므로 굉장히 효율적으로 이미지를 관리, 배포할 수 있게 되는 것이다.

## DockerFile
이미지를 생성하기 위해서는 DockerFile이라는 것을 작성해야 한다.
{% codeblock DockerFile %}
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
{% endcodeblock %}
샘플 코드를 보면 알 수 있듯이 어떤 이미지를 사용하고 워킹디렉토리는 어디로 할 것이며, 어떤 명령어를 사용하여 무엇을 하는지 등등 이미지 생성과정이 코드화되어 적혀있다. 
서버 생성, 설치, 설정 과정이 소스로 되어 있어 누구나 이해하고 변형할 수 있으며 서버 설치 과정을 버전 관리 할 수 있게 되어 따로 로그로 남기지 않아도 된다!
이 DockerFile를 빌드함으로써 언제든지 이미지를 생성하고 컨테이너화 할 수 있게 된다.

## 정리
도커의 장점은 위에서 기술한 것외에도 많고 활용도도 무궁무진하다.
~~역시 능력자들의 글을 보는게...~~
[서비큐라 기술블로그(초보를 위한 도커 안내서 - 도커란 무엇인가?) ](https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html)
[조영규의 블로그(도커란 무엇인가)](http://dev.youngkyu.kr/32)






 
