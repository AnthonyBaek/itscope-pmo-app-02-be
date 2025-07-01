# 상세 DTO 명세서 (DTOs)

## 개요
이 문서는 PMO 애플리케이션의 모든 DTO를 표 형식으로 상세하게 정리합니다.

---

## 요청 DTO (Request DTOs)

### LoginRequest

| 항목 | 상세 내용 |
|------|-----------|
| **클래스명** | `LoginRequest` |
| **용도** | 사용자 로그인 요청 |
| **사용 API** | `POST /api/auth/login` |
| **패키지** | `com.itscope.pmo.dto.auth.request` |

#### 필드 명세

| 필드명 | 타입 | 필수여부 | 설명 | 검증 어노테이션 | 에러 메시지 |
|--------|------|----------|------|----------------|-------------|
| `email` | String | Yes | 사용자 이메일 주소 | `@NotBlank`, `@Email` | "이메일은 필수 입력값입니다.", "올바른 이메일 형식이 아닙니다." |
| `password` | String | Yes | 사용자 비밀번호 | `@NotBlank`, `@Size(min=8)` | "비밀번호는 필수 입력값입니다.", "비밀번호는 최소 8자 이상이어야 합니다." |

#### JSON 예시

| 시나리오 | JSON |
|----------|------|
| **정상 요청** | `{"email": "test@test.com", "password": "password123"}` |
| **빈 이메일** | `{"email": "", "password": "password123"}` |
| **잘못된 이메일** | `{"email": "invalid-email", "password": "password123"}` |
| **짧은 비밀번호** | `{"email": "test@test.com", "password": "123"}` |

#### 검증 규칙 상세

| 검증 항목 | 규칙 | 통과 예시 | 실패 예시 |
|-----------|------|-----------|-----------|
| **이메일 형식** | RFC 5322 표준 | `user@domain.com` | `invalid-email` |
| **이메일 필수** | 빈 값 불허 | `test@test.com` | `""`, `null` |
| **비밀번호 길이** | 최소 8자 | `password123` | `123` |
| **비밀번호 필수** | 빈 값 불허 | `mypassword` | `""`, `null` |

---

## 응답 DTO (Response DTOs)

### LoginResponse

| 항목 | 상세 내용 |
|------|-----------|
| **클래스명** | `LoginResponse` |
| **용도** | 로그인 성공 응답 |
| **사용 API** | `POST /api/auth/login` (200 OK) |
| **패키지** | `com.itscope.pmo.dto.auth.response` |

#### 필드 명세

| 필드명 | 타입 | 필수여부 | 설명 | 예시 값 | 비고 |
|--------|------|----------|------|---------|------|
| `accessToken` | String | Yes | JWT 액세스 토큰 | `eyJhbGciOiJIUzI1NiIs...` | Base64 인코딩된 JWT |
| `tokenType` | String | Yes | 토큰 타입 | `Bearer` | 항상 "Bearer" 고정값 |
| `expiresIn` | Integer | Yes | 토큰 만료 시간 (초) | `3600` | 1시간 = 3600초 |

#### JSON 예시

| 시나리오 | JSON |
|----------|------|
| **정상 응답** | `{"accessToken": "eyJhbGciOiJIUzI1NiIs...", "tokenType": "Bearer", "expiresIn": 3600}` |

### MainResponse

| 항목 | 상세 내용 |
|------|-----------|
| **클래스명** | `MainResponse` |
| **용도** | 메인 페이지 데이터 응답 |
| **사용 API** | `GET /api/main` (200 OK) |
| **패키지** | `com.itscope.pmo.dto.main.response` |

#### 필드 명세

| 필드명 | 타입 | 필수여부 | 설명 | 예시 값 | 비고 |
|--------|------|----------|------|---------|------|
| `message` | String | Yes | 환영 메시지 | `안녕하세요, test@test.com 님!` | 동적 생성 메시지 |
| `user` | User | Yes | 사용자 정보 객체 | `{"email": "test@test.com", "name": "테스트 사용자"}` | User 도메인 모델 |

#### JSON 예시

| 시나리오 | JSON |
|----------|------|
| **정상 응답** | `{"message": "안녕하세요, test@test.com 님!", "user": {"email": "test@test.com", "name": "테스트 사용자"}}` |

### ErrorResponse

