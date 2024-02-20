---
description: https://paperswithcode.com/paper/fast-inference-of-mixture-of-experts-language
---

# Fast Inference of Mixture-of-Experts Language Models with Offloading

## Description

### Summary

* Mixtral-8x7B가 최종 모델임
* MoE는 좋은 방식이지만 모델이 너무 크다. 게이트 감안해도 크다.

\---

* link: [https://paperswithcode.com/paper/fast-inference-of-mixture-of-experts-language](https://paperswithcode.com/paper/fast-inference-of-mixture-of-experts-language)

## Introduction

MoE는 좋은 방식이지만 모델이 너무 크다. 게이트 감안해도 크다.

어떻게 하면 efficient하게 MoE를 실행시킬 수 있을까? 에 대한 해답 제시.

다음 두 가지 질문에 대한 답변을 제시하고 있다.

* MoE가 어떻게 각 experts를 쓸지 결정하는가?
* 어떻게 MoE를 더 efficient하게 쓸 수 있을까?

이 논문에서 타겟팅하는 하드웨어 셋업

* 시스템 메모리는 문제없음(vram말고 오프로드용 CPU로 예상)
* 11-16GB VRAM GPU 한 개
* 호스트-디바이스 통신 간 속도 8-16GB/s (아직 뭔지는 잘 모르겠음)

## Method

원래 dense model의 경우 임베딩이 현재 레이어를 돌고 있을 때 미리 다음 레이어를 GPU에 띄우기 시작한다. 이렇게 하면 생성과 동시에 로드를 하기 때문에 마치 pipelining처럼 레이어를 로드하는 시간이 단축되게 된다.

그러나 MoE의 경우 이게 쉽지 않다. 이유는

* 현 MoE의 특성상 이전 레이어의 생성 결과를 가지고 다음 레이어에서의 expert를 결정
* 따라서 현재 레이어가 끝나기 전까지 다음 레이어를 미리 GPU에 띄울 수 없음
* pipelining 불가

그렇기 때문에 캐시와 같은 몇 가지 예측을 통해서 expert를 로드하게 된다. 캐시의 장점은 만약에 맞추는 데 성공한 경우(hit), 모델 로드 시간이 줄어든다는 것이다. 만약에 맞추지 못한 경우에는? 그냥 해당 expert를 로드하면 된다. 따라서 해당 방식을 사용해서 답변이 달라진다거나 하는 일은 일어나지 않는다.

### LRU Caching

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

위의 이미지는 토큰에 따라 activate되는 expert와 LRU캐시를 가시화하고 있다. Layer 0을 보면 생각보다 많은 수의 expert가 cache hit인 것을 볼 수 있다. (layer 15의 의미는 잘 모르겠다)

LRU 캐시의 수는 2(12GB GPU), 4(16GB gpu)로 생각보다 적다.

### Speculative Expert Loading

위에서는 생각보다 cache hit이 많다고 했지만, 그래도 아직은 miss가 더 많아 보인다. 예측의 성공률을 높일 수 있는 방법이 있을까?

Transformer가 이전 값들을 계속해서 활용한다는 것을 기반으로 다음과 같은 heuristic을 사용해서 예측할 수 있다.

* 이전 레이어의 hidden state를 기반으로 다음 expert를 예측

### Implementation

* MoE Quantization: mixtral(이전 모델)이 quantization을 잘 먹었었기 때문에 이번에도 시도했음
  * mixtral에 HQQ가 잘 먹혔었기 때문에 HQQ를 사용하긴 했지만, 다른 quantization 기법을 시도해도 잘 될것으로 예상됨.
  * 그치만 1-bit compression은 영 아니었음.
* Expert Offloading
  * speculative expert loading: 현재 레이어가 돌고 있을 때 다음 레이어의 예상 expert를 선정(1-2개) 이 expert는 당장 cached expert와 대치되는 것은 아니고, 현재 레이어의 inference가 끝나고 다음 expert를 계산했을 때 hit이면 이거를 쓰고, cached expert와 대체함.
  * multi-device offloading을 지원함.
  * GPU로 로드할 때, LRU on-device expert를 다시 램에 로드한다.
  * speed-up techniques
    * allocate parameters to contiguous memory
    * pin memory buffer

## Experiments

### Implementation

#### Quantization

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

* trade-off를 봐야 하기 때문에 다양한 setup을 테스트 진행
* attention에 쓰이는 bit 수는 상대적으로 높게, expert bit 수는 상대적으로 낮게 설정해도 적절한 성능이 나왔음
* 초록으로 표시된 두 개의 세팅으로 작업

#### LRU & Speculative Loading

* LRU: 최적 cache-size 탐색
* Speculative loading: 예측 수 & 얼마나 앞까지 봐도 괜찮은가

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

* 캐시 수는 많을수록 좋고, 최소 2개는 되어야 반은 간다
* expert 수도 많을수록 좋지만, 생각보다 2-3개 차가 많지는 않아 보인다.
  * 직전 레이어의 hidden state를 사용할수록 좋다.

### Offload Performance

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

* T4 GPU(16GB, 코랩 무료버전이다) / RTX 3080 Mobile, RTX 3060(12GB) / A100-80GB
* full algorithm일 때, 모든 세팅에서 2-4 token/sec 을 달성했다.

## Discussion

* LRU Caching에서의 밑 이미지가 뭘 말하고 싶은지 아직 모르겠다.
* quantization 저 세팅 가져와서 써도 좋을듯
