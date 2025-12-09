# Ollama HTTP & HTTPS Dual-Port Deployment (via Docker Compose)

이 저장소는 **Ollama 모델 서버를 HTTP(포트 11434)** 로 제공하면서,
동시에 **HTTPS(포트 11435) 역프록시(Nginx)** 를 통해 안전한 TLS API 접근을 제공하는 구조입니다.

외부 클라이언트는 HTTPS 포트를 사용하여 데이터를 암호화하며, 내부/로컬 개발 환경에서는 HTTP로 빠르게 접근할 수 있습니다.
| 포트        | 프로토콜  | 설명                     |
| --------- | ----- | ---------------------- |
| **11434** | HTTP  | Ollama 모델 서버 직접 접근     |
| **11435** | HTTPS | 외부 클라이언트용 안전한 접근 (TLS) |

이 구성은 다음 장점을 제공합니다:

* 내부망/로컬에서는 빠른 HTTP 사용
* 외부(VPN/인터넷)에서는 안전한 HTTPS 강제
* Ollama는 HTTPS를 직접 지원하지 않아도 Nginx에서 TLS 처리 가능
* 운영/보안 요구사항 충족

---

**핵심 동작:**

```
HTTPS (11435) → Nginx → HTTP (ollama:11434)
```

---

## 📂 Directory Structure

아래 구조로 프로젝트 파일이 구성됩니다:

```
ollama-stack/
  docker-compose.yml
  nginx/
    nginx.conf
    certs/
      server.crt
      server.key
```

## 🔐 Self-Signed 인증서 생성

HTTPS 테스트용 인증서를 생성하려면:

```bash
cd ~/ollama-stack

openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 \
  -keyout nginx/certs/server.key \
  -out nginx/certs/server.crt \
  -subj "/C=KR/ST=Seoul/L=Seoul/O=GBCC/OU=AI/CN={IP_ADDR}"
```

> 실제 운용 환경에서는 회사 CA 인증서 또는 정식 TLS 인증서를 사용하는 것을 권장합니다.

---

## 🚀 서비스 실행

```bash
cd ~/ollama-stack
docker compose up -d
```

컨테이너 확인:

```bash
docker ps
```

정상적으로 실행되면:

* `ollama`
* `ollama-nginx`

두 컨테이너가 떠 있어야 합니다.

---

## 🧪 API 테스트

### ✔ 1. HTTP 직접 호출 (11434)

```bash
curl http://{IP_ADDR}:11434/api/tags
```

---

### ✔ 2. HTTPS 호출 (11435 — Self-signed 인증서이므로 -k 필요)

```bash
curl -k https://{IP_ADDR}:11435/api/tags
```

---

### ✔ 3. 모델 요청 테스트

```bash
curl -k https://{IP_ADDR}:11435/api/generate \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3:latest",
    "prompt": "안녕?",
    "stream": false
  }'
```
