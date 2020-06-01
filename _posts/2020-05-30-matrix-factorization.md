---
title: "추천 시스템 - Matrix Factorization 기초 원리와 구현 방법"
excerpt: "Python을 이용한 matrix factorization 구현"

categories:
  - recommend system
tags:
  - recommend, algorithm
last_modified_at: 2020-05-30T01:11:00-05:00
---

# 🗒 Table of Contents

- [Matrix Factorization 무어냐?](#matrix-factorization-이란)
- [Matrix Factorization의 기본 개념](#matrix-factorization의-기본개념)
  - [matrix factorization의 동작방식](#matrix-factorization의-동작방식)
- [Matrix Factorization 구현](#marix-factorization-구현)
- [reference](#reference)

# Matrix Factorization 이란?

# Matrix Factorization의 기본개념

> Matrix Factorization을 이해하기 위하여 필요한 개념들을 정리해보자. 오랜만에 선형대수학 다루니까 굉장히 생소(?)하면서 재밌다. 🤪 기하학적 의미, 데이터적으로 생각했을때... 이렇게 여러 관점에서 생각하니까 의외로 이해가 좀 쉽다.

## PCA (Principal Component Analysis)

PCA는 데이터의 구조를 **잘** 살려주면서 차원 감소를 할 수 있게 해주는 방법이다. 차원 감소를 위해선, 선형대수학적으로 **정사영**이라는 방법을 이용한다. 그럼 우리는 어떤 벡터에 정사영 해야 데이터 구조를 **잘** 살렸다고 소문이 날까? 🧐

![2D에서의PCA](https://t1.daumcdn.net/cfile/tistory/25388D40527C43DB0B)

위 xy축에 있는 데이터들의 분포를 보자. 저 검정색이 데이터들을 모아놓은 거라고 한다면, 이 데이터의 분포 특성을 2개의 벡터로 가장 잘 설명할 수 있는 방법은 e1, e2 두 개의 벡터로 데이터 분포를 설명하는 것이다. e1의 방향과 크기 & e2의 방향과 크기를 알면 데이터 분포가 어떤 형태 인지 가장 단순하고 효과적으로 파악 할 수 있다. (데이터 하나 하나하 분석하는 게 아니라, 하나의 분포를 이룰때를 분석하는 것). 여기서 e1과 e2는 `eigenvector`이다. (2차원이니까 eigenvector 2개) 아무리 데이터를 늘리고 (=선형변환해도) 크기에만 영향받지 방향에는 영향을 받지 않는 벡터. 우리는 그런 벡터들에 정사영 시켜 차원을 축소 할 수 있다. 여기서 e1, e2에 해당하는 벡터가 분포의 주축이다. (e1과 e2는 서로 수직한다) 그럼 이 둘 중 뭐에 정사영 시켜야 데이터 구조를 잘 살렸다고 말할 수 있을까? 선형대수학적으로 말하면, 정사영 시킨 데이터들의 퍼짐 정도(=variance)가 크면 클수록 데이터를 잘 살린 거다. 데이터적(?)으로 말하면, 그 데이터들이 각각의 특성을 정사영되더라도 잘 보여줄 수 있어야 구조를 잘 살리며 축소한 것이 되는 것이고. <b>주축</b>은 그 방향으로 데이터들의 분산(=variance)이 가장 큰 방향벡터를 의미하므로 여기서는 e1에 해당한다. 즉 e1에 데이터를 정사영시켜 1차원의 데이터를 만들었을 때 우리는 데이터 구조를 아주 잘 살리며 차원을 감소했다고 말할 수 있다.
<br>
PCA는 주성분을 분석하는건데, 그럼 주성분이 뭐야?! <b>주성분(Principle Component, Score)</b>은 사영변환된 후의 변수이다. 그리고 주성분은 원래 변수들의 선형 결합으로 이루어졌다.

<del>
여기서 우리가 주목해야하는 사실은 다음과 같다.
<br>
주성분의 linear combination으로 original matrix를 표현할 수 있다는 것. 즉, 모든 사람의 얼굴은 각 특징들을 조합해서 만들 수 있다는 것 (좀 무섭다😂)
</del>

### Latent Factors

PCA의 목적은 typical vector을 찾는것이다. 각 typical vector은 데이터에 해당하는 한 가지 특징을 가지고 있다. 예를 들어, 한 Typical guy는 **늙은**사람, 한 typical guy는 **안경 쓴**사람 등등...그리고 이러한 "늙었다" 거나 "안경을 쓴다" 는 특징을 **Latent Factor**라고 부른다. 즉, 각 주성분은 특정한 하나의 latent factor를 가지고 있다.

## SVD (Singular Value Decomposition)

SVD는 임의의 M x N 차원의 행렬에 대해서 행렬의 곱으로 행렬을 분해하는 방법 중 하나이다. SVD는 R을 입력 받아, 다음과 같은 M, Σ(대각행렬), U를 반환하는 알고리즘이다.

![SVD공식](https://t1.daumcdn.net/cfile/tistory/9909C6465B125C9E24)

시그마는 대각행렬이고, 시그마를 제외하고 얘기해도 크게 문제가 없다.

![SVD공식2](https://t1.daumcdn.net/cfile/tistory/99D5E34A5B1261A81F)

![rui공식](https://t1.daumcdn.net/cfile/tistory/99F01A445B12622833)

rui는 user가 item에 가지는 선호도라고 하자. pu는 user에 대한 정보, qi는 item에 대한 정보다.

# Matrix Factorization 구현

그럼 주어진 matrix R이 sparse하다면 빈 값을 어떻게 추정하여 구할 수 있을까? (= 추천시스템)
여러가지 방법이 있겠지만, Gradient Discent방식을 사용하여 구현해보자. (딥러닝에서 많이 쓰이는 방식) 랜덤한 값으로 p와 q를 셋팅하고 training하면서 RSME를 최소값으로 만드는 값을 찾는다. 아는 rui에 대해서 제일 좋은 p와 q를 찾아서, 나머지 빈 값을 계산하도록 한다.

# Reference

- https://worthpreading.tistory.com/57

- http://nicolas-hug.com/blog/matrix_facto_4

- https://darkpgmr.tistory.com/110

- http://nicolas-hug.com/blog/matrix_facto_1

- https://www.youtube.com/watch?v=jNwf-JUGWgg
