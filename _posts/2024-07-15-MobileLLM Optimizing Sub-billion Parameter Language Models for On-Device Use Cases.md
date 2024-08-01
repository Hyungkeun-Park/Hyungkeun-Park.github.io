---
title: "[Review] MobileLLM: Optimizing Sub-billion Parameter Language Models for On-Device Use Cases, ICML, 2024"
excerpt: "Sub-billion scale LLM"

toc: true
toc_sticky: true

use_math: true

categories:
    - Review
tags:
    - LLM
    - 경량화
    - Knowledge Distillation
last_modified_at: 2024-07-15T01:00:00
---

<!--bundle exec jekyll serve : 임시 확인-->

**[MobileLLM: Optimizing Sub-billion Parameter Language Models for On-Device Use Cases](https://arxiv.org/abs/2402.14905)**<br>
위 논문에서 적용한 기술에 대해 간단 정리한 내용입니다
<br>
<br>

해당 논문에서는
1. SwiGLU FFN
2. Forcing lanky (deep and thin) architectures
3. Revisiting embedding sharing method
4. Utilizing grouped query attention
5. Immediate block-wise layer-sharing method
위 방법들을 통해 sub-billion scale LLM의 성능을 향상시킬 수 있었다고 합니다
<br>
<br>

LLM이 핫한 만큼 LLM의 사이즈를 줄임과 동시에 성능은 최대한 유지하려고 하는 노력 또한 많이 보이고 있기에<br>
해당 연구에서는 어떤 방법들이 적용되었는지 궁금해서 한 번 살펴보려고 합니다
<br>
<br>

# SwiGLU FFN
---
단순한 feed-forward network (FFN)의 구조인 $FC \rightarrow ReLU \rightarrow FC$ 구조를<br>
125M 사이즈의 모델에서 SwiGLU로 변경함에 따라 zero-shot reasoning task에서의 성능이 42.6에서 43.9로 향상되었다고 합니다

SwiGLU는 Swish activation (Swi) + Gated Linear Units (GLU) 로<br>
최근 LLM에서 많이 사용되는 activation 이라고 합니다

+ ## [Swish activation](https://arxiv.org/abs/1710.05941)
<center><img width="700" alt="image" src="https://github.com/user-attachments/assets/cd5f1b19-72ed-4896-a3c7-b4b91c5d5f43"></center>

$$
Swish(x) = x\sigma(\beta x)
$$

$$
\left(\sigma(x)=Sigmoid=\frac{1}{1+e^{-x}}\right)
$$

Swish activation에서의 $\beta$는 상수 또는 learnable parameter로 설정할 수 있다고 합니다
+ $\beta=0$ : Scaled linear function $f(x)=\frac{x}{2}$
+ $\beta=1$ : Sigmoid-weighted Linear Unit (SiLU)
+ $\beta\rightarrow\infty$ : ReLU

$\beta$의 값은 위 그림에서와 같이, 도함수의 값이 0과 1에 도달하는 속도를 제어하는 역할을 하는데<br>
$\beta$를 learnable parameter로 모델(Mobile NASNet-A)을 학습시켰을 때, 학습된 $\beta$값이 1이 우세했다는 것은
ReLU가 최신 모델 구조에서는 최선이 아닐수도 있다 라는 점을 시사합니다

+ ## [Gated Linear Unit](https://arxiv.org/abs/1612.08083)
LSTM의 경우 gating mechanism을 통해 이전의 정보들을 계속해서 사용할 수 있도록 하였는데 반해<br>
Convolutional network에서는 이러한 류의 문제가 발생하지 않아 forget gate를 사용하지 않았다

따라서 output gate만을 갖는 모델에 대해 어떤 단어 or feature가 유용할지 선택하는 mechanism을 고안하였다<br>
<br>

$$
▽[tanh(X)⊗\sigma(X)]=tanh^\prime(X)▽X⊗\sigma(X)+\sigma^\prime(X)▽X⊗tanh(X)
$$

위 수식은 기존의 LSTM-style의 gating 방식에 대한 미분 식으로<br>
$tanh^\prime(X),\ \sigma^\prime(X)$ 항에 의해 layer를 쌓음에 따라 gradient vanishing 현상이 발생하였는데,<br>
이와 반대로 gated linear unit의 경우, 다음의 수식처럼 $\sigma(X)$가 곱해지는 식을 가져<br>
일종의 skip connection의 형식을 띄고 있음 알 수 있습니다

$$
▽[X⊗\sigma(X)]=▽X⊗\sigma(X)+X\sigma^\prime(X)▽X
$$

<br>
<br>

# Depth vs Width in architecture
---
Transformer 모델은 파라미터의 수가 많을수록, training data의 수가 많을수록 성능이 더 좋다 라는 믿음이 깔려있는데 반해,<br>
**`더 작은 모델의 경우에는 그렇지 않다`** 라고 저자들은 얘기하고 있습니다<br>

총 19개의 모델 (10개의 ~125M, 9개의 ~350M 모델)을 가지고 실험을 한 결과, **`모델의 너비 보다 깊이가 더 중요한`** 요소임을 밝혀냈다고 합니다
<center><img width="700" alt="image" src="https://github.com/user-attachments/assets/6a079c5e-d804-4039-97e6-88b4949c7813"></center>
<br>
<br>

# Embedding sharing
---
Embedding layer의 크기가 sub-billion scale의 언어 모델에서 차지하는 비율은 Billion-scale의 언어 모델에 비해 상당히커지게 됩니다<br>

+ Input embedding = Output embedding = vocab. $\rightarrow$ embedding

input-output embedding은 결국 동일한 사이즈(vocab., embedding)이기 때문에, 저자들은 embedding layer를 공유함으로써 parameter 수를 추가로 줄입니다<br>

공유에 따른 성능 악화가 존재하나, 감소한 만큼의 parameter로 depth를 추가로 쌓으면서 성능이 증가하여 오히려 최종적으로 이득을 보았다고 합니다
<br>
<br>

# [Grouped Query Attention](https://arxiv.org/abs/2305.13245)
---
기존 Multi-query attention (MQA)은 key-value head를 한 개만 사용함으로써 decoder의 속도를 끌어올린 방법입니다<br>
하지만, head를 한 개만 사용함에 따라 성능 또한 상당히 악화되는 부작용 또한 존재했는데<br>
<br>
Decoder의 크기와 속도, 성능 유지까지 모두 얻기 위해 multi-head attention (MHA)와 MQA의 중간인<br>
**`Grouped-query attention (GQA)`**라는 방식이 제안되었었고<br>
해당 방법을 MobileLLM에서 적용하였다고 합니다<br>
<center><img width="600" alt="image" src="https://github.com/user-attachments/assets/0fb99f05-e613-42bf-a2ff-d59f92cba527"></center>
<br>
각 group의 key와 value head는 기존 모델의 head들의 weight을 mean pooling함으로써 구현합니다
<br>
MobileLLM 논문의 저자들은 GQA로 모델의 크기를 감소시킴과 동시에, embedding size는 증가시킴으로써<br>
125M 모델에 대해 0.4% 성능 향상을 이뤄냈다고 합니다
<br>
<br>

# Immediate block-wise layer-sharing
---
실험적으로 모델의 깊이를 더 깊게 쌓는것이 중요하다고 밝힌데에 따라<br>
저자들은 layer sharing 기법을 추가적으로 제안하는데<br>

모델의 사이즈를 키우지 않으면서도 모델의 깊이를 증가시키기 위해 **`동일 layer를 여러 번 통과하게끔`** 하였습니다<br>

<center><img width="550" alt="image" src="https://github.com/user-attachments/assets/d039565e-ba0e-4315-a9af-7b3bb74a1e07"></center>

이 때, SRAM 자원이 transformer block 1개 정도만을 지니고 있을 수 있기에<br>
cache에 특정 block의 weight을 저장해둔 뒤, 해당 block에 대해 연산을 두 번 진행하는 것으로써 (위 그림의 (b))<br>
SRAM→DRAM간의 weight 전달 시간을 없앰과 동시에 성능을 올릴 수 있엇다고 합니다
