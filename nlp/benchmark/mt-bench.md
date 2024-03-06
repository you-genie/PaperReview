# MT-Bench

## Description

* name: Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena
* date: 23.06
* code: [https://github.com/lm-sys/FastChat/tree/main/fastchat/llm\_judge](https://github.com/lm-sys/FastChat/tree/main/fastchat/llm\_judge)
* [https://huggingface.co/spaces/lmsys/chatbot-arena-leaderboard](https://huggingface.co/spaces/lmsys/chatbot-arena-leaderboard)

## Introduction

원래 LLM chatbot 같은 경우 lm-evaluation-harness와 같은 방식을 주로 사용하였는데, 이게 실제 성능과는 괴리가 어느 정도 있다는 것이 주요 마이너스 요소였다.

예를 들면,

* LLM leaderboard에서는 (e.g. open ko-llm leaderboard) 높은 성능을 보이지만 실제로 대답하는 것을 보면 어색한 경우. 번역투의 데이터를 많이 집어넣으면 보통 이렇게 된다.
* 챗봇 형태(실제로 더 많이 쓰일)보다 PLM형태가 더 점수가 높다.

이런 이슈가 존재해서 '챗봇'을 위한 리더보드의 필요성이 생겼다.

그렇다면 왜 현재 하네스 식 리더보드에 이런 문제점이 생겼는가? 챗봇의 경우 대다수가 'instruction dataset'을 사용한다. 여기서 Instruction dataset은 아주 다양하고 섬세한 지시 사항도 모델이 따를 수 있도록 하는 것을 목적으로 한다. 영어로는 instruction alignment-로, 그냥 '말 잘 듣는 모델' 정도로 이해하면 된다. 지시를 잘 따르는 모델이라는 것은 말하자면 이런 능력을 가진 모델이라고 보면 되겠다.

* 지시문에 달린 여러 요구사항을 모두 만족할 것
* 답변이 정확하며 거짓을 담지 않을 것
* 원하는 뉘앙스로 응답을 생성할 것

이런 지시 사항 능력 강화를 위해 단순 FT를 넘어 DPO를 사용하기도 한다! 사실 '말을 잘 듣는다'라는 기준이 저렇게 딱딱 구조적으로 말하기 애매한 것들이 많아서, 그냥 모델한테 "야, 이게 내가 너에게서 원하는 답변이야" 라고 알려주는 거다.

이러한 일련의 과정을 거치고 나면 모델은 **지시문을 잘 따르는** 모델이 된다.

이번에는 리더보드에 많이 쓰이는 하네스 데이터 예시를 통해 **왜 좋은 챗봇이 좋은 점수를 받기 어려운지** 알아보자.

> 지시문: 정답을 고르세요. 바나나는 무슨 색인가? A) 노란색 B) 갈색 C) 검은색 D) 초록색
>
> 정답: 노란색
>
> 챗봇: 바나나는 익으면 노란색이기 때문에 정답은 A)입니다. 그러나 과하게 익으면 갈색-검은색이 되기도 하고, 덜 익으면 초록색이기도 합니다.

많이 과장하긴 했는데 이런 느낌이다. 챗봇은 보통 사고 후 대답하는 능력 또한 강화되었기 때문에 다음과 같이 대답할 가능성이 높고, 이는 정답과는 다르지만 사용자의 '의도'에는 더 맞는 듯한 느낌을 준다.

물론 챗봇도 할 수 있다. 입력을 다음과 같이 준다면...

> 지시문: 아래는 일반 상식을 평가하는 문제입니다. 정답에 해당하는 '단어'만을 출력하세요. 바나나는 무슨 색인가? A) 노란색 B) 갈색 C) 검은색 D) 초록색
>
> 챗봇: 노란색

하지만 문제 지문이 저렇게 되어 있지 않다. 따라서 챗봇 입장에서는 억울할 것.... 심지어 자기는 수학/과학문제도 풀 수 있는데 지문에 '안 써줬기 때문에' 대답하지 않을 뿐이다.



아무튼 이러한 이슈로 인해 챗봇이 '얼마나 더 지시를 따르는가?' 에 대한 평가를 따로 만들기로 했다. 사실 두 개 만들었는데, MT-bench와 Chatbot Arena이다.&#x20;

## Methods

간단하게 설명하면 MT-Bench는 답변 평가를 위한 벤치마크 질문지 모음이고, Chatbot Arena는 랜덤한모델과 생성 결과 비교를 할 수 있게 하는 플랫폼이다.

