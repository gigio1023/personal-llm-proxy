# AGENTS.md

이 문서는 이 저장소에서 작업하는 **AI agent / 자동화 도구**를 위한 작업 가이드입니다.

## 0) 이 프로젝트는 무엇인가?
`personal-llm-proxy`는 로컬에서 다음을 **한 번에** 띄우는 것을 목표로 합니다.

- **Langfuse (self-host)**: LLM observability / tracing UI
- **LiteLLM Proxy**: OpenAI-compatible proxy + `langfuse_otel` 콜백을 통해 Langfuse로 trace 전송

핵심은 **LiteLLM → (OTLP HTTP/protobuf) → Langfuse** 흐름이 로컬에서 바로 재현되는 것입니다.

---

## 1) 저장소 구조(현재)

- `docker-compose.yml`
  - Langfuse full stack (web/worker + postgres/redis/clickhouse/minio)
  - + `litellm-proxy` 서비스 포함 (같은 compose 네트워크에서 Langfuse로 전송)
- `docker-compose.yml.litellm`
  - LiteLLM 단독 실행용 compose 파일 (Langfuse는 외부/별도 실행 전제)
- `litellm_config.yaml`
  - LiteLLM Proxy 설정
  - `litellm_settings.callbacks: [langfuse_otel]` 활성화
- `langfuse-litellm-integrationn.md`
  - LiteLLM ↔ Langfuse 연동 문서(레퍼런스)
- `.env` (gitignored)
  - 로컬 전용 secrets/환경변수
- `.env.example`
  - `.env` 템플릿 (커밋 가능, 비밀 값 포함 금지)

---

## 2) 로컬 실행(기본 플로우)

### 2.1 사전 요구사항
- Docker 설치
- Docker Compose
  - 보통 `docker compose`를 사용하지만, 환경에 따라 `docker-compose`만 동작할 수 있음

### 2.2 환경변수 준비
1) `.env.example` → `.env` 복사

```bash
cp .env.example .env
```

2) 최소 설정
- `OPENAI_API_KEY`
- `LITELLM_MASTER_KEY` (로컬이라도 기본값 사용 지양)
- Langfuse 프로젝트 생성 후:
  - `LANGFUSE_PUBLIC_KEY`
  - `LANGFUSE_SECRET_KEY`

> 주의: `.env`는 gitignored 이어야 하며, AI agent는 **절대 .env를 커밋하거나 로그로 노출**하면 안 됩니다.

### 2.3 실행

Langfuse + LiteLLM 전체 스택:

```bash
# (권장) docker compose
# docker compose up

# (대안) docker-compose
docker-compose up
```

접속:
- Langfuse UI: http://localhost:3000
- LiteLLM Proxy: http://localhost:4000

---

## 3) Langfuse ↔ LiteLLM 연동 규칙(중요)

### 3.1 OTEL endpoint 규칙
LiteLLM의 `langfuse_otel` 통합은 `LANGFUSE_OTEL_HOST`에 **base URL만** 넣습니다.
- 예) `http://langfuse-web:3000` 또는 `http://localhost:3000`

LiteLLM이 내부적으로 `/api/public/otel`을 append 합니다.

### 3.2 Langfuse OTLP 프로토콜
Langfuse OTEL ingest는 **gRPC를 지원하지 않습니다**. (OTLP HTTP/protobuf 사용)

### 3.3 네트워크 기준값
- 같은 compose 내부: `LANGFUSE_OTEL_HOST=http://langfuse-web:3000`
- 호스트에서 접근: `LANGFUSE_OTEL_HOST=http://localhost:3000`

---

## 4) LiteLLM 설정 수정 가이드

### 4.1 모델 추가
`litellm_config.yaml`의 `model_list`에 모델을 추가합니다.

예시:
- `model_name`: 클라이언트가 호출할 이름
- `litellm_params.model`: 실제 provider/model 식별자
- provider key는 가능한 env var에서 주입

### 4.2 인증
`LITELLM_MASTER_KEY`가 설정되면 요청 시 보통 아래 헤더를 사용합니다.

```http
Authorization: Bearer <LITELLM_MASTER_KEY>
```

---

## 5) docker-compose 변경 규칙

AI agent는 compose 변경 시 반드시:
1) **포트 바인딩 정책** 유지: 외부 노출 최소화(가능하면 `127.0.0.1:` 바인딩)
2) **secrets 기본값을 무분별하게 추가하지 않기**
   - 로컬 편의 default는 허용하되, README에 반드시 경고/설명
3) Compose validation 수행
   - `docker compose config` 또는 `docker-compose -f <file> config`

---

## 6) 보안/운영 주의사항

- `.env` / credentials / API keys는 절대 커밋 금지
- 이 repo의 `docker-compose.yml`에는 로컬 개발 편의 default 값이 존재합니다.
  - 외부에 노출되는 환경에서 그대로 사용하면 위험합니다.
  - 실제 배포 시에는 `# CHANGEME` 항목을 모두 교체해야 합니다.

---

## 7) Troubleshooting

### 7.1 LiteLLM 트레이스가 Langfuse에 안 보임
- `litellm_config.yaml`에 `callbacks: [langfuse_otel]`가 있는지 확인
- `LANGFUSE_PUBLIC_KEY`, `LANGFUSE_SECRET_KEY`가 Langfuse 프로젝트 키인지 확인
- `LANGFUSE_OTEL_HOST`가 **scheme 포함(http/https)** 인지 확인
  - scheme이 없으면 LiteLLM이 https로 가정할 수 있음

### 7.2 docker compose 명령이 동작하지 않음
- 환경에 따라 `docker compose` 플러그인이 없을 수 있음
- 이 경우 `docker-compose` 사용

---

## 8) AI agent 작업 원칙(필수)

- 사용자 요청이 **명시적 구현**인지 확인 후 작업 시작
- 변경은 작고 명확하게 (버그 수정 중 리팩터링 금지)
- 검색은 내부 코드/문서 → 공식 문서/레퍼런스 순서로 수행
- 새로운 파일/문서 추가는 사용자 요청이 있을 때만
