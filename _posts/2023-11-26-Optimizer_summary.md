---
title: "Optimizer 정리"
excerpt: "알고 쓰자 Optimizer"

toc: true
toc_sticky: true

use_math: true

categories:
    - Optimizer
tags:
    - Optimizer
last_modified_at: 2023-11-26T16:52:00
---

<span style="font-size:200%">역전파(Back-Propagation)</span>
를 통한 Graident로 딥러닝 모델의 학습이 진행되는데<br>
각 Layer에 해당하는 **Graident를 어떻게 이용하여 Paramter를 업데이트 할 것인지**를 결정해 주는 것이<br>
바로 Optimizer의 역할이라고 볼 수 있다<br>
<br>
처음 딥러닝 공부를 시작할 때 각 Optimizer가 어떻게 고안됐는지는 나를 포함해 분명 다들 공부했을 것인데<br>

정작 사용할 때에는 `Adam`이 보통 성능이 잘나온다~ 라는 이유로 무작정 사용하다보니<br>
각 Optimzer가 정확히 어떤 역할을 하는지 까먹어서 정리를 다시 좀 해야겠구나 생각이 들었다<br>
<br>
<br>
<br>
깔끔하게 review한 다음 논문을 참고하여 정리해보도록 한다<br>
**[An overview of gradient descent optimization
algorithms](https://arxiv.org/abs/1609.04747)**
<br>
<br>
<br>

# 1. Gradient Descent 변형
---
우선 Gradient Descent의 변형들을<br>
얼마만큼의 데이터를 사용해 역전파를 진행하는지로 나누어 설명한다<br>

## 1) Batch Gradient Descent
역전파 라는 개념에 최초로 도입된 Optimizer로<br>
전체 학습 데이터에 대해 Gradient를 계산한 뒤 한 번만 파라미터를 업데이트한다<br>
<br>
즉, MNIST 데이터 셋으로 학습을 한다 생각하면<br>
학습 데이터 60,000장에 대해 다 봐야 파라미터 한 번 업데이트라는 것이다<br>
<br>
요즘 데이터를 억단위로 사용해서 학습하는 것을 고려한다면<br>
학습 속도 뿐만 아니라 메모리 또한 감당할 수 없다<br>
~~그러니 굳이 자세히 다룰 필요도 없다~~<br>
<br>

## 2) Stochastic Gradient Descent
Batch 전부를 사용해서 업데이트는 너무 비효율적이니<br>
**각 데이터 샘플에 대해서 파라미터를 업데이트하겠다**라는 것이다<br>
하지만 각 데이터에 제일 잘 맞는 방향을 찾아가는 꼴이다 보니 Fluctuation이 상당히 심하다<br>

<center><img width="193" alt="image" src="https://github.com/Hyungkeun-Park/Hyungkeun-Park.github.io/assets/21329629/c16ec8e2-00a5-4b49-b122-449488c76992"></center>

그렇기 때문에 우리가 원하는 Minimum에 도달하기 힘든데<br>
learning rate를 감소시키면 어느정도 minimum에 도달시킬 수 있을 것이다<br>
<br>

## 3) Mini-batch Gradient Descent
우리가 흔히 **프레임워크에서 사용하는 SGD가**<br>
**사실은 이 Mini-batch Gradient Descent에 해당한다**<br>
<span style="font-size:80%">~~엄연히 다르다는걸 정리하며 알게된건 함정~~<br></span>
<br>
Mini-batch 단위로 파라미터를 업데이트=일반적으로 잘 먹히는 방향으로 학습 진행<br>
<br>
당연히 1), 2)보다 안정적이고 효과도 좋을 수 밖에...<br>
<br>
<br>
<br>
하지만 Mini-batch Gradient Descent에게도 문제가 있었으니<br>

1. Learning rate 초기값 설정과
2. Learning rate 를 어떻게 스케쥴링할 것인지
3. 모든 파라미터를 동일한 learning rate로 업데이트하고
4. 안장점(Saddle point)에 걸릴 가능성이 크다

