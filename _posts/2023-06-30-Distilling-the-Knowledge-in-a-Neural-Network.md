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
last_modified_at: 2023-06-30T01:00:00
---
<br>
<br>
<br>
작성중
{: .text-center}
<br>
<br>
<br>

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

![image](https://github.com/Hyungkeun-Park/Hyungkeun-Park.github.io/assets/21329629/6322c5f6-8d0d-494e-809f-15b1e1099cd0)
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