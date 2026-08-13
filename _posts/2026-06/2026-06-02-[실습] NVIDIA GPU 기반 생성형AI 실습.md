---
title: "NVIDIA GPU 기반 생성형AI 실습"
date: 2026-06-02
tags: [NVIDIA, 생성형AI, GererativeAI, LLM, GPU, MLOps, Hugging Face, Transformer]
typora-root-url: ../
toc: true
categories: [NVIDIA]
---
최근 생성형 AI 개발의 중심은 단순히 LLM API를 호출하는 단계에서 벗어나고 있다. 실제 서비스 환경에서는 어떤 모델을 선택할 것인지뿐만 아니라, 어떻게 배포하고, 얼마나 빠르게 추론하며, GPU 자원을 어떻게 효율적으로 활용할 것인지가 중요한 경쟁력이 되고 있다.

LLM 애플리케이션 개발자, AI 인프라 엔지니어, MLOps 엔지니어에게 이제 필요한 역량은 모델 사용법이 아니라 LLM 운영 환경 전체를 이해하는 능력이다. 따라서, 이를 위해 생성형AI 실습을 공개했다. 생성형 AI와 GPU 기반 딥러닝 시스템을 실습 중심으로 학습할 수 있도록 구성했고, 실습 예제들은 한국어로 된 Jupyter Notebook 으로 제공한다.

이 실습 과정은 LLM 배포와 추론 서빙부터 시작해, vLLM 운영 환경, 양자화, 재현 가능한 학습, GPU 프로파일링, 성능 최적화, 모델 평가까지 실제 AI 서비스 운영 과정에서 필요한 핵심 기술을 단계적으로 다룬다.

# 1. NVIDIA 생성형 AI 실습(Hands-on Workshop)

* [[실습] vllm을 이용해 라마8b 서빙 배포](https://synabreu.github.io/nvidia/%EC%8B%A4%EC%8A%B5-vllm%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%B4-%EB%9D%BC%EB%A7%888b-%EC%84%9C%EB%B9%99-%EB%B0%B0%ED%8F%AC/)
* [[실습] NVIDIA Triton Inference Server를 이용한 추론](https://synabreu.github.io/nvidia/%EC%8B%A4%EC%8A%B5-NVIDIA-Triton-Inference-Server%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%B6%94%EB%A1%A0/)