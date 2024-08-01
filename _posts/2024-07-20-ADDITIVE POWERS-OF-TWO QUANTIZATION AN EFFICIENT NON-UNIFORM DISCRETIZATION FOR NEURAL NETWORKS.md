---
title: "[Review] ADDITIVE POWERS-OF-TWO QUANTIZATION: AN EFFICIENT NON-UNIFORM DISCRETIZATION FOR NEURAL NETWORKS, ICLR, 2020"
excerpt: "Non-uniform base-2 logarithmic quantization"

toc: true
toc_sticky: true

use_math: true

categories:
    - Quantization
tags:
    - 경량화
    - Quantization
last_modified_at: 2024-07-20T01:00:00
---

<!--bundle exec jekyll serve : 임시 확인-->

**[ADDITIVE POWERS-OF-TWO QUANTIZATION: AN EFFICIENT NON-UNIFORM DISCRETIZATION FOR NEURAL NETWORKS](https://arxiv.org/abs/1909.13144)**<br>
**`Logarithmic-scale quantization`**을 이용한 quantization 논문으로<br>
기존의 Power-of-Two (PoT) quantization의 문제점을 sum of PoT로 변경함으로써 문제점을 해결하고자 한 논문입니다

<br>
<br>

# Power-of-Two (PoT) Quantization
---
이전 게시물에서 다루었듯, 하드웨어 친화적인 quantization을 구현하기 위해<br>
bit shift를 이용한 quantization의 방법을 제안한 기존의 연구들이 존재하였는데<br>
이를 현재 논문에서 **`PoT`**라고 칭하고 있습니다<br>

PoT의 경우, quantization bit width가 증가할 경우, 분포 전반에 대한 resolution 증가가 아닌<br>
0에 가까운 값에서의 resolution만 증가하는 점을 문제로 지적하며, 이를 Rigid resolution 문제라고 정의합니다<br>
저자들은 위 문제를 해결하기 위해 **`PoT → sum of PoT 로 변경`**하여 해당 문제를 해결하고자 합니다<br>
<center><img width="650" alt="image" src="https://github.com/user-attachments/assets/da72fb54-7edf-4c08-8c25-119ef4b192db"></center>
<br>
<br>

# Additive Power-of-Two Quantization
---
기존의 PoT의 경우 다음과 같은 방식으로 quantization을 진행하였습니다

$$
Q^P(\alpha,b)=\alpha\times｛\ 0,±2^{-2^{b-1}+1},±2^{-2^{b-1}+2},\cdots,±2^{-1},±1\ ｝
$$

저자들이 제안하는 방식인 Additive PoT (APoT)는 다음과 같습니다

$$
Q^a(\alpha,kn)=\gamma\times\ ｛\sum _{i=0}^{n-1}p_i\ ｝\quad where\; p_i\in｛\ 0,\frac{1}{2^{i}},\frac{1}{2^{i+n}},\cdots,\frac{1}{2^{i+(2^k-2)n}}\ ｝
$$

식으로 봤을때에는 한 번에 이해가 잘 되지 않을수도 있습니다<br>

우선, 기존 PoT에서 사용하던 quantization하려는 bit인 b에 kn을 추가로 도입했다는 것을 알 수 있는데<br>
각각의 term이 의미하는 바는 다음과 같습니다
1. k : base bit-width (k=1 : uniform qunatization / k=b : PoT)
2. n : number of additive terms = b/k 

이를 논문에 나온 예시를 통해 이해해보면 조금 더 쉬워집니다<br>
b=4, k=2인 경우, APoT는 $p_0\in｛\ 0,2^0,2^{-2},2^{-4}\ ｝$, $p_1\in｛\ 0,2^{-1},2^{-3},2^{-5}\ ｝$의 power term을 갖게 됩니다<br>

즉, n은 몇개의 power term set을 갖고 quantization을 할 것인지<br>
b는 각각의 power term이 가질 수 있는 case는 몇개인지 (예시에서는 $2^2=4$)를 뜻한다고 보시면 됩니다<br>

결국 기존의 b=4인 PoT에서 갖는 power term 조합 $(2^4=16)$과 동일한 조합의 수를 얻게 됩니다 $(2^2\times2^2=16)$<br>

논문의 저자들은 실험에서 k=2를 사용하여 quantization을 진행하였는데<br>
k=2를 사용할 경우, 2n-bit의 quantization밖에 불가능해 2n+1 bit에 대한 일반화를 위해 n+1 PoT term을 추가합니다<br>

$$
Q^a(\alpha,2n+1)=\gamma\times\ ｛\sum _{i=0}^{n-1}p_i+\tilde{p}\ ｝\quad where\; p_i\in｛\ 0,\frac{1}{2^{i}},\frac{1}{2^{i+n}},\frac{1}{2^{i+2^n+1}}\ ｝,\ \tilde{p}\in｛\ 0,\frac{1}{2^{i+2n}}\ ｝
$$

<br>
<br>

# Reparameterization Clipping Function
---
Quantization에 있어서 clipping 범위를 잘 정하는 것이 성능 유지에 중요한 역할을 하게 되는데<br>
모든 layer에 static한 threshold를 정하기란 상당히 쉽지 않음을 예상할 수 있습니다<br>

이런 문제를 해결하고자 threshold와 weight을 같이 학습시키는 Straight-Through Estimator (STE)라는 기법이 이전에 연구됐습니다<br>

STE에서는 quantization된 weight에 대한 threshold로의 gradient 계산에 있어서<br>
clipping된 weight에 대한 gradient만을 학습하도록 하였습니다<br>

더욱 정교화된 학습을 위해, 저자들은 Reparameterized Clipping Function (RCF)를 다음과 같이 제안했습니다<br>

$$
\hat{W}=\alpha\prod _{Q(1,b)}\lfloor\frac{W}{\alpha},1\rceil
$$

<br>
우선 Weight을 $\alpha$로 나누어 ±1의 범위에서 clipping을 진행한 후<br>
해당 Weight을 Quantization한 뒤, 다시 $\alpha$를 곱하는 방식을 의미합니다<br>

위와 같은 방식으로 clipping을 진행할 경우 STE에 대한 $\alpha$로의 gradient는 다음과 같아집니다

$$
\frac{\partial\hat{W}}{\partial\alpha}=
\begin{cases}
sign(W) & if\; |W|>\alpha \newline
\prod_{Q(1,b)}\frac{W}{\alpha}-\frac{W}{\alpha} & if\; |W|\leq\alpha
\end{cases}
$$

<br>
결과적으로, $\alpha$ 값에 대한 gradient가 clipping 범위 안밖에 있는 모든 weight에 대해 생성되기 때문에<br>
더욱 정교한 gradient를 얻을 수 있다는 장점이 생기게 됩니다
<br>
<br>

# Weight Normalization
---
Weight의 분포는 상당히 급격하고, 학습에 따라 계속 변화하기 때문에<br>
clipping threshold와 weight을 동시에 학습시키는 것은 상당히 어렵다고 지적하며<br>

Activation qunatization에서 Batch normalization (BN)가 중추 역할을 하는 것처럼<br>
weight을 zero mean, unit variance로 바꿔주는 Weight Normalization (WN)을 저자들은 제안합니다<br>
(Weight quantization은 normalization 이후 진행한다고 합니다)<br>

Normalization은 일관되고 안정된 입력(weight) 분포를 제공하기 때문에 최적화에 도움이 되며<br>
weight의 중심을 0으로 맞춰줘 symmetric quantization의 이점을 더욱 끌어올릴 수 있다고 합니다<br>

서로 다른 layer에서의 clipping 결과가 normalization의 유무에 따라 얼마나 변화가 심한지를 실험적으로 보이며 WN의 필요성을 보여줍니다
<center><img width="700" alt="image" src="https://github.com/user-attachments/assets/0ca5f8c8-3057-41b5-b7e4-d9e22de21400"></center>