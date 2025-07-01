# 상세 JWT 토큰 명세서 (JWT Token)

## 개요
이 문서는 PMO 애플리케이션에서 사용되는 JWT 토큰의 모든 구조와 규칙을 표 형식으로 상세하게 정리합니다.

---

## JWT 토큰 기본 정보

### 토큰 개요

| 항목 | 상세 내용 |
|------|-----------|
| **토큰 표준** | JSON Web Token (RFC 7519) |
| **서명 알고리즘** | HMAC SHA-256 (HS256) |
| **토큰 타입** | Bearer Token |
| **만료 시간** | 3600초 (1시간) |
| **발급처** | `itscope-pmo-backend` |
| **사용 목적** | API 인증 및 권한 부여 |

### 토큰 구조 개요

| 구성요소 | 설명 | 인코딩 | 구분자 |
|----------|------|--------|--------|
| **Header** | 토큰 메타데이터 | Base64URL | `.` |
| **Payload** | 사용자 정보 및 클레임 | Base64URL | `.` |
| **Signature** | 토큰 무결성 검증 | Base64URL | - |

**완성된 토큰 형태:**  
`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ0ZXN0QHRlc3QuY29tIiwiaWF0IjoxNjQyMjQ4MDAwLCJleHAiOjE2NDIyNTE2MDAsImVtYWlsIjoidGVzdEB0ZXN0LmNvbSJ9.signature`

---

## JWT Header

### Header 구조

| 항목 | 상세 내용 |
|------|-----------|
| **알고리즘 필드** | `alg` |
| **타입 필드** | `typ` |
| **인코딩** | Base64URL |
| **역할** | 토큰 메타데이터 정의 |

### Header 필드 명세

| 필드명 | 타입 | 필수여부 | 설명 | 고정값 | 예시 |
|--------|------|----------|------|--------|------|
| `alg` | String | Yes | 서명 알고리즘 | `HS256` | `HS256` |
| `typ` | String | Yes | 토큰 타입 | `JWT` | `JWT` |

### Header JSON 예시

| 시나리오 | JSON |
|----------|------|
| **표준 헤더** | `{"alg": "HS256", "typ": "JWT"}` |

### Header Base64URL 인코딩

| JSON | Base64URL 인코딩 결과 |
|------|----------------------|
| `{"alg":"HS256","typ":"JWT"}` | `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9` |

---

## JWT Payload

### Payload 구조

| 항목 | 상세 내용 |
|------|-----------|
| **표준 클레임** | `sub`, `iat`, `exp` |
| **커스텀 클레임** | `email` |
| **인코딩** | Base64URL |
| **역할** | 사용자 정보 및 토큰 메타데이터 |

### Payload 필드 명세

| 필드명 | 타입 | 필수여부 | 클레임 타입 | 설명 | 예시 값 |
|--------|------|----------|-------------|------|---------|
| `sub` | String | Yes | 표준 | 토큰 주체 (사용자 이메일) | `test@test.com` |
| `iat` | Long | Yes | 표준 | 토큰 발급 시간 (Unix timestamp) | `1642248000` |
| `exp` | Long | Yes | 표준 | 토큰 만료 시간 (Unix timestamp) | `1642251600` |
| `email` | String | Yes | 커스텀 | 사용자 이메일 | `test@test.com` |

### 표준 클레임 상세

| 클레임 | 영문명 | 설명 | 형식 | 계산 방식 |
|--------|--------|------|------|-----------|
| `sub` | Subject | 토큰 주체 | String | 사용자 이메일 그대로 |
| `iat` | Issued At | 발급 시간 | Unix timestamp | `System.currentTimeMillis() / 1000` |
| `exp` | Expiration | 만료 시간 | Unix timestamp | `iat + 3600` |

### 커스텀 클레임 상세

| 클레임 | 설명 | 타입 | 제약조건 | 사용 목적 |
|--------|------|------|----------|-----------|
| `email` | 사용자 이메일 | String | RFC 5322 형식 | 사용자 식별 및 메인 페이지 데이터 조회 |

### Payload JSON 예시

| 시나리오 | JSON |
|----------|------|
| **기본 페이로드** | `{"sub": "test@test.com", "iat": 1642248000, "exp": 1642251600, "email": "test@test.com"}` |
| **관리자 페이로드** | `{"sub": "admin@itscope.com", "iat": 1642248000, "exp": 1642251600, "email": "admin@itscope.com"}` |

