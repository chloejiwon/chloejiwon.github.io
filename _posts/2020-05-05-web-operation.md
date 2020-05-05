---
title:  "주소 창에 www.naver.com 치면 일어나는 일들 "
excerpt: "WEB 동작 원리"

categories:
  - network
tags:
  - web, network
last_modified_at: 2020-05-05T01:11:00-05:00
---

# WEB 동작 원리

> 빽 투더 베이직이다 ~ 탄탄 탄탄🧘🏻 ‍(수양 중) 주소창에 www.naver.com을 검색하면 어떤 일이 일어날까? 결국엔 WEB이란 어떻게 동작할까를 알아보자.

## 🗒 Table of Contents
- [설마 이건 알겠지?](#설마-이건-알겠지)
    - [IP와 Domain, DNS](#ip-domain-dns)
- [WEB 동작방식](#web-동작방식)
- [reference](#reference)

## 설마 이건 알겠지?

> 아는 것 같은데 막상 설명해봐라! 하면 못할 수 있으니까 정리를 하자.

### IP Domain DNS

#### IP

Internet Protocol의 약자로, 네트워크상에서 컴퓨터가 다른 컴퓨터와 구별되어 지는 번호이다. 인터넷에 접속할때 컴퓨터 각각에 부여 받는 주소 혹은 전화번호라고 보면 됨.

#### Domain Name

IP주소가 숫자로만 구성되어 있으니 외우기 힘들기 때문에, 이 IP주소를 문자로 표현한 도메인 주소 등장.(ex www.naver.com) 예를 들어, 010-1234-1234 외우고 다니기 힘드니까 핸드폰 전화번호부에 이건 A 이렇게 저장해 두는 것. 우리가 핸드폰에서 A에게 통화하기를 누르면, 실제로 'A'한테 전화거는게 아닌 010-1234-1234에 전화를 거는 것처럼 우리가 www.naver.com하면 해당 IP주소로 변경해서 접속해야 한다. 이렇게 domain name과 IP주소쌍을 한 쌍으로 가지고 있는 데이터 시스템을 **DNS(Domain Name System)** 이라고 부른다.


## WEB 동작방식
<hr/>

![web동작방식](/assets/images/web-operation.png)

<hr/>

### 1. 사용자가 웹 브라우저에 도메인 네임을 입력한다. (주소창에 www.naver.com을 친다)

### 2. 웹 브라우저가 URL을 해석한다.

  - 어떤 프로토콜을 사용하는지 .. 
  (http, telnet, ftp ...)

  - ip 정보가 필요한지 (:다음에 //있으면 필요하다고 판단)

### 3. URL문법에 맞으면, punycode encoding을 url의 host부분에 적용하고 HSTS(HTTP strict Transport Security)목록 로드해서 확인한다.

  * url 문법 안맞으면 그냥 기본 검색엔진으로 넘어감

  * hsts 목록에 있으면 https로 보내고 아니면 http로 보냄

### 4. DNS (Domain Name Server)에 조회한다.

  * Browser에 해당 domain이 cache되어 있나 확인! 되어있으면 굳이 서버 조회 안해도 됨

  * 없으면, 로컬에 저장돼 있는 hosts파일에서 참조할 수 있는 Domain이 있는지 확인한다.

  * 모두 없으면 Network Stack에 구성되어 있는 DNS로 요청을 보낸다. 

  * DNS는 도메인 주소에 해당하는 ip 주소를 알려준다.

### 5. ARP(Address Resolution Protocol)로 대상의 IP와 MAC address를 알아낸다.

### 6. 대상과 TCP 통신을 통해 socket을 연다.

  1) 브라우저가 대상 서버의 IP 주소를 받으면 URL에서 해당 포트 번호(HTTP의 기본값은 80, HTTPS의 기본값은 443)를 가져와서, TCP Socket stream 요청

  2) TCP segment가 만들어지는 Transport Layer(OSI Model Layer 4)로 전달. 대상 포트 header에 추가되고 source port는 시스템에서 동적 포트 범위내에서 임의 지정

  3) TCP segment를 Network Layer(OSI Model Layer 3)로 전달. segment header에 대상 컴퓨터의 IP주소와 현재 컴퓨터의 IP주소가 삽입된packet 구성

  4) packet이 Link Layer(OSI Model Layer 2)로 전달. 시스템의 MAC address와 gateway(local router)의 MAC주소를 포함하는 Frame header 추가 (gateway의 MAC address를 모르는경우 ARP를 이용해 찾아야 한다.)

  5) packet이 ethernet, Wifi, Cellular data network 중 하나로 전송 

    - 광인터넷을 쓸 경우 modem을 통해 광신호로 변경된 후 network node로 직접 전송

  6) packet local subnet router 도착, AS(Autonomous System)경계 router들을 통과

      해당 라우터에선 packet의 IP header에서 target address를 추출하여 적당한 다음 hop으로 routing 

      IP header의 TTL(Time To Live)필드는 통과하는 라우터의 대해 하나씩 감소

      TTL필드가 0이 되거나 현재 router 대기열에 공간이 없으면 네트워크 정체로 packet 삭제

![TCP socket 통신 과정](/assets/images/tcp-operation.jpeg)

### 7. HTTPS인 경우 SSL(TLS) handshake가 추가된다. 

  - TCP socket 통신과정 전에 TLS 통신이 추가된다.

### 8. HTTP 프로토콜로 요청한다.

`
GET / HTTP/1.1
Host: google.com
Connection: close
[other headers]
`

서버로 부터 html 응답을 받는다.

### 9. HTTP서버가 응답한다.

HTTPD(HTTP daemon) 서버는 요청 / 응답을 처리하는 서버이다. 가장 일반적인 HTTPD 서버는 Linux의 경우 Apache 또는 Nginx이고 Windows의 경우 IIS이다.

### 10. 웹 브라우저가 그린다. 

- 서버가 리소스를 브라우저에게 제공하면 브라우저는 렌더링 등을 수행하여 화면을 그린다.

## reference

* https://sophia2730.tistory.com/entry/DNS-%EC%A3%BC%EC%86%8C%EC%B0%BD%EC%97%90-wwwnavercom%EC%9D%84-%EC%B9%98%EB%A9%B4-%EC%9D%BC%EC%96%B4%EB%82%98%EB%8A%94-%EC%9D%BC

* https://github.com/SantonyChoi/what-happens-when-KR#id7