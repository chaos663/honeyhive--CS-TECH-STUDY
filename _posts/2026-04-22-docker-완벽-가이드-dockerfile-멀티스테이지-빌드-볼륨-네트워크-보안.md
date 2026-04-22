---
layout: post
title: "Docker 완벽 가이드 — Dockerfile, 멀티스테이지 빌드, 볼륨, 네트워크, 보안"
date: 2026-04-22 16:39:06 +0900
excerpt: "VM vs Container: / VM: 하이퍼바이저가 하드웨어 에뮬레이션 → 수 GB, 수 분 부팅 / Container: 호스트 커널 공유, 프로세스 격리(네임스페이스+cgroups) → 수 MB, 수 초 실행 / Docker 핵심 객체: / Image: 불변 읽기 전용 레이어 스택 (실행 가능한 파일 시스템)"
tags: [OS, Network, Tech, cs-study]
image: "/assets/images/thumbnails/2026-04-22-docker-완벽-가이드-dockerfile-멀티스테이지-빌드-볼륨-네트워크-보안.svg"
series: "HoneyByte"
---
> "내 PC에서는 되는데요." 이 말을 없애기 위해 Docker가 존재한다. 환경을 코드로 정의하고, 어디서든 동일하게 실행한다. 컨테이너의 원리부터 프로덕션 보안까지 — 한 번에 정리한다.

## 핵심 요약 (TL;DR)

**VM vs Container:**
- VM: 하이퍼바이저가 하드웨어 에뮬레이션 → 수 GB, 수 분 부팅
- Container: 호스트 커널 공유, 프로세스 격리(네임스페이스+cgroups) → 수 MB, 수 초 실행

**Docker 핵심 객체:**
- **Image:** 불변 읽기 전용 레이어 스택 (실행 가능한 파일 시스템)
- **Container:** 이미지를 실행한 인스턴스 (읽기 쓰기 레이어 추가)
- **Volume:** 컨테이너 외부 데이터 영속 저장소
- **Network:** 컨테이너 간 통신 추상화

---

## Docker 아키텍처

```mermaid
graph TD
    CLI["Docker CLI\n(docker build, run, push)"]
    Daemon["Docker Daemon\n(dockerd)"]
    Registry["Docker Registry\n(Docker Hub, ECR, GCR)"]

    subgraph "Host OS (Linux Kernel)"
        subgraph "Container A"
            App1["Next.js App\n(Process)"]
            Layer1["읽기/쓰기 레이어"]
        end
        subgraph "Container B"
            App2["Spring Boot\n(Process)"]
            Layer2["읽기/쓰기 레이어"]
        end
        Shared["공유 이미지 레이어\n(읽기 전용, COW)"]

        NS["네임스페이스\n(pid, net, mnt, uts)"]
        CG["cgroups\n(CPU, 메모리 제한)"]
    end

    CLI -->|"REST API"| Daemon
    Daemon -->|"pull/push"| Registry
    Daemon --> Container A & Container B
    Layer1 & Layer2 --> Shared
    NS & CG -->|"프로세스 격리"| Container A & Container B

    style Shared fill:#c8e6c9,stroke:#388e3c
    style NS fill:#bbdefb,stroke:#1565c0
    style CG fill:#fff9c4,stroke:#f9a825
```

**레이어 캐싱(Layer Caching):** Dockerfile 각 명령이 레이어를 만든다. 내용이 변경되지 않으면 캐시를 재사용 → 빌드 속도 향상. **변경이 적은 레이어를 위에, 자주 변경되는 레이어를 아래에** 배치하는 것이 핵심이다.

---

## 기본 Docker 명령어

```bash
# ── 이미지 관리 ──────────────────────────────────────────
docker pull node:20-alpine               # 이미지 다운로드
docker images                           # 로컬 이미지 목록
docker image rm node:20-alpine          # 이미지 삭제
docker image prune -a                   # 사용하지 않는 이미지 전체 삭제

# ── 컨테이너 실행 ─────────────────────────────────────────
docker run -d \                          # -d: 백그라운드 실행
  --name honey-web \
  -p 3000:3000 \                         # 호스트:컨테이너 포트 매핑
  -e NODE_ENV=production \              # 환경변수
  -v honey-data:/app/data \             # 볼륨 마운트
  --memory=512m \                       # 메모리 제한
  --cpus=1 \                            # CPU 제한
  honey-web:latest

# ── 컨테이너 관리 ─────────────────────────────────────────
docker ps                               # 실행 중인 컨테이너
docker ps -a                            # 모든 컨테이너
docker logs -f honey-web               # 로그 스트리밍
docker exec -it honey-web sh           # 컨테이너 내부 접속
docker stop honey-web                  # 정상 종료 (SIGTERM)
docker rm honey-web                    # 컨테이너 삭제

# ── 정리 ────────────────────────────────────────────────
docker system prune -a --volumes       # 모든 미사용 리소스 삭제
```

