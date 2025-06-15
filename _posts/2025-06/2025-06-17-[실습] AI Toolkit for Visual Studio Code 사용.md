---
title: "[실습] AI Toolkit for Visual Studio Code 사용"
date: 2025-06-17
tags: [Microsoft, 마이크로소프트, Windows Foundry, Foundry Local, Windows Azure, Azure AI Foundry, AI Agent, Ollama, OpenAI, Visual Studio Code, LangChain, Open Web UI]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

이번 마이크로소프트 빌드 2025 행사는 AI 개발자들에게 좀 더 모델과 에이전트, 앱 개발을 사용하기 쉽게 하기 위해 촛점을 맞추었다.  VS Code 내에서 생성형 AI 앱 개발을 전방위 지원하기 위해 AI Toolkit for Visual Studio Code를 출시했다. 



## 1. AI Toolkit이란 무엇인가? 

* Visual Studio Code용 강력한 확장 기능으로, 에이전트 개발을 간소화해 줌
  *  Anthropic, OpenAI, GitHub 등 다양한 제공업체의 모델을 탐색하고 평가하거나, **ONNX 및 Ollama**를 사용해 로컬에서 모델을 실행할 수 있음
  * **프롬프트 생성, 빠른 시작 도구, MCP 툴과의 통합**을 통해 몇 분 만에 에이전트를 구축하고 테스트할 수 있음

* 주요 특징

  | 특징                                                         | 설명                                                         | 스크린샷                                                     |
  | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | [Model Catalog](https://code.visualstudio.com/docs/intelligentapps/models) | 다양한 소스에서 AI 모델을 탐색하고 접근할 수 있음.  GitHub, ONNX, Ollama, OpenAI, Anthropic, Google 모델을 **간편하게 검색하고 활용**할 수 있음 | <img src="/../images/2025-06/AIToolkit-01.png" alt="그림1 - Model Catalog" style="zoom:15%;" /> |
  | [Playground](https://code.visualstudio.com/docs/intelligentapps/playground) | 실시간 모델 테스트를 위한 인터랙티브 채팅 환경. 다양한 프롬프트, 파라미터, 이미지 및 첨부파일과 같은 멀티모달 입력을 활용하여 실험할 수 있음. | <img src="/../images/2025-06/AIToolkit-02.png" style="zoom:15%;" /> |
  | [Prompt(Agent) Builder](https://code.visualstudio.com/docs/intelligentapps/agentbuilder) | 간소화된 프롬프트 엔지니어링 및 에이전트 개발 워크플로우. 정교한 프롬프트를 생성하고 MCP 도구를 통합하며, 구조화된 출력으로 프로덕션 수준의 코드를 생성할 수 있음. | <img src="/../images/2025-06/AIToolkit-03.png" style="zoom:25%;" /> |
  | [Bulk Run](https://code.visualstudio.com/docs/intelligentapps/bulkrun) | 여러 모델을 대상으로 동시에 배치 프롬프트 테스트를 실행함. 다양한 입력 시나리오에서 모델 성능을 비교하고 대규모 테스트를 수행하는 데 이상적임. |                                                              |
  | [Evaluate an AI model with a dataset](https://code.visualstudio.com/docs/intelligentapps/evaluation) | 데이터셋과 표준 메트릭을 활용한 **포괄적인 모델 평가**를 제공함. 내장된 평가 도구(F1 점수, 연관성, 유사도, 일관성 등)를 사용하거나, **사용자 정의 평가 기준**을 생성하여 성능을 측정할 수 있음. |                                                              |
  | [Fine-tuning](https://code.visualstudio.com/docs/intelligentapps/finetune) | 특정 도메인과 요구사항에 맞게 모델을 **맞춤화하고 최적화**할 수 있음. 로컬에서 GPU를 활용해 모델을 학습하거나, **Azure Container Apps**를 이용해 클라우드 기반의 파인튜닝을 수행할 수 있음 |                                                              |
  | [Model Conversion](https://code.visualstudio.com/docs/intelligentapps/modelconversion) | 머신러닝 모델을 **로컬 배포용으로 변환, 양자화(quantize), 최적화**할 수 있음. Hugging Face 및 기타 소스의 모델을 변환하여 Windows 환경에서 **CPU, GPU, 또는 NPU 가속**을 활용해 효율적으로 실행할 수 있음. |                                                              |

  