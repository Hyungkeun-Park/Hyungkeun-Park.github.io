---
title: "[Review] Improved Feature Distillation via Projector Ensemble, NeurIPS, 2022"
excerpt: "FC Layer Ensemble을 이용한 Feature based KD"

toc: true
toc_sticky: true

use_math: true

categories:
    - Review
tags:
    - Review
    - Knowledge Distillation
last_modified_at: 2023-12-11T01:00:00
---

<span style="font-size:200%">지도(Supervised) 학습</span>은 특정 Task(classification, object detection, etc)를 수행하기 위해<br>
1. 데이터를 통한 Feature 학습 부분과(Conv. Layer)
2. 학습된 feature를 조합하여 사용한다고 볼 수 있는 Fully-connected(FC) Layer

로 크게 나누어 볼 수 있다<br>

**Knowledge Distillation에서 FC Layer를 앙상블(Ensemble)할 경우 성능을 증가시킬 수 있다**라는<br>

[Improved Feature Distillation via Projector Ensemble](https://arxiv.org/abs/2210.15274)

위 논문을 한번 간단히 리뷰해보려고 한다

><span style="font-size:150%">**앙상블(Ensemble)**</span><br>
앙상블이란 음악 용어로 2인 이상에 의한 가창이나 연주를 뜻하는 의미로 시작하여<br>
딥러닝에서는 여러 학습된 모델을 모아 성능을 더욱 좋게 만드는 것을 의미한다

그렇다면 앙상블이라는 말에서 알 수 있듯<br>
**FC Layer 여러 개를 한데 묶어 더 좋은 성능을 만들겠다**<br>
라는 것을 논문 제목에 빗대어 확인할 수 있다

# 1. Introduction
저자는 Knowledge Distillation을 크게 다음의 세 가지 종류로 분류하고 있다
1. Logit-based
2. Feature-based
3. Similarity-based

그 중에서도 2번 `Feature-based`방식이 다른 두 가지 방식에 비해 더 좋은 결과들을 보여왔는데<br>
`Feature-based`방식에서 **선생 모델과 학생 모델의 차원을 맞춰주는** 다양한 연구 결과들이 나왔는데<br>
차원을 맞춰주는 dimension projector인 **FC Layer를 동일 차원에 사용해도 성능이 증가한다**는 결과를 얻었다고한다<br>
<center><img width="458" alt="image" src="https://github.com/Hyungkeun-Park/Hyungkeun-Park.github.io/assets/21329629/c7d5e3f8-e043-4b97-8b5d-b45bd7faf3f6"></center>

저자는 Project를 추가하는 것이 선생과 학생 모델에서의 불일치(discrepancy)를 줄이는데 있어서 발생하는<br>
과적합(overfitting)문제를 완화시킨다는 가설을 조사하고 projector 앙상블 모델을 새로 제안한다<br>

# 3. Improved Feature Distillation 
우선 저자는 앞으로 사용할 표기법(notation)을 정리하고 시작한다<br>

$ S={s_1,s_2,\dots,s_i,\dots,s_b } \in R^{d*b}$ 는 학생 모델의 최종단에서의 feature를 뜻한다(d차원, batch size b)

$ T={t_1,t_2,\dots,t_i,\dots,t_b } \in R^{m*b}$ 는 선생 모델의 최종단에서의 feature를 뜻한다(m차원)

그리고 학생, 선생 모델의 차원을 맞춰주기 위한 projector는 $g(s_i)=\sigma(Ws_i)$로<br>
**Projector를 학생 모델에 붙이는 것이 더 좋다는 것을 실험적으로 확인**해서 그렇게 사용한다고 하며<br>
여기서 activation function인 $\sigma$는 ReLU를 사용한다고 한다

## 3.1 Feature Distillation as Multi-task Learning
