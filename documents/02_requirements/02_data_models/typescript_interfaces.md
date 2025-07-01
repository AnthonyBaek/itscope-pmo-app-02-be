# TypeScript 인터페이스 정의서

## 개요
이 문서는 프론트엔드에서 사용되는 TypeScript 인터페이스를 정의합니다. 백엔드 DTO와 1:1 대응됩니다.

---

## API 요청 인터페이스

### LoginRequest
**용도**: 로그인 API 요청  
**API**: `POST /api/auth/login`

```typescript
interface LoginRequest {
  email: string;
  password: string;
}
```

**Zod 스키마:**
```typescript
import { z } from 'zod';

export const loginRequestSchema = z.object({
  email: z
    .string()
    .min(1, '이메일은 필수 입력값입니다.')
    .email('올바른 이메일 형식이 아닙니다.'),
  password: z
    .string()
    .min(8, '비밀번호는 최소 8자 이상이어야 합니다.')
    .max(100, '비밀번호는 100자 이하여야 합니다.')
});

export type LoginRequest = z.infer<typeof loginRequestSchema>;
```

---

## API 응답 인터페이스

### LoginResponse
**용도**: 로그인 성공 응답  
**API**: `POST /api/auth/login` (성공 시)

```typescript
interface LoginResponse {
  accessToken: string;
  tokenType: string;
  expiresIn: number;
}
```

**Zod 스키마:**
```typescript
export const loginResponseSchema = z.object({
  accessToken: z.string(),
  tokenType: z.literal('Bearer'),
  expiresIn: z.number().positive()
});

export type LoginResponse = z.infer<typeof loginResponseSchema>;
```

### MainResponse
**용도**: 메인 페이지 데이터 응답  
**API**: `GET /api/main` (성공 시)

```typescript
interface MainResponse {
  message: string;
  user: User;
}
```

**Zod 스키마:**
```typescript
export const mainResponseSchema = z.object({
  message: z.string(),
  user: userSchema
});

export type MainResponse = z.infer<typeof mainResponseSchema>;
```

### ErrorResponse
**용도**: 에러 응답 표준화  
**API**: 모든 API (에러 발생 시)

```typescript
interface ErrorResponse {
  message: string;
  timestamp: string;
}
```

**Zod 스키마:**
```typescript
export const errorResponseSchema = z.object({
  message: z.string(),
  timestamp: z.string().datetime()
});

export type ErrorResponse = z.infer<typeof errorResponseSchema>;
```

---

## 도메인 모델 인터페이스

### User
**용도**: 사용자 정보  
**사용처**: MainResponse의 user 필드, AuthStore

```typescript
interface User {
  email: string;
  name: string;
}
```

**Zod 스키마:**
```typescript
export const userSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1)
});

export type User = z.infer<typeof userSchema>;
```

---

## 인증 상태 관리 인터페이스

### AuthState (Enum)
**용도**: 로그인 상태 관리

```typescript
enum AuthState {
  UNAUTHENTICATED = 'unauthenticated',
  AUTHENTICATING = 'authenticating', 
  AUTHENTICATED = 'authenticated'
}
```

### TokenData
**용도**: JWT 토큰 데이터 관리

```typescript
interface TokenData {
  accessToken: string;
  tokenType: string;
  expiresIn: number;
  expiresAt: number; // Unix timestamp
}
```

**헬퍼 함수:**
```typescript
export const createTokenData = (loginResponse: LoginResponse): TokenData => {
  const now = Date.now();
  const expiresAt = now + (loginResponse.expiresIn * 1000);
  
  return {
    ...loginResponse,
    expiresAt
  };
};

export const isTokenExpired = (tokenData: TokenData): boolean => {
  return Date.now() >= tokenData.expiresAt;
};
```

### AuthStore (Zustand)
**용도**: 인증 상태 전역 관리

```typescript
interface AuthStore {
  // 상태
  authState: AuthState;
  user: User | null;
  tokenData: TokenData | null;
  
  // 액션
  login: (credentials: LoginRequest) => Promise<void>;
  logout: () => void;
  setUser: (user: User) => void;
  setTokenData: (tokenData: TokenData) => void;
  checkAuthStatus: () => void;
  
  // 헬퍼
  isAuthenticated: () => boolean;
  isLoading: () => boolean;
  getAuthHeader: () => string | null;
}
```

---

## API 클라이언트 인터페이스

