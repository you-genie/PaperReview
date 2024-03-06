# Rotary Position Embedding (RoPE)

## Description

### Summary

* 현재 long context에 쓰이고 있는 RoPE란 무엇인가?

***

* name: RoFormer: Enhanced Transformer with Rotary Position Embedding
* link: [https://paperswithcode.com/paper/roformer-enhanced-transformer-with-rotary](https://paperswithcode.com/paper/roformer-enhanced-transformer-with-rotary)
* code: [https://github.com/lucidrains/performer-pytorch](https://github.com/lucidrains/performer-pytorch)

## Introduction

현 PLM의 문제점은 대체로 self-attention이 position 정보를 잘 잡지 못한다는 것이다. 기존 연구에서는 position embedding 등 position information을 더하기 위해 추가적인 position 값을 더해주었다. 이 논문에서는 rotary position embedding이라는 새로운 기법을 소개한다.

### RoPE의 contribution

* linear self-attention에 잘 적용되지 않던 기존 기법들과는 달리, RoPE는 잘 적용이 된다.
* flexible한 context 길이를 지원한다.

## Methods

### Relative Position Embedding

Position 정보를 attention 계산에 어떻게 사용할 수 있는가? 기존 방식에서는 key-query matching 행렬에 position embedding을 더해서 이를 해결했다.

기존의 position embedding은 단순하게 index를 기반으로 한다. 예를 들면, 0, 1, 2, 3, 4, 5, 6.

relative position embedding에서는 '현재 토큰'을 기준으로 상대적 거리를 표현한다. 예를 들어 idx 3번을 기준으로 한다면, -3, -2, -1, 0, 1, 2, 3 이런 식이다. 이렇게 연산하게 되면 상대적 거리를 고려하기 때문에 '단어와 단어 사이'를 조금 더 이해하는 데 도움을 준다.

물론 위의 기법들은 전부 기존 연산값에 **position 값을 추가**하는 식으로 이루어진다.

$$
q_mk_n = x_m^TW_q^TW_kx_n + x_m^TW_q^TW_kp_n + p_m^TW_q^TW_kx_n + p_m^TW_q^TW_kp_n
$$

가장 general한 연산식이며, 값-값 / 값-위치값 ... 등의 정보 관계를 다 더해서 연산한다.

Relative Position Embedding의 단점

* efficient attention에의 적용이 어렵다. efficient attention의 경우 기존 attention식을 틀어서 연산한다. $$sim(q_m, k_n) = exp(q_m^Tk_n/ \sqrt{d})$$이게 원래 transformers연산식인데, 이 sim을 분할해서 $$sim(q_m, k_n) = \phi(q_m)^T\varphi(k_n)$$으로 연산하게 하는 거라서... 위의 식처럼 바로 qk연산을 할 수 없다.
* 보면 알겠지만 기존의 attention값을 4배로 늘렸다. 따라서 연산량이 엄청 늘어났다. relative position embedding 다른 논문들을 보면 그래서 저 네 개의 항 중 몇 개를 필요없다고 없애려는 작업을 좀 한다.

### Rotary Position Embedding

efficient attention에서도 적용을 시키기 위해서 기존 접근법과 궤를 달리한다. 여기서는 내적을 사용하여 position 값을 추가하려고 한다.&#x20;

$$
<f_q(x_m, m), f_k(x_n, n)> = g(x_m, x_n, m-n)
$$

이 수식의 의미는 $$f$$를 적용한 두 벡터의 내적이 반드시 $$x_m, x_n, m-n$$의 관계식으로 표현할 수 있도록 해서 index의 차를 항상 반영하도록 하겠다는 것이다.&#x20;

왜 내적을 사용하는가?

논문에 정확히 나와 있지는 않지만, efficient attention에 적용을 시킨다는 점에서 유추해 볼 수는 있다. 위의 sim함수를 변환하게 되어도 RoPE의 목적 함수를 보면 이미 f함수로 wrapping되어 있는데, 이렇게 함수를 설정하면 sim함수와 동일하게 처리할 수 있다는 장점이 있다. 논문에서 나온 바로는 'normalization'에서 장점이 있다는 것. RoPE방식을 쓰면 normal값이 동일하기 때문에(벡터를 단순히 돌린 거니까)

아무튼 이 내적값을 설정하기 위해 저자들은 f함수로 '복소화'을 사용한다. 복소평면에서 두 벡터 사이의 각도를 position값차이로, 벡터값을 컨텍스트 값으로 설정하는 것이다. 따라서 특정 벡터가 특정 거리만큼 벌어지도록 하고 싶다면 해당 벡터를 거리만큼 돌리면(rotate)된다. 따라서 이름이 rotary PE가 된 것.

## Discussion

introduction을 보면 사실 이 기법은 position-awards embedding을 위한 것이라는 것을 알 수 있다. 그런데 개발하고 나서 보니, context길이를 flexible하게 잡을 수 있다는 장점 역시 존재했고, 이는 long llm으로 가는 가장 큰 단점인 '고정 길이를 늘려야 한다는 것'을 없애 주었다.

