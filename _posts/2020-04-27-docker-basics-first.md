---
title:  "docker 좋은데 ?! - 1 "
excerpt: "~ 아무것도 모르고 이미지 만들어보기 ~ "

categories:
  - docker
tags:
  - docker, container
last_modified_at: 2020-04-27T17:42:00-05:00
---
 
> 일단 docker의 기본 개념을 (대충) 알아보자. 나도 잘 모르니깐...😛 (대충) 해보니까 드는 생각은 너무 편하다 👏🏻 매번 환경설정 맞춰 줘야 되고.. 버전 맞춰줘야 될 일 없어서..! 하물며 강의 들으면서 공부하는데 조차 !!!! 패키지, 버전때문에 골치아플 일 없어서 너무 좋다. 휴~ 그리고 난 내 로컬에 뭐 잘못될때마다 자꾸 까는게 싫어... 도커로 개발하면서 알게된 내용은 계속 추가 예정이다.

## 🗒 Table of Contents
- [docker가 무어냐?](#docker란)
- [docker의 기본 개념](#docker의-기본개념)
    - [docker의 동작방식](#docker의-동작방식)
    - [docker의 이미지](#docker의-이미지)
    - [docker의 컨테이너](#docker의-컨테이너)
- [실제 수행기](#실제-수행기)
- [reference](#reference)

## docker란?

**컨테이너 기반의 오픈소스 가상화 플랫폼**이다. 다양한 프로그램, 실행환경을 컨테이너로 추상화하여 동일한 인터페이스를 제공해서 프로그램의 배포 및 관리를 단순화하게 해준다. 그리고 어디에서나 실행할 수 있도록 해준다. 내가 경험한 docker로 말하자면, 어떤 환경에서, 얼마나 패키지가 업데이트 진행된 상태더라도(=시간이  흘렀어도) 똑같은 프로그램이 똑같이 동작하게 된다는 거다. `내 코드에는 전혀 이상 없는것 같은데 왜 안돼?`하면서 구글링하며 삽질하는 시간을 save할 수 있다는 점 😌

## docker의 기본개념

### docker의 동작방식

docker는 OS-level 가상화 툴이다. 그럼 기존의 VMware, Virtual Box같은 가상화머신과 뭐가 다를까? 가장 큰 차이는 OS-level이냐, Hardaware-level이냐 하는 것. 어떤 수준의 가상화를 제공하는 것이냐! 이다. 가상머신은 호스트OS위에 게스트OS전체를 가상화하여 사용하는 방식이다. 사용법은 간단하나 무겁고 느리다.

> (참고) 너무 느리니까 CPU 가상화기술 HVM방식의 KVM (kernel-based virual machine)과 반가상화 방식의 Xen이 등장한다. 호스트형 가상화 방식에 비해 성능이 향상되었고, 이 기술들은 OpenStack이나, AWS 같은 클라우드서비스에서 가상 컴퓨팅 기술의 기반이 되었다.

추가적인 OS를 설치하여 가상화하는 건 아무리 해도 너무 느려! 그래서 **프로세스**를 격리하는 방식이 등장한다. 프로세스만 격리되기 때문에 빠르고, CPU나 메모리도 프로세스가 필요한 만큼만 추가로 사용되기 때문에 성능 손실이 거의 없다.

### docker의 이미지

이미지는 컨테이너 실행에 필요한 파일과 설정값등을 포함하고 있는 것으로 상태값을 가지지 않으며 변하지 않는다.

### docker의 컨테이너

Container 기술이 아주 중요한것 같지만 좀 방대한 것 같으므로, 일단 정의만 알아보자. 일단 컨테이너는 호스트OS상에 논리적인 구획을 만들고 어플리케이션을 작동시키기 위해 필요한 어플리케이션 등을 하나로 모아 마치 별도의 서버인 것처럼 사용할 수 있게 만든 것이다. 하드웨어 에뮬레이터 없이 리눅스 커널을 공유해서 바로 실행한다. 

## 실제 수행기 

> 공부하려니까 docker가 그냥 막연히 재밌어 보여서 이미지를 한번 만들어 보았다. 실제로 재밌었고, 나의 로컬을 깨끗이 하며 나의 딥러닝 프로젝트는 저 컨테이너에서 진행을 하겠다 !

일단 기본이 되는 ubuntu 18.04 이미지를 받아왔다. 

`docker pull ubuntu:18.04`

```zsh
➜  ~ docker pull ubuntu:18.04
18.04: Pulling from library/ubuntu
23884877105a: Pull complete 
bc38caa0f5b9: Pull complete 
2910811b6c42: Pull complete 
36505266dcc6: Pull complete 
Digest: sha256:3235326357dfb65f1781dbc4df3b834546d8bf914e82cce58e6e6b676e23ce8f
Status: Downloaded newer image for ubuntu:18.04
docker.io/library/ubuntu:18.04
```

내가 제대로 받아왔는지 봐보자
`docker image ls`

```zsh
➜  ~ docker image ls
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
ubuntu                          18.04               c3c304cb4f22        4 days ago          64.2MB
chloejiwon/deeplearningbasics   pytorch             2328f41a83f4        9 days ago          6.79GB
ubuntu                          latest              4e5021d210f6        5 weeks ago         64.2MB
deeplearningzerotoall/pytorch   latest              2b3fd7327ec7        13 months ago       4.14GB
hello-world                     latest              fce289e99eb9        16 months ago       1.84kB
```

위에 보면 ubuntu REPOSITORY에 18.04 TAG 되어있는 이미지가 잘 생성된 걸 확인할 수 있다. 그럼 이 이미지를 컨테이너로 실행시켜보자!

`docker run -i -it ubuntu /bin/bash`

oh ho~ 이제 컨테이너 실행시키고 그 안에 들어온 걸 알 수 있다.

```zsh
➜  ~ docker run -i -t ubuntu:18.04 /bin/bash
root@640d6ef0849d:/# 
```

이제 여기서 ubuntu에서 하는 것처럼 이것저것 내가 필요한 것들을 설치해주면 된다.
나는 이것들을 깔았다.

1. git
2. anaconda

되게 많이한 것 같은데 기억이 안나네;

~~ 자세한 부분은 추후 ~~

그럼 이제 내가 만든 이미지를 내 허브에 올려서 내가 로컬을 옮길때마다 다운받을 수 있도록 하면 좋겠다! 

`docker stop <<내가 실행한 컨테이너 이름>>`

(이름은 `docker ps -a`해서 알아낼 수 있음)

그리고 지금까지의 컨테이너를 이미지화 한다.

`docker commit -a "<<author name>>" <<container_id>> image_name/tag`

이제 `docker images`하면 아마 내가 만든 이름을 찾을 수 있을 것이다!

```zsh
➜  ~ docker commit -a "chloejiwon" busy_gould test1/test 
sha256:b06a2521461631c2c394f5a025ff7ba6825fa13c29444b47f615ed8d89fe762d
➜  ~ 
➜  ~ docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
test1/test                      latest              b06a25214616        3 seconds ago       64.2MB
ubuntu                          18.04               c3c304cb4f22        4 days ago          64.2MB
chloejiwon/deeplearningbasics   pytorch             2328f41a83f4        9 days ago          6.79GB
ubuntu                          latest              4e5021d210f6        5 weeks ago         64.2MB
deeplearningzerotoall/pytorch   latest              2b3fd7327ec7        13 months ago       4.14GB
hello-world      
```
그럼 이제 내가 잘 만든 이미지를 docker hub에 업로드 해보자

`docker tag test1/test chloejiwon/test:1.0`
우리가 생성한 이미지의 이름을 repository에 업로드하기 위해 고유 태그로 바꾼다.

`docker push chloejiwon/test:1.0`
하면, 이제 막 내 docker hub에 push 된다. 쭉쭉..


## reference
* https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html

* http://moducon.kr/2018/wp-content/uploads/sites/2/2018/12/leesangsoo_slide.pdf

* https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html