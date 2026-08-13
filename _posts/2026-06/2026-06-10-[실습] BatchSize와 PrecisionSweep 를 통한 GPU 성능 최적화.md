---
title: "[실습] BatchSize와 PrecisionSweep 를 통한 GPU 성능 최적화"
date: 2026-06-10
tags: [트랜스포머, Transformer, Inference, LLM, Optimization, Serving, vllm, gpu, k8s, kubernetes, triton, tgi, ollama, tensorRT-LLM]
typora-root-url: ../
toc: true
categories: [NVIDIA]
---

실습 8 번쨰로 배치 크기와 수치 정밀도 스윕을 통한 성능 최적점 탐색을 해 보자! **GPU 성능을 높이는 데 가장 큰 영향을 주는 조정 변수는 배치 크기(batch size)와 수치 정밀도(numerical precision)다.** 이 두 요소를 적절히 설정하면 모델 구조나 프레임워크를 변경하지 않고도 단일 GPU의 처리량(throughput)을 크게 높일 수 있다. 반대로 잘못 설정하면 GPU의 연산 자원과 전력 예산을 충분히 활용하지 못한다.

이 실습에서는 각 설정 조합에서 처리량과 출력 품질을 함께 측정한다. 예를 들어 FP16으로 변경한 뒤 코사인 유사도(cosine similarity)가 기준 이하로 떨어진다면, 이는 단순한 가속이 아니라 정확도 회귀(accuracy regression)로 판단해야 한다.

### 실습 내용

* **배치 크기 스윕(batch size sweep)** : 동일한 모델과 동일한 정밀도에서 배치 크기만 변경한다. 배치 크기에 따른 처리량, 최대 GPU 메모리 사용량, 지연 시간(latency)을 측정한다. 이후 처리량 증가가 둔화되는 무릎 지점(knee point)을 찾는다.
* **수치 정밀도 스윕(precision sweep)** : 고정된 배치 크기에서 FP32, FP16, BF16을 적용한다. 각 정밀도의 처리량과 FP32 기준 출력 대비 코사인 유사도를 측정한다.
* **복합 스윕(combined sweep)** : 배치 크기와 수치 정밀도를 함께 변경한다. GPU 메모리(VRAM) 예산을 만족하는 조합 중 처리량이 가장 높은 구성을 선택한다.
* **운영 환경 권장 구성(production recommendation)** : 최적 배치 크기와 정밀도, 메모리 부족(OOM, Out of Memory) 방지를 위한 안전 여유분, GPU 제품별 권장 정밀도, 정확도 통과 기준을 정리한다.

### 주요 수치 정밀도

| 정밀도 | 비트 구성                        | 표현 범위        | 장점                                                                                   | 한계                                                                                                                 |
| ------ | -------------------------------- | ---------------- | -------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| FP32   | 32비트(부호 1, 지수 8, 가수 23) | 약`1e±38`    | 기준 정밀도로 사용하기 좋고 수치  안정성이 높다.                                      | 메모리 사용량과 연산 비용이 가장 크다.                                                                               |
| FP16   | 16비트(부호 1, 지수 5, 가수 10)  | 약`6.5e4`      | 지원 GPU에서 텐서코어를 활용할 수 있다.                                                | <br />표현 범위가 좁아 오버플로와 언더플로가 발생할 수 있다. * 학습 시 손실 스케일링(loss scaling)이 필요할 수 있다. |
| BF16   | 16비트(부호 1, 지수 8, 가수 7)   | FP32와 유사      | FP32와 동일한 지수부를 사용하므로 표현 범위가 넓다. Ampere 이후 GPU에서 널리 사용한다. | 가수부가 짧아 개별 연산의 정밀도는 FP32보다 낮다.                                                                    |
| FP8    | 8비트                            | 형식에 따라 다름 | Hopper 이후 GPU의 행렬 연산에서 높은 처리량을 제공한다.                                | 텐서별 스케일링과 모델 보정(calibration)이 필요하다.                                                                 |

* *가수부(mantissa)는 부동소수점 값의 유효 숫자 정밀도를 결정한다.*
* *지수부(exponent)는 표현 가능한 수의 범위를 결정한다.*

### 용어 - 무릎 지점(knee point)

배치 크기가 작을 때는 커널 실행 오버헤드(kernel launch overhead)와 CPU·드라이버 처리 시간이 전체 실행 시간에서 큰 비중을 차지한다. 배치 크기가 증가하면 하나의 커널 실행으로 더 많은 입력을 처리하므로 처리량이 빠르게 증가한다.

그러나 일정 지점을 넘으면 GPU 연산 장치, 메모리 대역폭, 활성값 메모리가 포화 상태에 가까워진다. 이후 배치 크기를 더 늘리면 배치 지연 시간과 GPU 메모리 사용량은 증가하지만 처리량 증가는 작아진다. 이 구간이 무릎 지점이다.