### Payload Base64URL 인코딩

| JSON | Base64URL 인코딩 결과 |
|------|----------------------|
| `{"sub":"test@test.com","iat":1642248000,"exp":1642251600,"email":"test@test.com"}` | `eyJzdWIiOiJ0ZXN0QHRlc3QuY29tIiwiaWF0IjoxNjQyMjQ4MDAwLCJleHAiOjE2NDIyNTE2MDAsImVtYWlsIjoidGVzdEB0ZXN0LmNvbSJ9` |

---

## JWT Signature

### Signature 구조

| 항목 | 상세 내용 |
|------|-----------|
| **알고리즘** | HMAC SHA-256 |
| **입력 데이터** | `base64url(header) + "." + base64url(payload)` |
| **비밀키** | 서버 측 비밀키 (환경변수) |
| **출력** | Base64URL 인코딩된 서명 |

### Signature 생성 과정

| 단계 | 설명 | 입력 | 출력 |
|------|------|------|------|
| **1. 데이터 준비** | Header와 Payload 결합 | `header.payload` | `eyJhbGci...LmV5SnpkV0k...` |
| **2. HMAC 계산** | 비밀키로 HMAC-SHA256 실행 | `data + secret_key` | Binary hash |
| **3. Base64URL 인코딩** | 바이너리를 Base64URL로 변환 | Binary hash | `signature_string` |

### Signature 검증 규칙

| 검증 항목 | 규칙 | 성공 조건 | 실패 시 동작 |
|-----------|------|-----------|---------------|
| **서명 일치** | 재계산된 서명과 토큰 서명 비교 | 완전 일치 | 401 Unauthorized |
| **비밀키 유효성** | 서버 비밀키로만 검증 가능 | 키 일치 | 401 Unauthorized |
| **데이터 무결성** | Header, Payload 변조 검사 | 변조 없음 | 401 Unauthorized |

---

## 토큰 생명주기

### 토큰 발급 프로세스

| 단계 | 설명 | 입력 | 출력 | 소요시간 |
|------|------|------|------|---------|
| **1. 사용자 인증** | 이메일/비밀번호 검증 | LoginRequest | User 객체 | ~100ms |
| **2. 클레임 생성** | JWT Payload 구성 | User 객체 | JwtPayload | ~1ms |
| **3. 토큰 서명** | JWT 생성 및 서명 | JwtPayload + SecretKey | JWT Token | ~5ms |
| **4. 응답 반환** | LoginResponse 구성 | JWT Token | HTTP Response | ~1ms |

### 토큰 검증 프로세스

| 단계 | 설명 | 입력 | 검증 항목 | 실패 시 응답 |
|------|------|------|-----------|---------------|
| **1. 토큰 추출** | Authorization 헤더에서 추출 | `Bearer {token}` | 형식 검증 | 400 Bad Request |
| **2. 구조 검증** | JWT 3부분 구조 확인 | JWT Token | `.`으로 분할 | 400 Bad Request |
| **3. 서명 검증** | 무결성 및 진위 확인 | Header + Payload + Signature | HMAC 일치 | 401 Unauthorized |
| **4. 만료 검증** | 토큰 유효기간 확인 | `exp` 클레임 | 현재시간 < exp | 401 Unauthorized |
| **5. 사용자 추출** | 클레임에서 사용자 정보 추출 | `email` 클레임 | 이메일 형식 | 401 Unauthorized |

### 토큰 상태 전이

| 상태 | 설명 | 전이 조건 | 다음 상태 |
|------|------|-----------|-----------|
| **발급됨** | 새로 생성된 토큰 | 로그인 성공 | 활성 |
| **활성** | 유효한 토큰 | 만료시간 내 | 만료됨 |
| **만료됨** | 시간 초과된 토큰 | exp < 현재시간 | 무효 |
| **무효** | 사용 불가 토큰 | 서명 실패 등 | - |

---

## 토큰 보안 정책

### 비밀키 관리

| 항목 | 규칙 | 설정 방법 | 보안 수준 |
|------|------|-----------|-----------|
| **키 길이** | 최소 256비트 (32바이트) | 환경변수 | 높음 |
| **키 복잡도** | 영문+숫자+특수문자 조합 | 랜덤 생성 | 높음 |
| **키 저장** | 환경변수 또는 외부 키 저장소 | `JWT_SECRET_KEY` | 높음 |
| **키 순환** | 정기적 키 변경 (권장) | 운영 정책 | 중간 |

