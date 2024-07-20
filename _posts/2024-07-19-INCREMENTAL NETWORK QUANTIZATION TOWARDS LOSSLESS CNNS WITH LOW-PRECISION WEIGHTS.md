---
title: "[Review] INCREMENTAL NETWORK QUANTIZATION: TOWARDS LOSSLESS CNNS WITH LOW-PRECISION WEIGHTS, ICLR, 2017"
excerpt: "Non-uniform base-2 logarithmic quantization"

toc: true
toc_sticky: true

use_math: true

categories:
    - Quantization
tags:
    - 경량화
    - Quantization
last_modified_at: 2024-07-19T01:00:00
---

<!--bundle exec jekyll serve : 임시 확인-->

**[INCREMENTAL NETWORK QUANTIZATION: TOWARDS LOSSLESS CNNS WITH LOW-PRECISION WEIGHTS](https://arxiv.org/abs/1702.03044)**<br>
**`Logarithmic-scale quantization`**을 이용한 점진적 quantization을 제안한 논문을 간단 리뷰해보겠습니다<br>

재미있는 점은 해당 방법을 이용하면 기존 32bit으로 학습시킨 network의 성능보다 좋은 성능이 나온다는 점입니다
<center><img width="500" alt="image" src="https://github.com/user-attachments/assets/2b3536e4-6868-4703-8f9d-7b58fdc9e408"></center>
<br>
<br>

# Weight Quantization
---
$z
P_l= \left( ±2^{n_1},\cdots,±2^{n_2} ,0 \right)
$z
$P_l$의 값들로 model weight을 quantize하려고 할 때, $n_1$은 다음과 같이 설정한다고 합니다
$z
n_1 = floor(log_2(4s/3))
$z
$z
s=max(abs(W_l))
$z
위 식의 경우, $l-th$ layer에서의 max weight value를 가장 가까운 정수 값으로 quantize하기 위한 식으로 보여집니다<br>
$z
\begin{align}
Ex)\quad max(W_l)&=3.3 \newline
n_1=floor(log_2(4 \times 3.3/3))&=2 \newline
3.3\rightarrow 2^2&=4
\end{align}
$z

그리고 $n_1$의 값이 결정되고 나면 $n_2$의 값은 다음과 같이 정의된다고 합니다

$z
n_2 = n_1+1-2^{b-1}/2
$z
어떻게 위와 같은 식이 나오게 되었는가 고민을 좀 많이 해보았는데<br>
제가 해석하기로는 다음과 같습니다<br>

우선 논문에서 예시로 든 바와 같이 $b=3,\ n_1=-1,\ n_2=-2$라고 할 때<br>
총 3bit을 이용하여 model quantization을 하려는 것을 의미하며<br>
$P_l=\left(±2^{-1},±2^{-2},0\right)$이 됨을 알 수 있습니다<br>
3bit중에서 1bit은 0에 할당, 그리고 나머지 2bit을 통해 $±2^{-1},±2^{-2}$의 4가지 경우의 수를 표현할 수 있어<br>
$n_2$의 값이 위와 같이 설정된 것으로 판단하였습니다<br>

이렇게 구성된 $P_l$을 이용하여 model weight $W_l(i,j)$를 다음과 같이 quantize 한다고 합니다
$z
\hat{W}_l(i,j)=
\begin{cases}
{\beta sgn(W_l(i,j))} & if\;(\alpha+\beta)/2 ≤ abs(W_l(i,j)) ＜3\beta/2 \newline
{0} & otherwise
\end{cases}
$z
<br>
<br>

# Incremental Quantization Strategy
---
논문의 저자들은 pruning과 같이, lossless network quantization을 위해서는 추가적인 training이 필요하다고 얘기합니다<br>

따라서 저자들이 제안하는 Incremental Network Quantization (INQ)는 다음의 세 가지 step을 거친다고 합니다
1. Weight partition : group for quantization and re-train
2. Group-wise quantization
3. Re-training

Re-train을 위한 group에 대해 re-train을 거친 뒤, 해당 그룹에서 다시 1-3 의 과정인 partitioning & re-train을 계속 진행함으로써<br>
최종적으로 모든 weight이 quantization이 되도록 하게 만든다고 합니다

그렇다면 어떤 기준으로 partition을 나누느냐 가 관건일 것인데<br>
저자들은 random 방식과 pruning에서 영감을 받은 방식 두 가지를 사용해서 실험을 진행해보았다고 하며<br>
pruning에서 영감을 받은, larger absolute value를 가진 weight을 quantization하는 방식이 성능이 더 좋았다고 합니다
<center><img width="500" alt="image" src="https://github.com/user-attachments/assets/1bf664c0-e3ed-4839-b10f-7f2e1f01bc58"></center>
<br>
<br>

# Incremental Network Quantization (INQ) Algorithm
---
위에서 설명한 바와 같이, quantization하지 않은 group에 대해서는 re-training을 진행하게 되는데<br>
re-train group에 대해서만 training을 진행하기 위해 추가적인 matrix인 $T_l$을 추가로 도입하여<br>
back propagation시 해당 group에만 gradient가 전파되도록 masking하는 역할을 하게됩니다<br>

$z
\underset{W_l}{min}E(W_l)=L(W_l)+\lambda R(W_l)
$z
$z
s.t. W_l(i,j)\in P_l,\; if\; T_l(i,j)=0,\; 1\leq l\leq L
$z
최종적으로 위와 같은 object function을 만족하도록 학습이 진행되게 됩니다
<center><img width="550" alt="image" src="https://github.com/user-attachments/assets/d5c4cd55-066d-41e4-b0cd-c1884cbbd1b7"></center>