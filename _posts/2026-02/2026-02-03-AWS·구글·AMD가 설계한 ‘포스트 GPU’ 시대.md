---
title: "AWS·구글·AMD가 설계한 ‘포스트 GPU’ 시대"
date: 2026-02-03
tags: [nvidia, 엔비디아, 루빈, Rubin, GPU, HBM, AI, AWS, 구글, AMD, TPU, NPU, MIT, MIT TechnolgoyReview, Trainium, AIHyperComputer, AI하이퍼컴퓨터]
typora-root-url: ../
toc: true
categories: [NVIDIA]
---

지난 1월호가 NVIDIA GPU 중심의 AI 팩토리 였다면, [이번 MIT 테크 리뷰 저널 코리아 2월호는, NVIDIA GPU와 경쟁하고 있는 구글 TPU, AWS 트레니엄, 그리고 AMD 최신 칩 전략에 대해 살펴 보는 글을 기고했다.](https://www.technologyreview.kr/aws%C2%B7%EA%B5%AC%EA%B8%80%C2%B7amd%EA%B0%80-%EC%84%A4%EA%B3%84%ED%95%9C-%ED%8F%AC%EC%8A%A4%ED%8A%B8-gpu-%EC%8B%9C%EB%8C%80/) 

글의 핵심은 “GPU가 사라진다”가 아니라, AI 인프라 경쟁의 기준이 ‘최고 연산성능’에서 ‘비용·전력·메모리·데이터 이동·시스템 설계’로 이동하고 있다는 내용이다. 또한, 구글 TPU ‘아이언우드’를 예로 들며, AI 칩 설계가 단순 FLOPS 경쟁을 넘어 메모리와 데이터 이동, 시스템 구조 중심으로 바뀌고 있다고 설명한다.

## 1. 포스트-GPU의 본질은?

이것은 탈엔비디아가 아니라 탈범용 GPU 의존이다. 엔비디아 GPU는 여전히 강력하지만, 대규모 AI 서비스에서는 GPU 가격, 전력, 공급 제약, 운영비가 점점 부담이 된다. 그래서 AWS, 구글, AMD는 “GPU 왕좌를 바로 빼앗는 것”보다 각자 더 경제적인 AI 인프라 구조를 만드는 방향으로 움직이고 있다.

## 2. AWS의 전략은 성능보다 비용 계산이다.

AWS는 Trainium 계열로 학습·추론 비용을 낮추는 데 집중한다. AWS는 Trn2가 GPU 기반 EC2 P5e/P5en 대비 30~40% 더 나은 가격 대비 성능을 제공한다고 밝히고, Trainium2 UltraServer는 64개 Trainium2 칩, 6TB HBM, 185TB/s 메모리 대역폭을 제공한다고 설명한다. 즉 AWS의 핵심은 “가장 빠른 칩”이 아니라 “클라우드에서 가장 싸게 대규모 AI를 돌리는 구조”다.

## 3. 구글의 전략은 TPU와 AI 하이퍼컴퓨터다.

구글은 Ironwood TPU를 대규모 추론 시대에 맞춘 7세대 TPU로 소개한다. 특히 Ironwood는 단일 칩 성능보다 대규모 Pod, 메모리, 네트워크, 데이터 이동 효율을 중시한다. 구글 문서에 따르면 TPU7x Ironwood는 9,216개 칩 규모의 Pod 구성이 가능하며, 대규모 dense·MoE 모델, 사전학습, 샘플링, decode-heavy 추론에 맞춰 설계됐다.

## 4. AMD의 전략은 ‘대체 가능한 GPU 생태계’다.

AMD는 완전히 다른 ASIC보다 Instinct GPU 라인업으로 엔비디아의 대안이 되려 한다. AMD는 MI350 시리즈가 CDNA 4 기반으로 2025년 출시 예정이며, MI400 시리즈는 2026년 CDNA “Next” 기반으로 계획되어 있다고 밝혔다. 즉 AMD는 고객에게 “CUDA 독점에서 벗어날 수 있는 GPU 선택지”를 주는 방향이다.

## 5. 결론은?

AI 인프라 전쟁의 무대가 바뀌고 있다는 것이다. 앞으로의 경쟁은 “누가 가장 빠른 GPU를 만드느냐”보다 “누가 AI를 산업 규모로 더 싸고 안정적으로 운영하게 해주느냐”가 된다. 엔비디아는 여전히 CUDA와 GPU 생태계의 성벽을 갖고 있지만, AWS는 비용 구조, 구글은 시스템 최적화, AMD는 대체 GPU 생태계로 그 성벽 바깥에서 다른 판을 짜고 있다는 것이 글의 핵심이다.


* 컬럼: [AWS·구글·AMD가 설계한 ‘포스트 GPU’ 시대](https://www.technologyreview.kr/aws%C2%B7%EA%B5%AC%EA%B8%80%C2%B7amd%EA%B0%80-%EC%84%A4%EA%B3%84%ED%95%9C-%ED%8F%AC%EC%8A%A4%ED%8A%B8-gpu-%EC%8B%9C%EB%8C%80/)
