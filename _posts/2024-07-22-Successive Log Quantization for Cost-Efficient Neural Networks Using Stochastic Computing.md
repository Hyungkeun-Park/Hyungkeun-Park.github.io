---
title: "[Review] Successive Log Quantization for Cost-Efficient Neural Networks Using Stochastic Computing, DAC, 2019"
excerpt: "Successiv Log Quantization"

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

**[Successive Log Quantization for Cost-Efficient Neural Networks Using Stochastic Computing](https://dl.acm.org/doi/10.1145/3316781.3317916)**<br>

해당 논문에서는<br>
**`Log quantization 회로를 재사용`**을 이용한 quantization error를 최소화하는 방법에 대해 다루고 있습니다
<br>
<br>

# Succesive Log Quantization
---
Successive Log Quantization (SLQ)는 log quantization에 의해 발생하는 quantization error를<br>
quantize word의 series를 통해 error를 최소화 하려는 방법을 뜻합니다
<center><img width="550" alt="image" src="https://github.com/user-attachments/assets/250fda96-7fdf-4e22-a009-ef3a2d44effa"></center>
위 table에서 0.4375라는 값에 대해 log quantization을 적용할 경우<br>
가장 가까운 quantization point인 0.5로 quantize를 진행해 0.0625만큼의 quantization error가 발생하게 됩니다<br>

하지만 저자들이 제안한 SLQ를 적용할 경우<br>
0.5 + (-0.0625) (00001 + 11100(보수)) 또는 0.25 + 0.125 + 0.0625 (00010 00011 00100) 와 같이 표현이 가능해<br>
quantization error를 최소화 시킬 수 있습니다<br>
(이진수 값은 $2^{-i}$의 i에 해당)<br>

결과적으로 SLQ의 각 quantization step을 다음과 같이 표현할 수 있으며<br>
잔차(residue) 값인 $(x-\tilde{x} _i)$가 threshold $\theta$값 보다 클 경우<br>
그리고 설정해준 max length L보다 작을 경우<br>
계속해서 quantize 과정을 반복합니다<br>

$$
\begin{align}
\tilde{x} _0 &\equiv 0 \newline
q_i&=LogQuant(x-\tilde{x} _{i-1}),\quad i\geq 1
\end{align}
$$

위와 같이 가변 길이로 quantization을 진행할 경우<br>
**`얼마만큼의 길이로 해당 값을 quantize하는지`**를 알려줘야하기 때문에<br>
SLQ를 통해 quantize한 값의 길이를 알려주는 방식을 다음과 같이 두 가지 제안합니다
<br>
<br>

# Encoding
---
<center><img width="550" alt="image" src="https://github.com/user-attachments/assets/b07d38c3-8076-4489-a78b-faf85d49ac66"></center>
1. ## Tagging
Tagging 방식은 각 quantize word 앞에 1bit을 추가함으로써, 추가적인 word가 더 붙게 되는지 를 알려주는 방식을 뜻합니다<br>
간단히 1bit 추가만으로 표현할 수 있다는 장점이 있지만, 모든 word마다 1bit의 오버헤드가 추가로 필요로 하다는 단점이 존재합니다
2. ## Special Code
Special Code 방식은 quantized word가 한 개 이상 필요할 때, 특정 값(-M)을 삽입하는 방식을 뜻합니다