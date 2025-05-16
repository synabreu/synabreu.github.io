---
title: "초거대 언어 모델(LLM)을 Kubernetes로 확장하기"
date: 2025-04-15
tags: [구글, 구글 클라우드, LLM, 쿠버네티스, Kubernetes, 마이크로서비스, Microservice, 도커, Docker, 파이썬, Python, GCP, LLMOps]
typora-root-url: ../
toc: true
categories: [Kubernetes]
---

챗봇과 가상 비서의 구동, 문서 분석 자동화, 고객 참여 향상 등 다양한 분야에서 생성형 AI와 오픈 소스가 산업 전반에 걸쳐 커다란 영향을 끼치고 있다고 생각한다. 예를 들어, GPT-4와 같은 대형 언어 모델(LLM)은 자연어 처리, 대화형 AI, 콘텐츠 생성 분야에서 인공지능의 가능성을 혁신적으로 넓혀 놓았다. 

그러나 LLM이 지닌 막대한 잠재력에도 불구하고, 이를 실제 환경에 효과적으로 배포하는 일은 독특한 도전 과제를 동반한다. 이러한 모델은 막대한 연산 자원, 원활한 확장성, 그리고 효율적인 트래픽 관리가 요구되며, 이는 프로덕션 환경에서 특히 중요하다.



## **1. 쿠버네티스에서 LLM 앱 개발 배경**

업계에서 가장 널리 사용되는 컨테이너 오케스트레이션 플랫폼으로 현대의 모던 앱과 마이크로서비스 개발에서 주요한 클러스터 환경으로 발전한 쿠버네티스(kubernetes)는 클라우드 네이티브 환경에서 LLM 기반 애플리케이션을 동적으로 안정적으로 관리하고 확장할 수 있는 프레임워크를 제공한다. 

컨테이너화된 워크로드를 처리하는 Kubernetes의 능력은, 성능과 유연성을 유지하면서 AI 솔루션을 실제 운영 환경에 도입하고자 하는 조직에 있어 핵심적인 도구이다. Kubernetes를 활용해 LLM 기반 애플리케이션을 배포하고 확장하는 전 과정을 요약해서 순서별로 따라해 볼 수 있도록 정리해본다. 



## **2. 사전 준비 사항**

* **Kubernetes에 대한 기본 지식**: `kubectl`, `deployment`, `service`, `pod` 등의 개념과 사용법 숙지
* **Docker 설치 및 설정**: 여러분의 시스템에 Docker가 설치되어 있고 정상적으로 구성함
* **Kubernetes 클러스터 실행 환경**: 로컬 머신에서는 `minikube`와 같은 도구를, 클라우드 환경에서는 **AWS EKS**, **Google GKE**, **Microsoft AKS** 중 하나를 사용하여 Kubernetes 클러스터를 설치 및 실행해야 함
* **Python 환경에 OpenAI 및 Flask 설치**: LLM 애플리케이션을 만들기 위해 OpenAI와 Flask 라이브러리를 Python에 설치해야 함
* 필요한 Python 의존성 설치

```python
!pip install openai flask
```



## **3. LLM 기반 애플리케이션 생성**

* Python을 기반으로 한 간단한 API를 만들어  OpenAI의 GPT-4와 같은 LLM과 상호작용하는 애플리케이션을 빌드하기 위해 app.py 파일명으로 생성하기

```python
from flask import Flask, request, jsonify
import openai
import os
 
# Initialize Flask app
app = Flask(__name__)
 
# Configure OpenAI API key
openai.api_key = os.getenv("OPENAI_API_KEY")
 
@app.route("/generate", methods=["POST"])
def generate():
    try:
        data = request.get_json()
        prompt = data.get("prompt", "")
        
        # Generate response using GPT-4
        response = openai.Completion.create(
            model="text-davinci-003",
            prompt=prompt,
            max_tokens=100
        )
        return jsonify({"response": response.choices[0].text.strip()})
    except Exception as e:
        return jsonify({"error": str(e)}), 500
 
if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

* 이 Flask 애플리케이션은 `/generate` 엔드포인트를 통해 프롬프트를 입력받고 GPT-4의 응답을 반환하는 API 서버로 동작함.



## **4. 앱 컨테이너화**

* 애플리케이션을 Kubernetes에 배포하려면, 먼저 Docker 컨테이너로 패키징해야 함
* `app.py`와 동일한 디렉터리에 `Dockerfile`이라는 이름으로 파일을 생성함.

```python
# Use an official Python runtime as the base image
FROM python:3.9-slim
 
