---
description: https://arxiv.org/abs/2304.08244
---

# API-Bank: A Comprehensive Benchmark for Tool-Augmented LLMs

## Description

### Summary

\---

* link: [https://arxiv.org/abs/2304.08244](https://arxiv.org/abs/2304.08244)
* name: API-Bank: A Comprehensive Benchmark for Tool-Augmented LLMs
* code: [https://github.com/AlibabaResearch/DAMO-ConvAI/tree/main/api-bank](https://github.com/AlibabaResearch/DAMO-ConvAI/tree/main/api-bank)

## Introduction

이 논문에서는 세 가지 논제에 대해 답변하는 형식으로 되어 있다.

1. 현재 LLM은 얼마나 tool을 잘 다루는가?(How effective are current LLMs' ability to utilize tools?)\
   \= evaluation system 만듬
2. 어떻게 해만 LLM이 툴을 잘 다루게 할 수 있을까?(How can we enhance LLMs' ability to utilize tools?)\
   \= 학습을 위한 데이터 세트를 만듬(자동으로! Multi-agent)
3. LLM이 툴을 잘 다루기 위해서 해결해야 하는 문제점은 무엇인가?(What obstacles still need to be overcome for LLMs to effectively leverage tools?)

이 해결책들을 다 모은 것이 API-Bank

tool-usage에 대한 user requirements를 얻기 위해 우선 예상 사용자로부터의 interview를 진행. 요구 능력은 다음 세 가지.

* planning
* retrieving
* calling

또한 각 API에 대한 기준치는 다음 세 가지이다.

* domain diversity
* API diversity
* evaluation authenticity

첫 번째 논제에 대한 대답을 위해 evaluation system을 만듬.



## Method

### Design Principles

* 약 500명의 사람들에게 LLM tool use에 대한 인터뷰를 시행
* 해당 답변들을 바탕으로 tool-augmented llm의 requirements 작성

#### Ability Grading

두 개의 기준축을 설정

(1) API pool

* few: api개수가 애초에 적음(BNK? 연산(resoning) / retrieval 정도니까)
  * 이러면 사실 그냥 모든 콜을 진행한 다음에 생성을 해도 무방
* many: api 개수를 많이 사용&#x20;
  * 이러면 모든 콜을 하기에는 예산이 부족하므로 중간에 한 번 걸러 내는 작업 필요.
  * 대신 풀이 넓어서 좀 더 세부적인 정보 얻기 가능

(2) api calls per turn



## Experiments



## Discussion
