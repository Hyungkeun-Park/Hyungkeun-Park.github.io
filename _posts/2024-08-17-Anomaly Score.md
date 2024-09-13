---
title: "[Review] Anomaly Score: Evaluating Generative Models and Individual Generated Images based on Complexity and Vulnerability, CVPR, 2024"
excerpt: "Evaluation metric for generative models"

toc: true
toc_sticky: true
use_math: true

categories:
    - Review
tags:
    - Review
last_modified_at: 2024-08-17T00:00:01
---

<!--bundle exec jekyll serve : 임시 확인-->


**[Anomaly Score: Evaluating Generative Models and Individual Generated Images<br> based on Complexity and Vulnerability](https://openaccess.thecvf.com/content/CVPR2024/papers/Hwang_Anomaly_Score_Evaluating_Generative_Models_and_Individual_Generated_Images_based_CVPR_2024_paper.pdf)**<br>
<br>
<br>

해당 논문에서는<br>
생성형 모델로부터 생성된 이미지의 새로운 평가 방법으로<br>
**`Complexity`**와 **`Vulnerability`**를 이용한 **`Anomaly Score (AS)`**를 제안합니다

<br>
<br>

# 기존 평가 지표
---
크게 논문에서 제시한 기존의 평가 지표는 다음과 같습니다

1. FID : Inception 모델을 이용하여 실제/생성 이미지간의 wasserstein 거리를 측정하는 방식
2. Precision & Recall : 실제/생성 이미지가 동일 manifold상에 있는지를 판단
3. realism/rarity score : 실제/생성 이미지간의 관계/이미지의 희귀도 정도를 측정

위 기존 방식들의 문제점으로<br>
1. 사람들의 실제 인식과 일치하지 않는다는 점
2. 샘플간의 거리 측정을 위해 계산 비용이 많이 든다는 점
등을 들고 있습니다

위 문제를 해결하기 위해 저자들은 Complexity와 Vulnerability를 이용한 Anomaly Score를 제안하게 됩니다
<br>
<br>

# Complexity
---
학습된 딥러닝 모델의 경우<br>
입력 이미지에서의 선형 변환이 출력단에서는 비선형임을 예상할 수 있습니다<br>
(딥러닝 모델의 비선형 연산 때문)<br>

저자들은 입력 이미지에 노이즈를 추가함에 따라 출력에서 변화하는 정도를 측정해보았고<br>
노이즈가 더해진 횟수 k가 증가함에 따라 그 변화도는 감소함을 확인하였습니다<br>

<center><img width="400" alt="image" src="https://github.com/user-attachments/assets/cb9ae7b8-fcf7-430f-8d8d-52dce0d6e2b6"></center>

위와 같은 현상을 통해<br>
**실제 이미지 주변에서는 입력 이미지에 노이즈를 더함에 따라 출력에서의 변화도가 더 클 것**이다 라는 가정을 합니다<br>

이에 저자들은 Complexity라는 값을 다음과 같이 정의했습니다

$$
C(x)=\frac{1}{K-1} \sum_{k=1}^{K-1} \left( cos^{-1} \frac{(M(x^k)-M(x^{k-1}))\cdot(M(x^{k+1})-M(x^k))}{\lVert M(x^k)-M(x^{k-1})\rVert \lVert M(x^{k+1})-M(x^k) \rVert} \right)
$$

위 식에서 C()는 complexity, M()은 임의의 모델, $x^k$는 k번 노이즈가 더해진 이미지 를 뜻합니다

위에서 정의한 complexity를 실제/생성 이미지에 대해 ViT, ConvNeXt, DINO-V2에서 측정한 결과<br>
실제 이미지에서 complexity가 더 높았음을 확인할 수 있었습니다<br>

<center><img width="400" alt="image" src="https://github.com/user-attachments/assets/b930b8be-3536-4818-8fdd-3750b1d163f0"></center>

<br>
<br>

# Vulnerability
---
이미지 상에서 노이즈를 추가하는 adversarial pertubation은<br>
feature space 상에서 큰 변화를 야기할 수 있는데<br>

<center><img width="400" alt="image" src="https://github.com/user-attachments/assets/2d616c95-cd5e-4e78-8d3b-14eea41af494"></center>

생성된 이미지의 부자연스러운 부분이 pertubation에 대한 변화도가 가장 큰 것을 확인하였고<br>
이를 통해 vulnerability라는 값을 다음과 같이 정의했습니다

$$
V(x)=dist(M(x),M(x^J))
$$

위 식에서 J는 pertubation의 횟수를 뜻하고, dist는 L2 distance를 의미합니다

<br>
<br>

# Anomaly Score (AS) & AS for individual images
---
우선 Complexity와 Vulnerability를 값으로 갖는 Anomaly vector를 다음과 같이 정의합니다

$$
A(X)={[C(x),V(x)]}_{x\in X}
$$

그리고 Kolmogorov-Smirnov (KS)라는 정규성 검정 방식을 통해<br>
생성형 모델의 성능을 측정하는 Anomaly Score (AS)를 다음과 같이 정의합니다<br>

$$
AS = Sup |CDF(A(X^{real}))-CDF(A(X^{generated}))|
$$

생성 이미지에 대해 FID와 AS를 측정한 결과와, human error rate에 대해<br>
correlation을 확인해본 결과 저자들이 제안한 AS가 더 높은 상관관계를 갖고 있음을 확인했다고 합니다<br>
<br>
<br>

그리고 개별 생성 이미지에 대한 AS 값으로 AS-i라는 값을 다음과 같이 정의하는데

$$
AS-i(x) = \frac{V(x)}{C(x)}
$$

생성 이미지가 자연스러울수록 complexity는 증가하고, vulnerability는 감소하기에<br>
AS-i값이 높을수록 실제같은 이미지, 낮을수록 부자연스러운 이미지를 뜻하게 된다고 하며<br>
생성 이미지에 AS-i 값을 평가해본 결과 저자들이 의도한 바와 같이<br>
사람이 보기에 자연스러운 이미지에 대해서는 AS-i가 낮음을 확인했다고 합니다<br>

<center><img width="350" alt="image" src="https://github.com/user-attachments/assets/106e6e79-b93c-47bf-a192-e78e2f5c8a89"></center>