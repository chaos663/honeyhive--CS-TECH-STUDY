---
layout: post
title: "Next.js 인증과 미들웨어 — Auth.js v5, middleware.ts, 보호 라우트"
date: 2026-04-16 16:33:38 +0900
excerpt: "Auth.js v5(NextAuth v5) 는 Next.js App Router에 최적화된 인증 라이브러리다. `auth.ts` 하나로 설정을 관리하며, 미들웨어와 서버 컴포넌트에서 모두 세션을 읽을 수 있다. `middleware.ts` 는 Edge Runtime에서 실행되어 모든 요청을 가로채며, 인증된 사용자만 접근 가능한 보호 라우트를 구현한다. 단, Edge Runtime은 Node.js API(Prisma, bcrypt 등)를 사용할"
tags: [Application, cs-study]
image: "/assets/images/thumbnails/2026-04-16-nextjs-인증과-미들웨어-authjs-v5-middlewarets-보호-라우트.svg"
series: "HoneyByte"
---
> Next.js 인증에서 가장 흔한 실수 두 가지: 미들웨어에서 DB 쿼리를 실행하려는 것(Edge Runtime 제약), 그리고 CVE-2025-29927(미들웨어 우회 취약점)을 모르는 것. 올바른 구조를 처음부터 잡아야 나중에 고치지 않는다.

## 핵심 요약 (TL;DR)

**Auth.js v5(NextAuth v5)** 는 Next.js App Router에 최적화된 인증 라이브러리다. `auth.ts` 하나로 설정을 관리하며, 미들웨어와 서버 컴포넌트에서 모두 세션을 읽을 수 있다. **`middleware.ts`** 는 Edge Runtime에서 실행되어 모든 요청을 가로채며, 인증된 사용자만 접근 가능한 보호 라우트를 구현한다. 단, Edge Runtime은 Node.js API(Prisma, bcrypt 등)를 사용할 수 없으므로 **인증 로직과 DB 로직을 분리**하는 것이 핵심이다.

---

## 인증 흐름 전체 구조

```mermaid
flowchart TD
    Request["HTTP 요청"]

    Request --> MW["middleware.ts\n(Edge Runtime)\n세션 쿠키 검증만"]

    MW -->|"인증 안 됨 + 보호 경로"| Redirect["→ /login 리다이렉트"]
    MW -->|"인증됨 or 공개 경로"| App["Next.js App"]

    App --> SC["Server Component\nauth() 호출\n세션 읽기"]
    App --> SA["Server Action\nauth() 호출\n권한 검증"]
    App --> RH["Route Handler\nauth() 호출\nAPI 보호"]

    SC --> DB["DB / ORM\n(Prisma, Drizzle)\n서버에서만"]
    SA --> DB
    RH --> DB

    style MW fill:#fff9c4,stroke:#f9a825
    style Redirect fill:#fce4ec,stroke:#c62828
    style DB fill:#c8e6c9,stroke:#388e3c
```

**핵심 원칙:**
- `middleware.ts` (Edge): JWT 검증만 — DB 접근 없음
- Server Component / Server Action / Route Handler: 세션 확인 + DB 접근 가능

---

## 환경 설정

### 설치

```bash
# Auth.js v5 (NextAuth v5)
npm install next-auth@beta

# 환경변수 생성
npx auth secret  # AUTH_SECRET 자동 생성

# 소셜 로그인 + DB 어댑터 (선택)
npm install @auth/prisma-adapter prisma @prisma/client
```

### 환경변수 설정

```bash
# .env.local
AUTH_SECRET=your-generated-secret-here  # npx auth secret으로 생성

# Google OAuth
AUTH_GOOGLE_ID=your-google-client-id
AUTH_GOOGLE_SECRET=your-google-client-secret

# Kakao OAuth
AUTH_KAKAO_ID=your-kakao-client-id
AUTH_KAKAO_SECRET=your-kakao-client-secret

# DB (세션 DB 저장 시)
DATABASE_URL="postgresql://user:pass@localhost:5432/honeydb"
```

---

## Auth.js v5 설정 — 2파일 구조

Auth.js v5는 **Edge 호환 설정**(`auth.config.ts`)과 **Node.js 전용 설정**(`auth.ts`)을 분리한다. 이 분리가 핵심이다.

### `auth.config.ts` — Edge 호환 설정 (미들웨어용)

