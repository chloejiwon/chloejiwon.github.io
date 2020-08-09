---
title: "WebRTC로 실시간 영상 통화를 하며 Object Detection을 해보자 - 1탄"
excerpt: "webrtc 와 tensorflow js 모델로 object detection해보기"

categories:
  - 웹, 인공지능
tags:
  - tensorflow, javascript
last_modified_at: 2020-07-26T01:11:00-05:00

---



> 프로젝트 진행하면서 처음 접한 webrtc.. 실시간으로 영상 채팅하며tensorflow js 모델과 함께 객체 인식을 하는 기능을 구현해보는데... 👀 
>
> 왕왕 초보라 틀린 정보가 있을 수 있음을 미리 고지합니다

# 📌 Table of Contents

- [WebRTC가 뭐냐?](#🚦webrtc가-뭐냐)
  - [webrtc 동작방식](#webrtc-동작방식)
- [WebRTC구현하기](#webrtc-구현하기)
  - [N:N영상통화](#N:N영상통화)
- [Tensorflow js모델을 심자](#tensorflow-js모델을-심자)
- [reference](#reference)



# 🚦 webRTC가 뭐냐?

WebRTC는 web application간에 실시간 통신 기능을 제공하는 기술. 특히 마이크나 스피커, 영상같은 데이터를 실시간으로 peer connection으로 별도 서버 없이 가능하게 해준다.  (심지어 screen sharing도 가능하다고 함 ) 그럼 간단한 동작 방식을 살펴보자! 

## webRTC 동작 방식

![webrtc-topology](https://webrtc-security.github.io/images/diagram_1_en.png)

아주아주 간단한 webrtc topology는 위와 같다. Alice와 Bob이 각자 컴퓨터에서 browser를 키면, signaling server에 signal을 전송한다. Signaling은 RTCPeerConnection들이 적절하게 데이터를 교환할 수 있게 처리하는 복잡한 과정이다. Signaling server는 피어와 피어가 서로를 찾을 수 있도록 도와주는 서버. Signaling 단계에선 서로 Connection을 맺기 이전 아래와 같은 정보를 교환하도록 해줍니다.   



1. Network 정보 교환
   1. P2P 통신을 위해선 연결을 위한 각 client가 서로의 IP와 UDP port,  따위의 것을 알고 있어야 한다. WebRTC communication이 동작하기 전에 이 과정이 미리 되어 있어야 하는데, 이때 쓰이는 서버가 **STUN** 서버이다.  STUN서버로 인해 서로 각client가 자기 IP를 알게 된다.  
2. Media Capability 교환
3. Session Control Message 교환



결국 시그널링을 통해 정보를 주고 받고 P2P connection을 맺는다. 그리고 그 후부턴 각 peer 로 각자의 media, data를 전송하면 된다. 👏🏻



  

#  🎓 webRTC 구현하기 

프로젝트에서 사용한 기본적인 뼈대 코드를 webRTC 엄청 고수로 추정되는 유투버가 제공하는 tutorial 로 구현한 부분을 공유해본다. simple peer를 이용하여 아주 간단한 예제를 학습해봤다. 

코드 전문은 여기서 확인 가능하다. 😎 (여기서 signaling server는 server.js)

https://github.com/chloejiwon/coronabusters/tree/webrtc_basic_tutorial



## N:N 영상통화



동작 Flow부터 먼저 확인하자. 

1. client A가 브라우저에 접속하여, signaling server에 connection을 알린다. 
2. clinet B가 브라우저에 접속하여 signaling server에 알린다. 엇 근데 기다리는 client A가 있네? signaling server는 A의 socket 정보를 B에 보낸다. 
3. A의 정보를 받은 B는 A에 대해  createPeer 을 한다. 그리고 B의 peer signal를 signaling server에 전송한다. 
4. signaling server에서는 B의 signal을 A에게 전송한다. A또한 peer을 만들고, A의 signal을 signaling server에 전송하고, peer에도 signal한다. 
5. B가 signaling server에서 정보 받으면, A에게 signal한다. (이제 P2P연결이 맺어진 것. 각 peer로 data, media stream을 주고받을 수 있다.)



내가 github에 구현해놓은 부분은 N:N 다대다 통신이다. 당연히 p2p 니까, 얘네는 모두가 모두랑 연결되어 있으면 되는거다. Client 한 명이 방에 들어오면 방에 있는 모든 친구들의 socket id 리스트를 보내준다. 그럼 새로 방에 들어온 client가 각 친구들에게 signal을 보내 peer connection을 맺는다. 모두가 모두랑 연결되어 있으므로 N명이 있다면 전체 connection의 갯수는 N*(N-1)/2 이다. (친구를 그래프의 노드라고 생각하면 연결할 수 있는 모든 간선의 수) 여기까지 하면 아주 간단하게 친구의 stream을 얻어와 서로 데이터 공유할 수 있다. 🎉

root 폴더에서 아래 명렁어로 signaling server를 띄우고, (8000 포트 아무도 쓰지 않아야 함, 아님 설정을 바꿔주든지..?)

```
$node server.js
```

client/ 폴더에서는 view를 위해 아래 command 입력한다. (이건 3000번 포트 사용한다)

```
$npm run-script start
```



혹시 무슨 tensorflow coco-ssd 못 불러온다는 에러가 뜨면 아래 명령어 입력한다.

왜 package.json에 있는데 install 따로 해줘야할까..? 😞

```
$ npm install @tensorflow-models/coco-ssd
$ npm install @tensorflow/tfjs
```

하여튼 이렇게 하면 `http://location:3000`에 접속하면 create room화면이 뜨고, 누르면 자기 얼굴이 턱 등장. 

이제 나는 자기 비디오에 Tensorflow js 모델을 통해 object detection을 하고자 하는데 이거는 다음 편에 이어서 마무리하도록 하겠다. 🤟🏼

# 🎉 reference 

* http://jaynewho.com/post/36
* http://webrtc.org
* https://www.youtube.com/watch?v=R1sfHPwEH7A

