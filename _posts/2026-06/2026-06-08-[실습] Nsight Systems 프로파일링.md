---
title: "[실습] Nsight Systems 프로파일링"
date: 2026-06-08
tags: [트랜스포머, Transformer, Inference, LLM, Optimization, Serving, vllm, gpu, k8s, kubernetes, triton, tgi, ollama, tensorRT-LLM]
typora-root-url: ../
toc: true
categories: [NVIDIA]
---



# 실습 6 --- Nsight Systems Profiling: GPU 성능의 40%를 낭비하는 병목 찾기

성능 지표를 알아보는 "재현 가능한 학습(Reproducible Training)"편에서는
**무언가가 느리다**는 사실을 알려준다. 프로파일링(profiling)은 **정확히
어느 시점과 구간이 느린지**를 알려준다. **NVIDIA Nsight Systems**의
명령줄 도구인 `nsys`는 CUDA 환경에서 널리 사용하는 타임라인 수준
프로파일러(timeline-level profiler)다. 이 도구는 CUDA API 호출, 커널
실행(kernel launch), 메모리 복사(memory copy), NVTX 범위를 기록하고,
이를 하나의 통합 타임라인(unified timeline)에 표시한다.

이 실습에서는 의도적으로 비효율적으로 작성된 학습 루프(training loop)를
사용해 **프로파일링 후 최적화(profile-then-optimize)** 흐름을 학습한다.
CPU 데이터로더(dataloader)를 기다리느라 GPU가 유휴 상태가 되는 병목을
찾고, 한 줄 수준의 수정으로 처리량(throughput)이 얼마나 향상되는지
측정한다.

### `time.perf_counter()`만 사용하지 않고 Nsight Systems를 사용하는 이유

일반적인 시간 측정 함수는 전체 단계(step)의 경과 시간(wall-clock time)만
측정한다. Nsight Systems는 다음 정보를 제공한다.

-   **CUDA 커널 실행 시간(CUDA kernel execution time)** --- 각 커널의
    실행 시간을 집계해 어떤 커널이 GPU 시간을 가장 많이 사용하는지
    보여준다.
-   **실행 간격(launch gap)** --- 한 커널이 끝난 뒤 다음 커널이 시작될
    때까지의 시간이다. CPU 오버헤드 또는 동기화 장벽(sync barrier)을
    확인할 수 있다.
-   **메모리 전송(memory transfer)** --- 호스트-디바이스(H2D) 또는
    디바이스-호스트(D2H) 전송이 연산과 중첩되는지 확인한다.
-   **CPU 스레드 활동(CPU thread activity)** --- Python 작업 스레드가
    전역 인터프리터 잠금(GIL, Global Interpreter Lock) 때문에 대기하는지
    분석한다.
-   **NVTX 범위(NVTX range)** --- `torch.cuda.nvtx.range_push/pop`으로
    코드에 논리 구간을 표시해 순전파(forward), 역전파(backward),
    옵티마이저(optimizer) 단계를 구분한다.

이 실습에서는 다음 두 가지 사용 방식을 다룬다.

-   `nsys profile --stats=true ...` --- `.nsys-rep` 보고서를 생성하고
    요약 통계를 표준 출력(stdout)에 표시한다. 대부분의 자동화된 분석에서
    사용하는 방식이다.
-   Nsight Systems 그래픽 사용자 인터페이스(Nsight Systems GUI) ---
    데스크톱 애플리케이션에서 `.nsys-rep` 파일을 열어 대화형 타임라인을
    분석한다. 현재 실습 환경에는 화면이 없으므로 직접 실행하지 않고 참고
    자료에서 설명한다.

### 참고 자료

