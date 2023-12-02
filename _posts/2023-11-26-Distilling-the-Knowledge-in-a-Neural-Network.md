---
title: "[Review] Distilling the Knowledge in a Neural Network, NIPS, 2014"
excerpt: "Knowledge Distillation 분야의 시작"

toc: true
toc_sticky: true

use_math: true

categories:
    - Review
tags:
    - Review
    - 경량화
    - Model Compression
    - Knowledge Distillation
last_modified_at: 2023-11-26T01:00:00
---

<br>
오늘은 Deep Learning에서의 경량화 기법 중 한 종류인<br>
**Knowledge Distillation** (지식증류)의 첫 시작인 논문에 대해<br>
아주 아주 간단하게 리뷰하는 시간을 가져보려 한다<br>
<br>
# Knowledge Distillation (KD)
---
지식증류 라고도 불리우는 Knowledge Distillation은<br>
Input에 대한 정답 Label을 이용한 기존의 Supervised 학습 방법과 달리<br>
선생(Teacher) 모델의 정답 Label 이외의 정보에 대해서도<br>
학생(Student) 모델에게 학습을 시키는 것이 목적이다<br>

![image](https://github.com/Hyungkeun-Park/Hyungkeun-Park.github.io/assets/21329629/6322c5f6-8d0d-494e-809f-15b1e1099cd0){: with="70%", height="70%"}
KD의 방법들에는 위 사진과 같이 다양한 종류들이 존재하고<br>
오늘 다뤄볼 논문의 경우 *Distillation from one teacher - Knowledge from logits*에 해당한다<br> 

만일 필자와 같이 Knowledge Distillation 분야에 관심이 생겨 공부를 막 시작하는 사람이라면<br>
*Knowledge Distillation and Student-Teacher Learning for Visual Intelligence: A Review and New Outlooks*<br>
위 논문을 참고하여 공부를 시작하면 큰 그림을 이해하는데에 도움이 될 것이다<br>
<br>
## Convolutional Neural Network (CNN)
CNN을 통한 Image Classification task의 경우<br>
Input → Convolution Layer → Fully-Connetect Layer → 각 클래스에 대한 Logit<br>
최종적으로 Logit에 대한 Softmax를 통해 해당 Class에 대한 확률을 구하게 되는데<br>

KD에서는 정답 라벨 이외의 정보들을 보존하기 위해 Logit을 Temperature Scaling하고<br>
Temperature Scaling을 통해 broad해진 분포를 Student 모델의 분포와 KL-divergence를 통해 학습을 진행한다<br>
보통의 KD 논문에서 Temperature T=4로 두고 KD를 진행하고 있다<br>
$z
q_{i}=\frac{exp(z_{i}/T)}{\sum_{j}{z_j/T}},\; KL(T||S)=\sum T\log{\frac{T}{S}}
$z

$q_{i}$는 CNN을 통해 얻어진 Logit을<br>
각 클래스에 대한 확률로 변환해주는 `Softmax`를 수식으로 보여주고<br>
<br>
$KL(T||S)$는 KD의 목적함수인<br>
`쿨백-라이블러 발산(Kullback-Leibler Divergence)`로<br>
`T`라는 분포와 `S`라는 분포가 일치할 때 0이되는 일종의 분포간 거리에 대한 metric이다<br>
<br>

# 아주 간단히 정리하자면 !<br>
Image Classification에서 주어지는 Ground Truth에 대해 학습을 하면<br>
이미 학습된 선생모델(T)는 주어진 모든 클래스에 대해 확률값을 출력하게 되는데<br>
이 확률 분포가 곧 선생의 **암묵지(Dark Knowledge)**이기에<br>
**학생모델(S)이 해당 분포를 좇아가도록 하면**<br>
홀로 학습하는 것(보통 From-Scratch 라고 함)보다 **더 좋은 성능**이 나올 수 있다는 것이다<br>
<br>
<br>

### p.s.
KD의 최종 Loss항을 보면 KD-Div 항에 temperature의 제곱항이 곱해진 것을 볼 수 있는데<br>
Gradient가 역전파 될 때 Logit을 Temperature로 나누어 주었기에 이에 대한 보상으로 제곱항을 곱해준다<br>