---

## Dockerfile — 레이어 캐싱 최적화

```dockerfile
# ❌ 나쁜 예: package.json과 소스코드를 같이 복사
FROM node:20-alpine
WORKDIR /app
COPY . .                    # ← 소스 변경마다 npm install 재실행
RUN npm install
RUN npm run build
CMD ["node", "server.js"]

# ✅ 좋은 예: 의존성과 소스 분리 (레이어 캐싱 활용)
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./       # ← package.json이 변경될 때만 npm install
RUN npm ci --only=production
COPY . .                    # ← 소스 변경은 여기서만
RUN npm run build
CMD ["node", "server.js"]
```

---

## 멀티스테이지 빌드 — 이미지 크기 80% 절감

```dockerfile
# Next.js 프로덕션 멀티스테이지 Dockerfile
# 빌드 결과: ~1.2GB(node:20) → ~200MB(node:20-alpine + 빌드 산출물만)

# ── Stage 1: 의존성 설치 ─────────────────────────────────
FROM node:20-alpine AS deps
RUN apk add --no-cache libc6-compat  # native module 빌드에 필요

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci                           # ci: package-lock.json 엄격히 준수

# ── Stage 2: 빌드 ────────────────────────────────────────
FROM node:20-alpine AS builder
WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Next.js 출력 모드를 standalone으로 (최소 산출물)
# next.config.ts: output: 'standalone'
ENV NEXT_TELEMETRY_DISABLED=1
RUN npm run build

# ── Stage 3: 프로덕션 런타임 ─────────────────────────────
FROM node:20-alpine AS runner
WORKDIR /app

ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

# 🔐 보안: non-root 사용자로 실행
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# standalone 빌드 결과물만 복사
COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs  # non-root 전환

EXPOSE 3000
ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

# 헬스체크: 컨테이너 상태 모니터링
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
  CMD wget -qO- http://localhost:3000/api/health || exit 1

CMD ["node", "server.js"]
```

### Spring Boot 멀티스테이지 Dockerfile

```dockerfile
# Spring Boot 멀티스테이지 빌드
# Layered JAR: 의존성/스프링부트/앱 레이어 분리 → 캐시 효율 극대화

# ── Stage 1: Gradle 빌드 ─────────────────────────────────
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app

# Gradle wrapper와 빌드 파일 먼저 (의존성 캐싱)
COPY gradlew .
COPY gradle gradle
COPY build.gradle.kts settings.gradle.kts ./
RUN ./gradlew dependencies --no-daemon  # 의존성 다운로드만

# 소스 복사 + 빌드
COPY src src
RUN ./gradlew build -x test --no-daemon  # 테스트 스킵 (CI에서 이미 실행)

# Layered JAR 추출
RUN java -Djarmode=layertools -jar build/libs/*.jar extract

# ── Stage 2: 프로덕션 런타임 ─────────────────────────────
FROM eclipse-temurin:21-jre-alpine AS runner

# 🔐 보안: non-root 사용자
RUN addgroup -S spring && adduser -S spring -G spring
USER spring

WORKDIR /app

# Layered JAR 레이어 순서 (변경 빈도 낮은 순)
COPY --from=builder app/dependencies/ ./
COPY --from=builder app/spring-boot-loader/ ./
COPY --from=builder app/snapshot-dependencies/ ./
COPY --from=builder app/application/ ./

EXPOSE 8080

# JVM 최적화 (컨테이너 메모리 인식)
ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0 -Djava.security.egd=file:/dev/./urandom"

HEALTHCHECK --interval=30s --timeout=5s --start-period=60s \
  CMD wget -qO- http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS org.springframework.boot.loader.JarLauncher"]
```

---

