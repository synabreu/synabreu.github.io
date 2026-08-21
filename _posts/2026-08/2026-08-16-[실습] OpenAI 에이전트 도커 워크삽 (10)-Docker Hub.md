---
title: "[실습] OpenAI 에이전트 도커 워크삽 (10)-Docker Hub"
date: 2026-08-16
tags: [오픈AI, OpenAI, GPT-5.6, agenticai, aiagent, openrouter, docker, powershell, fastapi, swagger-ui, visual studio code, Linux, Windows11, Kubernetes, On-Premises ]
typora-root-url: ../
toc: true
categories: [openai]
---
이 실습은 도커 허브(Docker Hub) 또는 사설 레지스트리(Private Registery) 로 이동하는 방법이다. 참고로 도커 허브란 도커 이미지를 저장하고 공유하는 클라우드 저장소이다. 또한, 사설 레지스트리는 기업 조직 내부에서만 사용하는 비공개 도커 이미지 저장소를 말한다. 


# 1. 실습 11 - 도커 허브 또는 사설 레지스트리로 이동

예를 들어 Docker Hub를 사용할 경우:

```powershell
docker login

docker tag openai-agents-workshop:1.0 `
  YOUR_DOCKER_ID/openai-agents-workshop:1.0

docker push YOUR_DOCKER_ID/openai-agents-workshop:1.0
```

Public Cloud/On-Premises Linux 서버에서는:

```bash
docker pull YOUR_DOCKER_ID/openai-agents-workshop:1.0

docker run -d \
  --name openai-agents-workshop \
  -p 8000:8000 \
  -e OPENAI_API_KEY="$OPENAI_API_KEY" \
  -e OPENAI_DEFAULT_MODEL="gpt-5.6-luna" \
  YOUR_DOCKER_ID/openai-agents-workshop:1.0
```

같은 이미지를 사용하므로 애플리케이션 코드와 파이썬 종속성(Python dependency)를 서버마다 다시 설치할 필요가 없다.

# 2. 실습 12 - 퍼블릭 클라우드 및 온-프레미시스 배포 개념

```text
                  Container Registry
                         │
              openai-agents-workshop
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
       AWS/Azure        GCP          On-Prem
          │              │              │
    Docker / K8s   Docker / K8s   Docker / K8s
          └──────────────┼──────────────┘
                         │
                         ▼
                     OpenAI API
```

배포 환경에서 달라지는 것은 주로 다음이다.

- API Key / Secret 관리 방식
- 네트워크와 Proxy/Firewall
- TLS/Ingress
- 로그 수집
- Auto Scaling
- Container Registry
- Kubernetes resource 설정

애플리케이션 이미 자체는 동일하게 유지할 수 있다.

# 실습 13 - ARM64까지 지원하는 멀티 플랫폼 이미지 

AMD64 서버와 ARM64 서버를 동시에 지원하려면 Buildx를 사용할 수 있다.

```powershell
docker buildx build `
  --platform linux/amd64,linux/arm64 `
  -t YOUR_DOCKER_ID/openai-agents-workshop:1.0 `
  --push .
```

레지스트리에는 여러 아키텍처 매니페스트(architecture manifest)가 하나의 이미지 태그(image tag) 아래와 같이 저장된다.

```text
openai-agents-workshop:1.0
        │
        ├─ linux/amd64
        └─ linux/arm64
```

각 서버는 자신의 CPU 아키텍처에 맞는 변종(variant)를 자동으로 pull할 수 있다.
