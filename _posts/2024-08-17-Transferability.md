---
title: "[Review] Similarity of Neural Architectures using Adversarial Attack Transferability, ECCV, 2024"
excerpt: "Model Similarity"

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


**[Similarity of Neural Architectures using Adversarial Attack Transferability](https://arxiv.org/abs/2210.11407)**<br>
<br>
<br>

해당 논문은 모델간의 유사도를 측정하기 위한 Similarity by Attack Transferability (SAT) 라는 기법을 제안하는데<br>
adversarial attack에 대한 input gradient & decision boundary를 통해 SAT를 정의한다고 합니다

<br>
<br>

# Similarity by Attack Transferability (SAT)
기존에 모델의 유사도를 측정하기 위한 다양한 방법들이 존재했는데<br>
1. Loss landscape 비교 : 정량화 어려움
2. Prediction 비교 : DNN 모델들의 prediction이 유사해져 비교 어려움
3. Input gradient : Noisy하기 때문에 전처리가 필요
4. Decision boundary 비교 : 유한의 데이터로 상당히 고차원적인 차원을 추적하기 어려움
위와 같은 문제들이 있어 명확한 비교에 어려움이 있다고 합니다<br>

그래서 저자들은 adversarial attack을 이용한 모델 유사도 측정법에 대해 다음과 같은 제안을 합니다<br>

**`유사한 모델의 구조를 가졌다면, A (B) 모델에 적용된 attack이 B (A) 모델에 또한 적용될 것`이다**<br>

이에 따라 다음의 식을 SAT 라는 유사도 값으로 정의합니다

$$
SAT(A,B)=log[max\{\epsilon _s,100\times\frac{1}{2|X_{AB}|}\sum _{x\in X_{AB}}\{\mathbb{I}(A(x_B)\neq y)+\mathbb{I}(B(x_A)\neq y)\}\}]
$$

식이 복잡해 보이지만, 단순히 정리해보자면<br>
A, B 모델이 동일하게 정답을 맞추는 데이터($|X_{AB}|$)에 대해<br>
A, B 모델에 대한 attack을 가했을 때($x_A, x_B$), 각 모델에 대해 attack이 성공한 경우($A(x_B)\neq y,\ B(x_A)\neq y$)<br>
Indicator function($\mathbb{I}$)은 1을 출력하게 되어, 두 모델이 완벽하게 동일할 때 SAT=log100이 됩니다