## Docker Compose — 전체 스택 오케스트레이션

```yaml
# docker-compose.yml — 개발 환경
# HoneyBarrel 전체 스택: Next.js + Spring Boot + PostgreSQL + Redis + Nginx

services:

  # ── 프론트엔드 ────────────────────────────────────────
  frontend:
    build:
      context: ./frontend
      target: runner          # 멀티스테이지: runner 스테이지만 빌드
    container_name: honey-frontend
    environment:
      - NODE_ENV=production
      - NEXT_PUBLIC_API_URL=http://api:8080
    ports:
      - "3000:3000"
    depends_on:
      api:
        condition: service_healthy  # api 헬스체크 통과 후 시작
    networks:
      - honey-net
    restart: unless-stopped

  # ── 백엔드 ───────────────────────────────────────────
  api:
    build:
      context: ./backend
      target: runner
    container_name: honey-api
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/honeydb
      - SPRING_DATASOURCE_USERNAME=honey
      - SPRING_DATASOURCE_PASSWORD_FILE=/run/secrets/db_password
      - SPRING_REDIS_HOST=cache
      - SPRING_PROFILES_ACTIVE=prod
    ports:
      - "8080:8080"
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_healthy
    networks:
      - honey-net
    secrets:
      - db_password
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 512m
          cpus: '1'

  # ── PostgreSQL ───────────────────────────────────────
  db:
    image: postgres:16-alpine
    container_name: honey-db
    environment:
      POSTGRES_DB: honeydb
      POSTGRES_USER: honey
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    volumes:
      - postgres-data:/var/lib/postgresql/data  # 데이터 영속화
      - ./db/init:/docker-entrypoint-initdb.d   # 초기 SQL 스크립트
    networks:
      - honey-net
    secrets:
      - db_password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U honey -d honeydb"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  # ── Redis ────────────────────────────────────────────
  cache:
    image: redis:7-alpine
    container_name: honey-cache
    command: >
      redis-server
      --requirepass ${REDIS_PASSWORD}
      --maxmemory 256mb
      --maxmemory-policy allkeys-lru
    volumes:
      - redis-data:/data
    networks:
      - honey-net
    healthcheck:
      test: ["CMD", "redis-cli", "--no-auth-warning", "-a", "${REDIS_PASSWORD}", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5
    restart: unless-stopped

  # ── Nginx 리버스 프록시 ──────────────────────────────
  nginx:
    image: nginx:alpine
    container_name: honey-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/certs:/etc/nginx/certs:ro
    depends_on:
      - frontend
      - api
    networks:
      - honey-net
    restart: unless-stopped

# ── 볼륨 ─────────────────────────────────────────────
volumes:
  postgres-data:      # 명명된 볼륨: Docker가 관리, 컨테이너 삭제해도 유지
  redis-data:

# ── 네트워크 ─────────────────────────────────────────
networks:
  honey-net:
    driver: bridge    # 기본값. 같은 네트워크의 컨테이너는 이름으로 통신

# ── 시크릿 (프로덕션 민감 데이터) ──────────────────────
secrets:
  db_password:
    file: ./secrets/db_password.txt  # 로컬 파일에서 로드
    # 프로덕션: external: true + Docker Swarm/K8s Secret 사용
```

### Nginx 설정

```nginx
# nginx/nginx.conf
events { worker_connections 1024; }

http {
    upstream frontend { server frontend:3000; }
    upstream api      { server api:8080; }

    server {
        listen 80;
        server_name honeybarrel.co.kr;
        return 301 https://$host$request_uri;  # HTTP → HTTPS 리다이렉트
    }

    server {
        listen 443 ssl;
        server_name honeybarrel.co.kr;

        ssl_certificate     /etc/nginx/certs/fullchain.pem;
        ssl_certificate_key /etc/nginx/certs/privkey.pem;

        # API 요청 → Spring Boot
        location /api/ {
            proxy_pass http://api;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # 나머지 → Next.js
        location / {
            proxy_pass http://frontend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

---

## 볼륨과 네트워크 상세

```bash
# ── 볼륨 ─────────────────────────────────────────────
docker volume create honey-data         # 명명된 볼륨 생성
docker volume ls                        # 볼륨 목록
docker volume inspect honey-data        # 볼륨 상세 정보
docker volume rm honey-data             # 볼륨 삭제 (데이터 삭제 주의!)

