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

# 1. 왜 지금 추론 엔지니어링을 배워야 하는가?

과거 머신러닝 시스템에서는 모델 정확도가 가장 중요한 평가 기준이었다. 하지만 생성형 AI 시대에는 새로운 문제가 등장했다. 같은 LLM 모델이라도 다음 요소에 따라 실제 서비스 품질은 크게 달라진다.

* 응답 지연 시간(Latency)
* 초당 처리 토큰 수(Throughput)
* GPU 메모리 사용량(VRAM)
* 동시 사용자 처리 능력
* 운영 비용

예를 들어 Llama 3 8B 모델을 단순히 Hugging Face Transformers의 generate() 함수로 실행하는 것과 vLLM 기반 서버로 운영하는 것은 완전히 다른 경험을 제공한다. Generative AI Hands-on Lab의 첫 번째 실습에서는 이러한 차이를 직접 측정한다.

# [실습1. 프로덕션 환경에서 LLM 배포 및 서빙](<https://github.com/synabreu/genai-hands-on-lab/blob/main/Lab%201-Deploy-Serve-llama8b-vllm-KOR.ipynb>)

첫 번째 실습에서는 LLM 배포와 추로 서빙에 대해 기본적으로 이해하고, Llama 3 8B Instruct 모델을 대상으로 두 가지 방식을 비교한다.

### 순차 추론(Sequential Inference)

<pre class="overflow-visible! px-0!" data-start="1172" data-end="1234"><div class="relative w-full mt-4 mb-1"><div class=""><div class="contents"><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-(--code-block-surface) corner-superellipse/1.1 overflow-clip rounded-3xl [--code-block-surface:var(--bg-elevated-secondary)] dark:[--code-block-surface:var(--composer-surface-primary)] lxnfua_clipPathFallback"><div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1"></div><div class="relative"><div class="pe-11 pt-3"><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr"><div class="cm-scroller"><pre class="cm-content q9tKkq_readonly m-0"><code><span>Client
  |
  |
Transformers.generate()
  |
  |
GPU</span></code></pre></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></div></pre>

### 운영 서빙(Production Serving)

<pre class="overflow-visible! px-0!" data-start="1260" data-end="1395"><div class="relative w-full mt-4 mb-1"><div class=""><div class="contents"><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-(--code-block-surface) corner-superellipse/1.1 overflow-clip rounded-3xl [--code-block-surface:var(--bg-elevated-secondary)] dark:[--code-block-surface:var(--composer-surface-primary)] lxnfua_clipPathFallback"><div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1"></div><div class="relative"><div class="pe-11 pt-3"><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr"><div class="cm-scroller"><pre class="cm-content q9tKkq_readonly m-0"><code><span>Multiple Requests
        |
        |
   vLLM Server
        |
        |
Continuous Batching
        |
        |
       GPU</span></code></pre></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></div></pre>

이 실습에서는 요청별 지연 시간, 토큰 생성 속도, GPU 메모리 사용량, 처리량 변화 등을 직접 측정한다. 이를 통해 왜 실제 서비스 환경에서는 단순 모델 실행이 아니라 전용 추론 엔진이 필요한지 이해할 수 있다.
