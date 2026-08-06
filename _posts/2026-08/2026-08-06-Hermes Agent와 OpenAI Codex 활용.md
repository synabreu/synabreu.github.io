---
title: "Hermes Agent와 OpenAI Codex 활용"
date: 2026-08-06
tags: [Hermes, Agent, Codex, Harness, OpenAI, 오픈AI,  Ubuntu, Docker, Docker Compose, OpenAI Codex CLI] 
typora-root-url: ../
toc: true
categories: [openai]
---
앞의 블로그에서는 간단히 오픈AI Codex 와 Hermes Agent 구조에 대해 알아 보았다. 이번 글은 직접 Hermes 설치하고 완성까지 예를 보여주겠다. 또한, 서버 인프라(Ubuntu+Docker+HTTPS)를 세우고, Codex 콤비로 Hermes를 자동 설치한다.

# 1. 완성 후 구성

사용자 브라우저
    │ HTTPS 443
    ▼
Caddy 컨테이너
    │ HTTP, 호스트 내부에서만 접근
    ▼
Hermes Dashboard 127.0.0.1:9119
    │
    ├─ Hermes Gateway
    └─ 영구 데이터 /opt/hermes/data

서버 SSH 셸
    └─ OpenAI Codex CLI

외부에 공개하는 포트는 SSH 22, HTTP 80, HTTPS 443뿐이다. Hermes의 9119 대시보드 포트와 8642 API 포트는 127.0.0.1에만 바인딩한다.

# 2. 준비사항

* Ubuntu 24.04 LTS 서버 1대
* 권장 사양: 2 vCPU, 메모리 4 GB 이상, 저장공간 30 GB 이상
* sudo 권한이 있는 일반 사용자
* 서버 공인 IPv4 주소
* 본인이 관리하는 도메인 또는 하위 도메인. 예: hermes.example.com
* Hermes에서 사용할 모델 제공자 계정 또는 API 키
* Codex에 로그인할 ChatGPT 계정 또는 지원되는 다른 인증 수단

DNS 관리 화면에서 다음 레코드를 먼저 만든다.

| 유형 | 이름                    | 값                  |
| ---- | ----------------------- | ------------------- |
| A    | `<span>hermes</span>` | 서버 공인 IPv4 주소 |

hermes.example.com 대신 실제 도메인을 사용한다. DNS 전파는 다음 명령으로 확인한다.

`dig +short hermes.example.com`

출력된 주소가 서버 공인 IP와 같아야 한다.

# 3. Ubuntu 초기 설정

서버에 SSH로 접속한다. 브라우저 기반 VPS 콘솔은 붙여넣기 과정에서 특수문자가 변형될 수 있으므로 일반 SSH 접속을 권장한다.

`ssh ubuntu@SERVER_PUBLIC_IP`

운영체제를 업데이트하고 기본 도구를 설치한다.

```Shell
sudo apt update
sudo apt upgrade -y
sudo apt install -y ca-certificates curl gnupg git jq openssl ufw unattended-upgrades
```

시간대를 서울로 맞춘다.

```Bash
sudo timedatectl set-timezone Asia/Seoul
timedatectl
```

방화벽을 설정하고, 현재 SSH 접속이 끊기지 않도록 SSH를 먼저 허용한다.

```Bash
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status verbose
```

클라우드 사업자의 보안 그룹이나 네트워크 방화벽에서도 TCP 22, 80, 443을 허용해야 한다. 포트 번호 9119와 8642는 외부에 열지 않는다.

# 4. Docker Engine과 Compose 설치

Ubuntu 저장소의 오래된 패키지 대신 Docker 공식 APT 저장소를 사용한다.

```Bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" |
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

현재 사용자를 docker 그룹에 추가한다.

```Bash
sudo usermod -aG docker "$USER"
newgrp docker
```

설치를 검증한다.

```Bash
docker version
docker compose version
docker run --rm hello-world
```

주의: docker 그룹 사용자는 사실상 서버의 관리자 권한을 가질 수 있다. 신뢰할 수 있는 운영 계정만 추가한다.

# 5. OpenAI Codex CLI 설치

OpenAI 공식 standalone 설치 스크립트를 사용한다.

```Bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

