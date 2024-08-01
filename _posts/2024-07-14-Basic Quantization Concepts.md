---
title: "Basic Quantization Concepts"
excerpt: "Quantization 기본 컨셉 정리"

toc: true
toc_sticky: true

use_math: true

categories:
    - Quantization
tags:
    - Quantization
last_modified_at: 2024-07-14T01:00:00
---

<!--bundle exec jekyll serve : 임시 확인-->

**[A Survey of Quantization Methods for Efficient
Neural Network Inference](https://arxiv.org/abs/2103.13630)**<br>
위 서베이 논문 기반으로 간단 정리한 내용입니다
<br>
<br>

# Uniform vs Non-uniform Quantization
---
+ 기존 실수값의 weight/activation을 등간격의 특정 값으로 $\rightarrow$ Uniform quantization
+ 기존 실수값의 weight/activation을 비등간격의 특정 값으로 $\rightarrow$ Non-uniform quantization 
<br>
<br>

<center><img width="590" alt="image" src="https://github.com/user-attachments/assets/c3069942-331a-4675-8294-09d5b8261115">
</center>
<center>
좌 : Uniform / 우 : Non-uniform quantization<br>
실수축 r의 값은 Q축에 표시된 점들로 quantization 된다
</center>
<br>

$$
Quantization\ Q(r) = Int(r/S)-Z
$$

$$
Dequantization\ \tilde{r} = S(Q(r)+Z)
$$

$$
r:real\ value,\ \tilde{r}:recovered\ real\ value,\ S:scale,\ Z:zero\ point
$$

<br>
<br>

# Symmetric vs Asymmetric Quantization
---
+ Uniform quantization의 경우, 일정한 S(scale)로 quantization을 진행, $S=\frac{\beta-\alpha}{2^b-1}$
+ $\alpha, \beta$는 quantization할 실수(r)의 범위를 나타내고, b는 quantization할 bit를 의미함
+ $\alpha,\beta$의 값을 정하는 과정을 calibration이라 정의함
+ 이 때, $-\alpha=\beta$의 경우를 symmetric quantization, $-\alpha\neq\beta$의 경우를 asymmetric quantization 이라고 함
+ $\alpha,\beta$를 정하기 위한 방법으로는 `min/max`, `percentile`, `KL divergence`를 이용하는 방법들이 존재함
<br>
<br>

+ Symmetric의 경우, $Z=0$로, quantization이 간단해지는 장점이 존재함
+ Ex) IN8 quantization에서, $[-128,127]$의 범위를 모두 사용하는 경우를 `full range`, $[-127,127]$의 경우 `restricted range` quantization이라고 함
+ 예상대로, `full range` qunatization이 더 정확도가 높다

<br>
<br>

# Non-Uniform Quantization
---
+ $Q(r)=X_i,\ if\ r\in[\Delta_i,\Delta_{i+1})$
+ 실수 r이 $\Delta_i$와 $\Delta_{i+1}$ 사이의 값을 갖을 때, $X_i$로 해당 값을 quantize 함
+ 다음과 같은 Non-Uniform quantization 방식들이 존재
    1. 값들의 bell-shaped 분포를 가정한 방식
    2. 로그같은 분포를 사용하는 `Rule-based` 방식
    3. 이진 코드($r\approx\sum_{i=1}^m\alpha_i b_i$) 방식
    4. 일종의 opimization problem으로 설정하는 방식
    5. Clustering을 통한 방식

<br>
<br>

# Static vs Dynamic Quantization
---
+ `Dynamic` quantization은 $\alpha, \beta$의 값을 runtime에 동적으로 결정하는 방식
+ 위에서 제시한 min/max 등의 방식과 같이 real-time을 보장해야하기에 상당한 overhead가 될 수 있음
+ `Static` quantization은 미리 정한 $\alpha, \beta$ 값을 계속해서 사용하는 방식
+ 해당 값을 미리 정하기 위한 다음과 같은 방식들이 존재한다
1. `calibration input`을 이용하는 방식
2. `Mean Squared Error (MSE)`를 최소화 하는 방식
3. `Entropy`를 이용하는 방식
4. `학습하면서 범위를 학습/유추` 하는 방식<br>

<br>
<br>

# Quantization Granularity
---
다음과 같이 다양한 수준(layer 단위, channel 별 등)에서 quantization을 진행하는 법이 존재함
1. Layerwise
    + $\alpha, \beta$값을 동일 layer 안에 존재하는 모든 필터들의 weight을 고려하여 결정하는 방식
    + 적용하기 가장 쉬운 방식이나, 값의 범위가 작은 필터의 경우 resolution이 낮아짐에 따라 성능 악화의 원인이 됨
2. Groupwise
    + Layer 내부를 그룹으로 나누어 그룹별로 다른 clipping range를 적용
3. Channelwise
    + 각 필터 채널별 clipping range를 적용하는 방식
    + 다른 채널과 독립적인 clipping range를 갖기 때문에 다른 방식에 비해 성능이 높음

<br>
<br>

# Post-Training Quantization (PTQ) vs Quantization-Aware Training (QAT)
---
Quantization 이후, 모델의 파라미터를 조정하는 방식으로 retraining을 하는 Quantization-Aware Training (QAT)과, retraining을 하지 않는 Post-Training Quantization (PTQ)가 존재함
+ 
## QAT
+ Quantization으로 인해, FP(Floating Point)로 수렴된 지점과 다른 지점으로 이동할 수 있음 = 성능 악화 가능
+ 따라서, quantization 이후 재-학습을 통해 더 좋은 성능의 지점을 찾도록 함
+ Forward, backward를 FP에서 진행하여 gradient update(zero grad. or high error 방지를 위해) 한 뒤, 다시 quantization
+ Quantizer의 경우 piece-wise flat operator로, 대부분의 값에서 gradient는 0임
+ 이를 해결하기 위해 `Straight Through Estimator (STE)`를 통해 gradient를 계산
+ STE의 대체 방식으로 `stochastic neuron approach`, `combinatorial optimization`, `target propagation`, `Gumbel softmax` 등이 제안됨
+ 
## PTQ
+ Quantization 이후 아무런 fine-tuning 없이 weight을 조절하는 기법
+ Ex) 평균과 분산의 내재된 편향성을 보정하는 방법 제안
+ Ex) 다른 layer 또는 channel의 균등화 할 경우 quantization error가 작아짐을 보임