```typescript
// src/auth.config.ts
// ⚠️ Edge Runtime 호환: Prisma, bcrypt 등 Node.js 전용 모듈 import 금지
import type { NextAuthConfig } from 'next-auth'
import Google from 'next-auth/providers/google'
import Kakao from 'next-auth/providers/kakao'

export const authConfig: NextAuthConfig = {
  pages: {
    signIn: '/login',    // 커스텀 로그인 페이지
    error: '/auth/error',
  },

  callbacks: {
    // ── 라우트 보호 콜백 (미들웨어에서 호출됨) ────────────────
    authorized({ auth, request: { nextUrl } }) {
      const isLoggedIn = !!auth?.user
      const isOnDashboard = nextUrl.pathname.startsWith('/dashboard')
      const isOnAdmin = nextUrl.pathname.startsWith('/admin')
      const isOnLogin = nextUrl.pathname.startsWith('/login')

      // 대시보드: 로그인 필요
      if (isOnDashboard) {
        return isLoggedIn
      }

      // 관리자: ADMIN 역할 필요
      if (isOnAdmin) {
        return isLoggedIn && auth?.user?.role === 'ADMIN'
      }

      // 이미 로그인한 사용자가 로그인 페이지 → 홈으로
      if (isOnLogin && isLoggedIn) {
        return Response.redirect(new URL('/', nextUrl))
      }

      return true  // 나머지 공개 접근 허용
    },

    // ── JWT 토큰 커스터마이징 ──────────────────────────────
    async jwt({ token, user, account }) {
      if (user) {
        // 최초 로그인 시: DB의 사용자 정보를 토큰에 포함
        token.id = user.id
        token.role = (user as any).role ?? 'USER'
      }
      return token
    },

    // ── 세션 커스터마이징 ─────────────────────────────────
    async session({ session, token }) {
      if (token && session.user) {
        session.user.id = token.id as string
        session.user.role = token.role as string
      }
      return session
    },
  },

  // Edge Runtime에서 사용할 프로바이더만 (DB 접근 없는 것)
  providers: [Google, Kakao],
}
```

### `auth.ts` — 메인 설정 (Node.js 환경)

```typescript
// src/auth.ts
// Node.js 전용 — Prisma, bcrypt 사용 가능
import NextAuth from 'next-auth'
import { PrismaAdapter } from '@auth/prisma-adapter'
import Credentials from 'next-auth/providers/credentials'
import bcrypt from 'bcryptjs'
import { z } from 'zod'
import { prisma } from '@/lib/prisma'
import { authConfig } from './auth.config'

export const { handlers, auth, signIn, signOut } = NextAuth({
  ...authConfig,

  // DB 어댑터: 소셜 로그인 사용자 자동 저장
  adapter: PrismaAdapter(prisma),

  session: {
    strategy: 'jwt',  // 'database' 대신 jwt → Edge 미들웨어 호환
    maxAge: 30 * 24 * 60 * 60,  // 30일
  },

  // Credentials (이메일/비밀번호) 프로바이더 추가
  providers: [
    ...authConfig.providers,
    Credentials({
      async authorize(credentials) {
        // 입력 검증
        const parsed = z.object({
          email: z.string().email(),
          password: z.string().min(6),
        }).safeParse(credentials)

        if (!parsed.success) return null

        // DB에서 사용자 조회 (Node.js 환경에서만 가능)
        const user = await prisma.user.findUnique({
          where: { email: parsed.data.email },
        })

        if (!user?.password) return null  // 소셜 전용 계정

        // 비밀번호 검증
        const isValid = await bcrypt.compare(parsed.data.password, user.password)
        if (!isValid) return null

        return {
          id: user.id,
          email: user.email,
          name: user.name,
          image: user.image,
          role: user.role,
        }
      },
    }),
  ],
})
```

### Prisma 스키마

```prisma
// prisma/schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

// Auth.js 필수 모델
model User {
  id            String    @id @default(cuid())
  name          String?
  email         String?   @unique
  emailVerified DateTime?
  image         String?
  password      String?   // Credentials 로그인 시만 사용
  role          String    @default("USER")
  accounts      Account[]
  sessions      Session[]
  createdAt     DateTime  @default(now())
}

model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String? @db.Text
  access_token      String? @db.Text
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String? @db.Text
  session_state     String?
  user              User    @relation(fields: [userId], references: [id], onDelete: Cascade)
  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}
```

---

## `middleware.ts` — 보호 라우트 구현