### ApiClient
**용도**: API 호출 클라이언트

```typescript
interface ApiClient {
  // Auth API
  login(request: LoginRequest): Promise<LoginResponse>;
  
  // Main API  
  getMainData(): Promise<MainResponse>;
  
  // 헬퍼
  setAuthToken(token: string): void;
  clearAuthToken(): void;
}
```

### ApiError
**용도**: API 에러 처리

```typescript
class ApiError extends Error {
  public readonly status: number;
  public readonly response: ErrorResponse;
  
  constructor(status: number, response: ErrorResponse) {
    super(response.message);
    this.status = status;
    this.response = response;
    this.name = 'ApiError';
  }
}
```

---

## React Hook 인터페이스

### useAuth Hook
**용도**: 인증 관련 React Hook

```typescript
interface UseAuthReturn {
  // 상태
  authState: AuthState;
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  
  // 액션
  login: (credentials: LoginRequest) => Promise<void>;
  logout: () => void;
  
  // 에러
  error: string | null;
  clearError: () => void;
}
```

### useApi Hook
**용도**: API 호출 React Hook

```typescript
interface UseApiReturn<TData, TError = ApiError> {
  data: TData | null;
  error: TError | null;
  isLoading: boolean;
  isError: boolean;
  refetch: () => Promise<void>;
}
```

---

## 라우팅 인터페이스

### ProtectedRouteProps
**용도**: 보호된 라우트 컴포넌트

```typescript
interface ProtectedRouteProps {
  children: React.ReactNode;
  redirectTo?: string;
  requiredAuth?: boolean;
}
```

### RouteConfig
**용도**: 라우트 설정

```typescript
interface RouteConfig {
  path: string;
  element: React.ComponentType;
  protected: boolean;
  title?: string;
}

const routes: RouteConfig[] = [
  {
    path: '/login',
    element: LoginPage,
    protected: false,
    title: '로그인'
  },
  {
    path: '/main',
    element: MainPage,
    protected: true,
    title: '메인 페이지'
  }
];
```

---

## 폼 처리 인터페이스

### LoginFormData
**용도**: React Hook Form + Zod

```typescript
interface LoginFormData {
  email: string;
  password: string;
}

// React Hook Form 설정
const {
  register,
  handleSubmit,
  formState: { errors, isSubmitting }
} = useForm<LoginFormData>({
  resolver: zodResolver(loginRequestSchema)
});
```

---

## 환경 변수 인터페이스

### AppConfig
**용도**: 환경별 설정

```typescript
interface AppConfig {
  API_BASE_URL: string;
  API_TIMEOUT: number;
  TOKEN_STORAGE_KEY: string;
  IS_DEVELOPMENT: boolean;
}

export const config: AppConfig = {
  API_BASE_URL: import.meta.env.VITE_API_BASE_URL || 'http://localhost:8080',
  API_TIMEOUT: 30000,
  TOKEN_STORAGE_KEY: 'pmo_auth_token',
  IS_DEVELOPMENT: import.meta.env.DEV
};
```

---

## 유틸리티 타입

### API Response 유틸리티
```typescript
// API 응답 래퍼
type ApiResponse<T> = {
  data: T;
  status: number;
  headers: Record<string, string>;
};

// Async 상태 관리
type AsyncState<T, E = Error> = {
  data: T | null;
  error: E | null;
  loading: boolean;
};

// 옵셔널 필드 유틸리티
type Optional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;
```

---

## 파일 구조 예시

```
src/
├── types/
│   ├── api.ts           # API 관련 인터페이스
│   ├── auth.ts          # 인증 관련 인터페이스  
│   ├── user.ts          # 사용자 관련 인터페이스
│   └── common.ts        # 공통 타입
├── schemas/
│   ├── auth.ts          # Zod 스키마
│   └── validation.ts    # 검증 스키마
├── stores/
│   └── authStore.ts     # Zustand 스토어
├── hooks/
│   ├── useAuth.ts       # 인증 훅
│   └── useApi.ts        # API 훅
└── services/
    └── apiClient.ts     # API 클라이언트
```

---

## 관련 문서
- [DTO 정의서](./dto_definitions.md)
- [API 엔드포인트 명세](../../01_architecture/02_da/api_endpoints.md)
- [사용자 로그인 플로우](../01_user_flow/flow_001_signin.md)

---

**이 문서는 프론트엔드 개발 시 타입 안정성을 보장하는 기준이 됩니다.** 