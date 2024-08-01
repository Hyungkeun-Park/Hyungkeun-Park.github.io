---
title: "[Review] Averaging Weights Leads to Wider Optima and Better Generalization, IJAI, 2018"
excerpt: "Stocahstic Weight Averaging"

toc: true
toc_sticky: true

use_math: true

categories:
    - Review
tags:
    - Review
    - Weight Averaging
last_modified_at: 2024-07-23T01:00:00
---

<!--bundle exec jekyll serve : 임시 확인-->

**[Averaging Weights Leads to Wider Optima and Better Generalization](https://arxiv.org/abs/1803.05407)**<br>

해당 논문에서는<br>
학습을 통해 얻어지는 **`weight의 평균을 이용`**할 경우 더 좋은 일반화의 성능을 얻을 수 있다고 합니다
<br>
<br>

# Stochastic Weight Averaging (SWA)
---
<center><img width="550" alt="image" src="https://github.com/user-attachments/assets/50f72d1b-dc74-4d89-b534-a7a7149639ce"></center>

위와 같은 방식으로, 학습을 통해 얻어진 weight의 평균으로<br>
더욱 generalization이 잘 된 모델 weight을 얻을 수 있다고합니다<br>

Train, Test 간의 I.I.D (Independent Identical Distribution)을 가정하지만<br>
실제 Train, Test 간의 error (loss) surface를 확인해보면 약간의 shift가 존재하기에<br>
Shift된 test error surface에서의 더욱 최적화된 optima를 찾기에는 평균 weight을 사용하는 것이 좋다는 것입니다<br>

추가로, 기존 연구에 의하면 local optimum의 너비가 일반화 성능과 연관되어있다 라는 내용이 있는데<br>
(broad optima를 갖고 있어야 약간의 perturbation에 대해서도 optimal한 결과를 낼 수 있기 때문에)<br>

이와 관련하여, 저자들은 uniform random 10 방향에 대해 weight을 shift했을 때<br>
Stochastic Gradient Descent (SGD)로 학습한 weight에서의 shift보다<br>
**SWA로 얻어진 weight에서의 shift가 error (loss) 면에 있어서 broad한 optimum**을 보여줌을 확인하였습니다<br>

<center><img width="750" alt="image" src="https://github.com/user-attachments/assets/842c4f7b-cae7-4633-8f28-ba4f1163df50"></center>
<br>

SWA는 **`torchcontrib`** 라는 라이브러리를 추가로 설치할 경우, Pytorch에서도 바로 사용이 가능한 것으로 보여집니다<br>

해당 논문에서 제시한 바와 같이, 전체 training의 75%를 진행시킨 뒤<br>
cyclical learning rate (Lr)를 통해 학습을 진행시키며 minimum Lr에서의 model weight averaging을 하고<br>
최종적으로 얻어진 weight에 대한 Batch Normalization의 running mean, standard deviation을 계산합니다<br>
<br>

<center><img width="600" alt="image" src="https://github.com/user-attachments/assets/f27d4b4a-1f74-4cd5-8c92-cfcd1b3b86ba"></center>
자세한 사용법이나 내용은 다음의 URL을 참고하시면 됩니다 ([https://pytorch.org/blog/stochastic-weight-averaging-in-pytorch/](https://pytorch.org/blog/stochastic-weight-averaging-in-pytorch/))