# Set the working directory
WORKDIR /app
 
# Copy application files
COPY app.py /app
 
# Copy requirements and install dependencies
RUN pip install flask openai
 
# Expose the application port
EXPOSE 5000
 
# Run the application
CMD ["python", "app.py"]
```



## **5. Docker 이미지 빌드 및 레지스트리에 푸시하기**

* Docker 이미지를 빌드한 후, Docker Hub와 같은 컨테이너 레지스트리에 업로드해야 Kubernetes에서 사용할 수 있음

```bash
# Build the image
docker build -t your-dockerhub-username/llm-app:v1 .
 
# Push the image
docker push your-dockerhub-username/llm-app:v1
```

* Docker 이미지는 컨테이너 레지스트리에 업로드되었으며, Kubernetes 클러스터에서 이 이미지를 사용해 애플리케이션을 배포할 수 있음



## **6.  애플리케이션을 Kubernetes에 배포하기**

* Kubernetes에 애플리케이션을 배포하기 위해 **Deployment**와 **Service**를 생성함
* Deployment는 애플리케이션의 파드(pod)를 관리하고, Service는 외부에서 접근할 수 있도록 노출함
* `deployment.yaml`파일 작성

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: llm-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: llm-app
  template:
    metadata:
      labels:
        app: llm-app
    spec:
      containers:
      - name: llm-app
        image: your-dockerhub-username/llm-app:v1
        ports:
        - containerPort: 5000
        env:
        - name: OPENAI_API_KEY
          valueFrom:
            secretKeyRef:
              name: openai-secret
              key: api-key
---
apiVersion: v1
kind: Service
metadata:
  name: llm-app-service
spec:
  selector:
    app: llm-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: LoadBalancer
```

* **주의**: 위 설정은 OpenAI API 키를 `openai-secret`이라는 이름의 Kubernetes Secret에서 가져옵니다. 다음 단계에서 이를 생성 해야 함

```bash
kubectl create secret generic openai-secret --from-literal=api-key="your_openai_api_key"
```



* 참고로 이 YAML 파일을 적용하면 Kubernetes 클러스터에 2개의 LLM 애플리케이션 파드가 생성되고, 외부에서 접근 가능한 LoadBalancer 서비스가 구성되어야 함. 



## **7.  Deployment 및 Service 적용하기**

* 작성한 Kubernetes 설정 파일을 클러스터에 적용하여 애플리케이션을 실제로 배포함

```bash
kubectl apply -f deployment.yaml
 
Verify the deployment:
kubectl get deployments
kubectl get pods
kubectl get services
```

* **클라우드 제공자(AWS, GCP, Azure 등)를 사용하는 경우**: `EXTERNAL-IP` 값을 확인하라. 이 IP 주소를 통해 외부에서 애플리케이션에 접근할 수 있음.
* **Minikube를 사용하는 경우**: `NodePort` 값을 확인하세요. 해당 포트를 통해 로컬 호스트(`localhost:<NodePort>`)에서 애플리케이션에 접근할 수 있음 

```bash
kubectl get service
```



## **8.  오토스케일링 구성하기**

* Kubernetes의 **Horizontal Pod Autoscaler (HPA)** 기능을 사용하면, CPU 또는 메모리 사용률에 따라 파드 수를 자동으로 조절할 수 있음

```bash
kubectl autoscale deployment llm-app --cpu-percent=50 --min=3 --max=1	
```

* 리소스 사용량에 따라 파드 수를 자동으로 조절해주는 **수평 파드 오토스케일러(HPA, Horizontal Pod Autoscaler)**를 다음과 같이 적용하면,
  * `--cpu-percent=50`: 평균 CPU 사용률이 50%를 초과하면 스케일 아웃(확장)됨. 
  * `--min=2`: 최소 파드 수는 2개
  * `--max=10`: 최대 파드 수는 10개까지 확장될 수 있음

