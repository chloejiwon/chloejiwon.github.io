---
layout: post
title:  "BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding"
excerpt: "무시무시한 ML 논문 읽기"
comments: true

categories:
  - MachineLearning
tags:
  - Machine Learning
last_modified_at: 2021-02-03T17:42:00-05:00
---

> 이번엔 2019년도에 publish된 논문 BERT를 읽어보았다. 아무래도 attention 에 대한 지식이 거의 전무하다보니, 피상적으론 이해했으나 깊숙한 메커니즘은 이해하지 못한 것 같다. (논문이 쉽게 쓰여있어서 망정이지..) 하여튼, 다음 논문으로 attention is all you need를 읽어서 다시 싱크를 맞춰봐야겠다.

## Abstract

BERT : New language representation model, **B**idirectional **E**ncoder **R**epresentation from **T**ransformers

다른 Language representation model과는 달리, BERT는 모든 layer에서 unlabeled text를 가지고 left, right 양 옆 context를 공동으로 조절하며 deep bidirectional representation을 pre-train한다.

따라서, pre-trained BERT 모델은 추가로 output layer 하나만 더해서 fine-tune하면 다양한 task에 다 state-of-the-art 성능을 낼 수 있다.

## Introduction

Language model pre-training은 많은 NLP task에 효과적이다.

pre-trained language representation을 downstream task(본 문제)에 사용하는 방법은 두 가지가 있다.

<img src="../assets/images/ml-research-2-1.png" alt="ml-1" style="zoom:50%;" />

- **feature-based**

  `ELMo` - pre-trained representation을 additional feature로 포함시키는 task-specific architecture 사용

- **fine-tuning**

  `GPT`(Generative Pre-trained Transformer) - pre-trained parameter를 전부 fine-tuning 시킴

두 방법 모두 pre-training때 같은 objective function을 가지고 학습시키고, 무엇보다 **`unidirectional language model`** 을 general language representation을 학습할때 사용함.

⇒ pre-trained representation 사용의 효과(특히 fine-tuning approach에서)를 감소시킴. 모델이 unidirectional이기 때문에 pre-training할때 사용될 모델의 architecture의 선택의 폭을 좁힌다.

ex) GPT 의 경우, 모든 token이 그 전 token 에만 영향받을 수 있는 left-to-right architecture를 사용했다. question-answering task 와 같이 양 방향에서의 context 가 중요한 문제에서 좋지 못하다.

**In this paper, fine-tuning based approach를 좀 더 발전시키는 BERT를 제안한다.**

unidirectional하기 때문에 발생하는 제한을 `Masked Language Model (MLM)` pre-training objective 를 설정함으로써 극복한다.

- **Masked Language Model**

  input에서 어떤 token들을 랜덤하게 mask하고, objective는 이 masked word에서 오직 context로만 original vocabulary id를 예측하는 것이다.

  ⇒ left-right model과 달리 left, right context를 fuse할 수 있도록 해줌

  ⇒ 따라서, deep bidirectional Transformer를 pre-train시킬 수 있도록 도와준다

MLM 에 추가로, next sentence prediction task를 사용해서 text-pair representation을 pre-train 시킨다.

### Contributions

- language representation에서 bidirectional pre-training의 중요성을 보여준다
- pre-trained representation이 엔지니어링 수고가 많이 들어가는 task-specfic architecture의 필요를 줄여준다는 걸 보여준다 (왜냐면 BERT로 Fine-tuning만 하면 sentence-level & token-level tasks에 SOTA 가능하니까)
- BERT는 11 NLP task에서 S-O-T-A 기록

## Related Work

### 1. Unsupervised Feature-based Approaches

Pre-trained word embeddings, sentence embeddings, paragraph embeddings

**ELMo** - left-to-right 과 right-to-left 모델에서 *context-sensitive* feature를 추출한다. 그래서 contextual representation of each token은 left-to-right & right-to-left representation을 합친 것. (contextual word embedding + 기존에 있던 task-specific architecture를 잘 통합한 알고리즘) 성능이 좋다.