MT-Bench의 데이터는 다양한 분야(수학, 롤플레이, 생성, 지식 추출, reasoning...)에서의 생성 질문을 포함하고 있다.

Chatbot Arena의 경우 유저가 참가하게 되면 랜덤한 모델 두 개가 동일한 질문에 대해 같은 대답을 내놓는다. 그 이후 유저가 승패를 가르는 방식이다. [https://huggingface.co/spaces/lmsys/chatbot-arena-leaderboard](https://huggingface.co/spaces/lmsys/chatbot-arena-leaderboard) 바로 그 아레나.

### LLM as a Judge

위의 Chatbot Arena의 경우 사람이 직접적으로 참여하여 평가를 하는 방식이다. 이 과정은 정확도가 보장되지만 굉장히 품이 많이 드는 작업이기 때문에 이를 자동화하기 위한 'LLM Judge'를 고안하였다.



LLM Judge의 종류는 총 세 가지.

* pairwise comparison - 두 답변 중 어느 것이 더 좋은가?
* single answer grading - 답변의 퀄리티를 평가하라
* reference-guided grading - reference 답변을 주고, 이를 고려해서 답변의 퀄리티를 평가하가 하는 것. 뒤에 나오겠지만 수학 문제 등에 약하기 때문에 답변을 미리 주고 이를 감안하여 평가하라고 하는 것이다.

#### LLM Judge 사용의 장점?

사람이 평가 안해도 된다.

단순히 점수뿐만이 아니라 '왜' 그렇게 평가했는지도 같이 제시해준다.

#### LLM 사용의 단점?

**Position bias**: (왼쪽일때는 더 좋다고 그러다가 오른쪽으로 오니까 별로라고 하는 경우)가 생김. 모든 LLM이 이러한 position bias가 있음을 실험을 통해 밝혔으며, 대체로 앞에 오는 대답을 긍정적으로 평가하는 경우가 많았다. GPT-4 정도만 consistency를 65% 정도로 유지했다.&#x20;

**Verbosity bias**: LLM이 더 긴 모델을 더 좋다고 평가하는 경우가 많다. 평가를 위해 하나의 답변과 동일한 정보량을 가진 더 긴 답변 (리스트를 셔플해서 앞에 붙여놓음)을 준비했고, GPT-4를 제외한 다른 모델들은 이를 감별하는 데 실패했다.

**Self-enhancement bias**: 자기가 만든 답변을 더 좋게 여긴다는 이론. 대체로 한 모델이 특정 모델이나 자기 자신을 더 좋게 여기는 '경향'이 있기는 하지만 정확한 평가는 할 수 없다고 밝혔다. 이걸 평가하려면 '동일한 퀄리티'를 가진 '각 모델 스타일'의 답변을 준비해야 되는데, 퀄리티를 동일하게 하기가 쉽지 않기 때문.

Limited capability in grading math and reasoning questions: 수학이나 reasoning은 (자기도 못 풀기 때문에)채점을 잘 못 한다.

### Multi-turn Judge

MT-Bench의 모든 conversation은 2-turn으로 되어 있다. 그래서 평가 디자인 후보가 다음 두 가지였다.

1. 질문 - 턴 / 질문 - 턴 이렇게 2 row로 만들기
2. 멀티턴 자체를 한 row로 만들기

second turn(딸림질문에 따른 응답)에서 앞의 응답을 기반으로 대답을 한 것이기 때문에 앞의 턴을 빼고 데이터를 구성한다면(디자인 1) 판단에 문제가 생기게 된다. 따라서 멀티턴으로 데이터를 구성하게 했다.

## 사람과의 Agreement

LLM-Judge가 잘 작동하는지 보려면 사람이랑 평가가 일치하는지를 봐야 한다.

일치 기준 = 랜덤하게 선택된 문제에서 랜덤한 평가자가 일치하게 평가한 확률

재밌는 건 사람-사람의 일치도(81%)보다 평균적인 사람-GPT-4의 일치도(85%)가 더 높다는 것이다.

## As Benchmark

다른 MMLU / TruthfulQA / MT-Bench와 같은 벤치마크에도 실험했을 때, high-quality dataset을 쓰면 모델 퍼포먼스를 올릴 수 있고, 적은 high-quality conversation을 쓰더라도 GPT-4의 평가를 잘 받는 모델은 만들 수가 있지만 MMLU는 올릴 수 없다. 즉, 이러한 single benchmark(MMLU등)로는 모델의 성능을 온전히 평가하기에는 문제가 있으며 comprehensive 한 평가 기준 (예를 들어 MT-Bench!) 가 필요하다는 것을 시사한다.&#x20;
