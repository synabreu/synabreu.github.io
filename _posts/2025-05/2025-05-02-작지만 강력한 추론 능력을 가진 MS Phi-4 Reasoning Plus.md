---
title: "작지만 강력한 추론 능력을 가진 MS Phi-4 Reasoning Plus"
date: 2025-05-02
tags: [마이크로소프트, Microsoft, Phi4, Phi-4 Reasoning Plus, Chain-of-Thought, 오픈AI]
typora-root-url: ../
toc: true
categories: [Microsoft, Phi-4]
---



마이크로소프트도 OpenAI 외에 Phi-4 파운데이션 모델도 계속해서 업그레이드 시키고 있다. 이번에 새롭게 업그레이드된 Phi-4 Reasoning Plus는 작은 모델이지만, 강력한 추론 능력을 가진다. Phi-4 Reasoning Plus에 대해 다음과 같이 요약을 해본다.



## 1. Phi-4 Reasoning Plus 핵심 특징

* **크기를 뛰어넘는 성능:** Phi-4-Reasoning-Plus는 140억 파라미터의 공개 가중치(open-weights) 모델로, 아래와 같은 대형 모델들과 비교해 **동등하거나 뛰어난 성능**을 보여줌
  * 오픈모델: DeepSeek-R1-Distill-70B, QwQ-32B
  * 클로즈드모델: o1-mini, Claude-Sonnet-3.7, 특히 AIME 2025, HMMT, OmniMath, GPQA, LiveCodeBench 등의 벤치마크에서 우수한 성과를 보였고, 수학 분야에서는 무려 6,710억 파라미터의 DeepSeek-R1에 근접하는 성능을 보임.
* 오픈 & 접근성 용이: **MIT 라이선스**로 공개되어, **상업적·연구적 활용이 모두 자유롭다**
* **노트북에서도 실행 가능, 출력 해석 용이:** 고성능 노트북 GPU에서도 실행 가능하며, 사고 과정(Chain-of-Thought)를 `<think>...</think>` 태그로 구분하여 **출력 해석이 직관적**
* **추론은 전이 가능한 메타스킬:** TSP, 3-SAT, 미로 경로 찾기, 일정 계획 등 **학습하지 않은 작업**도 해결하며, **추론 능력이 일반화 가능함**을 보여줌. 
* **데이터 중심 접근 방식:** 약 1.4백만 개의 고품질 추론 예제(약 160억 토큰)를 o3-mini로부터 생성하고, 그 위에 단 6천 개 수학 문제를 사용하여 **GRPO 기반 RL로 미세 조정**하여 추론 스타일을 고정했음
* **RL의 실제 효과 입증:** 짧은 RL 훈련 한 번만으로도 AIME/HMMT 정확도가 **약 10%p 상승**, **설명 길이도 약 50% 증가**.**대규모 컴퓨팅 없이도 RL이 사고력 향상에 기여**할 수 있음을 보여줌



## 2. 애저 AI 파운더리에서 Phi-4

* 소형 언어 모델(SLM)의 가능성을 입증한 **Phi 시리즈** 공개 1주년을 맞아, **추론에 최적화되고 멀티스텝 사고에 특화**된 **새로운 모델군을 발표**함
* **Phi-4-reasoning**: 14B 파라미터, 고품질 reasoning 데이터로 SFT(Supervised Fine-Tuning) 진행
* **Phi-4-reasoning-plus**: 위 모델에 RL(Reinforcement Learning) 추가, **1.5배 더 많은 토큰 사용으로 정확도 향상**
* **Phi-4-mini-reasoning**: 3.8B 크기에도 불구하고 다양한 수학 벤치마크에서 **2배 이상 큰 모델들을 능가** , 모바일/에지에 적합
  * **OpenThinker-7B, Llama-3.2-3B, DeepSeek-R1 계열 등 다양한 모델을 성능 측면에서 능가함**
  * **OpenAI o1-mini와 유사하거나 우위의 성과**, 특히 Math-500, GPQA Diamond 등 수학 중심 테스트에서 강세
* Phi-4 모델들은 **Copilot+ PC에서 NPU 최적화 버전**인 **Phi Silica**로 제공되어 Windows 환경에서 빠르고 효율적인 실행 가능
* Azure AI Foundry 및 HuggingFace에서 공개:
  * [Phi-4-reasoning](https://huggingface.co/microsoft/Phi-4-reasoning)
  * [Phi-4-reasoning-plus](https://huggingface.co/microsoft/Phi-4-reasoning-plus)
  * [Phi-4-mini-reasoning](https://aka.ms/phi4-mini-reasoning/hf)

* **개발자 API 및 로컬 통합 도구**도 함께 제공되어 다양한 환경에 쉽게 접목 가능



## 3. 참고

* **논문**: [Phi-4 Reasoning 논문 링크](https://www.microsoft.com/.../2025/04/phi_4_reasoning.pdf)
* **마이크로소프트 공식 발표**: [1년간의 Phi 개발 성과](https://azure.microsoft.com/.../one-year-of-phi-small...)
* **언론 보도**: [VentureBeat 기사](https://venturebeat.com/.../microsoft-launches-phi-4...)





