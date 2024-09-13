---
title: "[Review] GENIE: Show Me the Data for Quantization, CVPR, 2023"
excerpt: "Zero Shot Quantization"

toc: true
toc_sticky: true
use_math: true

categories:
    - Review
tags:
    - Review
    - Quantization
last_modified_at: 2024-09-10T00:00:01
---

<!--bundle exec jekyll serve : 임시 확인-->


**[GENIE: Show Me the Data for Quantization](https://openaccess.thecvf.com/content/CVPR2023/html/Jeon_Genie_Show_Me_the_Data_for_Quantization_CVPR_2023_paper.html)**
<br>
<br>

해당 논문은
1. GENIE-D : Zero-shot Post-Training Quantization (PTQ)을 위한 Data Distillation
2. GENIE-M : Uniform quantization에서의 scale을 learnable parameter로 PTQ하는 방식
에 대한 논문입니다.

<br>
<br>

# GENIE-D : Data Distillation
---
일반적으로 Zero-shot Quantization을 위한 데이터 합성시<br>
모델의 batch normalization layers (BNS)에 학습된 $\mu$, $\sigma$ 값을 이용함으로써<br>
다음의 수식(BNS loss)을 만족하는 데이터를 합성해 낸다고 합니다<br>

$$
\mathcal{L}_{BNS}^D = \sum_{l=1}^L (\lVert \mu _l^s-\mu _l \rVert^2 + \lVert \sigma _l^s - \sigma_l \rVert^2)
$$

위 식에서, $\mu _l^s / \sigma _l^s$ 와, $\mu _l / \sigma _l$ 은 각각 합성된 데이터에 대한 파라미터와, 학습된 파라미터 값을 뜻하는 것으로<br>
합성한 데이터를 모델에 통과시켰을 때, 기존 모델에 학습된 파라미터와 최대한 동일한 입력을 찾아나간다고 보시면 됩니다<br>
<br>

데이터 합성 방식에는 크게 두 가지가 존재한다고 합니다
1. Generator-based approach (GBA) : Generator를 통한 데이터 합성
2. Distill-based approach (DBA) : 특정 loss를 만족하는 데이터 직접 합성

각각의 방법에는 장단점이 존재하는 것으로 보여지는데

+ Generator-based approach (GBA)<br>
    \+ Generator를 이용한 합성이므로 데이터를 무한대로 생성 가능함<br>
    \- Gen. 학습에 오랜 시간이 소요되며, Quantization에 필요한 정보들이 중복될 수 있음
+ Distill-based approach (DBA)<br>
    \+ 데이터에 대해 직접적으로 에러가 전파돼 수렴 속도가 빠름<br>
    \- 합성 데이터간 상호 관계가 없음<br>

저자들은 위 두 방식의 장점을 혼합 + Generative Latent Optimization (GLO)에서 영감을 받아<br>
**`Generator로 생성을 하되, Generator에 입력을 넣어주는 latent z에 대해 직접 에러를 전파`**하는 것을 제안합니다<br>

`그렇다면 Variational Auto-Encoder (VAE)와 다를 바가 무엇이냐?` 라는 물음에는<br>
VAE에서의 Decoder(Generator)는 실제 입력 $x$와, 생성 데이터 $x^\prime$간의 오차를 줄이고<br>
True posterior인 $p_\theta(z|x)$을 추정하는 encoder와의 상호작용을 통해서 variational inference를 진행하지만<br>

제안 방식에서는 variational inference 없이 $x$를 추정하며<br>
실제 샘플이 존재하지 않고, GBA는 BNS loss를 통해 실제 데이터의 분포를 간접적으로 계산한다<br>
라고 차이점을 언급하는 것으로 보여집니다<br>

<center><img width="600" alt="image" src="https://github.com/user-attachments/assets/a3b3fd83-d3a7-48dd-b72a-6c03f84f1dbe"></center>

## Swing Convolution
위에서 제안한 방식을 이용할 때, 사전 학습된 모델의 convolution layer을 **`Swing Convolution`**으로 대체합니다<br>
Swing convolution은 convolution layer의 stride 값 n을 `stochastic`하게 연산하도록 한다고 합니다<br>

처음에는 무슨 말인지 이해가 되지 않았는데 Figure 4를 보고 이해한 바로는<br>
1. Reflection padding을 통해 기존의 feature map보다 확장을 한 다음<br>
2. 원래의 feature map의 크기와 동일해지게끔 random crop
3. 그 다음 기존과 동일한 convolution 연산 진행
을 함으로써, 특정위치의 feature map과 동일한 연산을 하는 것이 아닌<br>
random하게 연산을 하게끔 변경해 줌으로써<br>
기존 distill 방식의 문제점인 `checkerboard` 현상을 개선한다고 합니다


<center><img width="400" alt="image" src="https://github.com/user-attachments/assets/a6a449ef-047c-4894-9cb1-047e5b8ca5ac"></center>

<br>
<br>


# GENIE-M : Quantization Algorithm
---
그 다음으로, 저자들은 Quantization algorithm도 추가로 제안합니다<br>

이는 기존의 AdaRound라는 방법과 크게 다른 것은 아닌 것으로 보여지는데<br>

기존 AdaRound의 경우<br>
Base integer를 다음과 같이 정의하고

$$
B:=clip\left(\lfloor\frac{W}{s}\rfloor,n,p\right)
$$

이에 softbit $V\in[0,1]^{m\times n}$ 을 추가함으로써 quantization을 진행합니다

$$
W^q = s \cdot (B+V)
$$

근데, 기존 AdaRound의 경우, scale s값과 softbit V를 동시에 최적화를 진행하지 않는데<br>
scale이 바뀔 경우, 그에 따른 B 값도 변화 → 결과적으로 softbit까지 변해야하는 문제가 발생하기 때문입니다<br>

이 문제를 해결하기 위해서 저자들은 단순히 B는 scale의 변화와 무관하게, 초기화만 진행해줌으로써 문제를 해결했다고 합니다

<center><img width="400" alt="image" src="https://github.com/user-attachments/assets/ee8a0510-daed-457d-b007-fe1aef583b1f"></center>