```bash
kubectl autoscale deployment llm-app --cpu-percent=50 --min=3 --max=1	
```

* HPA 상태를 점검하라.

```bash
kubectl get hpa
```

* 오토스케일러는 부하에 따라 `llm-app` 배포의 파드 수를 자동으로 조절



## **9.  모니터링 및 로깅**

* 모니터링과 로깅은 LLM 애플리케이션을 안정적으로 운영하고 문제를 신속하게 해결하기 위해 매우 중요함
* 모니터링 활성화하기 위해,
  * **Prometheus**와 **Grafana**와 같은 도구를 사용하면 Kubernetes 클러스터의 상태를 시각화하고 상세히 모니터링할 수 있음
  * **기본적인 리소스 사용량 모니터링**을 위해서는 **Kubernetes Metrics Server**를 사용할 수 있음

* Metrics Server 설치: 설치가 완료되면 HPA(오토스케일러)가 CPU 사용률을 기반으로 정상 동작하며, `kubectl top pods` 같은 명령어를 통해 파드의 리소스 사용량을 확인할 수 있음

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

* 로그 보기: 실행 중인 파드의 로그를 확인
  * 추천: 집계된 로그를 확인하려면 **Fluentd**, **Elasticsearch**, **Kibana**와 같은 도구를 사용하는 것

```bash
kubectl logs <pod-name>
```



## **10.  애플리케이션 테스트하기**

* `curl`이나 Postman과 같은 도구를 사용하여 LLM API를 테스트

```bash
curl -X POST http://<external-ip>/generate \
-H "Content-Type: application/json" \
-d '{"prompt": "Explain Kubernetes in simple terms."}'
```

* 출력 기대

```json
{
  "response": "Kubernetes is an open-source platform that manages containers..."
}
```



## **11.  Kubernetes를 넘어서 확장하기**

* 더 복잡한 워크로드를 처리하거나 여러 지역(리전)에 걸쳐 배포하려면 다음과 같은 고급 기능을 고려하라.
  * **서비스 메시(Service Mesh)** 사용: Istio와 같은 도구를 활용하여 마이크로서비스 간의 트래픽을 관리할 수 있음
  * **멀티클러스터 배포 구현**: KubeFed(페더레이션) 또는 Google Anthos와 같은 클라우드 제공자의 솔루션을 사용하면 멀티 Kubernetes 클러스터를 통합 관리할 수 있음 
  * **CI/CD 통합**: Jenkins, GitHub Actions, GitLab CI 등과 같은 도구를 이용해 배포 파이프라인을 자동화할 수 있음 



## **12.  결론**

* Kubernetes를 활용해 LLM 기반 애플리케이션을 배포하고 확장하는 전 과정을 살펴보았음
* AI 애플리케이션을 얼마나 효율적으로 확장할 수 있느냐는 연구 환경에 머무는 모델과 실제로 가치를 창출하는 프로덕션 모델을 가르는 핵심 요소임. 
* LLM 애플리케이션을 컨테이너화하고, Kubernetes에 배포하며, 수요 변화에 따라 자동 확장을 설정하고, 최적의 성능을 위해 사용자 트래픽을 관리하는 방법을 다루었다.
* LLM 기반 API를 만들고 이를 Kubernetes 클러스터에 배포하고 확장하는 전 과정을 통해, 신뢰할 수 있고 확장 가능하며 프로덕션 준비가 완료된 AI 애플리케이션을 만드는 설계도를 갖추게 되었음
* Kubernetes의 **자동 확장(Auto Scaling)**, **모니터링**, **서비스 디스커버리** 등의 기능 덕분에 실제 환경에서도 안정적인 운영이 가능함
  * **카나리 배포(Canary Deployment):** 새 버전을 전체에 적용하기 전에 소수의 사용자에게 먼저 배포하여 안정성을 검증하는 방식
  * **A/B 테스트:** 두 가지 버전(A와 B)을 사용자 그룹에 나눠 제공해 어떤 버전이 더 나은 성과를 내는지 비교하는 실험 방식
  * **Knative 등 서버리스 컴포넌트 통합:** Kubernetes 위에서 서버리스 애플리케이션을 손쉽게 배포하고 자동 확장할 수 있게 해주는 플랫폼

