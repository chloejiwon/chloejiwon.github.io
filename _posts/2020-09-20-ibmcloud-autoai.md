---
layout: post
title: "IBM Cloud - auto AI 실습"
excerpt: "autoAI 를 사용하여 최적의 모델 찾기"

categories:
  - ibmcloud
tags:
  - autoML
last_modified_at: 2020-09-20T01:11:00-05:00

---

> Auto AI 라고 하는 IBM Cloud watson이 선보이는 AutoML 서비스의 개념에 대해 알아보고, 사용 해서 배포까지 해보도록 합니다. 😎 전체적인 모든 내용은 [아주 친절한 유투브](#[]()) 를 참고했습니다.

# 준비물

- IBM Cloud 계정
- Dataset

ML이니 Dataset이 필요하겠죠? 무료인 kaggle에서 제공된 dataset을 이용해봅니다. 아래 링크에서 insurance.csv 파일을 이용할 겁니다. 여러 feature를 사용해서 보험 비용을 예측하는 Data입니다.

[Insurance Premium Prediction](https://www.kaggle.com/noordeen/insurance-premium-prediction)

# Demo Project

## Auto AI Experiment

일단 차근차근 또 따라가 봅시다.

IBM watson studio를 하나 만들고(이전 포스팅에 있어요!), New Project > empty project를 생성합니다.

<img src="../assets/images/autoai-1.png" alt="autoai-1" style="zoom: 50%;" />



아까 kaggle 사이트에서 다운받은 Data를 업로드합니다.

별거 없습니다. Add to project > Data선택해서 다운받은 insurance.csv 파일 업로드 해주면 돼요.

<img src="../assets/images/autoai-2.png" alt="autoai-2" style="zoom:50%;" />

그리고 우린 이제 본격적으로 AutoAI experiment를 해볼겁니다. Add to Project에서 Auto AI experiment를 클릭해봅니다.

<img src="../assets/images/autoai-3.png" alt="autoai-2" style="zoom:50%;" />

그러면 이렇게 뭐라 뭐라 뜹니다.

Watson Machine Learning Instance랑 연결해줘야 하는 모양이니 저 파란색 링크를 타고 들어가서 New Service > Machine Learning 서비스를 만들고 associate해주면 됩니다.  만들고 Reload를 누르세요.

<img src="../assets/images/autoai-4.png" alt="autoai-2" style="zoom:50%;" />

그러면 이런 페이지가 뜹니다. 🥳Data에는 Select from project눌러서 우리가 아까 올렸던 insurance data를 선택하세요.

<img src="../assets/images/autoai-5.png" alt="autoai-5" style="zoom:50%;" />

그러고 나면 요런 페이지가 뜨면서 configure해달라고 나옵니다. 여기가 중요합니다!

**What do you want to predict?**

우리 Dataset은 label되어 있기 때문에 supervised learning이고, correct answer이 있기 때문에 우리가 target으로 삼는 column명을 잘 선택 해줘야 합니다.

<img src="../assets/images/autoai-6.png" alt="autoai-6" style="zoom:50%;" />

target을 expenses로 설정하고 나면 제일 기본적인 머신러닝 모델을 이렇게 보여주는데, experiment setting에서 바꿀 수 있어요. 어떤 feature를 넣고 뺄건지도 정해서 experiment해볼 수 있고요. (좋다..🤭) 물론 metric도 정할 수 있습니다.

![autoai-7](../assets/images/autoai-7.png)

![autoai-8](../assets/images/autoai-8.png)

Regression 모델 중에서 이러한 머신러닝 알고리즘을 다 돌려보고 이중 제일 좋은 걸 찾을 수 있다는 거에요. 다 해봅시다.

![autoai-9](../assets/images/autoai-9.png)

선택하고 싶은 대로 선택하고 Run Experiment를 클릭합니다.

🥺진짜 너무 세상 좋네요. 한 5분 ~ 10분 정도 걸립니다.

하고 있는 중간 화면을 캡쳐 해봤어요. 아마 3-fold로 training data를 만들어서 사용한다는 거겠죠?

![autoai-10](../assets/images/autoai-10.png)

좀 더 지나니 pipeline이 만들어진 걸 알 수 있습니다. 아니 이쯤 되니까.. 내가 왜 ML 알고리즘을 공부해야 하는지 생각이 드네요. 😂머신러닝이 알아서 제일 좋은 걸로 골라주는 세상에..

![autoai-11](../assets/images/autoai-11.png)

오우 👀 밑에 리더보드를 보면 실시간으로 제일 좋은 결과를 내는 ML 알고리즘이 막 랭킹되고 있어요. 실시간으로 막 생겨나는 중입니다. 지금 1위는 Gradient Boosting regression이네요.

![autoai-12](../assets/images/autoai-12.png)

오 끝났네요. 🎉

지금 보면 1등이 바꼈어요. 마지막 8번 Pipeline의 Random Forset Regressor이 가장 적은 RSME를 기록했네요. 그럼 제일 성능이 좋았던 이 모델을 deploy해볼까요? 일단 deploy할 모델을 프로젝트에 save 해둡니다.

![autoai-13](../assets/images/autoai-13.png)

그러면 우리 프로젝트에 Model 이 생기며 이런 화면이 뜰거에요. Deploy를 해야 하는데 Deployment Space가 없네요.

![autoai-14](../assets/images/autoai-14.png)

메뉴에서 Create Deployment Space를 찾아 생성해줍니다.

![autoai-15](../assets/images/autoai-15.png)

그리고 Deployment에 우리가 만든 ML instance를 연결 시켜줘야 합니다.

![autoai-16](../assets/images/autoai-16.png)

그리고 다시 모델 페이지로 돌아가서 Promote to Deploy Space를 눌러 연결 해줍니다.

![autoai-17](../assets/images/autoai-17.png)

후에 Deployment Space에 들어가 연결되어 있는 모델을 또 Create a deployment를 눌러 줍니다. Online으로 한번 배포를 해볼게요.

![autoai-18](../assets/images/autoai-18.png)

Hooray~ 제가 만든 모델(?)이 (AutoAI가 만든 모델..) 배포가 되었습니다.

![autoai-19](../assets/images/autoai-19.png)

들어가면 API reference정보도 있고(→나중에 web app으로 만들 수도 있는!), Test할 수도 있게 되어 있네요. 간단하게 테스트를 해보겠습니다.

![autoai-20](../assets/images/autoai-20.png)

오호 요런 결과가 나왔네요. 6246 달러 정도?

![autoai-21](../assets/images/autoai-21.png)

# 마무리

다음에는 Deploy 된 모델 api 를 활용하여 간단한 flask 기반 web app을 만들어 봐야겠어요. 😎그리고 내가 본능적으로(?) 선택한 모델과 auto AI 가 선택한 모델이 같은 지 보는 식의 테스트도 재밌을 것 같네요.