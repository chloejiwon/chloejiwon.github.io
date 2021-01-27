---
title:  "Challenging Common Assumptions in the Unsupervised Learning of Disentangled Representations"
excerpt: "무시무시한 ML 논문 읽기"

categories:
  - MachineLearning
tags:
  - 머신러닝, 논문
last_modified_at: 2021-01-27T17:42:00-05:00

---

> 2019년 Top 10 논문에 뽑힌 Common Assumptions in the unsupervised learning of disentangled representations 논문을 살펴보자. 이번 논문은 제목 그대로 Common Assumption에 대해 도전하는 논문이다. 그래서 이걸 이해하려면 애초에 그럼 내가 Common Assumption이 있을 정도로 배경 지식이 있어야 했기 때문에(...) 배경 지식을 (논문을 이해할 만큼만) 간단히 정리해보았다. 여기서 🥝 로 중간 중간  내 의견, 참견들을 적어두었다. 시끄러우면 살포시 무시해주세요.

# 배경 지식

### representation learning

representation은 어떤 데이터 포인트를 다른 차원의 데이터 포인트로 바꾸는 것을 의미한다.

(x1, x2 ..)  → (z1, z2, ...)

주어진 데이터를 다른 차원에 **표현** 한다는 뜻으로 받아들이면 된다. 그리고 `learning`이니까 그 표현하는 방법을 학습하는 것이다. 주어진 데이터를 다른 데이터 포인트로 mapping하는 모델을 학습한다고 보면 되겠다.

![ML도표](/assets/images/ml-research-1.png)

### Disentanglement

Transforming from an uninterpretable space with entagled features to eigen spaces where features are independent.

*행렬의 고유 벡터(eigen vector)들이 형성하는 부분 공간 = 고유 공간(eigenspace)*

조금 더 풀어 설명하자면, 아래 그림에서 보듯이 x를 만들때 영향을 미치는 독립적인 요인인 z1, z2, .... 가 있다고 치자. 그래서 x가 주어졌을때 그의 표현인 r(x)를 만들때 (← r(x)만들때 z들이 영향을 끼치는건데) 한 요소가 바뀌면 딱 한 representation vector만 바뀌는 걸 의미한다.

다른 representation vector까지 바꿔버리면, z가 entagled되어 있다는 걸 의미한다.

![disentanglement](/assets/images/ml-research-2.png)

### Disentagled Representation

어떤 data(이미지)를 나타내는 latent variable이 여러 개로 분리 되어 각각 다른 데이터(이미지)의 특성에 관한 정보를 담고 있는 것을 의미

### latent variable

that describes original data

잠재 변수. Factor Analysis 를 할 때 많은 수의 변수들을 소수의 몇 개의 잠재된 변수로 찾아내는데, 그 변수가 latent variable.

### Autoencoder

for (Nonlinear) Dimensionality Reduction, Feature extraction, Representation learning

![autoencoder](/assets/images/ml-research-3.png)

### VAE (variational auto encoder)

생성모델을 다루는 기술 중 하나. GAN과 같이 각광받고 있음.

![vae](/assets/images/ml-research-4.png)

### downstream task

실제로, 구체적으로 풀고 싶은 문제(task)

요새는 대부분 pretrain 하고 그 모델에 우리가 풀고 싶어하는 downstream task 를 넣어서 fine tuning을 한다.

------

# Abstract

unsupervised learning of disentagled representation의 key idea :  real-world의 데이터가 소수의 explanatory factor(unsupervised learning 알고리즘으로 만들어지는)로 생성될 수 있다는 것.

여기서 넘어가기 전, prior과 posterior의 개념에 대해 짚고 넘어가자.

- **prior** : z
- **posterior(=likelihood)** : P(x|z)



**prior**는 어떤 중요한 factor(autoencoder 입장에서 보면 latent variable), **posterior**는 prior(=z)들이 주어졌을때, x를 z의 결합으로 표현하는 것이다.

in this paper, 이 분야의 최근 진행 상황을 살펴보고, 일반적인 가정에 challenge하겠다.

# Introduction

In representation learning, real-world observations x는 이렇게 2-step으로 생성된다고 생각한다.

