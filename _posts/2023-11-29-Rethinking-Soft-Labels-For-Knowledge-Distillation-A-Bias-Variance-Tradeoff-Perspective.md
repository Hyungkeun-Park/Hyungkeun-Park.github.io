---
title: "[Review] Rethinking Soft Labels For Knowledge Distillation: A Bias-Variance Tradeoff Perspective, ICLR, 2021"
excerpt: "Variance와 Bias 관점에서의 Knowledge Distillation"

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
last_modified_at: 2023-11-29T01:00:00
---
작성중
{: .notice}
<br>

# Variance VS Bias
---
우선 Variance와 Bias를 간단히 정리하고 넘어가면<br>
<br>

<span style="font-size:200%">`Variance`</span>는 주어진 데이터에 대해 모델이 얼마나 `flexible`하냐 라는 의미로 해석할 수 있는데<br>
`Variance`가 크다는 것은, 모델의 예측값이 주어진 데이터에 따라 변화하는 정도가 크다 라는 것을 뜻한다<br>
**즉, 모델이 데이터에 얼마나 의존적인지를 뜻한다**
<br>

<span style="font-size:200%">`Bias`</span>는 주어진 데이터와 모델의 추론값이 평균적으로 얼마나 떨어져있냐 라는 의미로 해석할 수 있으며<br>
`Bias`가 크다는 것은, 모델의 예측갑이 주어진 데이터에 따라 변화하는 정도가 작다 라는 것을 뜻한다<br>
**즉, 문제를 단순화 함에 따라 발생하는 오류** 라고 생각하면 된다<br>
<br>

그렇다면 당연히 **`Variance`와 `Bias` 모두 작은 모델이 가장 이상적인 모델**이라고 할 수 있다<br>
<br>
~~하지만 세상 만사가 다 그렇게 쉽지는 않다~~<br>
<br>

역시나 **`Variance`와 `Bias`는 서로 `Trade-off` 관계**를 갖고 있다<br>

>그렇다면, **학습하지 못한 데이터에 있어서도 좋은 성능을 보이는 것**이 딥러닝의 주 목표라고 할 수 있기에<br>
`Variance`가 낮아지게 학습을 하는 것이 좋은 방향이 아닐까?<br>

위와 같은 의문점을 Knowledge Distillation(KD)에 대해 접근한 논문을 이번에 살펴보고자 한다<br>
<br>
<br>
# Bias-Variance tradeoff for soft labels
---
우선 논문의 저자들은 KD에서의 Loss term인 $L_{CE}, L_{KD}$를 분석을 위해 다음과 같이 수식들을 진행한다<br>
~~다가오는 수식에 눈물이 앞을 가리지만 최대한 열심히 쫓아가 보자~~

우선 Cross Entropy(CE)로 학습한 모델의 입력 x에 대한 출력은 $\hat{y} _{ce}=f _{ce}(x;D)$로<br>
KD로 학습한 모델의 입력 x에 대한 출력은$\hat{y} _{kd}=f _{KD}(x;D,T)$로 표현한다<br>
<br>
이 때 $CE$와 $KD$에 대한 출력들의 평균 $\hat{y}$는 normalization 상수 $Z$와 함께 다음과 같이 표현 가능하다

$$
\bar{y} _{ce}=\frac{1}{Z _{ce}}\exp(E _D[\log\hat{y} _{ce}]),\quad \bar{y} _{kd}=\frac{1}{Z _{kd}}\exp(E _{D,T}[\log\hat{y} _{kd}]) \tag{1}
$$

그리고 CE에 대한 error항을 다음과 같이 $noise+bias+variance$항으로 decompositon을 진행한다<br>

$$
\begin{align}
error _{ce} = E _{x,D}[-y \log \hat{y} _{ce}] &= E _{x,D} [-y \log y + y \log \frac{y}{\bar{y} _{ce}} + y \log \frac{\bar{y} _{ce}}{\hat{y} _ce}] \newline
&=E _x[-y \log y] + E_x[y \log \frac{y}{\bar{y} _{ce}}]+E_D[E_x[y \log \frac{\bar{y} _{ce}}{\hat{y} _{ce}}]] \tag{2} \newline
&=E _x[-y \log y] + D _{KL}(y,\bar{y} _{ce}) + E _D[D _{KL}(\bar{y} _{ce},\hat{y} _{ce})] \newline
&=intrinsic\;noise+bias+variance
\end{align}
$$

위와 동일한 방식으로 KD에 대한 error항을 decomposition한다

$$
error _{kd} =E _x[-y \log y] + D _{KL}(y,\bar{y} _{ce}) +E _x[y \log (\frac{\bar{y} _{ce}}{\bar{y} _{kd}})] + E _{D,T}[D _{KL}(\bar{y} _{kd},\hat{y} _{kd})]
$$