---
title: "Amazon Nova Canvas: Virtual Try-on & Style Options 출시"
date: 2025-07-02
tags: [아마존, Amazon, AWS, Amazon Bedrock, Agent, 생성형AI, Generative AI, Agentic AI, Strands Agents SDK, Amazon Nova, Amazon Nova Canvas]
typora-root-url: ../
toc: true
categories: [AWS]
---

아마존에서 오랜만에 흥미로운 기술이 나왔다. Amazon은 **Nova Canvas**를 통해 AI 이미지 생성 경험을 크게 향상시키는 Virtual Try-on 과 Style Options 등 **두 가지 신기능**을 공개했다. 특히, GenAI Startup 들에게 많은 영감을 줄 수 있는 기능이라서 노트를 정리해 보겠다. 



## 1.  가상 피팅 (Virtual Try-on)

* **입력 이미지 2장**만으로 시뮬레이션 가능 -> **사용자 or 공간 이미지** + **상품 이미지 (의류, 가구 등)**
* **AI 기반 합성**: 착용/배치 후의 현실적인 모습을 자동으로 생성
* 특징
  * 자연스러운 **피팅**, **드레이핑**, **조명 및 배경 일관성**
  * **전자상거래, 홈데코, 뷰티, AR 쇼핑**에 최적화된 몰입형 경험 제공



## 2.  가상 피팅 (Virtual Try-on) 실습 예제

* 가상 피팅 실습 로직

  *  현재 Nova Canvas는 이미지 생성 기반 API이며, `boto3` 기반으로 호출함
  * 가상 피팅은 2장의 이미지를 입력(사용자 이미지 + 아이템 이미지)으로 업로드한 후, API를 통해 결과를 얻는 방식

* 사전 준비

  * AWS CLI 인증 완료 (`aws configure`)

  * `boto3` 설치

    ```bash
    pip install boto3
    ```

  * 이미지 파일 2개 준비

    * `user_image.jpg`: 인물 또는 공간 이미지
    * `item_image.png`: 옷, 가구 등 합성 대상 이미지 (배경 제거된 PNG 권장)

* Virtual Try-on 예제 소스

  ```python
  import boto3
  import base64
  
  # Bedrock runtime 클라이언트
  client = boto3.client(service_name="bedrock-runtime", region_name="us-east-1")
  
  # 이미지 파일 Base64 인코딩
  def encode_image(file_path):
      with open(file_path, "rb") as f:
          return base64.b64encode(f.read()).decode("utf-8")
  
  # 이미지 로드
  user_img_base64 = encode_image("user_image.jpg")
  item_img_base64 = encode_image("item_image.png")
  
  # Nova Canvas Virtual Try-on 프롬프트 구성
  payload = {
      "inputImage": user_img_base64,
      "overlayImage": item_img_base64,
      "taskType": "VIRTUAL_TRY_ON",
      "parameters": {
          "outputResolution": "1024x1024",
          "prompt": "Realistic try-on with natural lighting and fit"
      }
  }
  
  # Bedrock에서 제공하는 Nova Canvas 모델 ID
  model_id = "amazon.nova-canvas-v1"
  
  # API 호출
  response = client.invoke_model(
      modelId=model_id,
      body=bytes(str(payload), encoding="utf-8"),
      contentType="application/json",
      accept="application/json"
  )
  
  # 결과 이미지 디코딩 및 저장
  result = response["body"].read()
  output = eval(result)  # JSON string to dict
  
  # Base64 디코딩하여 저장
  output_image_data = base64.b64decode(output["images"][0])
  with open("virtual_tryon_result.jpg", "wb") as f:
      f.write(output_image_data)
  
  print("Virtual Try-on 이미지가 생성되었습니다: virtual_tryon_result.jpg")
  ```

  * 주의 사항
    * 현재 Nova Canvas는 Bedrock 콘솔에서 사용자 인터페이스를 통해 Virtual Try-on 기능을 체험할 수 있으며, API는 preview 상태일 수 있음
    * `taskType`, `inputImage`, `overlayImage` 등의 필드는 실제 Nova Canvas에서 정의한 스펙에 맞게 조정해야 함

  