### 토큰 보안 검증

| 보안 항목 | 검증 방법 | 위험도 | 대응 방안 |
|-----------|-----------|--------|-----------|
| **토큰 재사용** | 만료시간 검증 | 중간 | 짧은 만료시간 설정 |
| **토큰 변조** | 서명 검증 | 높음 | HMAC 서명 필수 |
| **중간자 공격** | HTTPS 강제 | 높음 | TLS 1.2+ 사용 |
| **XSS 공격** | HttpOnly 쿠키 | 중간 | 로컬스토리지 대신 쿠키 |

---

## 에러 시나리오

### 토큰 관련 에러

| 에러 상황 | HTTP 상태 | 에러 메시지 | 원인 | 해결방안 |
|-----------|-----------|-------------|------|-----------|
| **토큰 없음** | 401 | "토큰이 제공되지 않았습니다." | Authorization 헤더 누락 | 토큰 포함하여 재요청 |
| **잘못된 형식** | 400 | "토큰 형식이 올바르지 않습니다." | Bearer 스키마 오류 | "Bearer {token}" 형식 확인 |
| **서명 오류** | 401 | "유효하지 않은 토큰입니다." | 서명 불일치 | 새로운 토큰 발급 필요 |
| **토큰 만료** | 401 | "토큰이 만료되었습니다." | exp < 현재시간 | 재로그인 필요 |
| **사용자 없음** | 401 | "사용자를 찾을 수 없습니다." | 토큰의 사용자 정보 불일치 | 토큰 재발급 |

### 에러 응답 형식

| 에러 타입 | ErrorResponse JSON |
|-----------|-------------------|
| **토큰 만료** | `{"message": "토큰이 만료되었습니다.", "timestamp": "2024-01-15T10:30:00Z"}` |
| **서명 오류** | `{"message": "유효하지 않은 토큰입니다.", "timestamp": "2024-01-15T10:30:00Z"}` |
| **토큰 없음** | `{"message": "토큰이 제공되지 않았습니다.", "timestamp": "2024-01-15T10:30:00Z"}` |

---

## Spring Security 통합

### 토큰 처리 컴포넌트

| 컴포넌트 | 역할 | 주요 메서드 | 의존성 |
|----------|------|-------------|--------|
| `JwtTokenProvider` | 토큰 생성/검증 | `generateToken()`, `validateToken()` | `@Component` |
| `JwtAuthenticationFilter` | 요청 인터셉트 | `doFilterInternal()` | `OncePerRequestFilter` |
| `JwtAuthenticationEntryPoint` | 인증 실패 처리 | `commence()` | `AuthenticationEntryPoint` |

### 설정 매개변수

| 설정 항목 | 설정 키 | 기본값 | 설명 |
|-----------|---------|--------|------|
| **비밀키** | `jwt.secret` | - | HMAC 서명용 비밀키 |
| **만료시간** | `jwt.expiration` | `3600` | 토큰 유효기간 (초) |
| **발급자** | `jwt.issuer` | `itscope-pmo-backend` | 토큰 발급자 식별 |

---

## 향후 확장 계획

### Phase 2 확장 기능

| 기능 | 설명 | 우선순위 | 구현 방식 |
|------|------|----------|-----------|
| **Refresh Token** | 액세스 토큰 갱신 | 높음 | 별도 토큰 발급 |
| **토큰 블랙리스트** | 로그아웃된 토큰 관리 | 중간 | Redis 캐시 활용 |
| **역할 기반 권한** | 사용자 역할별 접근 제어 | 중간 | `role` 클레임 추가 |
| **토큰 암호화** | 페이로드 암호화 | 낮음 | JWE 표준 사용 |

### 확장된 클레임 구조 (계획)

| 클레임 | 타입 | 설명 | Phase |
|--------|------|------|-------|
| `role` | String | 사용자 역할 | Phase 2 |
| `permissions` | Array | 권한 목록 | Phase 2 |
| `jti` | String | JWT 고유 ID | Phase 2 |
| `iss` | String | 토큰 발급자 | Phase 2 |

---

## 관련 문서
- [상세 DTO 명세](./detailed_dtos.md)
- [도메인 모델 상세 명세](./detailed_domain_models.md)
- [API 엔드포인트 명세](../../01_architecture/02_da/api_endpoints.md) 