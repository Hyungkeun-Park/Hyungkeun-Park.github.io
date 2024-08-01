---
title: "[Review] Convolutional Neural Networks using Logarithmic Data Representation, arxiv, 2016"
excerpt: "Non-uniform base-2 logarithmic quantization"

toc: true
toc_sticky: true

use_math: true

categories:
    - Quantization
tags:
    - 경량화
    - Quantization
last_modified_at: 2024-07-17T01:00:00
---

<!--bundle exec jekyll serve : 임시 확인-->

**[Convolutional Neural Networks using Logarithmic Data Representation](https://arxiv.org/abs/1603.01025)**<br>
<br>
<br>

**`Logarithmic-scale quantization`**을 이용한 최초(?)의 non-uniform quantization 논문으로 보여져 공부해 보았습니다


<center><img width="550" alt="image" src="https://github.com/user-attachments/assets/99470aaa-1cfa-461a-bc91-33e001b7d56f"></center>

Convolution layer와 Fully-connected layer 모두 matrix dot product 연산을 진행하게 되는데<br>
하드웨어에서의 곱 연산을 bit shift 연산으로 변경해줄수만 있다면 하드웨어적 측면에서의 상당한 이득을 볼 수 있다는 점에<br>
해당 저자들은 다음과 같은 방식을 제안하게 됩니다

# Method 1.
---

$$
\begin{align}
w^Tx &\cong \sum_{i=1}^n w_i\times2^{\tilde{x}_i} \newline
&= \sum_i^nBitshift ( w_i, \tilde{x}_i )
\end{align}
$$

$\tilde{x}_i=Quantize(log_2(x_i))$ 라고 정의할 때<br>
기존 Conv. & FC layer에서의 dot product를 $\tilde{x}_i$만큼 shift한 $w_i$의 합 으로 표현이 가능하다는 것을<br>
위 식을 통해 보여주고 있습니다 $\left(Bitshift(a,b)=a>>b\right)$<br>

그렇게 되면 비싼 곱셈기 대신 간단한 bit shifter로 dot product가 구현이 가능하니<br>
우선 하드웨어적으로 상당한 이득을 보게됩니다<br>

그리고 Quantization에 대해서는 두 가지 옵션이 존재합니다
1. Floor : $\lfloor log_2(w) \rfloor$ $\rightarrow$ MSB로부터 첫 1 bit가 포착되는 지점 반환
2. Round : integer part 계산 후 fractional part 이용하여 가까운 값 반환

# Method 2.
---
그렇다면 동일하게 weight에도 quantization을 적용하게 될 경우<br>
지수간 곱셈이 더하기 연산으로 바뀌게 되고, 결국 1이라는 숫자에 대한 bit shift로 변환이 됩니다<br>

$$
\begin{align}
w^Tx &\cong \sum_{i=1}^n 2^{Quantize(log_2(w_i))+Quantize(log_2(x_i))} \newline
&= \sum_ {i=1}^n Bitshift ( 1, \tilde{w}_i+\tilde{x}_i )
\end{align}
$$

# Accumulation in log domain
---
Conv. & FC layer에서의 각 원소에 대한 dot product가 bit shift로 변환이 될 수 있음을 보였으니<br>
모든 원소들에 대한 합은 어떻게 변환을 할 수 있는지에 대해 저자들은 다음과 같이 일반화 하였습니다<br>

$$
\tilde{s}_n \cong max(\tilde{s} _{n-1},\tilde{p}_n)+Bitshift(1,-|\lfloor \tilde{s} _{n-1}\rfloor-\tilde{p}_n|)
$$

# Experiment
실험에서 재미있던 점 몇 가지를 보자면
<center><img width="550" alt="image" src="https://github.com/user-attachments/assets/75731887-5ca4-420e-821b-e72fa377499c"></center>
위 그래프에서 가로축은 quantization에 의한 error로 볼 수 있고 세로축은 해당 error에서의 count를 뜻합니다<br>
log quantization은 linear quantization에 비해 상대적으로 작은 quantization error를 띔을 볼 수 있습니다

<center><img width="450" alt="image" src="https://github.com/user-attachments/assets/a3bc9fa7-5eee-4804-90f7-35a0a1e18c5a"></center>
그리고 base를 2가 아닌 $\sqrt2$를 이용한 quantization을 적용해야 CNN에서의 accuracy drop이 덜 한 것을 볼 수 있는데<br>
convolution layer의 weight에 대한 quantization error가 누적됨에 따라 base-2에서는 accuracy drop이 더 컸음을 알 수 있습니다

<center><img width="450" alt="image" src="https://github.com/user-attachments/assets/73374615-2b53-4f12-a3ac-4a6ef848a9f0"></center>
마지막으로, quantization을 학습에도 적용해 보았을 때<br>
log-quantization의 경우 backpropagated gradient를 quantization해도 학습이 잘 되는 것을 볼 수 있으나<br>
linear-quanitzation의 경우 qunatize하지 않은 floating point gradient를 그대로 사용해야만 학습이 가능했다고 합니다