## 3. 스타일 옵션 (Style Options)

* 매번 프롬프트 입력 없이 **지정한 스타일 유지** 가능
* 현재 지원되는 **8가지 아트 스타일**:
  * 3D Animated Family Film
  * Design Sketch
  * Flat Vector Illustration
  * Graphic Novel Illustration
  * Maximalism
  * Midcentury Retro
  * Photorealism
  * Soft Digital Painting



## 4. Nova Canvas API를 활용해 ‘Style Options’ 기능 구현

* 정의: **Amazon Bedrock의 Nova Canvas API를 활용해 ‘Style Options’ 기능 중 ‘Photorealism’ 스타일을 적용**하여 이미지를 생성

* 목적

  * 입력 프롬프트에 `Photorealism` 스타일을 적용하여 일관된 결과 생성
  * Bedrock의 `amazon.nova-canvas-v1` 모델을 사용

* 사전 준비

  * AWS 계정에서 **Bedrock 권한** 활성화

  * Bedrock 모델이 배포된 리전 사용 (`us-east-1`, `ap-northeast-1`, `eu-west-1`)

  * 패키지 설치

    ```bash
    pip install boto3
    ```

* Style Option: Photorealism 코드

  ```python
  import boto3
  import json
  import base64
  
  # AWS Bedrock Runtime 클라이언트 초기화
  client = boto3.client("bedrock-runtime", region_name="us-east-1")  # 또는 ap-northeast-1 (도쿄)
  
  # Nova Canvas 모델 ID
  model_id = "amazon.nova-canvas-v1"
  
  # 이미지 생성 요청 payload
  payload = {
      "taskType": "TEXT_TO_IMAGE",
      "textToImageParams": {
          "text": "A futuristic living room with panoramic windows and smart furniture",
          "style": "PHOTOREALISM",  # Style Option 설정
          "cfgScale": 7.5,  # 스타일 강도 (선택적)
          "seed": 42,       # 재현 가능한 결과를 위한 시드값 (선택적)
          "resolution": "1024x1024"
      }
  }
  
  # 모델 호출
  response = client.invoke_model(
      modelId=model_id,
      contentType="application/json",
      accept="application/json",
      body=json.dumps(payload)
  )
  
  # 응답 파싱
  response_body = response["body"].read()
  output_json = json.loads(response_body)
  
  # Base64 디코딩 → 이미지 저장
  base64_image = output_json["images"][0]  # 하나의 이미지 생성
  image_data = base64.b64decode(base64_image)
  
  with open("photorealism_result.jpg", "wb") as f:
      f.write(image_data)
  
  print("스타일 'Photorealism'이 적용된 이미지가 생성되었습니다: photorealism_result.jpg")
  ```

* 지원 스타일 목록 (Style 옵션 이름) - 스타일은 반드시 **대문자 스네이크케이스**(`PHOTOREALISM`)로 지정해야 함

  * `PHOTOREALISM` 

  * `3D_ANIMATED_FAMILY_FILM` 

  * `DESIGN_SKETCH` 

  * `FLAT_VECTOR_ILLUSTRATION` 

  * `GRAPHIC_NOVEL_ILLUSTRATION` 

  * `MAXIMALISM` 

  * `MIDCENTURY_RETRO`

  * `SOFT_DIGITAL_PAINTING`

    

## 5. 활용 분야

* **패션**: 옷 입어보기, 스타일 추천
* **인테리어/홈데코**: 가구 배치 시뮬레이션
* **브랜드 콘텐츠 제작**: 일관된 아트 스타일 유지
* **마케팅 캠페인**: 몰입형 비주얼 생성



## 6. 참고

* [Amazon Nova Canvas adds virtual try-on and style options for image generation](https://aws.amazon.com/ko/about-aws/whats-new/2025/07/amazon-nova-canvas-virtual-try-on-style-options-image-generation/)

* [Build creative applications with Amazon Nova](https://aws.amazon.com/ai/generative-ai/nova/creative/)

* [Amazon Nova Canvas - AWS AI Service Cards](https://docs.aws.amazon.com/ai/responsible-ai/nova-canvas/overview.html)

  