```typescript
// src/middleware.ts (프로젝트 루트 or src/)
// ⚠️ Edge Runtime: Node.js API 사용 불가
import NextAuth from 'next-auth'
import { authConfig } from './auth.config'

// auth.config.ts의 Edge 호환 설정으로 미들웨어 생성
export const { auth: middleware } = NextAuth(authConfig)

// 미들웨어가 실행될 경로 패턴 (성능 최적화 — 정적 파일 제외)
export const config = {
  matcher: [
    /*
     * 다음을 제외한 모든 경로에 미들웨어 실행:
     * - _next/static (정적 파일)
     * - _next/image (이미지 최적화)
     * - favicon.ico, sitemap.xml
     * - 공개 이미지/아이콘
     */
    '/((?!_next/static|_next/image|favicon.ico|sitemap.xml|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)',
  ],
}
```

### 커스텀 미들웨어 (로깅, A/B 테스트 추가)

```typescript
// src/middleware.ts — 커스텀 로직 추가
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'
import NextAuth from 'next-auth'
import { authConfig } from './auth.config'

const { auth } = NextAuth(authConfig)

export default auth(async function middleware(req) {
  const { auth: session, nextUrl } = req

  // ── 1. Rate Limiting 헤더 추가 ────────────────────────
  const response = NextResponse.next()
  response.headers.set('X-Frame-Options', 'DENY')
  response.headers.set('X-Content-Type-Options', 'nosniff')

  // ── 2. 관리자 경로 보호 ───────────────────────────────
  if (nextUrl.pathname.startsWith('/admin')) {
    if (!session?.user) {
      return NextResponse.redirect(new URL('/login?callbackUrl=' + encodeURIComponent(nextUrl.pathname), nextUrl))
    }
    if (session.user.role !== 'ADMIN') {
      return NextResponse.redirect(new URL('/403', nextUrl))
    }
  }

  // ── 3. 인증 필요 경로 ─────────────────────────────────
  const protectedPaths = ['/dashboard', '/orders', '/profile']
  const isProtected = protectedPaths.some(path => nextUrl.pathname.startsWith(path))

  if (isProtected && !session?.user) {
    const loginUrl = new URL('/login', nextUrl)
    loginUrl.searchParams.set('callbackUrl', nextUrl.pathname)
    return NextResponse.redirect(loginUrl)
  }

  // ── 4. 로그인 사용자가 /login 접근 시 홈으로 ──────────
  if (nextUrl.pathname === '/login' && session?.user) {
    const callbackUrl = nextUrl.searchParams.get('callbackUrl') ?? '/'
    return NextResponse.redirect(new URL(callbackUrl, nextUrl))
  }

  return response
})

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico|.*\\.png$).*)'],
}
```

---

## Route Handler — API 엔드포인트 설정

```typescript
// src/app/api/auth/[...nextauth]/route.ts
import { handlers } from '@/auth'

export const { GET, POST } = handlers
// GET  /api/auth/session    → 현재 세션
// GET  /api/auth/providers  → 사용 가능한 프로바이더
// POST /api/auth/signin     → 로그인
// POST /api/auth/signout    → 로그아웃
// GET  /api/auth/callback/* → OAuth 콜백
```

---

## 서버 컴포넌트에서 세션 사용

```tsx
// src/app/dashboard/page.tsx
import { auth } from '@/auth'
import { redirect } from 'next/navigation'

export default async function DashboardPage() {
  // 서버 컴포넌트에서 세션 조회 (미들웨어 통과 후 2차 검증)
  const session = await auth()

  // 미들웨어가 이미 보호하지만, 방어적 프로그래밍으로 재확인
  if (!session?.user) redirect('/login')

  return (
    <div>
      <h1>안녕하세요, {session.user.name}님 👋</h1>
      <p>이메일: {session.user.email}</p>
      <p>역할: {session.user.role}</p>

      {session.user.role === 'ADMIN' && (
        <a href="/admin" className="text-red-600 font-bold">
          관리자 패널 →
        </a>
      )}
    </div>
  )
}
```

---

## 로그인 페이지 — 커스텀 UI