1. multivariate latent random variable z가 P(z) 분포로부터 샘플된다.
2. P(x|z)로부터 x가 샘플된다.

그니까, real-world에서는 우리가 모르는 요인인 z가 이미 있는데, 그 z로부터 x(실세계에 존재하는 고차원의 데이터)를 generate할 수 있다고 생각하는 것이다.

⇒ 즉, 고차원의 데이터 x가 저차원의 데이터 z(semantically meaningful latent variable)로 표현이 된다.

그러면, 여기서 목표와 왜를 한번 더 짚어보자.

- **Representation learning의 목표** : useful transformation r(x)를 찾는 것. 왜? Classifier나 다른 예측 모델을 만들 때, 어떤 x에 대해 유용한 information을 잘 추출하기 위해서.
- **그리고 disentaglement가 여기서 왜 중요하냐?** data에서 informative하고 구분되는 factor를 나눠야 좀 더 잘 생성될 수 있어 보인다.

### 이 논문에서 말하고자 하는 주장들의 summary (사실 이게 전부다...)

- Unsupervised learning of disentangled representation은 learning approach(모델,..)와 dataset에  inductive bias없이는 불가능하다. (이걸 이론적으로 증명할 것이다.)

- 그래서 current approach들과 inductive bias에 대해 조사하고 실험했다.

- 우리가 한 disentaglement_lib 도 배포하고, trained model도 여러개 배포했다.

- 우리는 이렇게 실험 결과를 분석했다.

  (i) 모든 method가

  (ii) random seed와 hyperparameter가 model choice보다 더 중요하다. 즉, reliable한 model이 없다.

  (iii) disentaglement가 downstream tasks에 유용한지 검증할 길이 없다.

- 그래서 further research에게는 followings가 있음 좋겠다.

  (i) inductive bias와 supervision의 역할이 명확해야 한다.: model selection이 Key question으로 남겠지.

  (ii) distenglement의 개념이 실질적으로 얼마나 이득을 가져오는지를 증명되어야 한다.

  (iii) 실험들은 모두 재생산할 수 있는 환경에서 시행되어야한다.

# Impossibility result

어떤 generative model도 unsupervised disentanglement learning이 가능한가? 를 증명해보자.

⇒ 답은 No이고 아래 theorem을 이용할 것이다.

![Theorem](/assets/images/ml-research-5.png)

(증명) P(x|z) 가 generative model이라고 하자. 그리고 z에 대해 완벽하게 disentagled된 representation r(x)를 찾아냈다고 가정하자.

근데 Theorem1에 의해서, equivalent한 generative model인 z_hat = f(z) 가 있을 것이다.

이 z_hat은 근데 z, r(x)와 완전히 entagled되어 있다.

왜냐하면, Theorem 1에서 말했듯이, f의 미분값이 대체로 0이 아니기 때문에 (=즉,  uj가 fi에 영향을 미친다는 뜻 = disentagle되려면 uj는 fi에 영향을 미치면 안됨).

그러니까, z의 한 차원을 변화시키면 z_hat의 모든 차원이 다 영향을 받아서 변화한다. (=entangled) 그리고 p(z) = p(z_hat) 이니까 두 generative model은 x에 대해 같은 marginal distribution을 가진다.

distenglement method는 관측값인 x에만 접근할 수 있기 때문에, 두 생성 모델 중 어떤게 제대로 disentagled된 아이인지 구별 할 수 가 없다. (둘 중 하나는 무조건 entagled되어 있다.)

⇒ f가 infinite family이니까 그 무한대인 entaglement 사이에서 진짜 disentagled된 생성모델을 찾을 수 없다.

→ **즉, inductive bias 와 supervision 없이는 unsupervised disentaglement learning이 불가능하다.**

# experiment design

기본적으로 VAE 를 사용하고, loss 는 VAE loss + regularizer

Gaussian encoder, Bernoulli decoder, latent dimension은 10으로 고정해서 구성

그리고 inductive bias가 필요하다는 걸 보여주기 위해, 여러 hyperparmeter set을 써서 실험을 할 것이다.

# Key experimental results

### 1) Can current methods enforce a uncorrelated aggregated posterior and representation?

