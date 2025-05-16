---
title: "AlphaEvolve: 알고리즘 설계와 자동 평가를 하는 Gemini 기반 코딩 에이전트"
date: 2025-05-15
categories: [Google, DeepMind, Gemini]
tags: [구글 클라우드, Gemini, AlphaEvolve, Paper, Coding Agent, LLM, Borg, TPU, Gemini Flash, Gemini Pro]
typora-root-url: ../
toc: true
---

이번 주 구글 딥마인드에서 아주 흥미로운 논문이 올라와서 내용을 한 번 정리해보겠다. 바로 대형 언어 모델의 창의력과 자동 평가 시스템을 결합해, 수학 및 컴퓨팅의 실제 응용에 사용할 알고리즘을 발전시키는 새로운 에이전트인 AlphaEvolve 이다. 

구글 딥마인드는 복잡한 수학 문제와 현대 컴퓨팅의 난제 해결을 목표로 하는 진화적 코딩 에이전트인 AlphaEvolve를 발표했다. AlphaEvolve는 Gemini 대규모 언어 모델의 창의적인 문제 해결 능력과 자동화된 평가자(Evaluator)를 결합하여 알고리즘 솔루션을 제안, 검증 및 개선한다. 이 시스템은 이미 구글의 데이터 센터, 하드웨어, AI 훈련 프로세스 등 다양한 분야에서 상당한 효율성 향상을 가져왔으며, 행렬 곱셈 알고리즘 개선과 같은 수학 분야의 새로운 발견에도 기여했다. AlphaEvolve는 알고리즘으로 설명되고 자동적으로 검증될 수 있는 모든 문제에 적용 가능하며, 향후 재료 과학, 신약 개발, 지속 가능성 등 더 넓은 분야에 혁신을 가져올 잠재력을 가지고 있다. 



## 1. **AlphaEvolve: LLM과 진화 알고리즘의 결합**

![그림1 - AlphaEvolve 전체 구성도](/../images/2025-05/AlphaEvolve01.webp)

* Gemini 대규모 언어 모델 (LLM)의 창의적인 아이디어 생성 능력과 자동화된 평가기를 활용하여 알고리즘을 발견하고 최적화하는 진화적 코딩 에이전트
* 아이디어 탐색의 폭을 넓히는 데 사용되고, Gemini Pro는 심층적인 제안을 제공하는 데 기여
* 최첨단 대규모 언어 모델의 앙상블을 활용. 가장 빠르고 효율적인 모델인 Gemini Flash는 탐색되는 아이디어의 폭을 극대화하는 반면, 가장 강력한 모델인 Gemini Pro는 통찰력 있는 제안으로 중요한 깊이를 제공함.



## 2. **자동화된 평가 및 검증**

* 제안된 프로그램은 자동화된 평가 메트릭을 사용하여 검증, 실행 및 점수가 매김

* 이러한 메트릭은 솔루션의 정확성과 품질에 대한 객관적이고 정량화 가능한 평가를 제공함

  

## 3. **Google 내부 시스템에 대한 상당한 영향**

![그림 2 - 구글 내부 디지털 생태계 시스템](/../images/2025-05/AlphaEvolve02.webp)

* Google의 컴퓨팅 생태계 전반에 배포되어 데이터 센터, 하드웨어 및 소프트웨어의 효율성을 향상시켰음
* **데이터 센터 스케줄링 개선:** Google의 방대한 데이터 센터를 보다 효율적으로 관리하기 위한 발견적인 방법을 찾아 전 세계 컴퓨팅 자원의 평균 0.7%를 지속적으로 회수했음
* **하드웨어 설계 지원:** 행렬 곱셈을 위한 주요 산술 회로에서 불필요한 비트를 제거하는 Verilog 재작성을 제안했으며, 이는 향후 Tensor Processing Unit (TPU)에 통합될 예정
* **AI 훈련 및 추론 향상:** 
  * Gemini 아키텍처에서 중요한 행렬 곱셈 연산 속도를 23% 향상시켜 Gemini 훈련 시간을 1% 단축했음
  * FlashAttention 커널 구현에서 최대 32.5%의 속도 향상을 달성했음

* **수학 및 알고리즘 발견의 새로운 개척**
  * 행렬 곱셈과 같은 복잡한 수학 문제에 대한 새로운 접근 방식을 제안함
  * 4x4 복소수 행렬을 48번의 스칼라 곱셈으로 곱하는 알고리즘을 발견하여 1969년 Strassen의 알고리즘을 개선했음
  * 수학 분석, 기하학, 조합론 및 정수론의 50개 이상의 미해결 문제에 적용되어 약 75%의 경우 최첨단 솔루션을 재발견하고 20%의 경우 기존 최적 솔루션을 개선했음
  * 1차원에서 593개의 외부 구체 구성을 발견하여 키싱 넘버 문제에 대한 새로운 하한을 설정했음
  * [더 빠른 행렬 곱셈 알고리즘을 발견하기 위해 제안한 변경 사항 목록](https://deepmind.google/api/blob/website/media/Code-Evolution-Illustration_compressed.mp4)



## 4. **향후 전망 및 잠재적 적용 분야**

*  LLM의 코딩 능력 향상에 따라 지속적으로 개선될 것으로 예상됨
* 현재 수학 및 컴퓨팅에 적용되고 있지만, 알고리즘으로 설명되고 자동적으로 검증될 수 있는 모든 문제에 적용될 수 있는 일반적인 성격을 가짐
* 선정된 학술 사용자를 위한 Early Access Program과 보다 광범위한 제공 가능성을 모색함



## 5. **결론**

* 대규모 언어 모델의 창의성과 자동화된 평가의 정확성을 결합하여 알고리즘 발견 및 최적화의 새로운 지평을 열고 있음
* Google 내부 시스템에 대한 즉각적이고 측정 가능한 영향을 넘어서, 수학 및 컴퓨터 과학의 근본적인 문제에 대한 진전을 이루고 있으며, 미래에는 다양한 과학 및 산업 분야에서 혁신적인 도구가 될 잠재력을 보여주고 있음



## 6. **참고 자료**

* 블로그: [AlphaEvolve: A Gemini-powered coding agent for designing advanced algorithms](https://deepmind.google/discover/blog/alphaevolve-a-gemini-powered-coding-agent-for-designing-advanced-algorithms/)

* 논문: [AlphaEvolve: A coding agent for scientific and algorithmic discovery](https://storage.googleapis.com/deepmind-media/DeepMind.com/Blog/alphaevolve-a-gemini-powered-coding-agent-for-designing-advanced-algorithms/AlphaEvolve.pdf)
* 구글 코랩: [AlphaEvolve's mathematical results](https://colab.research.google.com/github/google-deepmind/alphaevolve_results/blob/master/mathematical_results.ipynb)