새 셸을 열거나 설치 프로그램이 안내한 PATH 설정을 적용한 뒤 확인한다.

```Bash
codex --version
```

프로젝트 디렉터리에서 Codex를 처음 실행한다.

```Bash
mkdir -p ~/projects/hermes-stack
cd ~/projects/hermes-stack
codex
```

첫 실행 화면에서 Sign in with ChatGPT 또는 사용 가능한 다른 인증 방식을 선택한다. 원격 서버에서 브라우저를 직접 열 수 없으면 Codex가 표시하는 URL과 코드를 로컬 브라우저에서 처리한다. 로그인 후 Codex 안에서 다음 명령을 확인한다.

```Bash
/status
/permissions
/model
```

종료는 Ctrl+C를 누른다.

# 6. Hermes 영구 데이터 초기화

운영 파일은 /opt/hermes-stack, Hermes 영구 상태는 /opt/hermes/data에 둔다.

```Bash
sudo mkdir -p /opt/hermes-stack/caddy-data /opt/hermes-stack/caddy-config
sudo mkdir -p /opt/hermes/data
sudo chown -R "$USER":"$USER" /opt/hermes-stack /opt/hermes
cd /opt/hermes-stack
```

Hermes 공식 컨테이너의 대화형 설정 마법사를 한 번 실행한다.

```Bash
docker run -it --rm
  -v /opt/hermes/data:/opt/data
  nousresearch/hermes-agent setup
```

화면에서 다음을 설정한다.

* 사용할 모델 제공자를 고르고 제공자 API 키 또는 Nous Portal 로그인을 설정한다.
* 기본 모델을 고르고 필요한 도구와 메시징 채널을 선택한다.
* 설정을 저장하고 종료한다.

Nous Portal을 사용할 경우 다음 방식도 가능하다.

```Bash
docker run -it --rm
  -v /opt/hermes/data:/opt/data
  nousresearch/hermes-agent setup --portal
```

설정 파일이 생성되었는지 확인한다. 비밀값은 화면에 출력하지 않는다.

```Bash
find /opt/hermes/data -maxdepth 1 -type f -printf '%f\n'
```

일반적으로 .env, config.yaml, SOUL.md 등이 생성된다. /opt/hermes/data는 API 키, 대화 기록, 메모리, 스킬 및 로그가 보존되는 핵심 디렉터리다.

# 7. SecretValue 파일 만들기

Hermes 대시보드 인증용 비밀번호와 세션 비밀값을 생성한다.

```Bash
cd /opt/hermes-stack
umask 077
openssl rand -base64 24
openssl rand -hex 32
```

첫 번째 출력은 대시보드 로그인 비밀번호, 두 번째 출력은 세션 비밀값으로 사용한다. 다음 파일을 연다.

```Bash
nano /opt/hermes-stack/.env
```

아래 내용을 넣고 값을 교체한다.

```Bash
HERMES_DASHBOARD_USER=admin
HERMES_DASHBOARD_PASSWORD=REPLACE_WITH_RANDOM_PASSWORD
HERMES_DASHBOARD_SECRET=REPLACE_WITH_64_HEX_CHARACTERS
```

권한을 제한한다.

```Bash
chmod 600 /opt/hermes-stack/.env
```

`.env` 파일은 Git에 커밋하거나 다른 사람에게 전송하지 않는다.

# 8. Docker Compose 작성

`/opt/hermes-stack/compose.yaml` 을 만든다.