### 2. Unsupervised Fine-tuning Approaches

처음엔 feature-based 접근법과 마찬가지로 unlabeled text로부터 word embedding parameter를 pre-train시켰다.

pre-trained from unlabeled text —> fine-tuned for a supervised downstream task

이 방식의 좋은 점은 바닥부터 학습시킬 파라미터의 수가 얼마 없다는 것이다.

OpenAI GPT는 sentence-level task에서 많이 state-of-the-art 결과를 냈다.

### 3. Transfer Learning from Supervised Data

large dataset을 가지고 있는 supervised task로부터 효율적으로 transfer하는 방식을 보여주는 연구들도 있다. (Ex, machine translation, natural language inference)

Computer vision에서도 image net가지고 pre-trained된 모델을 fine-tune해서 사용하기도 한다. (효율적이니까)

## BERT

1. pre-training 2) fine-tuning 이 두가지 스텝을 거쳐 모델을 사용한다.
2. `pre-training` ⇒ 다양한 pre-training task의 unlabeled data로 모델을 학습시킨다.
3. `fine-tuning` ⇒ pre-trained parameter값으로 초기화시키고, downstream task의 labeled data를 사용해서 fine-tuning한다.

downstream task 마다 다른 fine-tuned model을 가진다. 여기서는 question-answering task를 가지고 설명할 것이다.

BERT의 특징 중 하나는 다양한 task에 통일된 하나의 architecture를 사용한다는 것

pre-trained architecture 과 final downstream architecture간의 차이가 거의 없다

- **Model Architecture**

  multi-layer bidirectional Transformer encoder (decoder는 사용하지 않음)

  layer수(Transformer Block) : L

  hidden size : H

  self-attention head : A

- **Input/Output Representation**

  <img src="../assets/images/ml-research-2-2.png" alt="ml-1" />

  BERT는 위 그림과 같이 세 가지 embedding 값을 합쳐서 input으로 사용한다.

  input은 sentence 하나나, pair of sentence가 올거고, 이걸 one token sentence로 표현해서 사용

  여기서 ***sentence***는 진짜 문장이 아니라 BERT의 입력값으로 사용되는 token sequence(한 문장 o 두 문장을 합쳐놓은거)를 의미한다

  WordPiece embedding with 30000 token vocabulary 을 사용했다.

  문장의 첫번째 token은 무조건 [CLS] (문장의 시작을 알리기 위해), 두 문장을 합쳐놓은 input에서 두 문장을 문맥적으로 구분하기 위해 중간에 [SEP] token을 넣고, 학습된 embedding을 각 token에 더한다. (각 token이 sentence A 에 속하는지, B에 속하는지 알려주는)

  <img src="../assets/images/ml-research-2-3.png" alt="ml-1" />

### Pre-training

2개의 unsupervised task로 pre-train시킬 것이다.

- **Task#1 : Masked LM(=Cloze task)**

  <img src="../assets/images/ml-research-2-4.png" alt="ml-1" />

  input token을 random하게 15%(여기서는)를 mask하고, 전체 input을 예측하는게 아니라 mask된 token만을 예측하는 방식으로 train한다.

  단점은 [MASK] token이 fine-tuning때는 없으니까 pre-training과 fine-tuning간의 mismatch를 발생시킨다는 것이다.

  ⇒ 이걸 해결하기 위해서, "masked" word를 그냥 [MASK] token으로 무조건 교체하는게 아니라, training data를 generate할 때 random 으로 mask시킬 token들이 선택되면,

  (i) 80% of time만 [MASK] Token으로 교체한다

  (ii) 10%는 random token으로 바꾼다

  (iii) 10%는 그냥 그대로 둔다.

