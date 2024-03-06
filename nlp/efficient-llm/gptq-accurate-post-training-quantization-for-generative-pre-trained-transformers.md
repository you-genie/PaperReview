---
description: https://paperswithcode.com/paper/gptq-accurate-post-training-quantization-for
---

# \[논문리뷰] GPTQ: ACCURATE POST-TRAINING QUANTIZATION FOR GENERATIVE PRE-TRAINED TRANSFORMERS

## Description

### Summary

* &#x20;

***

* name: [https://paperswithcode.com/paper/gptq-accurate-post-training-quantization-for](https://paperswithcode.com/paper/gptq-accurate-post-training-quantization-for)
* code: [https://github.com/ist-daslab/gptq](https://github.com/ist-daslab/gptq)
* date: 22(ICLR 23)

## Introduction

현재 양자화 하면 떠올리게 되는 기법인 GPTQ. Huggingface에 integrate되어있어서 사용이 편하다.&#x20;

Exllama 찾아보다가 GPTQ부터 읽는다. 일단 실험용으로 써보려고 했는데 중간에 가이딩 텍스트(C4 - google transformer에서 나온 그 데이터 맞다)를 넣는 이상한? 옵션이 있는데 실제 논문에서 제안했다고 한다. 안 쓰면 성능이 다소 떨어졌다고 한다. 이 텍스트가 대체 어떤 역할을 하는지 모르겠으니 읽어보기로 함. 실질적으로 22년 논문이니 꽤 오래된듯. 간단하게 읽고 넘길 예정.

### Background

#### Layer-Wise Quantization

* GPTQ 이전의 SOTA에 해당되는 기법에 나오는 objective. 레이어 별로 양자화를 진행한다.&#x20;

여기서 그 가이던스 데이터에 대한 정보가 나오는 것 같은데, 방식은 다음과 같다.

양자화된 레이어에 조그만 데이터를 통과시켜서 원본과의 격차를 줄이는 것을 objective로 한다.

$$
argmin_{\hat{W}}||WX - \hat{W}X||_2^2
$$

$\hat{W}$ 얘가 양자화된 레이어다. 이걸  찾는 것을 따로 **layer-wise quantization problem** 이라고 부르는 듯

#### Optimal Brain Quantization

요 기법을 베이스로 진행함. 가장 기본적인 개념만 보자면 다음과 같다.

1. 한 번에 한 개의 weight를 양자화함(round-nearest)
2. 아직 양자화되지 않은 모든 weight를 업데이트시킴

## Methods



## Discussion
