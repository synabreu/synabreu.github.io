---
title: "CPU만으로 동작하는 초경량 1비트 LLM, MS BitNet"
date: 2025-04-18
tags: [마이크로소포트, Microsoft, 비트넷, BitNet]
typora-root-url: ../
toc: true
categories: [Microsoft]
---

마이크로소프트가 아주 흥미로운 모델을 하나 공개했네요 🙂 지금까지 개발된 LLM들중 가장 작은 1비트 AI 모델인 '비트넷(BitNet)'을 개발했다고 해서 어떤 특징이 있는지 궁금해서 남겨 본다. 



## 1. BitNet의 주요 특징

* 해당 모델의 청식 명칭은 BitNet b1.58 2B4T 이다. 
* MIT 라이선스 하에 오픈소스로 공개되었음. 
* GPU는 지원하지 않지만, 애플 M2를 포함한 CPU에서만 실행 가능함. 
* 경량 하드웨어(On-device)에서도 작동할 수 있도록 압축된 AI 모델임.- 일반적인 AI 모델은 내부 구조를 정의하는 가중치(weight) 값들을 다양한 하드웨어에서 잘 작동하도록 양자화(quantization) 하는 데, 가중치를 표현하는 데 필요한 비트 수를 줄여, 적은 메모리와 더 빠른 연산이 가능한 칩에서도 모델을 실행할 수 있게 해줌.
* 비트넷은 가중치를 단 세 가지 값(-1, 0, 1) 만으로 표현함.(이론적으로는, 기존 대부분의 AI 모델보다 훨씬 더 메모리를 적게 차지하고 연산에 효율적)  



## 2. BitNet의 내부 아키텍처

* 20억 개의 파라미터(parameter)
* 4조 개의 토큰으로 학습됨 — 이는 약 3,300만 권의 책에 해당하는 분량
* 같은 크기의 기존 모델보다 우수한 성능을 보임
* 성능 비교 (벤치마크 테스트 기준): Meta의 Llama 3.2 1B, Google의 Gemma 3 1B, Alibaba의 Qwen 2.5 1.5B 등 보다 GSM8K(초등 수학), PIQA(물리적 상식 추론) 등의 테스트에서 더 나은 결과를 보였다고 MS가 주장함.  



## 3. BitNet의 제약사항

* 마이크로소프트의 전용 프레임워크 bitnet.cpp를 사용할 때에만 구현 가능함. 
* 해당 프레임워크는 현재 일부 하드웨어에서만 작동하며, GPU는 지원하지 않고 온리 CPU 모델임.



저의 생각은 현재 대규모 AI 인프라 대부분이 GPU에 의존하는 반면에, 이 모델은 주로 경량 하드웨어와 특정 목적으로 빠른 추론을 할 때 용도로 사용하지 않겠나 싶다. 



## 레퍼런스

* 논문: [BitNet: Scaling 1-bit Transformers for Large Language Models, 2023](https://arxiv.org/abs/2310.11453)
* 한글 논문 분석: [https://devhwi.tistory.com/66](https://devhwi.tistory.com/66)
* BinNet Github: [https://github.com/microsoft/BitNet](https://github.com/microsoft/BitNet)
* TechCrunch 기사: [Microsoft researchers say they’ve developed a hyper-efficient AI model that can run on CPUs](https://techcrunch.com/2025/04/16/microsoft-researchers-say-theyve-developed-a-hyper-efficient-ai-model-that-can-run-on-cpus/)  