```tsx
// src/app/login/page.tsx
import { signIn } from '@/auth'
import { redirect } from 'next/navigation'
import { AuthError } from 'next-auth'

interface LoginPageProps {
  searchParams: Promise<{ callbackUrl?: string; error?: string }>
}

export default async function LoginPage({ searchParams }: LoginPageProps) {
  const { callbackUrl, error } = await searchParams

  return (
    <div className="min-h-screen flex items-center justify-center bg-amber-50">
      <div className="bg-white rounded-2xl shadow-lg p-8 w-full max-w-md">
        <h1 className="text-2xl font-bold text-center mb-6 text-amber-900">
          🍯 HoneyBarrel 로그인
        </h1>

        {/* 에러 메시지 */}
        {error && (
          <div className="bg-red-50 text-red-600 rounded-lg px-4 py-2 mb-4 text-sm">
            {error === 'CredentialsSignin' ? '이메일 또는 비밀번호가 올바르지 않습니다.'
              : error === 'OAuthSignin' ? '소셜 로그인 중 오류가 발생했습니다.'
              : '로그인에 실패했습니다.'}
          </div>
        )}

        {/* 이메일/비밀번호 로그인 */}
        <form
          action={async (formData: FormData) => {
            'use server'
            try {
              await signIn('credentials', {
                email: formData.get('email'),
                password: formData.get('password'),
                redirectTo: callbackUrl ?? '/dashboard',
              })
            } catch (error) {
              if (error instanceof AuthError) {
                redirect(`/login?error=${error.type}`)
              }
              throw error
            }
          }}
          className="space-y-4 mb-6"
        >
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-1">
              이메일
            </label>
            <input
              name="email"
              type="email"
              required
              className="w-full border border-gray-300 rounded-lg px-3 py-2 focus:ring-2 focus:ring-amber-400 focus:border-transparent"
              placeholder="king@honeybarrel.co.kr"
            />
          </div>
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-1">
              비밀번호
            </label>
            <input
              name="password"
              type="password"
              required
              className="w-full border border-gray-300 rounded-lg px-3 py-2 focus:ring-2 focus:ring-amber-400"
              placeholder="••••••••"
            />
          </div>
          <button
            type="submit"
            className="w-full bg-amber-400 text-amber-900 py-2 rounded-lg font-semibold hover:bg-amber-500 transition-colors"
          >
            로그인
          </button>
        </form>

        <div className="relative mb-6">
          <div className="absolute inset-0 flex items-center">
            <div className="w-full border-t border-gray-200" />
          </div>
          <div className="relative flex justify-center text-sm">
            <span className="px-2 bg-white text-gray-500">또는</span>
          </div>
        </div>

        {/* 소셜 로그인 */}
        <div className="space-y-3">
          <form action={async () => {
            'use server'
            await signIn('google', { redirectTo: callbackUrl ?? '/dashboard' })
          }}>
            <button
              type="submit"
              className="w-full flex items-center justify-center gap-3 border border-gray-300 rounded-lg px-4 py-2 hover:bg-gray-50 transition-colors"
            >
              <GoogleIcon />
              Google로 계속하기
            </button>
          </form>

          <form action={async () => {
            'use server'
            await signIn('kakao', { redirectTo: callbackUrl ?? '/dashboard' })
          }}>
            <button
              type="submit"
              className="w-full flex items-center justify-center gap-3 bg-yellow-300 text-yellow-900 rounded-lg px-4 py-2 hover:bg-yellow-400 transition-colors font-semibold"
            >
              <KakaoIcon />
              카카오로 계속하기
            </button>
          </form>
        </div>
      </div>
    </div>
  )
}

function GoogleIcon() {
  return <span className="text-lg">G</span>
}
function KakaoIcon() {
  return <span className="text-lg">K</span>
}
```

---

## 클라이언트 컴포넌트에서 세션

```tsx
// src/components/UserMenu.tsx
'use client'

import { useSession, signOut } from 'next-auth/react'

export function UserMenu() {
  const { data: session, status } = useSession()

  if (status === 'loading') {
    return <div className="w-8 h-8 rounded-full bg-gray-200 animate-pulse" />
  }

  if (!session) {
    return <a href="/login" className="text-sm text-amber-900 font-medium">로그인</a>
  }

  return (
    <div className="relative group">
      <button className="flex items-center gap-2">
        {session.user?.image && (
          <img src={session.user.image} alt="" className="w-8 h-8 rounded-full" />
        )}
        <span className="text-sm font-medium">{session.user?.name}</span>
      </button>

      {/* 드롭다운 */}
      <div className="absolute right-0 mt-2 w-48 bg-white rounded-xl shadow-lg
                      opacity-0 invisible group-hover:opacity-100 group-hover:visible
                      transition-all duration-200">
        <a href="/dashboard" className="block px-4 py-2 text-sm hover:bg-gray-50">대시보드</a>
        <a href="/profile" className="block px-4 py-2 text-sm hover:bg-gray-50">프로필</a>
        <button
          onClick={() => signOut({ callbackUrl: '/' })}
          className="w-full text-left px-4 py-2 text-sm text-red-600 hover:bg-red-50"
        >
          로그아웃
        </button>
      </div>
    </div>
  )
}
```