**참고 자료** 
- [NVIDIA Mixed Precision Training](https://docs.nvidia.com/deeplearning/performance/mixed-precision-training/index.html) 
- [BFloat16: The secret to high performance on Cloud TPUs](https://cloud.google.com/blog/products/ai-machine-learning/bfloat16-the-secret-to-high-performance-on-cloud-tpus) 
- [FP8 Formats for Deep Learning](https://arxiv.org/abs/2209.05433) 
- [Tensor Core Matrix Multiplication Performance](https://docs.nvidia.com/deeplearning/performance/dl-performance-matrix-multiplication/)

## 1단계 - 배치 크기 스윕(batch size sweep)

소규모 트랜스포머 인코더(transformer encoder)의 순전파를 고정하고 배치 크기 `1`, `4`, `16`,`64`를 순차적으로 측정한다. 각 배치 크기에서 다음 지표를 수집한다.

- **처리량(throughput)** - 초당 처리 가능한 샘플 수(samples/s)다.
- **최대 GPU 메모리(peak VRAM)** - 실행 중 기록된 최대 GPU 메모리 할당량이다.
- **배치 지연 시간(batch latency)** - 하나의 배치를 처리하는 데 걸리는 시간이다.

배치 크기가 증가할 때 다음 변화를 확인해야 한다.

1. 초기에는 고정 오버헤드가 여러 샘플에 분산되므로 처리량이 빠르게 증가한다.
2. 배치 단위 지연 시간은 증가하지만 샘플당 처리 비용은 감소할 수 있다.
3. 활성값(activation) 메모리가 배치 크기에 비례해 증가하므로 GPU 메모리 사용량도 대체로 증가한다.

### 1-1. 모델과 고정 상수 정의

배치 크기 변화에 따른 차이를 명확하게 확인할 수 있도록 소규모 트랜스포머 인코더를 사용한다. 배치 크기 1에서는 커널 실행 오버헤드의 영향을 비교적 크게 받으며, 배치 크기가 증가하면 GPU 활용률이 개선되는 구조다.

모델 차원, 시퀀스 길이, 계층 수, 어휘 크기, 반복 횟수, 워밍업 횟수는 전역 상수로 정의한다. 더 큰 모델을 시험하려면 `D_MODEL`, `N_LAYERS`,`SEQ_LEN` 값을 조정하면 된다.

```python
import time
import warnings

import torch
import torch.nn as nn

# 중첩 텐서(nested tensor) 관련 경고가 실습 출력에 반복되지 않도록 숨긴다.
warnings.filterwarnings(
    'ignore',
    message='.*enable_nested_tensor.*',
)

# 모든 모델과 입력 텐서를 CUDA GPU에서 실행하도록 장치를 지정한다.
device = 'cuda'

# 트랜스포머 은닉 차원(hidden dimension)을 정의한다.
# 모델을 작게 유지해 배치 크기 1에서 커널 실행 오버헤드의 영향을 관찰한다.
D_MODEL = 256

# 각 입력 샘플의 토큰 길이(sequence length)를 고정한다.
SEQ_LEN = 64

# 트랜스포머 인코더 계층 수를 정의한다.
N_LAYERS = 2

# 임베딩 계층과 출력 계층에서 사용하는 어휘 크기를 정의한다.
VOCAB = 16_000

# 실제 실행 시간을 측정할 반복 횟수를 정의한다.
ITERS = 100

# CUDA 초기화와 내부 버퍼 할당 비용을 제외하기 위한 워밍업 횟수를 정의한다.
WARMUP = 10


class SmallTransformer(nn.Module):
    """배치 크기와 정밀도별 성능을 측정하기 위한 소규모 트랜스포머 인코더다."""

    def __init__(
        self,
        d=D_MODEL,
        nlayers=N_LAYERS,
        seq_len=SEQ_LEN,
        vocab=VOCAB,
    ):
        # nn.Module의 초기화 로직을 실행한다.
        super().__init__()

        # 정수 토큰 ID를 d차원 임베딩 벡터로 변환한다.
        self.embed = nn.Embedding(vocab, d)

        # 하나의 트랜스포머 인코더 계층을 정의한다.
        layer = nn.TransformerEncoderLayer(
            d_model=d,
            nhead=4,
            dim_feedforward=4 * d,
            dropout=0.0,
            batch_first=True,
            norm_first=True,
        )

        # 동일한 구조의 인코더 계층을 nlayers개 쌓는다.
        self.enc = nn.TransformerEncoder(
            layer,
            num_layers=nlayers,
        )

        # 각 토큰의 은닉 벡터를 어휘 크기의 출력 로짓(logit)으로 변환한다.
        self.head = nn.Linear(d, vocab)

    def forward(self, x):
        # 토큰 ID를 임베딩 벡터로 변환한다.
        embedded = self.embed(x)

        # 임베딩 시퀀스를 트랜스포머 인코더에 전달한다.
        encoded = self.enc(embedded)

        # 인코더 출력을 어휘별 로짓으로 변환해 반환한다.
        return self.head(encoded)
```

```text
/usr/local/lib/python3.11/dist-packages/torch/cuda/__init__.py:65: FutureWarning: The pynvml package is deprecated. Please install nvidia-ml-py instead. If you did not install pynvml directly, please report this to the maintainers of the package that installed pynvml for you.
  import pynvml  # type: ignore[import]
```

### 1-2. 성능 측정 함수

`bench(model, batch, dtype)` 함수는 처리량, 최대 GPU 메모리 사용량, 배치 지연 시간을 반환한다.

정확한 CUDA 성능 측정을 위해 다음 두 가지가 중요하다.

1. **측정 전 워밍업(warmup)을 수행한다.**초기 반복에는 CUDA 컨텍스트 생성, 커널 초기화, 내부 작업
   공간(workspace) 할당 비용이 포함될 수 있다.
2. **측정 직전과 직후에 `torch.cuda.synchronize()`를 호출한다.**
   CUDA 연산은 기본적으로 비동기식(asynchronous)으로 실행된다.
   동기화하지 않으면 실제 GPU 실행 시간이 아니라 CPU가 작업을 제출한
   시간만 측정될 수 있다.

```python
def bench(model, batch, dtype=torch.float32):
    # 이전 측정에서 사용하지 않는 CUDA 캐시 메모리를 해제한다.
    # 캐시 해제는 측정 간 메모리 상태를 비슷하게 맞추기 위한 용도다.
    torch.cuda.empty_cache()

    # 최대 GPU 메모리 통계를 0으로 초기화한다.
    torch.cuda.reset_peak_memory_stats()

    # 모델을 지정한 GPU 장치와 수치 정밀도로 이동한다.
    model = model.to(
        device=device,
        dtype=dtype,
    )

    # 지정한 배치 크기와 시퀀스 길이를 가진 임의 토큰 입력을 GPU에 생성한다.
    x = torch.randint(
        0,
        VOCAB,
        (batch, SEQ_LEN),
        device=device,
    )

    # 추론 벤치마크이므로 기울기 계산과 자동 미분 그래프 생성을 비활성화한다.
    with torch.no_grad():
        # 초기 CUDA 컨텍스트, 커널, 내부 작업 공간 할당 비용을 측정에서 제외한다.
        for _ in range(WARMUP):
            _ = model(x)

        # 워밍업에서 제출한 모든 비동기 CUDA 연산이 끝날 때까지 기다린다.
        torch.cuda.synchronize()

        # 실제 측정 구간의 시작 시각을 기록한다.
        start_time = time.perf_counter()

        # 동일한 입력으로 순전파를 반복 실행한다.
        for _ in range(ITERS):
            _ = model(x)

        # 반복 실행한 CUDA 연산이 모두 완료될 때까지 기다린다.
        torch.cuda.synchronize()

        # 전체 측정 구간의 경과 시간을 계산한다.
        elapsed = time.perf_counter() - start_time

    # 실행 중 기록된 최대 GPU 메모리 할당량을 MiB 단위로 변환한다.
    peak_vram_mib = (
        torch.cuda.max_memory_allocated()
        // (1024 * 1024)
    )

    # 전체 처리 샘플 수를 경과 시간으로 나눠 초당 처리량을 계산한다.
    samples_per_s = (
        ITERS * batch
        / elapsed
    )

    # 한 번의 배치 순전파에 걸린 평균 시간을 밀리초 단위로 계산한다.
    latency_ms = (
        1000 * elapsed
        / ITERS
    )

    # 처리량, 최대 GPU 메모리, 배치 지연 시간을 반환한다.
    return samples_per_s, peak_vram_mib, latency_ms
```

### 1-3. 배치 크기 스윕과 무릎 지점 탐색

배치 크기 `1`, `4`, `16`, `64`에서 처리량을 측정한다. 현재 코드에서는
이전 측정값 대비 처리량 증가율이 10% 이상인 마지막 배치 크기를 무릎
지점으로 선택한다.

이 기준은 실습용 휴리스틱(heuristic, 정확한 수학적 최적해를 구하는 대신
경험적 기준이나 간단한 규칙으로 빠르게 판단하는 방법)이다. 운영
환경에서는 서비스 수준 목표(SLO, Service Level Objective), 최대 지연
시간, 동시 요청 수, GPU 메모리 여유분을 함께 고려해야 한다.

```python
# 동일한 모델 구조를 한 번 생성해 배치 크기별 측정에 사용한다.
model = SmallTransformer()

# 각 배치 크기의 측정 결과를 저장할 목록을 생성한다.
batch_sweep = []

# 작은 배치부터 큰 배치까지 순차적으로 성능을 측정한다.
for batch_size in [1, 4, 16, 64]:
    # FP32 정밀도에서 처리량, 최대 GPU 메모리, 배치 지연 시간을 측정한다.
    throughput, peak_vram, latency = bench(
        model,
        batch_size,
        dtype=torch.float32,
    )

    # 후속 분석에 사용할 수 있도록 측정 결과를 구조화해 저장한다.
    batch_sweep.append({
        'batch_size': batch_size,
        'throughput_samples_per_s': throughput,
        'peak_vram_mib': peak_vram,
        'latency_ms': latency,
    })

    # 현재 배치 크기의 주요 성능 지표를 출력한다.
    print(
        f'배치={batch_size:3d}: '
        f'{throughput:8.1f} 샘플/초 | '
        f'최대 VRAM {peak_vram:5d} MiB | '
        f'지연 시간 {latency:5.2f} ms/배치'
    )

# 첫 번째 배치 크기를 초기 무릎 지점 후보로 설정한다.
knee_batch_size = batch_sweep[0]['batch_size']

# 인접한 두 측정값의 처리량 증가율을 순차적으로 계산한다.
for previous, current in zip(
    batch_sweep[:-1],
    batch_sweep[1:],
):
    # 이전 구성 대비 현재 구성의 처리량 증가율을 계산한다.
    gain = (
        current['throughput_samples_per_s']
        - previous['throughput_samples_per_s']
    ) / previous['throughput_samples_per_s']

    # 처리량 증가율이 10% 이상이면 현재 배치를 유효한 확장 지점으로 기록한다.
    if gain >= 0.10:
        knee_batch_size = current['batch_size']

# 실습용 기준으로 찾은 무릎 지점을 출력한다.
print(
    f'\n무릎 지점: 배치={knee_batch_size} '
    f'(이전 측정 대비 처리량 증가율이 10% 이상인 마지막 지점)'
)
```

```text
  batch=  1:   3456.4 samples/s | peak VRAM    55 MiB | latency  0.29 ms/batch
  batch=  4:  12270.0 samples/s | peak VRAM    79 MiB | latency  0.33 ms/batch
  batch= 16:  34768.3 samples/s | peak VRAM   173 MiB | latency  0.46 ms/batch
  batch= 64:  44832.1 samples/s | peak VRAM   551 MiB | latency  1.43 ms/batch

Knee found at batch=64 (last size where doubling gave >=10% gain)
```

## 2단계 --- 수치 정밀도 스윕(precision sweep)

동일한 모델, 동일한 가중치, 동일한 입력, 동일한 배치 크기를 사용하고
수치 정밀도만 변경한다. 각 정밀도에서 처리량, 최대 GPU 메모리 사용량,
FP32 기준 출력 대비 코사인 유사도를 측정한다.

- **FP32** --- 기준 출력(reference output)을 생성하는 기본 정밀도다.
- **FP16** --- 메모리 사용량과 연산량을 줄일 수 있지만 표현 범위가
  좁다.
- **BF16** --- FP32와 같은 지수부를 사용해 표현 범위가 넓으며 Ampere
  이후 GPU에서 널리 사용한다.

처리량이 증가하더라도 출력 유사도가 허용 기준보다 낮아지면 안전한
최적화로 볼 수 없다.

### 잘못된 비교 방식

서로 다른 정밀도의 텐서를 그대로 비교하면 비교 연산 자체가 낮은
정밀도에서 수행되거나 자료형 불일치 오류가 발생할 수 있다. 따라서 출력
텐서를 모두 FP32로 변환한 뒤 코사인 유사도를 계산해야 한다.

### FP32 기준 출력과 코사인 유사도 계산

정밀도에 따른 출력 차이만 측정하려면 다음 조건을 고정해야 한다.

- 동일한 입력을 사용한다.
- 동일한 모델 가중치를 사용한다.
- FP32 출력 결과를 기준값으로 사용한다.
- 코사인 유사도 계산은 FP32에서 수행한다.

코사인 유사도는 두 출력 벡터의 방향이 얼마나 유사한지 나타낸다. 값이 1에
가까울수록 FP32 기준 출력과 유사하다.

```python
import torch.nn.functional as F

# 여러 정밀도에서 동일한 입력을 생성할 수 있도록 난수 시드를 고정한다.
torch.manual_seed(42)

# 정밀도 비교에 사용할 고정 배치 크기를 정의한다.
FIXED_BATCH = 16

# 모든 정밀도에서 공통으로 사용할 토큰 입력을 GPU에 생성한다.
fixed_input = torch.randint(
    0,
    VOCAB,
    (FIXED_BATCH, SEQ_LEN),
    device=device,
)

# FP32 기준 모델을 생성하고 추론 모드로 전환한다.
model = SmallTransformer().to(
    device=device,
    dtype=torch.float32,
).eval()

# 자동 미분 그래프를 생성하지 않고 FP32 기준 출력을 계산한다.
with torch.no_grad():
    fp32_out = model(fixed_input).float()


def cosine_vs_fp32(out):
    # 기준 출력과 비교 출력 모두 FP32로 변환한 뒤 1차원 벡터로 펼친다.
    # 비교 지표 자체가 낮은 정밀도에서 계산되는 문제를 방지한다.
    reference = fp32_out.float().flatten()
    candidate = out.float().flatten()

    # 두 출력 벡터 사이의 코사인 유사도를 계산하고 Python 실수로 반환한다.
    return F.cosine_similarity(
        reference,
        candidate,
        dim=0,
    ).item()
```

### 정밀도별 벤치마크 함수

`bench_with_dtype(dtype)` 함수는 지정한 자료형으로 모델을 변환한 뒤
처리량, 최대 GPU 메모리 사용량, FP32 기준 코사인 유사도를 반환한다.

모든 정밀도에서 동일한 가중치를 사용하기 위해 FP32 모델의 `state_dict`를
대상 정밀도로 변환해 불러온다. 따라서 측정값의 차이는 가중치 초기화가
아니라 수치 표현 형식에서 발생한다.

```python
def bench_with_dtype(dtype):
    # 이전 정밀도 측정에서 남은 CUDA 캐시를 비운다.
    torch.cuda.empty_cache()

    # 최대 GPU 메모리 사용량 통계를 초기화한다.
    torch.cuda.reset_peak_memory_stats()

    # 측정 대상 정밀도로 새로운 모델을 생성하고 추론 모드로 전환한다.
    candidate_model = SmallTransformer().to(
        device=device,
        dtype=dtype,
    ).eval()

    # FP32 기준 모델의 가중치를 대상 정밀도로 변환해 불러온다.
    # 모든 정밀도에서 동일한 가중치를 사용해 공정하게 비교한다.
    converted_state = {
        name: value.to(dtype)
        for name, value in model.state_dict().items()
    }
    candidate_model.load_state_dict(converted_state)

    # 추론 측정이므로 기울기 계산을 비활성화한다.
    with torch.no_grad():
        # 초기화와 내부 버퍼 할당 비용을 제외하기 위해 워밍업한다.
        for _ in range(WARMUP):
            _ = candidate_model(fixed_input)

        # 워밍업 CUDA 연산이 모두 끝날 때까지 기다린다.
        torch.cuda.synchronize()

        # 정밀도별 측정 시작 시각을 기록한다.
        start_time = time.perf_counter()

        # 동일한 고정 입력으로 순전파를 반복 실행한다.
        for _ in range(ITERS):
            output = candidate_model(fixed_input)

        # 모든 비동기 CUDA 연산이 끝날 때까지 기다린다.
        torch.cuda.synchronize()

        # 전체 반복 실행 시간을 계산한다.
        elapsed = time.perf_counter() - start_time

    # 실행 중 기록된 최대 GPU 메모리를 MiB 단위로 변환한다.
    peak_vram_mib = (
        torch.cuda.max_memory_allocated()
        // (1024 * 1024)
    )

    # 초당 처리 가능한 샘플 수를 계산한다.
    throughput = (
        ITERS * FIXED_BATCH
        / elapsed
    )

    # 마지막 출력과 FP32 기준 출력 사이의 코사인 유사도를 계산한다.
    cosine_similarity = cosine_vs_fp32(output)

    # 처리량, 최대 GPU 메모리, 출력 유사도를 반환한다.
    return throughput, peak_vram_mib, cosine_similarity
```

### FP32, FP16, BF16 비교

각 정밀도에서 처리량과 출력 유사도를 측정한다. 일반적으로 BF16은 FP32와
표현 범위가 같아 수치 안정성이 높고, FP16은 일부 모델이나 입력에서
오버플로 또는 언더플로가 발생할 수 있다. 실제 결과는 GPU 아키텍처,
PyTorch 버전, 커널 구현, 모델 구조에 따라 달라질 수 있다.

```python
# 정밀도별 측정 결과를 저장할 목록을 생성한다.
precision_sweep = []

# FP32, FP16, BF16을 순차적으로 측정한다.
for precision_name, dtype in [
    ('fp32', torch.float32),
    ('fp16', torch.float16),
    ('bf16', torch.bfloat16),
]:
    # 현재 정밀도의 처리량, 최대 GPU 메모리, 출력 유사도를 측정한다.
    throughput, peak_vram, cosine_similarity = bench_with_dtype(dtype)

    # 후속 비교와 최적 구성 선택에 사용할 결과를 저장한다.
    precision_sweep.append({
        'precision': precision_name,
        'throughput_samples_per_s': throughput,
        'peak_vram_mib': peak_vram,
        'output_cosine_sim_vs_fp32': cosine_similarity,
    })

    # 현재 정밀도의 측정 결과를 출력한다.
    print(
        f'{precision_name}: '
        f'{throughput:7.1f} 샘플/초 | '
        f'VRAM {peak_vram:5d} MiB | '
        f'FP32 대비 코사인 유사도={cosine_similarity:.4f}'
    )
```

```text
  fp32: 37723.8 samples/s | VRAM   272 MiB | cos-sim vs fp32 = 1.0000
  fp16: 70799.1 samples/s | VRAM   191 MiB | cos-sim vs fp32 = 1.0000
  bf16: 67884.5 samples/s | VRAM   191 MiB | cos-sim vs fp32 = 1.0000
```

## 3단계 --- 배치 크기와 정밀도의 복합 스윕(combined sweep)

배치 크기만 변경하거나 정밀도만 변경하면 부분적인 결과만 얻는다. 실제
운영 구성은 배치 크기와 정밀도의 조합으로 결정된다.

복합 스윕에서는 다음 절차를 수행한다.

1. 배치 크기와 정밀도의 모든 조합을 실행한다.
2. 각 조합의 처리량과 최대 GPU 메모리 사용량을 측정한다.
3. GPU 메모리 예산을 초과한 조합을 제외한다.
4. 남은 조합 중 처리량이 가장 높은 구성을 선택한다.

운영 환경에서는 전체 GPU 메모리를 모두 사용하지 않는다. CUDA 컨텍스트,
메모리 단편화(memory fragmentation), 입력 길이 증가, 일시적 버퍼 할당을
고려해 일정한 안전 여유분을 남겨야 한다.

### GPU 메모리 예산 설정

엔비디아 관리 라이브러리(NVML, NVIDIA Management Library)를 사용해 전체
GPU 메모리를 조회한다. 전체 용량의 85%만 사용 가능한 예산으로 설정하고
나머지 15%는 다음 항목을 위한 안전 여유분으로 남긴다.

- CUDA 컨텍스트와 런타임 오버헤드
- 메모리 할당자 단편화
- 평소보다 긴 입력 또는 큰 요청
- 일시적인 중간 버퍼와 작업 공간

GPU 메모리를 100%에 가깝게 사용하면 입력 형태 변화나 메모리 단편화로
인해 간헐적인 메모리 부족(OOM)이 발생할 수 있다.

```python
import pynvml

# NVML 라이브러리를 초기화한다.
pynvml.nvmlInit()

# 첫 번째 GPU 장치의 핸들을 가져온다.
gpu_handle = pynvml.nvmlDeviceGetHandleByIndex(0)

# GPU의 전체 메모리 용량을 바이트에서 MiB 단위로 변환한다.
total_vram_mib = (
    pynvml.nvmlDeviceGetMemoryInfo(gpu_handle).total
    // (1024 * 1024)
)

# 전체 VRAM의 85%만 사용 가능한 예산으로 설정한다.
# 나머지 15%는 CUDA 컨텍스트, 단편화, 입력 크기 변동을 위한 여유분으로 남긴다.
VRAM_BUDGET_MIB = int(total_vram_mib * 0.85)

# 전체 GPU 메모리와 실제 스윕에 사용할 메모리 예산을 출력한다.
print(
    f'전체 VRAM: {total_vram_mib} MiB | '
    f'사용 예산(85%): {VRAM_BUDGET_MIB} MiB\n'
)
```

```text
Total VRAM: 24564 MiB | Budget (85%): 20879 MiB

```

### 배치 크기 × 수치 정밀도 조합 측정

배치 크기 `1`, `4`, `16`, `32`와 정밀도 `FP32`, `FP16`, `BF16`을 조합해
총 12개 구성을 측정한다.

일부 구성은 GPU 메모리가 작은 환경에서 메모리 부족(OOM)을 일으킬 수
있다. 코드는 `try/except` 구문으로 해당 오류를 처리하고 실행 가능한
구성만 결과 목록에 저장한다.

```python
# 배치 크기와 정밀도 조합별 결과를 저장할 목록을 생성한다.
combined_sweep = []

# 네 가지 배치 크기를 순차적으로 측정한다.
for batch_size in [1, 4, 16, 32]:
    # 각 배치 크기에서 세 가지 수치 정밀도를 측정한다.
    for precision_name, dtype in [
        ('fp32', torch.float32),
        ('fp16', torch.float16),
        ('bf16', torch.bfloat16),
    ]:
        # 이전 측정에서 사용하지 않는 CUDA 캐시를 비운다.
        torch.cuda.empty_cache()

        # 최대 GPU 메모리 통계를 현재 조합에 맞게 초기화한다.
        torch.cuda.reset_peak_memory_stats()

        try:
            # 현재 정밀도로 새로운 모델을 생성하고 추론 모드로 전환한다.
            candidate_model = SmallTransformer().to(
                device=device,
                dtype=dtype,
            ).eval()

            # 현재 배치 크기에 맞는 임의 토큰 입력을 GPU에 생성한다.
            input_ids = torch.randint(
                0,
                VOCAB,
                (batch_size, SEQ_LEN),
                device=device,
            )

            # 자동 미분 그래프 생성을 비활성화한 상태에서 성능을 측정한다.
            with torch.no_grad():
                # 초기 CUDA 실행과 내부 버퍼 할당 비용을 제외하기 위해 워밍업한다.
                for _ in range(WARMUP):
                    _ = candidate_model(input_ids)

                # 워밍업 연산이 모두 완료될 때까지 기다린다.
                torch.cuda.synchronize()

                # 실제 측정 시작 시각을 기록한다.
                start_time = time.perf_counter()

                # 동일한 입력을 반복 실행해 평균 성능을 측정한다.
                for _ in range(ITERS):
                    _ = candidate_model(input_ids)

                # 반복 실행한 CUDA 연산이 모두 끝날 때까지 기다린다.
                torch.cuda.synchronize()

                # 전체 실행 시간을 계산한다.
                elapsed = time.perf_counter() - start_time

            # 현재 조합에서 기록된 최대 GPU 메모리를 MiB 단위로 변환한다.
            peak_vram_mib = (
                torch.cuda.max_memory_allocated()
                // (1024 * 1024)
            )

            # 현재 조합의 초당 샘플 처리량을 계산한다.
            throughput = (
                ITERS * batch_size
                / elapsed
            )

            # 정상 실행된 조합의 결과를 목록에 추가한다.
            combined_sweep.append({
                'batch_size': batch_size,
                'precision': precision_name,
                'throughput_samples_per_s': throughput,
                'peak_vram_mib': peak_vram_mib,
            })

            # 현재 구성의 측정 결과를 출력한다.
            print(
                f'배치={batch_size:3d} '
                f'{precision_name}: '
                f'{throughput:7.1f} 샘플/초 | '
                f'VRAM {peak_vram_mib:5d} MiB'
            )

        except torch.cuda.OutOfMemoryError:
            # 현재 조합이 GPU 메모리를 초과하면 오류를 기록하고 다음 조합으로 진행한다.
            print(
                f'배치={batch_size:3d} '
                f'{precision_name}: '
                f'OOM 발생 — 해당 구성을 건너뛴다.'
            )

            # OOM 이후 남아 있을 수 있는 캐시를 비워 다음 측정에 영향을 주지 않도록 한다.
            torch.cuda.empty_cache()
```

```text
  batch=  1  fp32:  4432.9 samples/s | VRAM   151 MiB
  batch=  1  fp16:  4316.2 samples/s | VRAM   169 MiB
  batch=  1  bf16:  4275.8 samples/s | VRAM   148 MiB
  batch=  4  fp32: 17570.5 samples/s | VRAM   167 MiB
  batch=  4  fp16: 17347.1 samples/s | VRAM   180 MiB
  batch=  4  bf16: 17133.1 samples/s | VRAM   154 MiB
  batch= 16  fp32: 37569.5 samples/s | VRAM   210 MiB
  batch= 16  fp16: 70260.2 samples/s | VRAM   227 MiB
  batch= 16  bf16: 68194.6 samples/s | VRAM   177 MiB
  batch= 32  fp32: 40635.7 samples/s | VRAM   274 MiB
  batch= 32  fp16: 130696.9 samples/s | VRAM   291 MiB
  batch= 32  bf16: 137718.0 samples/s | VRAM   209 MiB
```

### GPU 메모리 예산 내 최적 구성 선택

GPU 메모리 예산을 초과하지 않는 구성만 필터링한 뒤 처리량이 가장 높은
항목을 선택한다.

선택된 구성은 `FP32, 배치 크기 1` 기준 구성과 비교해 전체 성능 향상
배수를 계산한다. 이 값은 배치 처리와 낮은 정밀도를 함께 적용했을 때 얻은
총 효과를 나타낸다.

```python
# GPU 메모리 예산 이하인 구성만 선택한다.
within_budget = [
    entry
    for entry in combined_sweep
    if entry['peak_vram_mib'] <= VRAM_BUDGET_MIB
]

# 예산 내 구성 중 처리량이 가장 높은 항목을 선택한다.
best = max(
    within_budget,
    key=lambda entry: entry['throughput_samples_per_s'],
)

# FP32 정밀도와 배치 크기 1인 기준 구성의 처리량을 찾는다.
baseline_throughput = next(
    entry
    for entry in combined_sweep
    if (
        entry['precision'] == 'fp32'
        and entry['batch_size'] == 1
    )
)['throughput_samples_per_s']

# 최적 구성에 메모리 예산과 기준 대비 성능 향상 배수를 추가한다.
best_config = {
    **best,
    'vram_budget_mib': VRAM_BUDGET_MIB,
    'speedup_vs_fp32_batch1': (
        best['throughput_samples_per_s']
        / baseline_throughput
    ),
}

# 선택된 최적 구성과 주요 지표를 출력한다.
print(
    f"\n★ GPU 메모리 예산 내 최적 구성: "
    f"{best_config['precision']} / "
    f"배치={best_config['batch_size']}"
)
print(
    f"  처리량: "
    f"{best_config['throughput_samples_per_s']:.1f} 샘플/초"
)
print(
    f"  FP32 배치 1 대비: "
    f"{best_config['speedup_vs_fp32_batch1']:.2f}배"
)
print(
    f"  최대 VRAM: "
    f"{best_config['peak_vram_mib']} / "
    f"{VRAM_BUDGET_MIB} MiB"
)
```

```text

★ Best within budget: bf16 @ batch=32
  throughput:  137718.0 samples/s
  vs fp32@1:   31.07x
  peak VRAM:   209 / 20879 MiB budget
```

## 4단계 --- 운영 환경 권장 구성 생성

성능 측정 결과를 운영팀이 바로 사용할 수 있는 형태로 정리한다. 이
실습에서는 다음 세 가지 결과물을 생성한다.

1. **운영 환경 권장 구성(production recommendation)**권장 배치 크기, 권장 정밀도, 메모리 안전 여유분, 선택 이유, 주요
   절충 사항을 기록한다.
2. **GPU 제품별 권장 정밀도(SKU-to-precision mapping)**GPU 아키텍처가 지원하는 텐서 코어 형식에 따라 기본 정밀도를
   정리한다.
3. **정확도 통과 기준(accuracy gate)**
   정밀도 변경 후에도 유지해야 하는 최소 출력 품질을 정의한다. 기준을
   통과하지 못하면 지속적 통합(CI, Continuous Integration) 단계에서
   구성을 거부한다.

### 권장 구성과 선택 근거

단순히 "BF16과 배치 크기 N을 사용한다"라고 기록하는 것만으로는 충분하지
않다. 다음 담당자가 다른 모델이나 서비스 수준 목표에 적용할 수 있도록
선택 이유와 절충 사항을 함께 제공해야 한다.

주요 절충 사항은 처리량, 요청 지연 시간, GPU 메모리 사용량, 수치 안정성,
GPU 아키텍처 지원 여부다.

```python
# 스윕 결과를 운영팀이 사용할 수 있는 권장 구성으로 정리한다.
production_recommendation = {
    # GPU 메모리 예산 내에서 가장 높은 처리량을 보인 배치 크기를 기록한다.
    'preferred_batch': best_config['batch_size'],

    # 최적 구성에서 사용한 수치 정밀도를 기록한다.
    'preferred_precision': best_config['precision'],

    # 입력 증가와 메모리 단편화에 대비해 남겨 둘 VRAM 비율을 기록한다.
    'oom_margin_pct': 15,

    # 기준 구성 대비 성능 향상과 메모리 사용량을 선택 근거로 설명한다.
    'reason': (
        f"({best_config['precision']}, "
        f"배치={best_config['batch_size']}) 구성이 "
        f"FP32 배치 1 대비 "
        f"{best_config['speedup_vs_fp32_batch1']:.2f}배의 처리량을 제공하고, "
        f"최대 VRAM {best_config['peak_vram_mib']} MiB로 "
        f"{VRAM_BUDGET_MIB} MiB 예산을 만족한다."
    ),

    # 운영 환경에서 함께 검토해야 하는 주요 절충 사항을 목록으로 기록한다.
    'tradeoffs': [
        (
            '배치 크기를 늘리면 처리량은 증가할 수 있지만 요청 지연 시간도 증가한다. '
            'p99 지연 시간 SLO가 100ms라면 순전파 시간이 충분한 여유를 두고 '
            '100ms 아래에 유지되는 배치 크기를 선택해야 한다.'
        ),
        (
            'BF16은 FP32와 표현 범위가 같아 학습 시 손실 스케일링이 일반적으로 필요하지 않다. '
            'FP16은 일부 구형 GPU에서 효율적일 수 있지만 수치 안정성을 별도로 검증해야 한다.'
        ),
        (
            '15%의 VRAM 여유분은 보수적인 기본값이다. '
            '입력 크기와 메모리 사용 패턴이 안정적으로 관리되면 10% 수준을 검토할 수 있다.'
        ),
        (
            'GPU 드라이버, CUDA, PyTorch, 커널 구현이 바뀌면 최적 지점도 달라질 수 있으므로 '
            '주기적으로 동일한 스윕을 다시 수행해야 한다.'
        ),
    ],
}

# 운영 환경 권장 구성의 기본 항목을 출력한다.
print('=== 운영 환경 권장 구성 ===')

for key, value in production_recommendation.items():
    # 절충 사항은 별도 목록 형태로 출력하므로 이 반복에서는 제외한다.
    if key != 'tradeoffs':
        print(f'  {key:>22s}: {value}')

# 절충 사항 목록을 사람이 읽기 쉬운 형태로 출력한다.
print('  tradeoffs:')

for tradeoff in production_recommendation['tradeoffs']:
    print(f'    - {tradeoff}')
```

```text
=== Production recommendation ===
         preferred_batch: 32
     preferred_precision: bf16
          oom_margin_pct: 15
                  reason: (bf16, batch=32) delivered 31.07x throughput vs fp32@1 while staying 209/20879 MiB below budget.
  tradeoffs:
    - Larger batches improve throughput but increase per-request latency linearly; if SLO is p99<100ms, cap batch size at whatever fits under 80ms forward-pass time.
    - bf16 is same-range as fp32 — no loss scaling needed. fp16 can be slightly faster on older GPUs (T4, V100) but requires AMP for training stability.
    - OOM margin of 15% is conservative for dev; production can drop to 10% if memory-usage patterns are well-understood.
    - Re-run this sweep quarterly — driver + framework updates silently change optimal points.
```

### GPU 제품별 권장 정밀도

GPU 아키텍처에 따라 효율적으로 지원하는 정밀도가 다르다.

- Turing(T4)과 Volta 세대(V200)는 FP16 중심으로 사용한다.
- Ampere(A100)와 Ada 세대(L4, L40)는 BF16을 기본 선택으로 사용할 수
  있다.
- Hopper 세대(H100, H200)는 FP8 텐서 코어를 지원한다.
- Blackwell 세대(B200, B300)는 더 낮은 정밀도 형식을 지원하지만, 모델
  품질 검증과 소프트웨어 지원 수준을 함께 확인해야 한다.

이 표는 일반적인 시작점이며 실제 운영 구성은 모델별 벤치마크와 정확도
검증을 통해 결정해야 한다.

```python
# GPU 제품별로 일반적으로 검토할 수 있는 기본 정밀도를 정의한다.
# 실제 적용 전에는 모델별 처리량과 정확도 검증이 필요하다.
sku_to_preferred_precision = {
    # Turing 세대는 BF16을 기본 하드웨어 형식으로 지원하지 않으므로 FP16을 사용한다.
    'T4 (Turing)': 'fp16',

    # Volta 세대의 텐서 코어는 FP16 중심으로 설계되어 있다.
    'V100 (Volta)': 'fp16',

    # Ampere 세대부터 BF16 텐서 코어 연산을 기본적으로 지원한다.
    'A10 (Ampere)': 'bf16',
    'A100 (Ampere)': 'bf16',

    # Ada Lovelace 세대에서도 BF16을 안정적인 기본 선택으로 사용할 수 있다.
    'L4 (Ada Lovelace)': 'bf16',
    'L40 (Ada)': 'bf16',

    # Hopper 세대는 FP8 텐서 코어를 지원하므로 보정 후 FP8을 검토할 수 있다.
    'H100 (Hopper)': 'fp8',
    'H200 (Hopper)': 'fp8',

    # Blackwell 세대는 더 낮은 정밀도도 지원하지만 안정적인 기본값으로 FP8을 기록한다.
    'GB200 (Blackwell)': 'fp8',
}

# GPU 제품별 권장 정밀도를 표 형태로 출력한다.
print('\n=== GPU 제품별 권장 정밀도 ===')

for sku, precision in sku_to_preferred_precision.items():
    print(f'  {sku:<22s} → {precision}')
```

```text

=== SKU → preferred precision ===
  T4 (Turing)            → fp16
  V100 (Volta)           → fp16
  A10 (Ampere)           → bf16
  A100 (Ampere)          → bf16
  L4 (Ada Lovelace)      → bf16
  L40 (Ada)              → bf16
  H100 (Hopper)          → fp8
  H200 (Hopper)          → fp8
  GB200 (Blackwell)      → fp8
```

### 정확도 통과 기준(accuracy gate)

단일 입력의 코사인 유사도만으로 정밀도 변경의 안전성을 충분히 입증할
수는 없다. 운영 환경에서는 별도의 평가 데이터셋(eval set)을 사용해 여러
샘플의 품질 지표를 측정해야 한다.

예제에서는 1,024개 평가 샘플의 평균 코사인 유사도가 `0.995` 이상인지
확인한다. 임베딩(embedding), 재순위화(reranking), 검색 모델처럼 미세한
출력 차이에 민감한 작업은 `0.999` 이상의 더 엄격한 기준이 필요할 수
있다.

```python
# 지속적 통합(CI)에서 사용할 최소 출력 품질 기준을 정의한다.
accuracy_gate = {
    # FP32 기준 출력과 비교할 품질 지표 이름을 지정한다.
    'metric': 'output_cosine_sim_vs_fp32',

    # 허용 가능한 최소 평균 코사인 유사도를 지정한다.
    'threshold': 0.995,

    # 평가 데이터와 집계 방법을 명시해 측정 절차를 재현할 수 있게 한다.
    'measurement_method': (
        '1,024개 평가 샘플에서 FP32 기준 출력 대비 '
        '배치 평균 코사인 유사도를 계산한다.'
    ),

    # 기준 미달 시 배포 구성이 병합되지 않도록 CI 동작을 정의한다.
    'enforcement': (
        '각 정밀도 후보에 대해 CI 작업을 실행하고, '
        '측정값이 임계값보다 낮으면 병합을 차단한다.'
    ),
}

# 정확도 통과 기준의 세부 항목을 출력한다.
print('\n=== 정확도 통과 기준 ===')

for key, value in accuracy_gate.items():
    print(f'  {key:>20s}: {value}')
```

```text

=== Accuracy gate ===
                metric: output_cosine_sim_vs_fp32
             threshold: 0.995
    measurement_method: batch-average cosine similarity over a 1024-sample eval set vs. fp32 reference
           enforcement: CI job runs this per precision candidate; blocks the merge if below threshold
```

---

## 실습 결과

이 실습에서는 다음 구성 요소를 구현했다.

- 처리량 증가가 둔화되는 무릎 지점을 찾는 **배치 크기 스윕(batch size
  sweep)**
- 처리량과 FP32 기준 출력 유사도를 함께 측정하는 **수치 정밀도
  스윕(precision sweep)**
- GPU 메모리 예산 내에서 최적 조합을 선택하는 **복합 스윕(combined
  sweep)**
- GPU 제품별 권장 정밀도와 정확도 통과 기준을 포함하는 **운영 환경
  권장 구성**

## 다른 구성 요소와의 연계

- 추론 서빙(inference serving)에서는 `preferred_batch` 값을 Triton
  Inference Server 또는 vLLM의 최대 배치 크기 설정에 활용할 수 있다.
- 비용 분석(cost analysis)에서는 동일한 처리량을 더 적은 GPU 시간으로
  처리할 수 있으므로 처리량 향상을 비용 절감 효과로 환산할 수 있다.
- 컨테이너와 배포 파이프라인에서는 새로운 모델 빌드마다
  `accuracy_gate`를 실행해 정밀도 변경으로 인한 품질 회귀를 방지할 수
  있다.