-   **[Nsight Systems User
    Guide](https://docs.nvidia.com/nsight-systems/UserGuide/index.html)**
    --- 전체 기능과 명령줄 옵션을 설명한다.
-   **[Karpathy --- Recipe for Training Neural
    Networks](http://karpathy.github.io/2019/04/25/recipe/)** ---
    엔사이트 시스템즈 전용 문서는 아니지만, 최적화 전에 프로파일링해야
    하는 이유를 설명한다.
-   **[NVTX ranges in
    PyTorch](https://pytorch.org/docs/stable/cuda.html#torch.cuda.nvtx)**
    --- PyTorch 코드에 NVTX 범위를 추가하는 방법을 설명한다.
-   **[GPU Performance
    Background](https://docs.nvidia.com/deeplearning/performance/dl-performance-gpu-background/index.html)**
    --- 프로파일링 결과를 이해하기 위한 GPU 성능 기본 개념을 설명한다.

------------------------------------------------------------------------

## 1단계 --- NVTX 범위로 학습 작업 계측하기

NVTX가 없으면 엔사이트 시스템즈 프로파일은 수많은 `cudaLaunchKernel`
이벤트로만 표시된다. NVTX 범위를 추가하면 `data_load`, `forward`,
`backward`, `optimizer`와 같은 논리적 실행 단계를 타임라인의 색상 막대로
구분할 수 있다. 실무 CUDA 프로젝트에서는 이러한 계측(instrumentation)을
일반적으로 코드에 포함한다.

이 단계에서는 NVTX 주석(annotation)이 포함된 소규모 학습 스크립트를
파일로 저장하고, `nsys` 명령이 실행 경로(PATH)에 등록되어 있는지
확인한다.

``` python
import subprocess
import shutil
import os

# 실행 경로(PATH)에서 nsys 명령의 위치를 찾는다.
nsys_path = shutil.which("nsys")

if nsys_path is None:
    print("nsys를 찾을 수 없다.")
    print("엔사이트 시스템즈(Nsight Systems)가 설치되지 않았거나 PATH에 등록되지 않았다.")
    print("다음 명령으로 설치 경로를 확인할 수 있다.")
    print("  which nsys")
    print("  ls /usr/local/cuda/bin/nsys")
    print("  ls /usr/local/cuda-*/bin/nsys")
else:
    print(f"nsys 경로: {nsys_path}")

    # 설치된 nsys 버전을 확인한다.
    nsys_result = subprocess.run(
        [nsys_path, "--version"],
        capture_output=True,
        text=True,
    )

    print("표준 출력:", nsys_result.stdout.strip())
    print("표준 오류:", nsys_result.stderr.strip())
```

``` text
nsys를 찾을 수 없습니다.
Nsight Systems가 설치되어 있지 않거나 PATH에 등록되지 않았습니다.
확인 명령:
  which nsys
  ls /usr/local/cuda/bin/nsys
  ls /usr/local/cuda-*/bin/nsys
```

``` python
import subprocess
import os

# nsys 명령을 실행할 수 있는지 확인한다.
nsys_result = subprocess.run(
    ['nsys', '--version'],
    capture_output=True,
    text=True,
)
nsys_version = nsys_result.stdout.strip()
print(f'nsys 버전: {nsys_version}')

# 프로파일링할 작업은 workspace/scripts/bad_training.py에 있다.
# 이 스크립트에는 다음과 같은 비효율이 의도적으로 포함되어 있다.
# - 반복마다 메인 스레드를 차단하는 CPU 전처리
# - 페이지 고정 메모리(pinned memory)를 사용하지 않는 느린 H2D 전송
# - 모든 작업을 동기식으로 처리하는 일반 for 루프 데이터 로더
workload_script_path = '/home/labuser/workspace/scripts/bad_training.py'

assert os.path.exists(workload_script_path), (
    f'작업 스크립트를 찾을 수 없다: {workload_script_path}'
)

print(
    f'작업 스크립트: {workload_script_path} '
    f'({os.path.getsize(workload_script_path)}바이트)'
)
```

``` text
[0;31m---------------------------------------------------------------------------[0m
[0;31mFileNotFoundError[0m                         Traceback (most recent call last)
Cell [0;32mIn[2], line 4[0m
[1;32m      1[0m [38;5;28;01mimport[39;00m [38;5;21;01msubprocess[39;00m[38;5;241m,[39m [38;5;21;01mos[39;00m
[1;32m      3[0m [38;5;66;03m# Confirm nsys is available[39;00m
[0;32m----> 4[0m nsys_result [38;5;241m=[39m [43msubprocess[49m[38;5;241;43m.[39;49m[43mrun[49m[43m([49m[43m[[49m[38;5;124;43m'[39;49m[38;5;124;43mnsys[39;49m[38;5;124;43m'[39;49m[43m,[49m[43m [49m[38;5;124;43m'[39;49m[38;5;124;43m--version[39;49m[38;5;124;43m'[39;49m[43m][49m[43m,[49m[43m [49m[43mcapture_output[49m[38;5;241;43m=[39;49m[38;5;28;43;01mTrue[39;49;00m[43m,[49m[43m [49m[43mtext[49m[38;5;241;43m=[39;49m[38;5;28;43;01mTrue[39;49;00m[43m)[49m
[1;32m      5[0m nsys_version [38;5;241m=[39m nsys_result[38;5;241m.[39mstdout[38;5;241m.[39mstrip()
[1;32m      6[0m [38;5;28mprint[39m([38;5;124mf[39m[38;5;124m'[39m[38;5;124mnsys version: [39m[38;5;132;01m{[39;00mnsys_version[38;5;132;01m}[39;00m[38;5;124m'[39m)

File [0;32m/usr/lib/python3.11/subprocess.py:548[0m, in [0;36mrun[0;34m(input, capture_output, timeout, check, *popenargs, **kwargs)[0m
[1;32m    545[0m     kwargs[[38;5;124m'[39m[38;5;124mstdout[39m[38;5;124m'[39m] [38;5;241m=[39m PIPE
[1;32m    546[0m     kwargs[[38;5;124m'[39m[38;5;124mstderr[39m[38;5;124m'[39m] [38;5;241m=[39m PIPE
[0;32m--> 548[0m [38;5;28;01mwith[39;00m [43mPopen[49m[43m([49m[38;5;241;43m*[39;49m[43mpopenargs[49m[43m,[49m[43m [49m[38;5;241;43m*[39;49m[38;5;241;43m*[39;49m[43mkwargs[49m[43m)[49m [38;5;28;01mas[39;00m process:
[1;32m    549[0m     [38;5;28;01mtry[39;00m:
[1;32m    550[0m         stdout, stderr [38;5;241m=[39m process[38;5;241m.[39mcommunicate([38;5;28minput[39m, timeout[38;5;241m=[39mtimeout)

File [0;32m/usr/lib/python3.11/subprocess.py:1026[0m, in [0;36mPopen.__init__[0;34m(self, args, bufsize, executable, stdin, stdout, stderr, preexec_fn, close_fds, shell, cwd, env, universal_newlines, startupinfo, creationflags, restore_signals, start_new_session, pass_fds, user, group, extra_groups, encoding, errors, text, umask, pipesize, process_group)[0m
[1;32m   1022[0m         [38;5;28;01mif[39;00m [38;5;28mself[39m[38;5;241m.[39mtext_mode:
[1;32m   1023[0m             [38;5;28mself[39m[38;5;241m.[39mstderr [38;5;241m=[39m io[38;5;241m.[39mTextIOWrapper([38;5;28mself[39m[38;5;241m.[39mstderr,
[1;32m   1024[0m                     encoding[38;5;241m=[39mencoding, errors[38;5;241m=[39merrors)
[0;32m-> 1026[0m     [38;5;28;43mself[39;49m[38;5;241;43m.[39;49m[43m_execute_child[49m[43m([49m[43margs[49m[43m,[49m[43m [49m[43mexecutable[49m[43m,[49m[43m [49m[43mpreexec_fn[49m[43m,[49m[43m [49m[43mclose_fds[49m[43m,[49m
[1;32m   1027[0m [43m                        [49m[43mpass_fds[49m[43m,[49m[43m [49m[43mcwd[49m[43m,[49m[43m [49m[43menv[49m[43m,[49m
[1;32m   1028[0m [43m                        [49m[43mstartupinfo[49m[43m,[49m[43m [49m[43mcreationflags[49m[43m,[49m[43m [49m[43mshell[49m[43m,[49m
[1;32m   1029[0m [43m                        [49m[43mp2cread[49m[43m,[49m[43m [49m[43mp2cwrite[49m[43m,[49m
[1;32m   1030[0m [43m                        [49m[43mc2pread[49m[43m,[49m[43m [49m[43mc2pwrite[49m[43m,[49m
[1;32m   1031[0m [43m                        [49m[43merrread[49m[43m,[49m[43m [49m[43merrwrite[49m[43m,[49m
[1;32m   1032[0m [43m                        [49m[43mrestore_signals[49m[43m,[49m
[1;32m   1033[0m [43m                        [49m[43mgid[49m[43m,[49m[43m [49m[43mgids[49m[43m,[49m[43m [49m[43muid[49m[43m,[49m[43m [49m[43mumask[49m[43m,[49m
[1;32m   1034[0m [43m                        [49m[43mstart_new_session[49m[43m,[49m[43m [49m[43mprocess_group[49m[43m)[49m
[1;32m   1035[0m [38;5;28;01mexcept[39;00m:
[1;32m   1036[0m     [38;5;66;03m# Cleanup if the child failed starting.[39;00m
[1;32m   1037[0m     [38;5;28;01mfor[39;00m f [38;5;129;01min[39;00m [38;5;28mfilter[39m([38;5;28;01mNone[39;00m, ([38;5;28mself[39m[38;5;241m.[39mstdin, [38;5;28mself[39m[38;5;241m.[39mstdout, [38;5;28mself[39m[38;5;241m.[39mstderr)):

File [0;32m/usr/lib/python3.11/subprocess.py:1955[0m, in [0;36mPopen._execute_child[0;34m(self, args, executable, preexec_fn, close_fds, pass_fds, cwd, env, startupinfo, creationflags, shell, p2cread, p2cwrite, c2pread, c2pwrite, errread, errwrite, restore_signals, gid, gids, uid, umask, start_new_session, process_group)[0m
[1;32m   1953[0m     err_msg [38;5;241m=[39m os[38;5;241m.[39mstrerror(errno_num)
[1;32m   1954[0m [38;5;28;01mif[39;00m err_filename [38;5;129;01mis[39;00m [38;5;129;01mnot[39;00m [38;5;28;01mNone[39;00m:
[0;32m-> 1955[0m     [38;5;28;01mraise[39;00m child_exception_type(errno_num, err_msg, err_filename)
[1;32m   1956[0m [38;5;28;01melse[39;00m:
[1;32m   1957[0m     [38;5;28;01mraise[39;00m child_exception_type(errno_num, err_msg)

[0;31mFileNotFoundError[0m: [Errno 2] No such file or directory: 'nsys'
```

``` python
from IPython.display import Code

# 프로파일링 대상 학습 스크립트의 소스 코드를 표시한다.
Code(
    filename='/home/labuser/workspace/scripts/bad_training.py',
    language='python',
)
```

## 2단계 --- `nsys profile`로 Nsight Systems 프로파일 수집하기

대표적인 명령은 다음과 같다.

``` bash
nsys profile \
    --trace=cuda,nvtx,osrt \
    --sample=cpu \
    --stats=true \
    --output=/tmp/profile.nsys-rep \
    --force-overwrite=true \
    python your_script.py
```

각 옵션의 기능은 다음과 같다.

-   `--trace=cuda,nvtx,osrt` --- CUDA API 호출, NVTX 범위, 운영체제
    런타임(OS runtime)의 스레드 생성과 뮤텍스 경합(mutex contention)을
    수집한다. cuDNN 또는 cuBLAS를 분석하려면 `,cudnn,cublas`를 추가한다.
-   `--sample=cpu` --- CPU 호출 스택(call stack)을 샘플링해 GPU 활동과
    함께 Python 스택 정보를 표시한다.
-   `--stats=true` --- 프로파일링 완료 후 텍스트 요약 통계를 생성한다.
    이 실습에서는 해당 출력을 파싱한다.
-   `--output=...` --- 바이너리 형식의 `.nsys-rep` 파일을 저장할 경로를
    지정한다.
-   `--force-overwrite=true` --- 기존 파일을 확인 없이 덮어쓴다.
    자동화된 스크립트 실행에 필요하다.

생성된 보고서는 데스크톱의 엔사이트 시스템즈 그래픽 사용자
인터페이스(Nsight Systems GUI)에서 열어 대화형 타임라인으로 분석할 수
있다. 지속적 통합(CI)이나 자동 분석 환경에서는 텍스트 요약만으로도 많은
문제를 진단할 수 있다.

``` python
profile_report_path = '/tmp/profile.nsys-rep'

# 이전 실행에서 생성된 보고서와 SQLite 파일을 삭제한다.
for path in [profile_report_path, profile_report_path + '.sqlite']:
    if os.path.exists(path):
        os.remove(path)

# .nsys-rep 파일은 임시 디렉터리(/tmp)에 저장한다.
# 실제 프로파일링 대상 스크립트만 작업 공간(workspace)에 존재한다.
cmd = [
    'nsys', 'profile',
    '--trace=cuda,nvtx,osrt',
    '--sample=cpu',
    '--stats=true',
    f'--output={profile_report_path}',
    '--force-overwrite=true',
    'python', 'scripts/bad_training.py',
]

print('실행 명령: ' + ' '.join(cmd))
print()

# 엔사이트 시스템즈 프로파일링 명령을 실행한다.
result = subprocess.run(
    cmd,
    capture_output=True,
    text=True,
    cwd='/home/labuser/workspace',
)

# --stats=true를 사용하면 요약 통계가 표준 출력 또는 오류에 기록된다.
stats_output = result.stdout + result.stderr

print('=== nsys 출력: 앞부분 4,000자 ===')
print(stats_output[:4000])

# CUDA API 요약을 실습 검증에 사용한다.
cuda_api_summary = stats_output

print(
    f'\n보고서 크기: '
    f'{os.path.getsize(profile_report_path)/1024/1024:.1f}MB'
)
```

## 3단계 --- NVTX 범위 보고서를 파싱해 병목 찾기

실제 AI 학습 작업 중 상당수는 CPU 제약(CPU-bound)의 영향을 받는다.
이러한 경우에는 **NVTX 범위 요약(NVTX range summary)**이 적합하다.
1단계에서 표시한 `data_load`, `forward`, `backward`, `optimizer` 구간별
실행 시간을 집계하며, 전체 시간이 가장 긴 구간이 우선적인 분석 대상이다.

별도의 커널 수준 보고서인 `--report=cuda_gpu_kern_sum`은 어떤 GPU 커널이
가장 많은 시간을 사용하는지 보여준다. 이는 **GPU 제약(GPU-bound)**
작업을 분석할 때 적합하다. NVTX 범위 분석과 커널 수준 분석은 서로
보완한다.

> **컨테이너 환경 참고:** 일부 컨테이너 환경에서는 CUDA 커널 추적을 위해
> `CAP_SYS_ADMIN` 권한 또는 `/proc/sys/kernel/perf_event_paranoid` 설정
> 변경이 필요하다. *"does not contain CUDA kernel data"* 메시지가
> 표시되더라도 NVTX 범위는 사용할 수 있으므로, CPU 측 병목을 진단하는
> 데는 문제가 없다.

### 🐛 일반적인 실수: 전체 시간이 아니라 평균 시간만 확인하는 경우

`nsys stats`는 범위별 평균 시간과 전체 시간을 모두 보여준다. **병목
분석에서는 전체 시간(total time)을 기준으로 정렬해야 한다.** 호출당
5밀리초(ms)가 걸리는 범위가 1,000번 실행되면 총 5초가 소요된다. 반면
500밀리초가 걸리는 범위가 한 번만 실행되면 총 500밀리초다. 실제 최적화
우선순위는 첫 번째 범위다.

``` python
# CUDA 커널 추적 권한이 없어도 사용할 수 있는 NVTX 범위 요약을 요청한다.
nvtx_result = subprocess.run(
    [
        'nsys', 'stats',
        '--force-export=true',
        '--report=nvtx_sum',
        '--format=csv',
        profile_report_path,
    ],
    capture_output=True,
    text=True,
)

csv_output = nvtx_result.stdout

print('=== NVTX 범위 요약: 앞부분 3,000자 ===')
print(csv_output[:3000])
```

``` text
=== NVTX range summary (first 3000 chars) ===
Generating SQLite file /tmp/profile.sqlite from /tmp/profile.nsys-rep
Processing [/tmp/profile.sqlite] with [/opt/nvidia/nsight-compute/2024.1.1/host/target-linux-x64/reports/nvtx_sum.py]... 
Time (%),Total Time (ns),Instances,Avg (ns),Med (ns),Min (ns),Max (ns),StdDev (ns),Style,Range
27.7,1631355508,1,1631355508.0,1631355508.0,1631355508,1631355508,0.0,PushPop,step_0
19.3,1135904229,50,22718084.6,22240097.0,22026050,28786412,1417632.9,PushPop,data_load
11.8,694057698,50,13881154.0,1392122.5,1182862,620707110,87570041.1,PushPop,backward
9.4,556649880,50,11132997.6,796834.0,712771,514425376,72629095.4,PushPop,forward
8.8,519399446,50,10387988.9,850835.5,733613,471776961,66582345.8,PushPop,optimizer
0.7,40617226,50,812344.5,761949.0,679299,1675859,157588.8,PushPop,h2d_transfer
0.6,33841972,1,33841972.0,33841972.0,33841972,33841972,0.0,PushPop,step_1
0.6,33315513,1,33315513.0,33315513.0,33315513,33315513,0.0,PushPop,step_2
0.5,30233228,1,30233228.0,30233228.0,30233228,30233228,0.0,PushPop,step_8
0.5,29237542,1,29237542.0,29237542.0,29237542,29237542,0.0,PushPop,step_3
0.5,29021646,1,29021646.0,29021646.0,29021646,29021646,0.0,PushPop,step_7
0.5,28113356,1,28113356.0,28113356.0,28113356,28113356,0.0,PushPop,step_21
0.5,27951987,1,27951987.0,27951987.0,27951987,27951987,0.0,PushPop,step_35
0.5,27811552,1,27811552.0,27811552.0,27811552,27811552,0.0,PushPop,step_4
0.5,27272080,1,27272080.0,27272080.0,27272080,27272080,0.0,PushPop,step_32
0.5,27172466,1,27172466.0,27172466.0,27172466,27172466,0.0,PushPop,step_6
0.5,26999536,1,26999536.0,26999536.0,26999536,26999536,0.0,PushPop,step_31
0.5,26950962,1,26950962.0,26950962.0,26950962,26950962,0.0,PushPop,step_5
0.5,26925268,1,26925268.0,26925268.0,26925268,26925268,0.0,PushPop,step_36
0.5,26917479,1,26917479.0,26917479.0,26917479,26917479,0.0,PushPop,step_11
0.5,26824314,1,26824314.0,26824314.0,26824314,26824314,0.0,PushPop,step_24
0.5,26748055,1,26748055.0,26748055.0,26748055,26748055,0.0,PushPop,step_29
0.5,26663766,1,26663766.0,26663766.0,26663766,26663766,0.0,PushPop,step_12
0.5,26660171,1,26660171.0,26660171.0,26660171,26660171,0.0,PushPop,step_43
0.5,26575885,1,26575885.0,26575885.0,26575885,26575885,0.0,PushPop,step_10
0.5,26536851,1,26536851.0,26536851.0,26536851,26536851,0.0,PushPop,step_34
0.4,26488730,1,26488730.0,26488730.0,26488730,26488730,0.0,PushPop,step_14
0.4,26473351,1,26473351.0,26473351.0,26473351,26473351,0.0,PushPop,step_39
0.4,26444862,1,26444862.0,26444862.0,26444862,26444862,0.0,PushPop,step_26
0.4,26385586,1,26385586.0,26385586.0,26385586,26385586,0.0,PushPop,step_42
0.4,26316501,1,26316501.0,26316501.0,26316501,26316501,0.0,PushPop,step_33
0.4,26263542,1,26263542.0,26263542.0,26263542,26263542,0.0,PushPop,step_48
0.4,26220985,1,26220985.0,26220985.0,26220985,26220985,0.0,PushPop,step_23
0.4,26214122,1,26214122.0,26214122.0,26214122,26214122,0.0,PushPop,step_25
0.4,26185669,1,26185669.0,26185669.0,26185669,26185669,0.0,PushPop,step_20
0.4,26150390,1,26150390.0,26150390.0,26150390,
```

### `nsys stats` 출력 파싱 --- CSV 구간 추출

`nsys stats`는 실제 CSV 데이터 앞에 진행 상태와 로그 메시지를 추가한다.
다음 보조 함수는 첫 번째 `Time (%)` 헤더 이전의 내용을 제거해 나머지
셀에서 `csv.reader`로 데이터를 처리할 수 있게 한다.

운영 환경의 프로파일링 파이프라인에서는 이러한 파서를 한 번 작성한 뒤
`ops/` 디렉터리 등에 저장해 재사용하는 것이 일반적이다.

``` python
import csv
import io

# nsys stats의 로그 메시지를 제거하고 실제 CSV 구간만 추출한다.
def extract_csv(text: str) -> str:
    lines = text.splitlines()

    for index, line in enumerate(lines):
        if line.startswith('Time (%)'):
            return '\n'.join(lines[index:])

    return ''

csv_only = extract_csv(csv_output)
```

### 전체 시간을 기준으로 NVTX 범위 순위 지정

CSV를 파싱해 각 NVTX 범위의 이름, 전체 실행 시간, 실행 횟수를 추출한다.
전체 단계를 감싸는 `step_*` 범위는 구조상 가장 긴 시간이 기록되므로 분석
대상에서 제외한다.

상위 10개 범위를 출력했을 때 첫 번째 항목이 가장 중요한 병목이다. 이후
항목부터는 일반적으로 최적화 효과가 점차 작아지므로, 가장 큰 병목부터
수정해야 한다.

``` python
# 실습 검증 코드와의 호환성을 위해 변수 이름은 top_kernels를 유지한다.
# 실제로는 NVTX 범위별 실행 시간을 저장한다.
top_kernels = []

reader = csv.reader(io.StringIO(csv_only))
header = None

for row in reader:
    # 비어 있는 행은 건너뛴다.
    if not row or not row[0].strip():
        continue

    # 첫 번째 유효 행을 CSV 헤더로 사용한다.
    if header is None:
        header = [column.strip() for column in row]
        continue

    try:
        row_dict = dict(zip(header, row))

        # 전체 실행 시간을 나노초(ns) 단위로 변환한다.
        total_ns = int(
            float(
                row_dict.get('Total Time (ns)', '0')
                .replace(',', '')
                .strip() or '0'
            )
        )

        # NVTX 범위 이름과 실행 횟수를 추출한다.
        name = (row_dict.get('Range') or row[-1]).strip()
        instances = int(
            float(
                row_dict.get('Instances', '0')
                .replace(',', '')
                .strip() or '0'
            )
        )

        # 전체 단계를 감싸는 래퍼(wrapper) 범위는 제외한다.
        if name.startswith('step_'):
            continue

        if total_ns > 0:
            top_kernels.append({
                'name': name,
                'total_time_ns': total_ns,
                'instances': instances,
            })

    except (ValueError, KeyError):
        # 형식이 예상과 다른 행은 건너뛴다.
        continue

# 전체 실행 시간이 긴 순서로 상위 10개 범위를 선택한다.
top_kernels.sort(key=lambda item: -item['total_time_ns'])
top_kernels = top_kernels[:10]

print(
    f'전체 실행 시간 기준 상위 NVTX 범위 {len(top_kernels)}개 '
    f'(step_* 래퍼 제외)'
)

total_time = sum(item['total_time_ns'] for item in top_kernels)

for rank, item in enumerate(top_kernels, 1):
    percentage = (
        100 * item['total_time_ns'] / total_time
        if total_time
        else 0
    )
    average_ms = (
        item['total_time_ns']
        / max(item['instances'], 1)
        / 1e6
    )

    print(
        f"  #{rank} {item['name']:>15}: "
        f"{item['total_time_ns']/1e6:8.1f}ms 전체 | "
        f"{item['instances']:4}회 | "
        f"{average_ms:6.2f}ms/호출 | "
        f"{percentage:5.1f}%"
    )

print()
print('첫 번째 NVTX 범위가 가장 큰 병목이다. 이 구간부터 수정한다.')
```

## 4단계 --- 프로파일링 결과에 따라 수정하고 성능 향상 측정하기

프로파일링 결과는 CPU 데이터 로더가 약 20밀리초 동안 메인 스레드를
차단하는 반면, GPU 커널은 약 2밀리초만 실행되어 **GPU가 대부분의 시간
동안 유휴 상태**임을 보여준다. 대표적인 해결 방법은 다음 두 가지다.

1.  **데이터 로딩과 연산 중첩(overlap data loading with compute)** ---
    Python의 `DataLoader(num_workers=N)`가 백그라운드 작업자(worker)에서
    다음 배치를 미리 가져온다. GPU가 N번째 단계를 완료할 때 N+1번째
    데이터가 준비되도록 한다.
2.  **데이터를 GPU에 유지(keep data on GPU)** --- 데이터를 한 번 GPU로
    전송한 뒤 재사용한다. 데이터가 GPU 메모리에 들어갈 만큼 작거나
    캐시할 수 있을 때만 사용할 수 있다.

이 실습에서는 구현이 간단한 두 번째 방법을 적용하고 단계 실행 시간이
얼마나 개선되는지 측정한다. 대규모 운영 학습에서는 일반적으로 첫 번째
방법을 사용하며, 최대 처리량이 필요할 때는 DALI(Data Loading Library)를
적용할 수 있다.

``` python
import time
import torch
import torch.nn as nn

# 비효율적인 데이터 로딩이 포함된 수정 전 단계 시간을 측정한다.
device = torch.device('cuda')

model = nn.Sequential(
    nn.Linear(1024, 4096),
    nn.ReLU(),
    nn.Linear(4096, 4096),
    nn.ReLU(),
    nn.Linear(4096, 1024),
).to(device)

optimizer = torch.optim.AdamW(model.parameters(), lr=1e-3)
loss_fn = nn.MSELoss()

def bad_step():
    # 느린 CPU 전처리를 20밀리초 대기로 모의한다.
    time.sleep(0.02)

    # CPU 메모리에 입력과 목표 텐서를 생성한다.
    x = torch.randn(256, 1024)
    y = torch.randn(256, 1024)

    # 매 단계마다 데이터를 CPU에서 GPU로 전송한다.
    x = x.to(device)
    y = y.to(device)

    # 순전파, 손실 계산, 역전파, 파라미터 갱신을 수행한다.
    out = model(x)
    loss = loss_fn(out, y)
    optimizer.zero_grad(set_to_none=True)
    loss.backward()
    optimizer.step()

# 최초 CUDA 초기화 비용을 제외하기 위해 워밍업한다.
for _ in range(3):
    bad_step()

torch.cuda.synchronize()

# 30회 실행의 평균 단계 시간을 측정한다.
N = 30
start_time = time.perf_counter()

for _ in range(N):
    bad_step()

torch.cuda.synchronize()

before_avg_step_ms = (
    (time.perf_counter() - start_time) / N * 1000
)

print(
    f'수정 전: 느린 CPU 데이터 로더 — '
    f'{before_avg_step_ms:.2f}ms/단계'
)
```

``` text
BEFORE (slow CPU dataloader): 25.55 ms/step
```

``` python
# 프로파일링 결과에 따라 데이터를 GPU에 미리 할당한다.
x_pool = torch.randn(256, 1024, device=device)
y_pool = torch.randn(256, 1024, device=device)

def good_step():
    # 이미 GPU에 있는 데이터를 사용해 H2D 전송을 제거한다.
    out = model(x_pool)
    loss = loss_fn(out, y_pool)
    optimizer.zero_grad(set_to_none=True)
    loss.backward()
    optimizer.step()

# 수정 후 코드도 동일하게 워밍업한다.
for _ in range(3):
    good_step()

torch.cuda.synchronize()

# 수정 후 평균 단계 시간을 측정한다.
start_time = time.perf_counter()

for _ in range(N):
    good_step()

torch.cuda.synchronize()

after_avg_step_ms = (
    (time.perf_counter() - start_time) / N * 1000
)

print(
    f'수정 후: GPU에 데이터 사전 할당 — '
    f'{after_avg_step_ms:.2f}ms/단계'
)
print(
    f'성능 향상: '
    f'{before_avg_step_ms/after_avg_step_ms:.2f}배'
)
```

``` text
AFTER (data pre-allocated on GPU): 9.13 ms/step
Speedup: 2.80x
```

------------------------------------------------------------------------

## 실습 결과

다음과 같은 전체 **프로파일링 후 수정(profile-then-fix)** 흐름을
구성했다.

NVTX로 계측한 작업 → `nsys profile` 수집 → `nsys stats` 파싱 → 상위 병목
분석 → 대상 최적화 → 수정 전후 성능 비교

실제 GPU 최적화 프로젝트에서도 이 과정을 반복적으로 수행한다.

------------------------------------------------------------------------

## 참고 자료

-   **[Nsight Systems User
    Guide](https://docs.nvidia.com/nsight-systems/UserGuide/index.html)**
    --- 전체 옵션과 시각화 기능을 설명한다. 전문가 시스템(Expert
    Systems) 검사는 일반적인 성능 안티패턴(anti-pattern)을 자동으로
    감지한다.
-   **[Nsight
    Compute](https://docs.nvidia.com/nsight-compute/NsightCompute/index.html)**
    --- 커널 수준 프로파일러(kernel-level profiler)다. 엔사이트
    시스템즈가 어떤 구간이 느린지 보여준다면, 엔사이트 컴퓨트는 특정
    커널이 느린 원인을 메모리 병목(memory-bound), 워프 분기(warp
    divergence), 점유율(occupancy) 관점에서 분석한다.
-   **[NVTX in
    PyTorch](https://pytorch.org/docs/stable/cuda.html#torch.cuda.nvtx)**
    --- 적은 코드로 PyTorch 모델에 NVTX 범위를 추가하는 방법을 설명한다.
-   **[PyTorch
    Profiler](https://pytorch.org/tutorials/recipes/recipes/profiler_recipe.html)**
    --- 실습 I-4에서 다룬다. Python 수준의 병목 구간을 쉽게 분석할 수
    있으며 엔사이트 시스템즈와 상호 보완적이다.
-   **[Making Deep Learning Go
    Brrrr](https://horace.io/brrr_intro.html)** --- GPU 성능을 분석하는
    사고방식을 설명한다.