- **Task#2 : Next Sentence Prediction (NSP)**

  Question Answering(QA) 나 Natural Language Inference(NLI)같은 downstream task는 두 sentence의 관계를 이해하는 것에서 출발한다. (language modeling으로 바로 보이지는 않음)

  관계를 알기 위해서,  monolingual corpus에서 생성된 *binarized* next sentence prediction task로 pre-train한다.

  좀 더 자세히 어떻게 pre-train하냐면, sentence A, B가 있을때 (i) 50%는 B가 A다음 온다(labeled as IsNext). (ii) 50%는 random sentence(labeled as NotNext)

  Prior work랑 다른 점은 prior work는 sentence embedding만 downstream task에 transfer되는데, BERT는 모든 parameter를 전부 transfer한다.

- **pre-training data**

  BooksCorpus(800M words) & English Wikipedia (2500M words) 사용

### Fine-tuning

보통 text pair에 대한 모델을 fine-tuning하는 방식은 1) 독립적으로 text pair를 encoding하고 2) bidirectional cross attention에 적용하다.

하지만 BERT는 이 두 단계를 self-attention mechanism을 통해 하나로 합쳤다.

pre-training에 비해 굉장히 저렴하다. single Cloud TPU 사용해서 최대 1시간, GPU로는 몇시간 정도만 학습하면 Fine-tuning된다.

## Ablation Studies

### Effect of Pre-training Tasks

<img src="../assets/images/ml-research-2-5.png" alt="ml-1" />

- No NSP ⇒ NSP의 중요성
- LTR(MLM대신에 Left-to-Right LM, left-context-only model) & No NSP ⇒ Bidirectional Representation의 중요성

it would be possible to train 각 LTR, RTL model and each token을 저 두 모델의 concatenation으로 사용 (ELMo처럼)

근데, 이렇게 하면 **(a)** single bidirectional-model보다 두배 더 비싸고, **(b)** QA같은 task에 non-intuitive하다. (RTL model은 question에 답을 못하니까) 그리고 **(c)** 모든 layer에서 left, right context밖에 사용 못하니까 deep bidirectional model보다 별로 좋지 않다.

### Effect of Model Size

fine-tuning task accuracy를 통해서 model size의 영향을 조사했다.

BERT model의 레이어 수, hidden units, attention heads를 다르게 해서 실험했다. (다른 hyperparameter하고 training procedure는 똑같이 두고)

<img src="../assets/images/ml-research-2-6.png" alt="ml-1" />

- larger model이 accuracy 가 월등히 좋았다.
- BERT(base)는 110M parameter, BERT(large) 는 340M parameter
- LM perlexity*를 비교해보면 모델 사이즈가 크면 클 수록 large-scale task(machine translation, language modeling)에 좋다. 근데 small task도 모델 사이즈가 클수록 크게 성능이 좋아진다.

**perplexity* : 언어 모델을 평가하기 위한 내부 평가 지표. 낮을 수록 언어 모델의 성능이 좋다는걸 말한다.

### Feature-based Approach with Bert

지금까지 모든 결과는 fine-tuning approach를 사용해서 실험된 결과이다. (simple classification layer가 pre-trained model에 더해지고, 모든 pre-trained model parameter가 적용되어서 downstream task에 대해 fine-tune)

Feature-based approach(pre-trained model에서 fixed feature가 추출되는)도 장점이 있다.

(1) 모든 task가 Transformer encoder 구조로 잘 표현되는게 아니라서, task-specific architecture가 필요할 수 있다.

(2) 비싼 training data의 representation을 계산을 먼저 하고, 이 representation을 사용한 cheaper model에서 여러 실험을 하는게 computational cost입장에서 훨씬 이득이다.

<img src="../assets/images/ml-research-2-7.png" alt="ml-1" />

결과적으로, BERT가 fine tuning과 feature based approach 모두에서 성능이 좋다.

## Conclusion

우리 연구의 가장 큰 성취는 NLP task에 잘 적용될 수 있는 pre-trained model를 deep bidirectional architecture로 이루었다는 것에 있다.