# 볼륨 타입 비교:
# named volume: Docker 관리, 컨테이너 간 공유, 백업 용이
# bind mount:   호스트 경로 직접 마운트, 개발 시 HMR에 유용
# tmpfs:        메모리 마운트, 임시 데이터, 컨테이너 종료 시 삭제

# ── 네트워크 ─────────────────────────────────────────
docker network create honey-net         # 브리지 네트워크 생성
docker network ls                       # 네트워크 목록
docker network inspect honey-net        # 연결된 컨테이너 확인

# 네트워크 타입:
# bridge:   기본값, 컨테이너 간 격리 + 이름으로 통신
# host:     호스트 네트워크 직접 사용 (성능 최적화, 격리 포기)
# none:     네트워크 완전 격리
# overlay:  Swarm/K8s 멀티호스트 컨테이너 통신
```

---

## 보안 강화

```dockerfile
# 🔐 프로덕션 보안 체크리스트

# 1. 특정 버전 고정 (latest 금지)
FROM node:20.18.0-alpine3.20  # ✅
# FROM node:latest             # ❌

# 2. non-root 사용자
USER nextjs   # ✅

# 3. 불필요한 패키지 최소화
# alpine 이미지 사용, 빌드 도구는 멀티스테이지로 분리

# 4. 시크릿을 ENV로 넣지 않기
# ENV DB_PASSWORD=secret123  # ❌ docker inspect로 노출
# Docker Secrets 또는 런타임 주입 사용  # ✅

# 5. .dockerignore
```

```
# .dockerignore — 빌드 컨텍스트에서 제외
node_modules
.next
.git
.env*           # 환경변수 파일 제외
*.log
coverage
.nyc_output
Dockerfile*
docker-compose*
README.md
```

```bash
# 6. Trivy로 취약점 스캔
docker pull aquasec/trivy
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image honey-web:latest

# CI에서 자동 스캔 (GitHub Actions)
- name: Trivy vulnerability scan
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'honey-web:latest'
    severity: 'CRITICAL,HIGH'
    exit-code: '1'  # 취약점 발견 시 CI 실패
```

---

## 실행 및 운영

```bash
# 개발 환경 (hot reload)
docker compose up --build

# 특정 서비스만 재빌드
docker compose up --build api

# 백그라운드 실행
docker compose up -d

# 로그 모니터링
docker compose logs -f api

# 스케일 아웃 (api 컨테이너 3개)
docker compose up --scale api=3 -d

# 서비스 중지 (데이터 유지)
docker compose stop

# 서비스 + 컨테이너 삭제 (데이터 유지)
docker compose down

# 서비스 + 컨테이너 + 볼륨 전체 삭제
docker compose down -v
```

---

## 이미지 최적화 결과

```
멀티스테이지 빌드 효과 (Next.js):
  일반 빌드:   node:20 → ~1.2 GB
  멀티스테이지: node:20-alpine (runner only) → ~200 MB
  절감: ~83%

Spring Boot Layered JAR:
  첫 빌드: 전체 다운로드 (느림)
  소스만 변경 후 재빌드: 앱 레이어만 갱신 → 빌드 시간 70% 절감

.dockerignore 효과:
  node_modules 제외: ~300MB 빌드 컨텍스트 절감
```

---

## 레퍼런스

### 공식 문서
- [Dockerfile Best Practices — Docker Docs](https://docs.docker.com/build/building/best-practices/) — Docker 공식 모범 사례
- [Multi-stage Builds — Docker Docs](https://docs.docker.com/get-started/docker-concepts/building-images/multi-stage-builds/) — 멀티스테이지 빌드 공식 가이드

### 기술 블로그
- [Dockerfile Best Practices: Security & Multi-Stage Builds — owais.io](https://www.owais.io/blog/2025-10-03_dockerfile-best-practices-security-production/) — non-root + Trivy 보안 강화 가이드 (2025.10)
- [Next.js with Docker, Compose & NGINX — Docker Blog](https://www.docker.com/blog/how-to-build-and-run-next-js-applications-with-docker-compose-nginx/) — Docker 공식 블로그 Next.js 가이드

---

*이 포스트는 [HoneyByte](https://blog.honeybarrel.co.kr) 개발 블로그의 일부입니다.*
