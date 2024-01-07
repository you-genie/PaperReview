# KwaiAgents

## Descriptions

### Summary

* information retrevial agent

\---

* name: KwaiAgents: Generalized Information-seeking Agent System with Large Language Models
* link: [https://arxiv.org/pdf/2312.04889v1.pdf](https://arxiv.org/pdf/2312.04889v1.pdf)
* code: [https://github.com/KwaiKEG/KwaiAgents](https://github.com/KwaiKEG/KwaiAgents)
* date: 23.12



## Introduction

* agent가 잘 동작하는 것은 알려져 있지만, 상대적으로 작은(7B, 13B)오픈소스 모델에서는 프롬프트에 따른 동작 변화 등 불안정한 측면이 있음
* 지식 검색 기반 llm은 해당 지식을 걸러 듣지는 못한다는 단점이 있음

이 두개를 해결할 수 있는 지식검색 agent sytem을 제시&#x20;

## Method

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption><p>overview</p></figcaption></figure>

총 세 가지 요소로 구성된다.

1\) KAgentSys: loop형식이며, 사실상 핵심

2\) KAgenetLMs: llm

3\) KAgentBench: 각각의 lm의 성능 평가.

### KAgentSys

#### LLM

* 응대
* user requirement파악 등

#### Memory Bank

세 종류의 메모리 존재

1. Knowledge Memory: 외부 지식 등등
2. Conversation Memory: 담화
3. Task Memory: agent가 다음 task를 뭘 할지 지정하는데, 그 기록들

#### Tool Library

기본적으로 2개의 툴, 유저가 원한다면 custom tool 사용 가능.

factuality 툴은 지식검색관련. 위키피디아, 브라우저, 비디오 뒤지기 등등 어디로 search할 것인지를 찾음. 이 때 검색은 hybrid search라고 해서 기존의 검색 방식과 entity search를 합친 방식.

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

두 번째 툴은 시간 관련 툴로, 달력, 휴일, 시간, 날씨 등에 실시간으로 접근할 수 있는 툴들이다.



#### Agent Loop

Agent는 유저의 쿼리에 대해 다음과 같은 loop를 반복한다.

1. memory update: (입력이 주어졌으니) 메모리를 업데이트
2. memory retrieval: 현재의 쿼리와 interaction을 바탕으로, 메모리에서 유의미한 정보를 가져옴
3. task planning: llm을 위한 prompt만들기. llm은 다음 'task\_name'을 생성하도록 유도됨
4. tool execution: llm이 만든 task\_name 실행
5. concluding: 루프를 종료하고 응답을 생성하는 단계

### Meta-Agent Tuning (MAT)

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

시드 쿼리를 가지고 좋은 모델(GPT-4)이 instruction을 만들도록 함. 물론 랭크를 먹이는 것도 좋은 모델이 함.

이 데이터를 가지고 open llm 학습

## Experiments

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>



## Discussion