### SessionProvider 설정

```tsx
// src/app/layout.tsx
import { SessionProvider } from 'next-auth/react'
import { auth } from '@/auth'

export default async function RootLayout({ children }: { children: React.ReactNode }) {
  const session = await auth()  // 서버에서 세션 프리페칭

  return (
    <html lang="ko">
      <body>
        {/* 서버에서 가져온 세션을 클라이언트에 전달 → 불필요한 재요청 방지 */}
        <SessionProvider session={session}>
          {children}
        </SessionProvider>
      </body>
    </html>
  )
}
```

---

## TypeScript 세션 타입 확장

```typescript
// src/types/next-auth.d.ts
import type { DefaultSession, DefaultUser } from 'next-auth'
import type { DefaultJWT } from 'next-auth/jwt'

declare module 'next-auth' {
  interface Session {
    user: {
      id: string
      role: string
    } & DefaultSession['user']
  }

  interface User extends DefaultUser {
    role?: string
  }
}

declare module 'next-auth/jwt' {
  interface JWT extends DefaultJWT {
    id?: string
    role?: string
  }
}
```

---

## ⚠️ 보안 주의사항 — CVE-2025-29927

```
CVE-2025-29927: Next.js 미들웨어 인증 우회 취약점

공격: x-middleware-subrequest 헤더 조작으로 미들웨어 실행 건너뜀
영향: 미들웨어만으로 인증을 구현한 경우 완전 우회 가능
패치: Next.js ≥ 15.2.3 (2025년 3월)

대응:
1. Next.js 버전 업데이트 (≥ 15.2.3)
2. 미들웨어에만 의존하지 말고 Server Component/Action에서도 auth() 검증
3. CDN/리버스 프록시에서 x-middleware-subrequest 헤더 차단
```

```typescript
// ✅ 방어적 프로그래밍 — 미들웨어 + 서버 컴포넌트 이중 검증
export default async function AdminPage() {
  const session = await auth()

  // 미들웨어가 보호하더라도 서버에서 재검증
  if (!session?.user || session.user.role !== 'ADMIN') {
    redirect('/403')
  }

  // ...
}
```

---

## 트레이드오프 비교

| 방식 | 장점 | 단점 | 적합한 경우 |
|------|------|------|------------|
| **Auth.js v5** | 소셜/Credentials 통합, 검증됨 | 설정 복잡, 2파일 구조 | 대부분의 Next.js 프로젝트 |
| **Clerk** | 가장 쉬운 설정, 관리 UI 제공 | 유료 (엔터프라이즈), 벤더 종속 | 빠른 MVP |
| **jose 직접 구현** | 완전한 제어, 외부 의존성 없음 | 구현 공수 많음 | 커스텀 인증 요구사항 |
| **NextAuth v4** | 레거시 코드베이스 | App Router 비최적화 | 기존 Pages Router 유지 |

---

## 시리즈 안내

| Part | 주제 | 상태 |
|------|------|------|
| Part 1 | App Router 시작하기 | ✅ 완료 |
| Part 2 | 데이터 페칭과 캐싱 | ✅ 완료 |
| **Part 3** | **인증과 미들웨어** | 현재 글 |
| Part 4 | 성능 최적화 | 예정 |
| Part 5 | 테스트 전략 | 예정 |
| Part 6 | 배포와 운영 | 예정 |

---

## 레퍼런스

### 공식 문서
- [Authentication — Next.js Guides](https://nextjs.org/docs/app/guides/authentication) — Next.js 공식 인증 가이드 (jose + 직접 구현)
- [Auth.js — Migrating to v5](https://authjs.dev/getting-started/migrating-to-v5) — NextAuth v5 마이그레이션 가이드
- [Next.js Configuration — NextAuth.js](https://next-auth.js.org/configuration/nextjs) — 미들웨어 설정 공식 레퍼런스

### 기술 블로그
- [Next.js Session Management 2025 — Clerk](https://clerk.com/articles/nextjs-session-management-solving-nextauth-persistence-issues) — CVE-2025-29927 분석 + 세션 관리 모범 사례

---

*이 포스트는 [HoneyByte](https://blog.honeybarrel.co.kr) Next.js Deep Dive 시리즈의 일부입니다.*