4번의 경우, 3차원으로 생각해보면<br>
x축으로는 최대점이고 y축으로는 최저점일 경우<br>
그냥 거기 머무르는게 optimizer한테는 어찌보면 최선이 되는 것이다<br>

그래서 이런 한계를 개선하기 위해 Optimizer의 알고리즘을 변형한 버전이 등장한다<br>
<br>
<br>

# 2. Gradient Descent 최적화 알고리즘 변형
---
## 1) 어디로 가야하오
아래 다룰 두 Optimizer는 SGD의 방향성에 대한 문제를 개선한 버전이다<br>

### (1) Momentum
**물리학에서 등장하는 그 모멘텀 개념**이 맞다<br>
왜 운동량이란 개념이 Optimizer에 등장하게 됐을까?<br>
<br>
SGD(mini-batch Gradient Descent)는 다음과 같은 oscillation 문제가 발생한다<br>
<center><img width="358" alt="image" src="https://github.com/Hyungkeun-Park/Hyungkeun-Park.github.io/assets/21329629/be4759fe-ea07-4e86-8b27-ced81cdc82cb"></center>
수평축으로는 계속 동일한 방향으로 진행한다 한들<br>
수직축에서의 경사가 급하다 보니 계속 요동치며 진행한다는 것이다<br>
<br>
경사면에서 움직이는 공을 생각해보자<br>
<br>
같은 방향으로 계속 진행할 때에는 가속도가 붙고<br>
경사를 다 내려가고 다시 올라가는 방향이 생길 경우에는<br>
그 반대로 힘을 받아 속도가 줄어드는 것을 쉽게 떠올릴 수 있다<br>
<br>
그런 물리적인 개념을 Optimizer에 적용한 것이 바로 Momentum이라고 할 수 있다<br>

$$
v_t=\gamma v_{t-1}+\eta \nabla_{\theta} J(\theta)
$$

$$
\theta=\theta-v_t
$$

이전 Gradient에 대한 정보를 현재 업데이트에 사용하겠다는 뜻이다 ($\gamma$는 보통 0.9 사용)<br>
그렇다면 위 그림과 같이 Gradient의 방향이 역전되는 경우에는 속도($v_t$)가 줄고<br>
Gradient의 방향이 일정한 경우 속도가 점점 가속되게 만들 수 있다<br>
<br>
p.s. Pytorch의 경우 SGD에 momentum 값을 주면 momentum으로 작동한다
<br>
<br>

### (2) Nesterov Accelerated Gradient (NAG)
momentum에서는 경사를 올라가야 속도가 감소하는데<br>
**경사에 올라 타기 전부터 속도를 줄이자** 라는게 NAG의 아이디어다<br>

$$
v_t=\gamma v_{t-1}+\eta \nabla_{\theta} J(\theta-\gamma v_{t-1})
$$

$$
\theta=\theta-v_t
$$

수식을 살펴보면<br>
현재 파라미터($\theta$)에서의 경사로 속도를 구하는 것이 아닌<br>
업데이트되면 갈 지점($\theta-\gamma v_{t-1}$)에서의 경사($\nabla_\theta J$)를 통해 속도($v_t$)를 미리 구하고<br>
파라미터를 업데이트 하겠다라는 뜻이다<br>
<center><img width="259" alt="image" src="https://github.com/Hyungkeun-Park/Hyungkeun-Park.github.io/assets/21329629/32c3eb77-8440-4b9d-8069-4334f3e708f8"></center>

위 그림에서 파란 선은 momentum의 $\theta$ 업데이트를 보여주는데<br>
$\eta \nabla_{\theta} J(\theta)$(짧) + $\gamma v_{t-1}$(긴)를 순서대로 벡터의 합으로 나타낸 모습이고<br>
<br>
초록 선은 NAG의 업데이트를 보여주는데<br>

$\gamma v_{t-1}$(갈색)+$\eta \nabla_{\theta} J(\theta-\gamma v_{t-1})$(주황색)의 벡터 합으로 나타낸 모습이다<br>
<br>

## 2) 얼마나 가야하오
다음으로 다룰 Optimizer는 파라미터별로 다른 learning rate를 사용한다<br>
(왜? : 학습이 덜 된 feature에는 상대적 가중을 주고 싶기 때문)<br>