```yaml
services:
  hermes:
    image: nousresearch/hermes-agent:latest
    container_name: hermes
    restart: unless-stopped
    command: ["gateway", "run"]
    volumes:
      - /opt/hermes/data:/opt/data
    ports:
      - "127.0.0.1:8642:8642"
      - "127.0.0.1:9119:9119"
    environment:
      HERMES_DASHBOARD: "1"
      HERMES_DASHBOARD_HOST: "0.0.0.0"
      HERMES_DASHBOARD_PORT: "9119"
      HERMES_DASHBOARD_BASIC_AUTH_USERNAME: "${HERMES_DASHBOARD_USER}"
      HERMES_DASHBOARD_BASIC_AUTH_PASSWORD: "${HERMES_DASHBOARD_PASSWORD}"
      HERMES_DASHBOARD_BASIC_AUTH_SECRET: "${HERMES_DASHBOARD_SECRET}"
    healthcheck:
      test: ["CMD-SHELL", "curl -fsS http://127.0.0.1:8642/health || exit 1"]
      interval: 30s
      timeout: 5s
      retries: 5
      start_period: 30s

  caddy:
    image: caddy:2
    container_name: caddy
    restart: unless-stopped
    network_mode: host
    depends_on:
      - hermes
    volumes:
      - /opt/hermes-stack/Caddyfile:/etc/caddy/Caddyfile:ro
      - /opt/hermes-stack/caddy-data:/data
      - /opt/hermes-stack/caddy-config:/config
```

8642는 선택적인 Hermes API 및 상태 확인 포트다. 호스트의 루프백에만 연결되므로 인터넷에서는 직접 접근할 수 없다.

# 9. Caddy HTTPS 설정

`/opt/hermes-stack/Caddyfile` 을 만들고 실제 도메인으로 바꾼다.

```
hermes.example.com {
    encode zstd gzip

    header {
        Strict-Transport-Security "max-age=31536000; includeSubDomains"
        X-Content-Type-Options "nosniff"
        Referrer-Policy "strict-origin-when-cross-origin"
        -Server
    }

    reverse_proxy 127.0.0.1:9119
}
```

Caddy는 DNS가 서버를 가리키고 외부에서 80과 443에 접근할 수 있으면 TLS 인증서를 자동으로 발급하고 갱신한다.

Compose 설정을 문법 검사한다.

```Bash
cd /opt/hermes-stack
docker compose config >/dev/null
echo "Compose configuration: OK"
```

# 10. Hermes와 HTTPS 서버 실행

도커 이미지를 내려받고 서비스를 실행한다.

```Bash
cd /opt/hermes-stack
docker compose pull
docker compose up -d
```

상태와 로그를 확인한다.

```Bash
docker compose ps
docker compose logs --tail=100 hermes
docker compose logs --tail=100 caddy
```

hermes가 healthy가 되지 않으면 실제 이미지의 상태 확인 경로가 변경되었을 수 있다. 이 경우 다음 명령으로 응답을 확인하고 healthcheck를 조정한다.

```Bash
curl -i http://127.0.0.1:8642/health
docker inspect --format '{{json .State.Health}}' hermes | jq
```

# 11. 설치 검증

### 11.1 로컬 포트 노출 확인

```Bash
sudo ss -lntp | grep -E ':(80|443|8642|9119)\b'
```

`8642`와 `9119`는 `127.0.0.1`에만 보여야 한다. `80`과 `443`은 Caddy가 수신한다.

### 11.2 HTTPS 확인

```Bash
curl -I https://hermes.example.com
```

브라우저에서 https://hermes.example.com을 열고 `Hermes` 로그인 화면이 표시되는지 확인한다. `.env`에 지정한 사용자 이름과 비밀번호로 로그인한다.

### 11.3 Hermes 대화 확인

실행 중인 데이터 디렉터리를 사용해 일회성 CLI 세션을 연다.

```Bash
docker run -it --rm
  -v /opt/hermes/data:/opt/data
  nousresearch/hermes-agent
```

다음과 같이 간단한 요청을 보내 응답을 확인한다.

`현재 날짜를 알려주고, 사용 가능한 도구 이름을 요약해줘.`

### 11.4 Codex 확인

```Bash
cd ~/projects/hermes-stack
codex
```

Codex에 다음처럼 요청한다.

`/opt/hermes-stack`의 Compose 구성을 읽기 전용으로 검토하고,
외부 노출 포트, 비밀값 처리, 재시작 정책과 HTTPS 구성을 점검해줘.
파일은 수정하지 말고 개선점만 알려줘.

