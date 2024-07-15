---
title: "[Review] Anomaly Detection via Reverse Distillation from One-Class Embedding, CVPR, 2022"
excerpt: "Knowledge Distillation을 이용한 Anomaly detection"

toc: true
toc_sticky: true

use_math: true

categories:
    - Review
tags:
    - Review
    - Anomaly Detection
    - Knowledge Distillation
last_modified_at: 2023-12-02T01:00:00
---

<span style="font-size:200%">Anomaly Detection (AD)</span>는 뜻 그대로 '변칙'인, 비정상 데이터 검출을 의미하는데<br>
신기하게도 AD에서 Knowledge Distillation을 이용하여 이상치를 검출할 수 있다고 하여<br>

아래 논문을 한번 간단히 리뷰해보려고 한다

[Anomaly Detection via Reverse Distillation from One-Class Embedding](https://arxiv.org/abs/2201.10703)


><span style="font-size:150%">**Hypothesis**</span><br>
Unsupervised AD에서 학생(Student) 모델을 일반 샘플에 대해서 학습하면<br>
비정상 데이터에 대해 선생(Teacher) 모델과 다른 representation을 생성할 것이다

라는 아이디어에서 출발한다<br>
<br>
하지만<br>
1. 동일하거나 유사한 선생 학생 모델의 구조
2. 선생-학생 모델간의 동일한 데이터 플로우

로 인해 정확히 비정상을 검출하지 못한다고 저자는 지적한다<br>
<br>
위 문제를 해결하기 위해 아래와 같은<br>
**`Teacher Encoder - Student Decoder`구조**를 제안한다고 한다<br>
<center><img width="568" alt="image" src="https://github.com/Hyungkeun-Park/Hyungkeun-Park.github.io/assets/21329629/4a54a1f9-6b3c-4ad9-b295-8860a65cb4a0"></center>
<br>
데이터를 T와 S에 직접 넣어주는 전형적인 KD 구조와는 달리<br>
T를 통해 뽑아진 representation을 S를 통해 decoding하는<br>
말그대로 `Reverse Distillation` 구조인 것이다<br>
<br>
위 구조를 통해<br>
T-S의 구조 유사성으로 인한 문제점을 해결하고<br>
AutoEncoder(AE) 기반의 AD와는 달리 deep features를 이용하여 검출하기 때문에<br>
pixel-by-pixel 검출보다 더욱 효과적인 판별 정보를 제공한다고 한다<br>
<br>

그리고 **`One-Class Bottleneck Embedding`** 이라는 모듈을 제안하는데<br>

`OCBE`는 `Multi-scale Feature Fusion` +`One-Class Embedding`으로 구성되어
+ MFF<br>
일반 패턴을 복원하기 위한 Low & High level features를 집계(aggregate)
+ OCE<br>
학생 모델이 선생 모델의 representation을 decode하기에 유리한 정보를 유지하도록 함

위 설명만 봐서는 아직 이해가 안되기에 차근차근 보도록 하자<br>
<br>
# Our Approach
---
<center><img width="578" alt="image" src="https://github.com/Hyungkeun-Park/Hyungkeun-Park.github.io/assets/21329629/497c4249-4299-4618-9a7e-93caf338a2ca"></center>
우선 $I^t$를 정상 이미지 데이터로, $I^q$를 정상 & 비정상 샘플이 담긴 데이터라고 하면<br>
이미 트레이닝 된 T 모델(E)로 부터 $I^t$에 속한 이미지로 부터 multi-scale featre를 뽑아내고<br>
학생 모델(D)로 해당 이미지를 복원하는 방법으로 학습이 진행된다<br>

## 1. Reverse Distillation
학생 모델의 경우 비정상 데이터로 학습을 한 적이 없으니<br>
당연히 비정상 데이터에 대해 상당히 다른 representation을 내놓기를 기대할 것이다<br>
하지만 선생 학생 모델이 서로 유사한 데이터 플로우와 구조를 갖는 경우<br>
비정상에 대한 activateion이 사라진다고 한다<br>
(구조가 같으니 너무 다른 결과에 대해 Teacher도 복원하지 못하기 때문?)<br>
<br>
그래서 AE구조를 갖는 reversed distillation을 제안한다고 한다<br>
<br>
학생 모델(D)는 선생 모델(E)와 동일한 구조로 순서만 뒤집힌 구조를 갖는데<br>
이러한 구조가 비정상에 대한 response를 제거하기 용이하게 해준다고 한다<br>
<br>
그리고 학생 모델을 학습시키는 **Loss로는 Cosine similarity**    를 사용했다<br>
<br>

## 2. One-Class Bottleneck Embedding
<center><img width="281" alt="image" src="https://github.com/Hyungkeun-Park/Hyungkeun-Park.github.io/assets/21329629/bd39c2a0-9781-4a01-b6fb-c00a45f36f29"></center>
선생 모델(E)의 representation을 학생 모델(D)에서 그대로 decode할 경우<br>
1. E의 풍부한 representaion으로 인해<br>
비정상 데이터에 대한 feature 또한 복원 될 가능성이 있음
2. E의 최종 representation은 고차원 feature만을 담고 있기에<br>
D가 복원할 때 Low-feature에 대해 복원하기 상당히 어려울 것

위와 같은 문제가 존재하여<br>
1. MFF<br>
다양한 scale의 feature를 모은 뒤 convolution을 통해 차원을 맞춰준다<br>
2. OCE<br>
MFF에 모아준 feature를 D에서 decode할 수 있는 최소한의 정보만을 담도록 만들어준다

위 두 모듈을 통해 문제점을 해결하도록 한다<br>
(동일하게 OCBE 또한 training 진행)<br>
<br>

## 3. Anomaly Scoring
<center><img width="248" alt="image" src="https://github.com/Hyungkeun-Park/Hyungkeun-Park.github.io/assets/21329629/5a7fa609-e7b9-4c79-b262-75143448b6d6"></center>
위 식을 통해 E와 D 각 블록에서의 anomaly map을 구했는데<br>
비정상 이미지에 해당 map을 맞추기 위해 bilinear interpolation을 통해<br>
원래 이미지 사이즈로 upscaling한다고 한다<br>
<br>
Anomaly detection score를 매기는데 있어<br>
이미지 사이즈로 스케일링한 map을 단순 average 취할 경우<br>
비정상 데이터가 상당히 적은 이미지의 경우 평균값이 상당히 낮아지기 때문에<br>
최대값으로 anomaly score를 정했다고 한다