| 항목 | 상세 내용 |
|------|-----------|
| **클래스명** | `ErrorResponse` |
| **용도** | 에러 응답 표준화 |
| **사용 API** | 모든 API (에러 발생 시) |
| **패키지** | `com.itscope.pmo.dto.common.response` |

#### 필드 명세

| 필드명 | 타입 | 필수여부 | 설명 | 형식 | 예시 값 |
|--------|------|----------|------|------|---------|
| `message` | String | Yes | 에러 메시지 | 사용자 친화적 메시지 | `이메일 또는 비밀번호가 일치하지 않습니다.` |
| `timestamp` | String | Yes | 에러 발생 시간 | ISO 8601 형식 | `2024-01-15T10:30:00Z` |

#### HTTP 상태코드별 사용

| 상태코드 | 사용 상황 | 메시지 예시 |
|----------|-----------|-------------|
| **400 Bad Request** | 입력 검증 실패 | `이메일은 필수 입력값입니다.` |
| **401 Unauthorized** | 인증 실패 | `이메일 또는 비밀번호가 일치하지 않습니다.` |
| **500 Internal Server Error** | 서버 내부 오류 | `서버에 일시적인 오류가 발생했습니다.` |

#### JSON 예시

| 시나리오 | 상태코드 | JSON |
|----------|----------|------|
| **인증 실패** | 401 | `{"message": "이메일 또는 비밀번호가 일치하지 않습니다.", "timestamp": "2024-01-15T10:30:00Z"}` |
| **검증 실패** | 400 | `{"message": "이메일은 필수 입력값입니다.", "timestamp": "2024-01-15T10:30:00Z"}` |
| **서버 오류** | 500 | `{"message": "서버에 일시적인 오류가 발생했습니다.", "timestamp": "2024-01-15T10:30:00Z"}` |

---

## 데이터 검증 어노테이션 매트릭스

### Bean Validation 어노테이션

| 어노테이션 | 용도 | 적용 필드 | 메시지 | 비고 |
|------------|------|-----------|--------|------|
| `@NotBlank` | 빈 문자열 검증 | `email`, `password` | "필수 입력값입니다." | null, 빈 문자열, 공백만 있는 문자열 모두 거부 |
| `@Email` | 이메일 형식 검증 | `email` | "올바른 이메일 형식이 아닙니다." | RFC 5322 표준 |
| `@Size(min=8)` | 문자열 길이 검증 | `password` | "비밀번호는 최소 8자 이상이어야 합니다." | 최소 길이 제한 |
| `@Size(max=100)` | 문자열 최대 길이 | `password` | "비밀번호는 100자 이하여야 합니다." | 최대 길이 제한 |
| `@NotNull` | null 값 검증 | 모든 필수 필드 | "필수 입력값입니다." | null 값만 거부 |

### 커스텀 검증 어노테이션 (향후 확장)

| 어노테이션 | 용도 | 적용 대상 | 구현 예정 |
|------------|------|-----------|-----------|
| `@ValidPassword` | 비밀번호 복잡도 검증 | `password` | Phase 2 |
| `@ValidEmail` | 이메일 도메인 제한 | `email` | Phase 2 |
| `@Sanitized` | XSS 방지 | 모든 String 필드 | Phase 2 |

---

## DTO 생성자 및 빌더 패턴

### 권장 구현 방식

| 패턴 | 사용 상황 | 예시 코드 |
|------|-----------|-----------|
| **기본 생성자** | JSON 직렬화/역직렬화 | `public LoginRequest() {}` |
| **모든 필드 생성자** | 테스트 코드 | `public LoginRequest(String email, String password)` |
| **빌더 패턴** | 복잡한 객체 생성 | `LoginRequest.builder().email("...").password("...").build()` |

### Lombok 어노테이션

| 어노테이션 | 효과 | 권장 사용 |
|------------|------|-----------|
| `@Data` | getter, setter, toString, equals, hashCode 자동 생성 | ✅ 권장 |
| `@Builder` | 빌더 패턴 자동 생성 | ✅ 권장 |
| `@NoArgsConstructor` | 기본 생성자 자동 생성 | ✅ 필수 |
| `@AllArgsConstructor` | 모든 필드 생성자 자동 생성 | ⚠️ 선택적 |

---

## 관련 문서
- [도메인 모델 상세 명세](./detailed_domain_models.md)
- [JWT 토큰 상세 명세](./detailed_jwt_token.md)
- [TypeScript 인터페이스](./typescript_interfaces.md) 