Codex가 `/opt/hermes-stack`을 읽을 권한이 없다면 해당 디렉터리를 현재 사용자가 읽을 수 있는지 확인한다. 비밀값이 들어 있는 .env의 내용은 프롬프트나 로그에 붙여 넣지 않는다.

# 12. Codex를 이용한 반복 운영

설정 변경 전 백업과 Git 체크포인트를 만든다. 비밀값과 영구 데이터는 Git에서 제외한다.

```Bash
cd /opt/hermes-stack
cat > .gitignore <<'EOF'
.env
caddy-data/
caddy-config/
backups/
EOF

git init
git add compose.yaml Caddyfile .gitignore
git commit -m "Add Hermes Docker HTTPS stack"
```

Codex에 한 번에 너무 넓은 권한을 주지 말고, 다음처럼 단계별로 요청한다.

1) compose.yaml과 Caddyfile을 읽고 현재 구조를 설명해줘.
2) 변경 계획과 예상 영향만 제안해줘. 아직 수정하지 마.
3) 승인한 항목만 수정하고 docker compose config로 검증해줘.
4) 변경 diff를 보여주고 롤백 방법을 알려줘.

Codex가 Docker 명령을 실행하도록 허용하기 전에는 변경 범위와 컨테이너 재시작 여부를 확인한다.

# 13. 업데이트

### 13-1. Codex 업데이트

공식 설치 명령을 다시 실행한다.

```Bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
codex --version
```

### 13-2. Hermes와 Caddy 업데이트

먼저 데이터를 백업한다.

```Bash
sudo tar -C /opt/hermes -czf "/opt/hermes-stack/hermes-backup-$(date +%F-%H%M).tar.gz" data
```

그다음 새 이미지를 내려받아 재생성한다.

```Bash
cd /opt/hermes-stack
docker compose pull
docker compose up -d --force-recreate
docker compose ps
docker compose logs --tail=100
```

Docker판 Hermes는 컨테이너 내부에서 hermes update를 실행하는 방식이 아니라 새 이미지를 pull한 뒤 컨테이너를 재생성하는 방식으로 업데이트한다. 영구 상태는 /opt/hermes/data에 남는다.

# 14. 백업과 복구

다음과 같이 백업한다.

```Bash
sudo tar -C /opt/hermes -czf "$HOME/hermes-data-$(date +%F).tar.gz" data
```

복구 시에는 먼저 서비스를 중지한 뒤 기존 데이터와 섞이지 않게 별도 디렉터리에서 백업 내용을 검사한다.

```Bash
cd /opt/hermes-stack
docker compose down
mkdir -p "$HOME/hermes-restore-check"
tar -xzf "$HOME/hermes-data-YYYY-MM-DD.tar.gz" -C "$HOME/hermes-restore-check"
find "$HOME/hermes-restore-check" -maxdepth 2 -type f | head
```

내용을 확인한 뒤 실제 복구 대상을 명확히 정하고 교체한다. API 키가 포함된 백업 파일은 암호화된 저장소에 보관한다.

# 15. 문제 해결

### 15-1. HTTPS 인증서가 발급되지 않는다

```Bash
dig +short hermes.example.com
sudo ufw status
docker compose logs --tail=200 caddy
```

* DNS A 레코드가 현재 서버 IP를 가리키는지 확인한다.
* 클라우드 보안 그룹에서 80, 443이 열렸는지 확인한다.
* 다른 웹 서버가 80, 443을 점유하지 않았는지 sudo ss -lntp로 확인한다.

### 15-2. Hermes 대시보드가 시작되지 않는다

```Bash
docker compose logs --tail=200 hermes
docker compose exec hermes env | grep '^HERMES_DASHBOARD_' | sed 's/=.*$/=<set></set>/'
```

비루프백 주소에 바인딩된 최신 Hermes 대시보드는 인증 제공자가 없으면 안전을 위해 시작을 거부한다. 사용자 이름, 비밀번호, 세션 비밀값 세 변수가 모두 전달됐는지 확인한다.

