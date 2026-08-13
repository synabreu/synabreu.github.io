---
title: "NVIDIA GPU 기반 생성형AI 실습"
date: 2026-06-02
tags: [NVIDIA, 생성형AI, GererativeAI, LLM, GPU, MLOps, Hugging Face, Transformer, Jupyter Notebook, vLLM, HOL]
typora-root-url: ../
toc: true
categories: [NVIDIA]
---
최근 생성형 AI 개발의 중심은 단순히 LLM API를 호출하는 단계에서 벗어나고 있다. 실제 서비스 환경에서는 어떤 모델을 선택할 것인지뿐만 아니라, 어떻게 배포하고, 얼마나 빠르게 추론하며, GPU 자원을 어떻게 효율적으로 활용할 것인지가 중요한 경쟁력이 되고 있다.

LLM 애플리케이션 개발자, AI 인프라 엔지니어, MLOps 엔지니어에게 이제 필요한 역량은 모델 사용법이 아니라 LLM 운영 환경 전체를 이해하는 능력이다. 따라서, 이를 위해 생성형AI 실습을 공개했다. 생성형 AI와 GPU 기반 딥러닝 시스템을 실습 중심으로 학습할 수 있도록 구성했고, 실습 예제들은 한국어로 된 Jupyter Notebook 으로 제공한다.

이 실습 과정은 NVIDIA Generative AI Professional Certified 인증 시험을 준비하는 과정에 맞춰 LLM 배포와 추론 서빙부터 시작해, vLLM 운영 환경, 양자화, 재현 가능한 학습, GPU 프로파일링, 성능 최적화, 모델 평가까지 실제 AI 서비스 운영 과정에서 필요한 핵심 기술을 단계적으로 다룬다.

# 1. NVIDIA 생성형 AI 실습(Hands-on Workshop)

* [[실습] vllm을 이용해 라마8b 서빙 배포](https://synabreu.github.io/nvidia/%EC%8B%A4%EC%8A%B5-vllm%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%B4-%EB%9D%BC%EB%A7%888b-%EC%84%9C%EB%B9%99-%EB%B0%B0%ED%8F%AC/)
* [[실습] NVIDIA Triton Inference Server를 이용한 추론](https://synabreu.github.io/nvidia/%EC%8B%A4%EC%8A%B5-NVIDIA-Triton-Inference-Server%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%B6%94%EB%A1%A0/)
* [[실습] GPU 기반 vLLM 프로덕션 서빙](https://synabreu.github.io/nvidia/%EC%8B%A4%EC%8A%B5-GPU-%EA%B8%B0%EB%B0%98-vLLM-%ED%94%84%EB%A1%9C%EB%8D%95%EC%85%98-%EC%84%9C%EB%B9%99/)
* [[실습] GPU 기반 LLM 양자화 및 최적화](https://synabreu.github.io/nvidia/%EC%8B%A4%EC%8A%B5-GPU-%EA%B8%B0%EB%B0%98-LLM-%EC%96%91%EC%9E%90%ED%99%94-%EB%B0%8F-%EC%B5%9C%EC%A0%81%ED%99%94/)
* [[실습] GPU 기반 재현 가능한 학습](https://synabreu.github.io/nvidia/%EC%8B%A4%EC%8A%B5-GPU-%EA%B8%B0%EB%B0%98-%EC%9E%AC%ED%98%84-%EA%B0%80%EB%8A%A5%ED%95%9C-%ED%95%99%EC%8A%B5/)
* [[실습] Nsight Systems 프로파일링](https://synabreu.github.io/nvidia/%EC%8B%A4%EC%8A%B5-Nsight-Systems-%ED%94%84%EB%A1%9C%ED%8C%8C%EC%9D%BC%EB%A7%81/)
* [[실습] PyTorch 프로파일러와 텐서보드 활용](https://synabreu.github.io/nvidia/%EC%8B%A4%EC%8A%B5-PyTorch-%ED%94%84%EB%A1%9C%ED%8C%8C%EC%9D%BC%EB%9F%AC%EC%99%80-%ED%85%90%EC%84%9C%EB%B3%B4%EB%93%9C-%ED%99%9C%EC%9A%A9/)
* [[실습] BatchSize와 PrecisionSweep 를 통한 GPU 성능 최적화](https://synabreu.github.io/nvidia/%EC%8B%A4%EC%8A%B5-BatchSize%EC%99%80-PrecisionSweep-%EB%A5%BC-%ED%86%B5%ED%95%9C-GPU-%EC%84%B1%EB%8A%A5-%EC%B5%9C%EC%A0%81%ED%99%94/)
* [[실습] LLM 평가 및 벤치마킹](https://synabreu.github.io/nvidia/%EC%8B%A4%EC%8A%B5-LLM-%ED%8F%89%EA%B0%80-%EB%B0%8F-%EB%B2%A4%EC%B9%98%EB%A7%88%ED%82%B9/)

# 2. 전체 소스

* [NVIDIA Generative AI Hands-on Lab](https://github.com/synabreu/genai-hands-on-lab)
