# MINIGPT-4: ENHANCING VISION-LANGUAGE UNDERSTANDING WITH ADVANCED LARGE LANGUAGE MODELS

## Description

### Summary

* 비전 모델(freeze) + 어댑터 레이어(tuning) + 언어 모델(freeze)

***

* name: MINIGPT-4: ENHANCING VISION-LANGUAGE UNDERSTANDING WITH ADVANCED LARGE LANGUAGE MODELS
* code: [https://github.com/Vision-CAIR/MiniGPT-4](https://github.com/Vision-CAIR/MiniGPT-4)

## Introduction

ChatGPT4는 대체 어떻게 이미지를 입력으로 받을 수 있었는가? -> LLM의 능력을 빌린 것으로 추정한다.

따라서 공개 모델들을 가지고도 ChatGPT4에 상응하는 모델을 만들 수도 있겠다! 라는 아이디어. 언어 모델의 능력이 충분히 좋으므로, 약간의 변수들만 학습하고도 충분히 언어 모델은 이미지 feature에 대해 이해할 수 있다.

## Method

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

진짜 '약간의' 레이어만을 사용했는데, 저 Linear Layer는 **고작 한 개의 projection layer**이다.

두 단계의 학습 스테이지를 거쳤다.

### Stage 1: 이미지에 대해 이해하기

* image-text pair 데이터로 이미지 정보에 대해 이해하는 과정을 거침(약간 PLM과 유사한 학습법)
* 4 A100(80GB)에 10시간 학습을 돌렸다고 한다.

### Stage 2: QA를 할 수 있게 하기

마치 instruction dataset을 줘서 SFT를 만드는 느낌

#### 데이터 제작

* stage 1의 데이터를 가지고 시작한다.&#x20;
* 다음과 같이 포맷시켜준다 \
  `###Human: Describe this image in detail. Give as many details as possible. Say everything you see. ###Assistant:`
* 위 프롬프트로 1차적 데이터를 만든 후, ChatGPT에 통과시켜 데이터를 한 번 더 처리한다 \
  `Fix the error in the given paragraph. Remove any repeating sentences, meaningless characters, not English sentences, and so on. Remove unnecessary repetition. Rewrite any incomplete sentences. Return directly the results without explanation. Return directly the input paragraph if it is already correct without explanation.`

데이터를 만든 다음에는 instruction tuning과 똑같이 포맷팅을 거쳐 학습시킨다. 프롬프트는 위와 똑같다. `###Human: Describe this image in detail. Give as many details as possible. Say everything you see. ###Assistant:`

## Evaluation

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

모델의 생성 능력은 어디에 가장 많이 영향을 받을까? 에 대한 실험으로 이미지 인코더를 함께 학습해 보았다(그 데이터로)

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

생성 능력(말의 길이 등)은 미니지피티(이미지를 쓰지 않았더라도!) 훨씬 좋았음을 알 수 있으며, 블립은 학습을 시켜도 그게 그거인걸 알 수 있다.

게다가 Q-Former를 학습시키거나 레이어 수를 늘리면 오히려 성능이 살짝 떨어진다. 1레이어가 충분하다는 실험 결과.

## Discussion

멀티모달 학습할 때 왜 다른 데서 PEFT를 쓴 데가 있는지 알 것 같다. 아주 작은 레이어의 업데이트면 충분하다는 의미로 보임. 그럼 LoRA보다는 IA3로 시도해보는 것도 나쁘지 않을 수 있겠다.