질문의 뜻은, 현재 방식들이 aggregated posterior와 representation이 각각 uncorrelate 하게 해주냐?

보통의 method에서, Gaussian Encoder에서 나온 value를 sample해서 사용하는 게 아니라 mean vector를 representation으로 사용한다.

mean vector로 학습 진행했을 때와, 그 분포에서 sample을 얻어 진행했을때의 결과를 살펴보면 아래 표와 같다. 여기서 value는 total correlation이다.

왼쪽(sample)은 regularization strength가 강해질수록(=일반화의 중요도가 강해질수록) value가 감소하고 오른쪽(mean)은 반대다.

🥝 : 여기서 regularization strength ↔ value 사이의 관계를 어떻게 해석하는 거지? 일반화가 잘 되면, 당연히 correlation score도 낮아지는 게 일반적이라고 보는건가?

⇒ 아! 그게 아니라, regularzation strength는 hyper parameter 이니까 hyperparam choice에 따라 correlation score가 달라진다고 보는게 맞다.

![실험결과1](/assets/images/ml-research-6.png)

- **Implications**

현 방식들은 aggregated posterior가 uncorrelate한 방향으로 시행되는건 맞지만, 그렇다고 해서 mean representation의 dimension이 uncorrelate하다는 뜻은 아니다.

### 2) How much do the disentanglement metrics agree?

disentanglement의 확실한 정의가 없기 때문에 disentaglement의 metric score가 얼마나 통일된 결과를 보이는가를 실험해봤다.

아래 figure는 Noisy-dSprites dataset에 대한 각 metric의 correlation을 matrix로 보인것이다.

Modularity정도 빼놓고는 metric들이 서로 엄청 correlate되어 있음을 알 수 있다.

🥝 : 그니까 correlate되어 있다는 거는, 대충 metric score가 통일된 결과를 보여주고 있다는 거겠지? Modularity같은 애 빼놓곤 disentanglement 가 확실히 정의되어 있지 않는데도 불구하고 (= 그러니까 당연히 그걸 측정할 metric도 확실하게 있는게 아닌데) 대부분 비슷한 결과를 보이더라.

![실험결과2](/assets/images/ml-research-7.png)

- **Implications**

Modularity를 제외한 모든 disentaglement metrics가 dataset에 따라 정도는 다르지만, 서로 correlate되어 있다.

### 3) How important are different models and hyperparameters for disentaglement?

우리가 실험한 이런 method들을 사용하는 이유는 Disentanglement를 잘하기 위해 하는 거니까, disentanglement 성능이 model choice, hyperparameter selection, randomness(=random seed)에 얼마나 영향을 많이 받나를 실험해봤다.

![실험결과3](/assets/images/ml-research-8.png)

왼쪽 표를 보면, Model에 따른(hyperparameter, 여기선 regularization strength)을 달리해서 실험한) score 표인데, 같은 모델이어도 엄청 분포가 0.95 상위부터 0.60정도의 하위까지 걸쳐져 있는 것을 볼 수 있음. variance가 크다! hyperparmeter에 엄청 영향을 많이 받는 다는 걸 알 수 있다.

오른쪽 표는 regularization strength를 다르게 해서 한 model에 대해 random seed를 다르게 하여 실험한 결과. 왼쪽보다는 낫지만 random seed에도 score가 영향을 받는 다는 걸 알 수 있다.

score로 따졌을때, good run with a bad hyperparmeter > bad run with a good hyperparmeter

- **Implication**

unsupervised learning의 disentanglement score은 randomness(in the form of random seed)와 hyperparmeter choice(in the form of the regularization strength)에 영향을 막대하게 받는다.

### 4) Are there reliable recipes for model selection?

앞서 hyperparameter와 model choice가 엄청 중요한 것을 알았으니, 그럼 어떻게 어떻게 좋은 hyperparameter를 고르나? 이 연구에서는 (1) good learning model과 (2) regularization strength corresponding to that loss function 이 두가지를 집중해서 살펴본다.

- **General recipes for hyperparameter selection**

![ML도표](/assets/images/ml-research-9.png)

이 표를 Regularization strength가 달라질 때마다 좋은 성능을 내는 model 이 계속 바뀌는 걸 알 수 있다. (...) 일관되게 좋은 성능을 내는 model이 없고, regularization strength를 고를때도 성능을 최대로 끌어올리는 그런 전략이 없다.

