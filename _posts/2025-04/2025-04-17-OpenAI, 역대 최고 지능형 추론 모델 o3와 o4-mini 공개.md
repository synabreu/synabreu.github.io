---
title: "OpenAI, 역대 최고 지능형 추론 모델 o3와 o4-mini 공개"
date: 2025-04-17
tags: [오픈AI, OpenAI, 챗GPT, ChatGPT, 멀티모달, SWE-Bench, Claude 3 Sonnet, o3, o4-mini]
typora-root-url: ../
---

OpenAI가 지금까지 등장한 모델 중 가장 지능적인 추론 모델 o3와 경량 고효율 모델 o4-mini를 새롭게 공개했다. 이번 모델들은 단순한 언어 처리 능력을 넘어서, 도구 사용 능력과 시각적 추론 기능까지 통합하며 AI의 새로운 진화를 보여주고 있다. 요약하자면 다음과 같다. 



🚀 **모델 공개 및 적용 대상**

OpenAI는 공식 블로그를 통해 `o3`와 `o4-mini` 모델을 공개하고, ChatGPT Plus, Teams, Pro 등 **유료 사용자에게 즉시 제공**한다고 밝혔음. 



💡 **주요 특징 요약**

![✅](https://static.xx.fbcdn.net/images/emoji.php/v9/t33/1/16/2705.png) **도구 사용 능력 내장**

* 웹 브라우징, 파이썬 코드 실행, 이미지 분석 및 생성 등 ChatGPT의 모든 내장 도구 사용 가능
* API 함수 호출을 통해 사용자 정의 도구까지 호출 가능
* 모델 스스로 언제 도구를 사용할지 판단하고, 실제로 도구를 사용해 추론을 진행함

![✅](https://static.xx.fbcdn.net/images/emoji.php/v9/t33/1/16/2705.png) **내장 멀티모달 추론과 이미지 기반 사고**

* 텍스트 뿐만 아니라 이미지 기반 추론까지 수행
* 흐릿하거나 반전된 이미지도 도구를 활용해 분석 및 이해
* OpenAI는 이를 두고 "이미지로 사고하는 모델" 이라 표현

![✅](https://static.xx.fbcdn.net/images/emoji.php/v9/t33/1/16/2705.png) **강화 학습 기반 성능 최적화**

* 반복된 강화 학습을 통해 추론 능력 강화
* 동일한 비용과 지연 시간으로도 더 높은 성능 발휘
* 기존보다 더 유연하고 전략적인 문제 해결 방식 제공

![✅](https://static.xx.fbcdn.net/images/emoji.php/v9/t33/1/16/2705.png) **성능 및 벤치마크**

* SWE-Bench (코딩 테스트) 성능: o3: 69.1%, o4-mini: 68.1% → 기존 o3-mini (49.3%) 및 Claude 3 Sonnet (62.3%) 대비 높은 수치

![그림 1 - SME-Bench 코딩 테스트 벤치마크](../images/2025-04/2025-04-17_01.png)

![✅](https://static.xx.fbcdn.net/images/emoji.php/v9/t33/1/16/2705.png) **비용**

o3는 o1보다 성능은 향상, 지연 시간과 비용은 동일 또는 더 저렴함. 

![그림 2 - API 사용 요금](../images/2025-04/2025-04-17_02.png)

![✅](https://static.xx.fbcdn.net/images/emoji.php/v9/t33/1/16/2705.png) **향후 계획**

o3와 o4-mini는 단순한 언어 모델이 아닌, 멀티모달 능력과 도구 활용 능력까지 갖춘 진정한 추론 AI로 평가받고 있다. 특히, 이미지로 사고하고, 상황에 따라 전략적으로 도구를 활용하는 능력은 앞으로의 AI 활용 방식에 큰 전환점을 가져올 것이다. 

저의 생각은 앞으로 AI가 단순한 답변만 생성하는 것을 넘어, 스스로 문제를 정의하고 도구를 활용해 해결하는 에이전트형 AI로 진화하는 흐름을 가중화할 것이다.  