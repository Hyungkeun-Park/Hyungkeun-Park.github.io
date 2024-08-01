---
title: "Fixed vs Floating point"
excerpt: "소수점 표현 방식"

toc: true
toc_sticky: true

use_math: true

categories:
    - Quantization
tags:
    - Quantization
last_modified_at: 2024-07-13T01:00:00
---

<!--bundle exec jekyll serve : 임시 확인-->

소수점 표현 방식은 크게 두 가지로 나뉘게 되는데<br>
**`Fixed point`, 고정 소수점 방식**과<br>
**`Floating point`, 부동 소수점 방식**이다.<br>

컴퓨터에서는 2진수인 binary로 값을 표현하기 때문에<br>
10진수의 정수 데이터 또한 2진수의 합으로 표현하듯이<br>
실수 데이터 또한 2진수의 합으로써 표현하게 된다<br>

$$
\begin{align}
10_2 &= 1 * 2^3 + 1 * 2^1 \newline
1.25_2 &= 1 * 2^0 + 1 * 2^{-2}
\end{align}
$$

# Fixed Point, 고정 소수점
---
+ 정수를 표현하는 비트, 소수를 표현하는 비트 수를 미리 정함 (고정)
+ Sign 1bit + Integer 15bit + Fraction 16bit = 32bit 으로 구성
+ Int. Frac. 각각에 대해 고정된 bit 할당에 따른 **표현 범위 제한**

# Floating Point, 부동 소수점
---
+ Sign 1bit + Exponent(지수부) 8bit + Mantissa(가수부) 23bit = 32bit 으로 구성
+ 다음과 같은 방식으로 실수를 표현하게 됨
1. 표현하고자 하는 숫자의 부호에 따라 Sign bit에 값을 할당<br>
    Ex) $-118.625$ : 음수 $\rightarrow$ sign bit = 1
2. 표현하고자 하는 숫자를 정규화<br>
    Ex) $-118.625 = 1110110.101 \rightarrow 1.110110101 * 2^6$
3. 지수값에 Bias를 더한 값을 Exponent 항에 할당<br>
    Ex) $6+127(IEEE\ 754) = 133 = 10000101_2$
4. 정규화 한 값의 소수점 오른쪽 값을 Mantissa 항에 할당<br>
    Ex) $110110101 \rightarrow 110110101 + 00000000000000$
5. $-118.625 = 1 / 10000101 / 11011010100000000000000$