- **Model selection based on unsupervised scores**

다른 방식으로 접근해보자. Hyperparmeter를 unsupervised score(ex, reconstruction error, KL divergence between the prior and the approximate posterior, 등등) 를 보고 고르는 것이다. 그러려면 최종 목표는 disentanglement score를 높이는 것이니 disentanglement metric이랑 unsupervised score랑 correlate하고 있음 unsupervised score기반으로 결정을 내려도 되곘지? 하지만 표를 보면, correlation이 거의 없어서 unsupervised score를 쓰기에 적절치 않다는 결론을 내렸다.

![ML도표](/assets/images/ml-research-10.png)

- **Hyperparameter selection based on transfer**

🥝 : 여기까지 오다니...

앞의 두 방법 다 적절치 않아서, 마지막으로 transferring good settings across data sets 이 전략을 취해봤다. 핵심 아이디어는 좋은 hyperparameter setting은 label이 있는 dataset에서 추론되어 새로운 dataset에 적용될 수 있다는 것이다.

우린 이 transfer based approach to hyperparameter selection vs random model selection 비교를 이렇게 했다.

1. 50개 random seed중 한 sample과 random disentanglement metric, random dataset을 뽑는다. 그리고 요것들로 성능을 최대로 끌어올리는 hyperparameter setting을 찾는다.
2. randomly selected model과 우리가 1)에서 고른 hyperparameter setting을 비교한다.
3. transfer strategy가 random model selection보다 좋은 성능을 보일 때를 10000 trial 중 몇 퍼센트인지 확인할 것이다.

**결과** ⇒ same metric, same dataset (+ different random seed)를 고르면, 80.7% 나온다. same metric을 전체 dataset에 transfer한다고 하면 59.9%, metric과 dataset 전체를 transfer하면 54.9%로 떨어진다.

- **Implications**

Unsupervised model selection은 unsolved problem이다. good hyperparameter를 transfer하는 방법은 별로 좋은 방법이 아닌 것 같다. 왜냐하면 good or bad random seed를 구분할 수 있는 방법이 없기 때문이다.

### 5) Are these disentagled representations useful for downstream tasks in terms of the sample complexity of learning?

그러면 진짜로 disentangled representation이 downstream task에 유용하냐?

- **Implications**

여기서 우리는 interpretability 나 fairness 의미를 가진 usefulness를 확인하지 않았기 때문에, 이걸 고려하면 다른 결과를 추론할 수도 있다.

하지만, 여러 different dataset에 disentangeld representation score 와 downstream task performance 의 correlation을 따지면 거의 관계가 없는 걸로 나온다. 그니까, disentanglement때문에 downstream task에 좋은 성능이 나온다고 말할 수 없다.

# Conclusion

이 논문에선 이론적으로 unsupervised learning of disentanged representation이 inductive bias 없이는 불가능하다는 걸 증명했다.

그리고, 대규모 실험 (6 state-of-the-art disentanglement methods와 metric로)을 진행하며 이런 결론을 내렸다.

1. posterior를 factorizing하는 것이 꼭 representation의 차원이 uncorrelated되어있다는걸 암시하지 않는다.
2. Random seed와 hyperparmeter가 model selection보다 영향을 많이 미치고, tuning할땐 supervision이 필요하다.
3. disentanglement가 잘 된다고 해서 downstream task의 학습이 잘된다는 걸 의미하지 않는다.

그래서 앞으로 있을 연구에는 다음의 것들을 기대한다.

(i) inductive bias와 supervision의 역할이 명확해야 한다.: model selection이 Key question으로 남겠지.

(ii) distenglement의 개념이 실질적으로 얼마나 이득을 가져오는지를 증명되어야 한다.

(iii) 실험들은 모두 재생산할 수 있는 환경에서 시행되어야한다.

# reference

[배경 지식]에서 사용된 이미지를 제외한 모든 이미지는https://arxiv.org/pdf/1811.12359.pdf 에서 가져왔다.



[배경 지식]의 첫번째 이미지(도표)를 제외하고 내가 그린 그림이다.