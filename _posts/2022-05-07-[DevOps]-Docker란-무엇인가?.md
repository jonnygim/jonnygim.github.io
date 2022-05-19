---
title: Docker 란 무엇인가
date: 2022-05-05 23:00:00 +09:00
categories: [DevOps, Docker]
tags: [DevOps, Docker]
---

![Docker](/assets/img/docker-logo.png)

처음 백엔드 개발을 접했을 때는 서버에 개념조차 이해하기 어려웠는데 조금 공부하다 보니 
도커에 대한 이야기를 정말 보게 되는 것 같다. 도커는 과연 무엇일까?


## 도커(Docker)

도커는 컨테이너 기반의 오픈소스 가상화 플랫폼이다.<br><br>

도커에서는 컨테이너라는 개념이 등장하는데 이는 격리된 공간에서 프로세스가 동작하는 기술을 일컫는다.<br>
가상머신(VM)을 이용하여 윈도우나 macos에서 또 다른 os 를 사용해 본적이 있다면 조금은 이해가 갈 수도 있지만,<br>
도커에서의 컨테이너는 같은 가상화 기술이면서도 방식이 다르다.
가상머신(VM)은 '하이퍼바이저'를 사용하여 논리적으로 분할된 공간에서 독립된 가상환경을 만들어서 Guest OS를 설치하고 , Host OS가 이를 구동하고 모니터링하는 방식이다. 가상머신은 전체 OS를 가상화해서 매우 무겁고 속도가 느리기 떄문에 서버를 구성하기에는 적절하지 못하다.


## 컨테이너(Container)
컨테이너는 격리된 공간에서 프로세스가 동작하는 기술이다. 가상화 기술의 하나지만 기존 방식과는 차이가 있다. 
기존의 가상화 방식은 주로 OS를 가상화해서 사용했지만 무겁고 느려서 운영환경에서는 사용할 수 없었다. 이것을 개선하기 위해 CPU의 가상화 기술(HVM)을 이용한 KVM(Kernel-based Virtual Machine)과 반가상화(Paravitualization)방식의 Xen이 등장한다.

 이러한 방식은 전체 OS를 가상화하는 방식이 아니기 때문에 성능이 향상되었다. 하지만 전가상화든 반가상화든 추가적인 OS를 설치하여 가상화하는 방법은 성능에 문제가 있었기 때문에 이를 개선하기 위해 프로세스를 격리하는 방식이 등장한다. 리눅스에서는 이 방식을 리눅스 컨테이너(LXC)라고 불렀고 도커는 이 리눅스 컨테이너를 기반으로 시작해서 0.9버전에서는 자체적인 libcontainer 기술을 사용하였다. 이 기술은 추후 runC 기술에 합쳐지게 되었다.

## 이미지(Image)
도커에서 가장 중요한 개념은 컨테이너와 더불어 이미지라는 개념이다. 

이미지는 컨테이너 실행에 필요한 파일과 설정 값 등을 포함하고 있는 것이다. 상태 값을 가지지 않으며 변하지 않는다(Immutable). 이러한 이미지를 실행한 상태를 컨테이너라고 볼 수 있다. 
추가되거나 변하는 값은 컨테이너에 저장되며 같은 이미지에서 여러개의 컨테이너를 생성할 수 있다. 컨테이너의 상태가 바뀌거나 컨테이너가 삭제되더라도 이미지는 변하지 않고 그대로 남아있다. 
이처럼 이미지는 컨테이너를 실행하기 위한 모든 정보를 가지고 있기 때문에 더이상 의존성 파일을 컴파일하고 이것저것 설치할 필요가 없다. 새로운 서버가 추가되면 미리 만들어 놓은 이미지를 다운받고 컨테이너를 생성만 하면 되는 것이다.

## 레이어(Layer)

![Docker](/assets/img/docker-layer.png)

도커 이미지는 컨테이너를 실행하기 위한 모든 정보를 가지고 있기 때문에 보통 용량이 수백 메가(MB)이다. 처음에 이미지를 다운 받고 나서 기존 이미지에 파일 하나를 추가하며 다시 큰 용량을 다운 받는 것은 매우 비효율적이기 때문에 도커는 레이어(Layer)라는 개념을 사용한다.
도커는 유니온 파일 시스템을 이용하여 여러개의 레이를 하나의 파일시스템으로 사용할 수 있게 해준다. 이미지는 여러 개의 읽기 전용(read only) 레이어로 구성되고 파일이 추가되거나 수정되면 새로운 레이어가 생성되는 방식이다. 
ubuntu 이미지가 A + B + C 라면
Ubuntu 이미지를 베이스로 만든 nginx 이미지는 A + B + C + nginx,
webapp 이미지를 nginx 기반으로 만들었다면 A + B + C + nginx + source 가 된다.
webapp 소스를 수정하면 A, B, C, nginx 레이어를 제외한 새로운 source(v2) 레이어만 다운받으면 되기 때문에 굉장히 효율적으로 이미지를 관리할 수 있다. 

컨테이너를 생성할 때도 레이어 방식을 사용한다.
기존의 이미지 레이어 위에 읽기/쓰기(read-write) 레이어를 추가 한다. 이미지 레이어를 그대로 사용하면서 컨테이너가 실행 중에 생성하는 파일이나 변경된 내용은 읽기/쓰기 레이어에 저장되므로 여러개의 컨테이너를 생성해도 최소한의 용량만 사용한다. 

## 이미지 경로

![Docker](/assets/img/docker-url.png)

이미지는 url 방식으로 관리하며 태그를 붙일 수 있다. ubuntu 14.04 이미지는 docker.io/library/ubuntu:14.04 또는 docker.io/library/ubuntu:trusty 이고 docker.io/library 는 생략가능하여 ubuntu:14.04 로 사용할 수 있다. 

```dockerfile
# vertx/vertx3 debian version
FROM subicura/vertx3:3.3.1
MAINTAINER chungsub.kim@purpleworks.co.kr

ADD build/distributions/app-3.3.1.tar /
ADD config.template.json /app-3.3.1/bin/config.json
ADD docker/script/start.sh /usr/local/bin/
RUN ln -s /usr/local/bin/start.sh /start.sh

EXPOSE 8080
EXPOSE 7000

CMD ["start.sh"]
```

도커는 이미지를 만들기 위해 Dockerfile 이라는 파일에 자체 DSL(Domain-specific language)언어를 이용하여 이미지 생성 과정을 적는다. 서버에 어떤 프로그램을 설치하려고 많은 의존성 패키지를 설치하고 설정파일을 만들던 작업을 Dockerfile 로 관리하면 된다. 

## Docker hub
Docker hub는 도커에서 운영하는 이미지 저장소 서비스이다. 도커 이미지를 저장하는 원격 스토리지를 Image Registry 라고 부르며, Docker hub 는 도커의 기본이자 공식 Image Registry 이다. 

<br>
<br>
<br>


* 출처 <br>
초보를 위한 도커 안내서 - 도커란 무엇인가? https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html (2022-05-07) <br>
https://www.lainyzine.com/ko/article/

