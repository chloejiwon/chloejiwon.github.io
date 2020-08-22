---
title: "WebRTC로 실시간 영상 통화를 하며 Object Detection을 해보자 - 2탄"
excerpt: "webrtc 와 tensorflow js 모델로 object detection해보기"

categories:
  - 웹, 인공지능
tags:
  - tensorflow, javascript
last_modified_at: 2020-08-23T01:11:00-05:00




---



> 프로젝트 진행하면서 처음 접한 webrtc.. 실시간으로 영상 채팅하며tensorflow js 모델과 함께 객체 인식을 하는 기능을 구현해보는데... 👀 
>
> 2탄. 대망의 tensorflow js 모델로 object detection 성공해보자
>
> 왕왕 초보라 틀린 정보가 있을 수 있음을 미리 고지합니다

# 📌 Table of Contents

- [Tensorflow js모델을 심자](#tensorflow-js-모델을-심자)
  - [Tensorflow js](#tensorflow-js)
  - [WebRTC + Tensorflow.JS](#webrtc-+-tensorflow.js)
  - [Tensorflowjs 모듈 설치](#tensorflowjs-모듈-설치 )
- [reference](#reference)



# Tensorflow js 모델을 심자



## Tensorflow js



Tensorflow.js는 자바스크립트 머신러닝 라이브러리다. JS로 ML모델을 개발하고 브라우저 / node.js에서 바로 ML 모델을 사용해볼 수 있다. 🎉 ML관련 웹 서비스에 대한 아이디어가 있다면 바로 활용하기 좋을듯 ! 나름대로 Pretrained된 모델 및 API도 많이 제공해주고 있으니 공식 홈페이지 구경해보면 재밌을 거 같다. 데모도 제공하는데 귀여운 아이디어와 UI로 구성된 페이지들이 많다.



바로 사용할 수 있는 JS 모델을 node.js에 같은 데 넣어서 사용해도 되고(내가 했던 부분) 혹은 직접 모델을 만들었다면 Python Tensorflow모델을 변환하기도 된다는데, 이건 안해봐서 모르겠다. 



(아래는 Demo 페이지들. 구경해보세요 👀)

<https://www.tensorflow.org/js/demos?hl=ko>



## WebRTC + Tensorflow.JS



그럼 나는 뭘 하고 싶냐면, 내가 1탄에서 만들었던 WebRTC기반 화상채팅 시스템에다가 남들 웹 캠 stream말고 내 웹 캠에서 나오는 stream에서 object detection을 해 볼 것이다. 추가로 더 하고자하는 건 object detection해서 나오는 정보를 peer에 data를 실어 보내면(?) peer connection을 맺고 있는 나머지 사람들도 내 정보를 받게끔 하고 싶다. 



나는  TensorFlow.JS 측에서 제공해주고 있는 **객체 감지 모델(CoCo SSD)**을 이용할 것이다. CoCo SSD 가 어떻게 train 됐고 어쩌고 저쩌고는 사실 관심 없고(이미지 관련 머신러닝은 왜케 땡기지가 않는지 모르겠다🤔.. 궁금한 사람은 TensorflowJS 공식 홈페이지나 github repo가면 나름 설명되어 있는것 같으니 참조하세요), 이용해서 내 웹캠에 어떻게 띄우냐가 중요한 부분이니까 빠르게 skip한다. 

   

### Tensorflowjs 모듈 설치 



1탄에서 한거에서 package.json의 dependencies에다가 tensorflow-model cocossd 넣어준다.    



⛔️ 요기서 잠깐 스탑!!   



 **@tensorflow/tfjs** 요거 버전 원래 2.0.1 사용하니까, 화면 띄울때 (=React실행되면서..) 텐서플로우 라이브러리가 import되면서 페이지를 몇 번씩 불러오는(=compDidMount 가 두번 이상 불려!!) 거가 아니겠는가 🤬 그래도 기능은 돌아가야 하는데 문제는 webRTC로 인해서 p2p connection 맺어야하니까 자꾸 시그널을 몇번씩 보내는거다.  아주 성가신... 하여튼, 버전을 다시 1.7.4로 하니까 이런 일은 없어졌다. 별 이슈 없다면 1.7.4를 사용하시길~ 

```javascript
  "dependencies": {
    "@tensorflow-models/coco-ssd": "^2.1.0",
    "@tensorflow/tfjs": "^1.7.4",
```

그 담에 우리 화면 띄워줄 js 파일에 import 해준다.

```javascript
import * as cocoSsd from '@tensorflow-models/coco-ssd'
import '@tensorflow/tfjs'
```



이렇게 dependency 추가해주고 npm install 한번 해주면 우리가 사용할 Tensorflow library는 import 완료.    



## 모델 활용 

이제 내 웹캠 화면을 하나씩 뽑아서 object detect를 해보자. 나는 promise를 사용했음. CompDidMount()내에서 Promise로 내 `navigator.mediaDevices.GetUserMedia(this.streamConstraints)`가지고 Model를 load해서 처리하는 함수들을 만들었다. 

모델 load는 이렇게 하면 되고, 

```javascript
const loadlModelPromise = cocoSsd.load()
```

Promise를 사용해서 로드한 cocoSSD 와 mediaDevices의 localVideo stream을 detect하는 함수에 넘겼다. 

```javascript
    // resolve all the Promises
    Promise.all([loadlModelPromise, webcamPromise])
      .then(values => {
        detectFromVideoFrame(values[0], this.localVideo.current)
      })
      .catch(error => {
        console.error(error)
      })
```

그리고 실제 모델 가지고 화면으로 predict하는 부분은 여기! 

알다시피... requestAnimationFrame() 이게 1 frame마다 불려서 너무 성능이 구려지기에 120 frame마다 돌아가게 했다. 여기다 timing넣는 방법은 못찾고 구글링하니 다들 저렇게 쓴다 그러길래 그냥 나도 따라 씀.

```javascript
    const detectFromVideoFrame = (model, video) => {
      count = (count + 1) % 120
      if (count % 120 === 0) {
        model.detect(video).then(predictions => {
          showDetections(predictions)
        }, (error) => {
          console.log('Couldn\'t start the webcam')
          console.error(error)
        })
      }
      requestAnimationFrame(() => {
        detectFromVideoFrame(model, video)
      })
    }
```

**model.detect(video)**하면 predictions 값을 return한다. 어떤 형식 이냐면 아래와 같다. bbox의 화면의 위치들이 나오고, class로 predict한 object의 분류?값이 나온다. 참고로, 얼굴이 있어야 사람으로 잡는 것 같고(당연한가?🙂) 기본 제공되는 모델이다 보니 성능이 미친듯이 좋지는 않다.

```json
[{
  bbox: [x, y, width, height],
  class: "person",
  score: 0.8380282521247864
}, {
  bbox: [x, y, width, height],
  class: "kite",
  score: 0.74644153267145157
}]
```



그리고 나는 model.detect해서 나오는 predictions 값을 확인하는 로직을 세워서 다른 peer에게 전송해야 되는 값이라면, peer에 data를 실어서 보내줬다. 내가 사용한 simple peer에선 `peer.send(JSON.stringify(data))` 해주면 되더라! 

받는 peer 쪽에서는 'data'로 받아주면 된다.

 `peer.on('data', data => { 어쩌고저쩌고 `



어떤 사람은 네모 박스 만들어서 Video 위 Canvas를 띄워주기도 하는데 나는 tutorial따라하다가 실제로는 안쓰긴 했다. 후후.   



어쨌든 결론은 1:N 화상채팅은 WebRTC로 구현 (1탄에선 N:N이긴 했으나... 1:N은 p2p connection을 1 - N (N끼리는 안하고) 으로만 해주면 되는 심플한 일.. ), 그다음 각 local video를 object detection해서 몇가지 특정 값은 peer에 보내주는 것 까지 구현 성공했다. 자기 webcam stream을 object detection하는 튜토리얼은 많다! 나는 여기 참고했다. 전체 구조랑 다 배웠다;  

* <https://nanonets.com/blog/object-detection-tensorflow-js/>

우리 팀이 프로젝트에서 한거는 요거! frontend랑, signaling-server참고하면 된다. 🤟🏼

* <https://github.com/CheatingBusters>   

#  reference 

* <https://github.com/tensorflow/tfjs-models/tree/master/coco-ssd>
* <https://www.tensorflow.org/js/models?hl=ko>
* <https://webrtchacks.com/webrtc-cv-tensorflow/>
* <https://nanonets.com/blog/object-detection-tensorflow-js/>

