# AGENTS.md 분리로 컨텍스트 효율 높이기
## 하나의 프로젝트에 여러 개의 AGENTS.md 두기

AI가 **지금 작업 중인 디렉터리**에 가까운 AGENTS.md를 우선 참고하게 하면, 전체 프로젝트 설명을 한 번에 넣지 않아도 되어 **컨텍스트 효율**이 올라갑니다.

---

## 폴더 구조 예시

```
my-project/
├── AGENTS.md                    # 1) 프로젝트 전체: 목적, 스택, 공통 규칙
├── .cursor/
│   └── rules/                   # (선택) Cursor Rules와 병행 시
│       ├── global.mdc
│       └── ...
├── apps/
│   ├── web/
│   │   └── AGENTS.md            # 2) 웹 앱 전용: 프레임워크, API 연동 방식
│   └── api/
│       └── AGENTS.md            # 3) API 서버 전용: 라우팅, 인증, DB 접근
├── packages/
│   ├── shared-ui/
│   │   └── AGENTS.md            # 4) 공용 UI: 컴포넌트 규칙, 스토리북
│   └── shared-utils/
│       └── AGENTS.md            # 5) 공용 유틸: 네이밍, 에러 처리
├── docs/
│   └── AGENTS.md                # 6) 문서화: 작성 스타일, 템플릿
└── scripts/
    └── AGENTS.md                # 7) 스크립트: 실행 환경, 의존성
```

- **루트 `AGENTS.md`**: 프로젝트 전체 목적, 기술 스택, 레포 구조, 공통 컨벤션(브랜치, 커밋, 테스트).
- **하위 `AGENTS.md`**: 해당 폴더만의 책임, 사용 기술, 파일/네이밍 규칙, 자주 쓰는 패턴.

이렇게 두면 `apps/web`에서 작업할 때는 루트 + `apps/web/AGENTS.md` 위주로, `packages/shared-ui`에서 작업할 때는 루트 + `packages/shared-ui/AGENTS.md` 위주로 컨텍스트가 잡혀 **토큰을 덜 쓰고** 더 맞는 지시를 줄 수 있습니다.

---

## 각 AGENTS.md에 넣을 내용 예시

### 1) 루트 `AGENTS.md`

```markdown
# My Project

## 목적
- B2B 대시보드 + 공개 API 서비스
- Monorepo (pnpm workspace)

## 스택
- apps/web: Next.js 14, React 18
- apps/api: Node.js, Fastify
- packages/*: 공용 코드

## 공통 규칙
- 브랜치: feature/-, fix/-, chore/-
- 테스트: Vitest. 변경된 패키지만 테스트 실행
- 환경 변수: .env.example 참고, 비밀값은 코드에 넣지 말 것
```

### 2) `apps/web/AGENTS.md`

```markdown
# Web App (Next.js)

## 역할
- 대시보드 UI, 공개 랜딩 페이지
- API는 apps/api 호출 (내부 fetch, 경로는 env NEXT_PUBLIC_API_URL)

## 규칙
- 페이지: app router. 레이아웃은 app/(dashboard)/layout.tsx
- 컴포넌트: packages/shared-ui 사용 우선, 새 컴포넌트는 해당 패키지 제안 후 추가
- 스타일: Tailwind. 공통 색/간격은 tailwind.config에서 토큰 참고
```

### 3) `apps/api/AGENTS.md`

```markdown
# API Server (Fastify)

## 역할
- REST API. 인증: JWT (Authorization header)
- DB: PostgreSQL, Drizzle ORM. 마이그레이션은 packages/db

## 규칙
- 라우트: src/routes/ 하위에 도메인별 파일. 핸들러는 src/handlers/
- 에러: HTTP 상태 코드 + { code, message } JSON. 4xx/5xx는 공통 핸들러 사용
- 새 엔드포인트 추가 시 OpenAPI 스펙(spec.yaml) 동기 업데이트
```

### 4) `packages/shared-ui/AGENTS.md`

```markdown
# Shared UI

## 역할
- 버튼, 폼, 모달 등 공용 React 컴포넌트
- Storybook: pnpm storybook

## 규칙
- export는 index.ts만. 컴포넌트별 폴더 (Button/, Input/)
- props 타입은 해당 컴포넌트의 types.ts 또는 인터페이스로 정의
- 스타일: Tailwind. variants는 cva 사용
```

### 5) `packages/shared-utils/AGENTS.md`

```markdown
# Shared Utils

## 역할
- 날짜, 포맷, 검증 등 순수 함수
- 앱/API 공용

## 규칙
- 사이드 이펙트 없음. 테스트 필수
- 네이밍: getXxx, formatXxx, isXxx, validateXxx
- 에러는 throw new AppError(code, message), catch는 호출 측에서 처리
```

---

## 사용 시 참고

- **도구 동작**: Cursor 등에서는 보통 **현재 열린 파일/선택된 디렉터리** 기준으로 가까운 AGENTS.md(또는 rules)를 자동으로 포함할 수 있습니다. 정확한 동작은 도구 문서 확인.
- **중복 최소화**: 루트에는 “전체 공통”만 두고, 도메인/폴더별 차이는 해당 AGENTS.md에만 두면 컨텍스트가 짧고 명확해집니다.
- **버전 관리**: AGENTS.md도 코드처럼 커밋해 두면 팀 전체가 같은 컨텍스트로 AI를 쓸 수 있습니다.

이렇게 **한 프로젝트 안에 여러 개의 AGENTS.md를 두고**, 폴더별로 나누어 쓰면 컨텍스트 효율을 높일 수 있습니다.
