# Baro EC2 Deployment

프론트엔드는 Vercel에 배포된 상태를 전제로 하고, EC2에서는 백엔드 API, Baro-AI, PostgreSQL만 Docker Compose로 실행한다.

## 구조

```text
사용자 브라우저
  -> Vercel Frontend
  -> EC2 Backend API :8000
      -> Baro-AI :8000 (Docker internal)
      -> PostgreSQL :5432 (Docker internal)
```

## EC2 디렉터리 구조

`docker-compose.prod.yml` 기준으로 아래 경로가 필요하다.

```text
.
├── Baro-AI/
├── baro-backend/
├── docker-compose.prod.yml
└── .env.prod
```

AI와 서버가 별도 레포라면 EC2에서 다음처럼 폴더명을 맞춘다.

```bash
mkdir baro
cd baro
git clone https://github.com/capston-design-baro/Baro-AI.git Baro-AI
git clone https://github.com/capston-design-baro/Baro-Server.git baro-backend
```

## 환경 변수

```bash
cp .env.prod.example .env.prod
```

`.env.prod`에 실제 운영 값을 입력한다.

```env
POSTGRES_USER=baro
POSTGRES_PASSWORD=change-this-long-random-password
POSTGRES_DB=baro_db

SECRET_KEY=change-this-secret-key-min-32-characters
ENCRYPTION_KEY=change-this-fernet-key

OPENAI_API_KEY=sk-your-openai-api-key
OPENAI_MODEL=gpt-5-mini
RAG_DB_URL=https://your-presigned-rag-db-url/chroma.sqlite3

CORS_ORIGINS=https://baro-front-five.vercel.app
BACKEND_PORT=8000
```

## 실행

```bash
docker compose --env-file .env.prod -f docker-compose.prod.yml up -d --build
```

상태 확인:

```bash
docker compose --env-file .env.prod -f docker-compose.prod.yml ps
```

로그 확인:

```bash
docker compose --env-file .env.prod -f docker-compose.prod.yml logs -f baro-backend
docker compose --env-file .env.prod -f docker-compose.prod.yml logs -f baro-ai
```

백엔드 헬스 체크:

```text
http://<EC2_PUBLIC_IP>:8000/health
```

## Vercel 설정

Vercel 프론트엔드 프로젝트의 환경 변수에 EC2 백엔드 API 주소를 설정한다.

```env
VITE_API_BASE_URL=http://<EC2_PUBLIC_IP>:8000/api
```

환경 변수 수정 후 Vercel에서 재배포해야 반영된다.

