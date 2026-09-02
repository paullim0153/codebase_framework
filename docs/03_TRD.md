# Technical Requirements Documents
<!-- TRD: "특정 기능/모듈을 어떻게 구현할 것인가?" -->
<!-- 코드 작성 전에 먼저 이 문서에 설계를 명시한다. Design-First 원칙 -->

## 사용 방법

1. 새 기능 구현 전 아래 템플릿을 복사하여 TRD 항목을 추가한다.
2. `CLAUDE.md`의 CRITICAL 규칙: **TRD 미작성 기능은 구현하지 않는다.**
3. 구현 완료 후 **Status를 `Done`으로 업데이트**하고, 실제 구현과 다른 부분이 있으면 TRD을 수정한다.

---

## TRD-001: {기능/모듈명}
<!-- 예: 사용자 인증 플로우 -->

**Status**: `Proposed` | `In Progress` | `Done` | `Cancelled`  
**작성일**: {YYYY-MM-DD}  
**작성자**: {이름}  
**관련 PRD 기능**: [F-001](01_PRD.md#3-core-features--mvp-scope-핵심-기능)  
**관련 ADR**: [ADR-004](02_ADR.md#adr-004-인증인가-전략)

---

### 1. Overview (개요)
{이 기능이 무엇을 하는지 2~3문장으로 요약}

> 예: 이메일/패스워드 기반 로그인 및 회원가입 플로우를 구현한다. Supabase Auth를 사용하며, 세션은 Next.js Middleware에서 서버 사이드로 검증한다.

---

### 2. API Interfaces (인터페이스 정의)

#### Server Actions / API Routes
```typescript
// 예시: app/api/auth/login/route.ts
POST /api/auth/login
Request:  { email: string; password: string }
Response: { user: User; session: Session } | { error: string }

// 예시: app/api/auth/logout/route.ts
POST /api/auth/logout
Response: { success: boolean }
```

#### Service Layer Functions
```typescript
// 예시: services/auth.service.ts
async function loginUser(email: string, password: string): Promise<Result<User, AuthError>>
async function logoutUser(): Promise<void>
async function getCurrentUser(): Promise<User | null>
```

#### Component Props
```typescript
// 예시: components/features/LoginForm.tsx
interface LoginFormProps {
  onSuccess: (user: User) => void;
  onError: (error: string) => void;
  redirectTo?: string;
}
```

---

### 3. Data Model (데이터 모델)

```sql
-- 예시: Supabase 테이블 정의
CREATE TABLE profiles (
  id          UUID REFERENCES auth.users(id) PRIMARY KEY,
  email       TEXT NOT NULL,
  role        TEXT NOT NULL DEFAULT 'user' CHECK (role IN ('admin', 'user')),
  created_at  TIMESTAMPTZ DEFAULT NOW(),
  updated_at  TIMESTAMPTZ DEFAULT NOW()
);

-- RLS 정책
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users can view own profile" ON profiles
  FOR SELECT USING (auth.uid() = id);
```

---

### 4. State Management (상태 관리)

```typescript
// 예시: Zustand store
interface AuthStore {
  user: User | null;
  isLoading: boolean;
  setUser: (user: User | null) => void;
}
```

---

### 5. Error Handling & Security (에러 처리 & 보안)

| 시나리오 | 처리 방법 | HTTP 상태코드 |
|---------|----------|--------------|
| {예: 잘못된 자격증명} | {예: 제네릭 에러 메시지 반환 (정보 노출 방지)} | 401 |
| {예: 유효하지 않은 세션} | {예: 자동 로그아웃 및 로그인 페이지 리다이렉트} | 401 |
| {예: Rate Limit 초과} | {예: 1분간 요청 차단, 사용자에게 안내} | 429 |
| {예: 서버 오류} | {예: Sentry 에러 로깅, 사용자에게 일반 오류 메시지} | 500 |

**보안 체크리스트**:
- [ ] 비밀번호는 절대 로그에 남기지 않는다
- [ ] API 응답에서 비밀번호 해시 필드 제외
- [ ] HTTPS 전용 (HTTP Strict Transport Security)
- [ ] CSRF 토큰 적용

---

### 6. Test Strategy (테스트 전략)

```typescript
// 유닛 테스트 (Vitest)
describe('AuthService', () => {
  it('유효한 자격증명으로 로그인 성공', async () => { ... });
  it('잘못된 비밀번호로 AuthError 반환', async () => { ... });
  it('존재하지 않는 이메일로 AuthError 반환', async () => { ... });
});

// E2E 테스트 (Playwright)
test('로그인 → 대시보드 이동 플로우', async ({ page }) => { ... });
test('로그인 실패 시 에러 메시지 표시', async ({ page }) => { ... });
```

---

### 7. Implementation Checklist (구현 체크리스트)

- [ ] Service layer 함수 작성 (`services/auth.service.ts`)
- [ ] API Route 핸들러 작성 (`app/api/auth/`)
- [ ] Middleware 세션 검증 로직 작성
- [ ] LoginForm 컴포넌트 작성
- [ ] 에러 처리 및 로딩 상태 처리
- [ ] 유닛 테스트 작성 및 통과
- [ ] E2E 테스트 작성 및 통과
- [ ] `npm run lint` 통과
- [ ] `npm run type-check` 통과

---

## TRD-002: {다음 기능명}

**Status**: `Proposed`  
**작성일**: {YYYY-MM-DD}

> 위 TRD-001 템플릿을 복사하여 작성한다.

---

## TRD 추가 템플릿

```markdown
## TRD-{번호}: {기능명}

**Status**: Proposed
**작성일**: YYYY-MM-DD
**작성자**: {이름}
**관련 PRD 기능**: [F-{번호}](01_PRD.md#...)
**관련 ADR**: [ADR-{번호}](02_ADR.md#...)

### 1. Overview
### 2. API Interfaces
### 3. Data Model
### 4. State Management
### 5. Error Handling & Security
### 6. Test Strategy
### 7. Implementation Checklist
```

---

## Change Log

| 날짜 | 내용 | 작성자 |
|------|------|--------|
| {YYYY-MM-DD} | 최초 작성 | {이름} |
