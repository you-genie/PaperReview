# LLMs in the Imaginarium: Tool Learning through Simulated Trial and Error

## Description

### Summary

* tool-llm을 위한 tool data augmentation기법
* trial-and-error을 통해 더 퀄리티 높은 tool data를 만듬

## Introduction

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

생각보다 tool llm의 tool 사용 정확도가 높지 않다. API를 잘못 쓰거나(API Match) 잘못된 답을 뱉거나(Correctness)의 문제가 발생한다.

가장 높은 ChatGPT4도 상대적으로 낮은 정확도와 API Match를 기록했다.

위 논문에서는 모델이 API를 더 정확하고 잘 쓰기 위한 데이터 생성 기법을 소개한다.

* 해당 논문에서는 한 query당 한 개의 API call만을 하며, call이 반드시 존재하는 시나리오만 있다.
* 생성된 데이터는 학습 / few-shot에 쓰이게 된다.

아이디어는 똑똑한 동물들(실제로 intelligent animals라고 했다)이 어떤 식으로 도구에 대해 학습하는지에서 따왔다고 한다.

* 우선 원하는 점에 대해 가능한 여러 시나리오를 탐색한다
* 실패할 수도 있는데, 실패를 바탕으로 자신의 행동을 교정해 나간다
* 이 지식들을 바탕으로 툴을 쓰는 방법을 개선해 나간다.

## Methods

<figure><img src="../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

exploration과 exploitation 두 개의 단계로 나뉘어진다.

사실 중요한 부분은 exploration.

보면 a / b / c 세 단계로 나뉘어 있는데, b를 계속 진행하고 쌓인 b 정보를 c에 쓴다고 생각하면 된다. b내에서 진행하는 'trial'이 a이다.

### a) trial

한 개의 쿼리에 대하여 진행한다. 처음에 API call을 진행한 후 그 결과를 가지고 다시 api call을 진행하도록 한다.

예를 들어,

`weather('last week')` -> input에 int값을 넣어주세요 -> `weather(-7)` &#x20;

이런 식으로 한 개의 쿼리에 대해서 계속해서 진행한다.

### b) short-term memory

API에 대해 쿼리를 바꿔 가며 진행한다. 여기서 참조하는 것은 해당 'episode'의 과거 정보들이고, 이 정보는 \[a), a), a), ...]이런 식으로 구성되어 있다.

### c) long-term memory

short-term memory에서 나온 정보들을 간략화하여(trial 에서 나온 query / 실패-성공 여부) 저장한다. 이 정보는 나중에 trial을 계산할 때 (첫 api call시에) 만 참조된다.

이렇게 나온 정보들(query / 응답 / 실패-성공 여부 등) 을 가지고 하나의 큰 데이터를 만든다. 이 데이터로 바로 사용하는 것이 아니라, ChatGPT를 통해 퀄리티 filtering을 한 번 거친다.

그 이후 fine-tuning에 쓰거나 ICL에 쓰거나인데, ICL(few-shot) 에서는 nearest-neighbor demonstration을 통해 관련 query-응답 데이터를 가지고 few-shot에 이용하게 된다.