### 15-3. Hermes 모델이 응답하지 않는다

```Bash
docker run -it --rm
  -v /opt/hermes/data:/opt/data
  nousresearch/hermes-agent model
```

모델 제공자, 모델 이름, API 키, 계정 한도와 네트워크 연결을 확인한다. 기본 대화가 성공하기 전에는 메시징 게이트웨이, cron, 추가 스킬을 붙이지 않는 편이 문제 범위를 줄이기 쉽다.

### 15-4. 컨테이너가 반복 재시작한다

```Bash
docker compose ps
docker inspect hermes --format '{{.State.ExitCode}} {{.State.Error}}'
docker compose logs --tail=300 hermes
```

설정 파일 권한과 `/opt/hermes/data`의 소유권을 확인한다.

```Bash
ls -ld /opt/hermes /opt/hermes/data
find /opt/hermes/data -maxdepth 1 -printf '%M %u:%g %f\n'
```

# 16. 운영 보안 체크리스트

* SSH는 키 기반 로그인을 사용하고 가능하면 비밀번호 로그인을 끈다.
* 클라우드 방화벽과 UFW에서 `22, 80, 443`만 공개한다.
* Hermes `9119`와 `8642`를 `0.0.0.0`에 직접 공개하지 않는다.
* 대시보드 인증 비밀번호는 충분히 긴 난수로 설정한다.
* `.env`, `/opt/hermes/data`, 백업 파일을 Git에 올리지 않는다.
* 운영 서버에서는 Hermes의 도구 반복 중단 설정을 활성화한다.
* Docker 이미지 업데이트 전 영구 데이터를 백업한다.
* Codex가 제안한 명령과 diff를 검토한 후 실행을 승인한다.
* 여러 Hermes gateway 컨테이너가 같은 `/opt/hermes/data`를 동시에 쓰지 않게 한다.

무인 실행 환경에서는 `/opt/hermes/data/config.yaml`에 다음 정책을 검토한다. 기존 항목이 있으면 중복으로 추가하지 말고 YAML 구조를 유지한다.

```YAML
tool_loop_guardrails:
  hard_stop_enabled: true
  hard_stop_after:
    exact_failure: 5
    idempotent_no_progress: 5
```

변경 후 재시작한다.

```Bash
cd /opt/hermes-stack
docker compose restart hermes
```

# 17. 완료 기준

아래 항목이 모두 충족되면 구축이 완료된 것이다.

* `docker compose ps`에서 Hermes와 Caddy가 실행 중이다.
* `https://hermes.example.com`에 유효한 TLS 인증서로 접속된다.
* Hermes 로그인 화면이 나타나고 인증 후 대시보드에 진입한다.
* Hermes CLI에서 모델 응답을 받는다.
* `codex --version`이 정상 출력되고 Codex 로그인이 완료된다.
* 외부 공개 포트는 `22, 80, 443`뿐이다.
* `/opt/hermes/data` 백업을 만들 수 있다.

# 18. 공식 참고 문서

* OpenAI Codex CLI: https://learn.chatgpt.com/docs/codex/cli
* Hermes Agent Quickstart: https://hermes-agent.nousresearch.com/docs/getting-started/quickstart
* Hermes Agent Docker: https://github.com/NousResearch/hermes-agent/blob/main/website/docs/user-guide/docker.md
* Hermes Agent Configuration: https://hermes-agent.nousresearch.com/docs/user-guide/configuration
* Docker Engine Ubuntu 설치: https://docs.docker.com/engine/install/ubuntu/
* Caddy 자동 HTTPS: https://caddyserver.com/docs/automatic-https

# 20. 참고

Hermes Agent와 Codex CLI는 서로 다른 제품이다. 이 구성에서는 Hermes가 Docker 기반 상시 에이전트와 웹 대시보드 역할을 하고, Codex CLI는 서버의 배포 파일 검토, 생성, 수정 및 검증을 돕는 운영 도구 역할을 한다. Codex 인증 정보를 Hermes 컨테이너와 자동 공유하지 않는다.
