---
title: "[Review] Data-Free Quantization via Pseudo-label Filtering, CVPR, 2024"
excerpt: "Zero Shot Quantization"

toc: true
toc_sticky: true
use_math: true

categories:
    - Review
tags:
    - Review
    - Quantization
last_modified_at: 2024-09-11T00:00:01
---

<!--bundle exec jekyll serve : 임시 확인-->


**[Data-Free Quantization via Pseudo-label Filtering](https://openaccess.thecvf.com/content/CVPR2024/html/Fan_Data-Free_Quantization_via_Pseudo-label_Filtering_CVPR_2024_paper.html)**
<br>
<br>

해당 논문은<br>
Zero shot quantization을 위한 데이터 생성 및 quantization을 위한 training 제안 논문으로<br>
1. Self-Entropy을 이용한 데이터 필터링
2. Confidence에 따른 psuedo labeling
위 두 가지를 이용하여 quantized 모델의 성능에 있어서 state-of-the-art를 달성했다고 합니다<br>

<br>
<br>

# Reliability Filtering
---
Zero-shot quantization을 위한 데이터 생성시, 일반적으로<br>
Batch-Norm의 statistics (mean, statndard deviation)을 이용하여<br>
기존 pre-train된 모델의 statistics와 동일한 값을 갖는 데이터를 합성해 내도록 학습시킵니다<br>

$$
\mathcal{L}_{BN} = \sum _{l=1}^L(\lVert\mu _l^P - \mu _l^s\rVert _2^2 +\lVert \sigma _l^P -\sigma _l^s \rVert _2^2)
$$

$$
\mathcal{L}_{DATA} = \mathcal{L}_{BN} + \gamma\cdot\mathcal{L}_{IL}
$$

위의 loss식을 만족하는 data를 생성하는데<br>
$\mathcal{L}_{IL}=\sum _{i=1}^N CE(P(\hat{x}_i),\hat{y}_i)$ 은 pre-train된 모델의 Cross-entropy loss가 작아지는 데이터를 생성하도록 합니다<br>

그런데, pre-train된 모델의 경우, 원 이미지 데이터셋에 대해 entropy가 작아지도록 학습하였기 때문에<br>
entropy가 낮은 합성 데이터가 quantized 모델의 지도학습에 도움이 될 것이고<br>
entropy가 높은 합성 데이터의 경우 모델의 robustness를 발전시킬 수 있을 것이라고 저자들은 얘기합니다<br>

따라서 학습의 초반에는 모델의 성능에 영향을 많이 끼치기 때문에<br>
high-reliable 데이터의 기준을 정하는 threshold의 값을 높게 정하고<br>
학습이 진행됨에 따라, threshold 값을 낮추는 다음과 같은 방식을 사용한다고 합니다<br>

$$
t(threshold)=T_u-f_t(epoch)(T_u-T_l)
$$

$T_u$ 와 $T_l$ 는 각각 theshold의 상하한을 정하는 값이고, $f_t(epoch)=\frac{epoch}{E}$를 뜻합니다

<br>
<br>

# Multiple Pseudo-Labels & Labeling
---
위에서 말한, 데이터에 대한 필터링을 거친 뒤에 해당 데이터에 대한 라벨링에 다음의 처리를 해준다고 합니다<br>

1. High-reliable 데이터에 대해서는 pre-train 모델의 argmax 클래스 라벨을 그대로 사용
2. Low-reliable 데이터에 대해서는 pre-train 모델의 가장 높은 2개의 클래스 라벨을 사용

High-reliable 데이터에 대해서는 다음과 같이 일반적인 CE를 사용하고

$$
\mathcal{L}_{CE}^h = \frac{1}{N_h}\sum _{i=1}^{N_h}CE(P_q(\hat{x}_i^h),\hat{y}_i^m) 
$$

Low-reliable 데이터에 대해서는 2개의 클래스를 라벨로 사용한다고 했는데<br>
Cross-entropy loss에서 각각의 라벨에 대한 loss의 비율을 다음과 같이 조절한다고 합니다<br>
(p : primary label, s : secondary label)

$$
\begin{align}
\mathcal{L}_{CE}^l = \frac{1}{N}\sum _{i=1}^{N_l}&(\lambda _i^P\cdot CE(P_q(\hat{x}_i^l),\hat{y}_i^p) \\
&+\lambda _i^s\cdot CE(P_q(\hat{x}_i^l),\hat{y}_i^s))
\end{align}
$$

$$
\lambda _i^p=\frac{P(\hat{x}_i^l,p)}{P(\hat{x}_i^l,p)+P(\hat{x}_i^l,s)} \\
\lambda _i^s=\frac{P(\hat{x}_i^l,s)}{P(\hat{x}_i^l,p)+P(\hat{x}_i^l,s)}
$$

결과적으로 모델 학습에 사용되는 CE는 다음과 같이 두 CE의 합을 사용한다고 합니다

$$
\mathcal{L}_{CE}^{total}=\mathcal{L}_{CE}^h + \beta \cdot \mathcal{L}_{CE}^l
$$

<br>

그리고, quantized 모델이 원래의 FP32 모델과 유사한 feature map을 갖도록 학습시키기 위해서<br>
layer 별 MSE loss를 추가하고, 추론 결과에 대한 KL-divergence를 취함으로써 distillation을 한다고 합니다<br>

이 때, MSE loss에 대한 정도는 hyperparameter $\tau$로 조절을 한다고 합니다

$$
\mathcal{L}_{MSE}=\sum _{k=1}^L\left( \frac{1}{N}\sum _{i=1}^N(f_k(\hat{x}_i-f_k^q(\hat{x}_i))^2 \right)
$$

<br>
<br>
<center><img width="700" alt="image" src="https://github.com/user-attachments/assets/44a2d55d-2dd2-4b0c-91fa-00ed353983f6"></center>