### (1) Adagrad
Adagrad는 파라미터 업데이트의 빈도에 따라 learning rate를 다르게 주겠다 라는 아이디어이기에<br>
희소(sparse) 데이터를 다루는데 잘 맞다고 알려져있다<br>
$g_{t,i}$를 시간 t에서의 gradient라고 하고 $G_t$를 gradient의 제곱합(Sum of square)을 갖는 대각 행렬이라고 하면<br>
Adagrad는 다음과 같이 learning rate를 자체적으로 조절하게 된다(초기값은 보통 0.01)<br>

$$
g_{t,i}=\nabla_{\theta_t}J(\theta_{t,i})
$$

$$
\theta_{t+1,i}=\theta_{t,i}-\frac{\eta}{\sqrt{G_{t,ii}+\epsilon}}g_{t,i},\ \epsilon≒1e-8
$$

하지만 Adagrad 또한 취약점이 존재하는데<br>
gradient의 제곱합을 계속 누적하기 때문에 **학습을 할 수록 learning rate는 감소하다가 결국 0으로 수렴**하게 된다<br>
<br>
이런 문제점을 개선하기 위해 Adadelta가 등장하게 된다<br>
<br>
<br>

### (2) Adadelta
Adagrad가 이전의 모든 gradient를 제곱 누적하기 때문에 문제가 발생했으므로<br>
**최근의 graident만 누적하자** 라는게 Adadelta의 아이디어이다<br>
<br>
최근 gradient만 누적하자 = 최근 값을 저장해야함<br>
→ **sum of sqaure를 줄여나가자 (직전 값만 저장하면 됨)**<br>

$$
E[g^2]_t=\gamma E[g^2] _{t-1}+(1-\gamma)g^2
$$

$$
\Delta\theta_t=-\frac{\eta}{\sqrt{E [ g^2 ]_t+\epsilon}} g_t
$$

$$
RMS[g]_t=\sqrt{E [ g^2 ]_t+\epsilon}
$$

그런데, 한 가지 저자가 생각했던 문제점은<br>
**Parameter의 unit과 graident의 unit이 같지 않다**는 점을 지적했다<br>
따라서, 위 식이 아래와 같이 바뀌어야 한다고 말한다<br>

$$
E[\Delta \theta^2]_t=\gamma E[\Delta \theta^2] _{t-1}+(1-\gamma)\Delta \theta^2
$$

$$
RMS[\Delta \theta]_t=\sqrt{E [ \Delta \theta^2 ]_t+\epsilon}
$$

하지만 위 식에서 $RMS[\Delta \theta]_t$를 알 수 없으니<br>
($E[\Delta\theta^2]_t$를 구하기 위해 $\Delta\theta$필요 ↔ $\Delta\theta$구하기 위해 $E[\Delta\theta^2]_t$필요)<br>
$RMS[\Delta \theta]_t$을 $RMS [ g ]_t$로 근사하고, $\eta$대신 $RMS[\Delta \theta ] _{t-1}$로 대체하여

$$
\Delta \theta_t=-\frac{RMS [ \Delta \theta ] _{t-1}}{RMS [ g ]_t } g_t
$$

$$
\theta_{t+1}=\theta_t+\Delta \theta_t
$$

최종적으로 위와 같이 parameter를 업데이트 진행하게 된다<br>
<br>
<br>

### (3) RMSprop
RMSprop은 우리들의 힌턴 선생님께서 제안한 방식으로 Adadelta와 거의 동시에 개발되었다고 한다<br>
<br>
근데 신기하게도 상당히 Adadelta와 유사한 모습을 띄고있다(ㄷㄷ)<br>

$$
E[g^2]_t=0.9 E[g^2] _{t-1}+0.1g^2
$$

$$
\theta _{t+1} = \theta _t-\frac{\eta}{\sqrt{E [ g^2 ]_t+\epsilon}} g_t
$$

힌턴 선생님께서 말씀하시길 $\gamma=0.9, \eta=0.001$이 좋다고 하셨다고 한다<br>
<br>
<br>

### (4) Adam
