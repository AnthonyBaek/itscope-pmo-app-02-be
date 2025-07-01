# DTO 정의서 (Data Transfer Objects)

## 개요
이 문서는 PMO 애플리케이션의 API에서 사용되는 모든 데이터 전송 객체(DTO)를 정의합니다.

---

## 요청 DTO (Request DTOs)

### LoginRequest
**용도**: 사용자 로그인 요청  
**API**: `POST /api/auth/login`

```java
public class LoginRequest {
    @NotBlank(message = "이메일은 필수 입력값입니다.")
    @Email(message = "올바른 이메일 형식이 아닙니다.")
    private String email;
    
    @NotBlank(message = "비밀번호는 필수 입력값입니다.")
    @Size(min = 8, message = "비밀번호는 최소 8자 이상이어야 합니다.")
    private String password;
    
    // getters, setters, constructors...
}
```

**JSON 예시:**
```json
{
  "email": "test@test.com",
  "password": "password123"
}
```

**검증 규칙:**
- `email`: 필수값, RFC 5322 이메일 형식
- `password`: 필수값, 최소 8자 이상

---

## 응답 DTO (Response DTOs)

### LoginResponse  
**용도**: 로그인 성공 응답  
**API**: `POST /api/auth/login` (성공 시)

```java
public class LoginResponse {
    private String accessToken;
    private String tokenType;
    private Integer expiresIn;
    
    // getters, setters, constructors...
}
```

**JSON 예시:**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "tokenType": "Bearer",
  "expiresIn": 3600
}
```

**필드 설명:**
- `accessToken`: JWT 액세스 토큰 (String)
- `tokenType`: 토큰 타입, 항상 "Bearer" (String)
- `expiresIn`: 토큰 만료 시간 (초 단위, Integer)

### MainResponse
**용도**: 메인 페이지 데이터 응답  
**API**: `GET /api/main` (성공 시)

```java
public class MainResponse {
    private String message;
    private User user;
    
    // getters, setters, constructors...
}
```

**JSON 예시:**
```json
{
  "message": "안녕하세요, test@test.com 님!",
  "user": {
    "email": "test@test.com",
    "name": "테스트 사용자"
  }
}
```

**필드 설명:**
- `message`: 환영 메시지 (String)  
- `user`: 사용자 정보 객체 (User)

### ErrorResponse
**용도**: 에러 응답 표준화  
**API**: 모든 API (에러 발생 시)

```java
public class ErrorResponse {
    private String message;
    private String timestamp;
    
    // getters, setters, constructors...
}
```

**JSON 예시:**
```json
{
  "message": "이메일 또는 비밀번호가 일치하지 않습니다.",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

**필드 설명:**
- `message`: 에러 메시지 (String)
- `timestamp`: 에러 발생 시간, ISO 8601 형식 (String)

**HTTP 상태코드별 사용:**
- `400 Bad Request`: 입력 검증 실패
- `401 Unauthorized`: 인증 실패
- `500 Internal Server Error`: 서버 내부 오류

---

## 도메인 모델 (Domain Models)

### User
**용도**: 사용자 정보 표현  
**사용처**: MainResponse의 user 필드

```java
public class User {
    private String email;
    private String name;
    
    // getters, setters, constructors...
}
```

**JSON 예시:**
```json
{
  "email": "test@test.com",
  "name": "테스트 사용자"
}
```

**필드 설명:**
- `email`: 사용자 이메일 주소 (String)
- `name`: 사용자 이름 (String)

---

## JWT 토큰 구조

### JWT Header
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### JWT Payload
```java
public class JwtPayload {
    private String sub;        // Subject (사용자 이메일)
    private Long iat;          // Issued At (발급 시간)
    private Long exp;          // Expiration (만료 시간)
    private String email;      // 사용자 이메일
    
    // getters, setters, constructors...
}
```

**JSON 예시:**
```json
{
  "sub": "user_email@test.com",
  "iat": 1642248000,
  "exp": 1642251600,
  "email": "user_email@test.com"
}
```

**필드 설명:**
- `sub`: 토큰 주체 (사용자 이메일)
- `iat`: 토큰 발급 시간 (Unix timestamp)
- `exp`: 토큰 만료 시간 (Unix timestamp)  
- `email`: 사용자 이메일 (커스텀 클레임)

---

## 데이터 검증 어노테이션

### Bean Validation 사용
```java
// 이메일 검증
@NotBlank(message = "이메일은 필수 입력값입니다.")
@Email(message = "올바른 이메일 형식이 아닙니다.")
private String email;

// 비밀번호 검증  
@NotBlank(message = "비밀번호는 필수 입력값입니다.")
@Size(min = 8, max = 100, message = "비밀번호는 8자 이상 100자 이하여야 합니다.")
private String password;

// 필수값 검증
@NotNull(message = "필수 입력값입니다.")
private String requiredField;
```

### 커스텀 검증 어노테이션 (향후 확장)
```java
// 향후 필요 시 구현
@ValidPassword  // 비밀번호 복잡도 검증
@ValidEmail     // 이메일 도메인 제한 검증
```

---

## DTO 변환 가이드

### Entity ↔ DTO 변환
```java
// User Entity → User DTO 변환 예시
public static User toDto(UserEntity entity) {
    return User.builder()
        .email(entity.getEmail())
        .name(entity.getName())
        .build();
}

// LoginRequest → 인증 처리용 객체 변환 예시  
public static AuthenticationRequest toAuthRequest(LoginRequest loginRequest) {
    return AuthenticationRequest.builder()
        .email(loginRequest.getEmail())
        .password(loginRequest.getPassword())
        .build();
}
```

### MapStruct 사용 (권장)
```java
@Mapper(componentModel = "spring")
public interface UserMapper {
    User entityToDto(UserEntity entity);
    UserEntity dtoToEntity(User dto);
}
```

---

## 향후 확장 DTO

### Phase 2 추가 예정
```java
// 토큰 갱신 요청
public class RefreshTokenRequest {
    private String refreshToken;
}

// 토큰 갱신 응답
public class RefreshTokenResponse {
    private String accessToken;
    private String refreshToken; 
    private Integer expiresIn;
}

// 비밀번호 변경 요청
public class ChangePasswordRequest {
    private String currentPassword;
    private String newPassword;
    private String confirmPassword;
}

// 사용자 프로필 응답
public class UserProfileResponse {
    private String email;
    private String name;
    private String role;
    private LocalDateTime createdAt;
    private LocalDateTime lastLoginAt;
}
```

---

## 관련 문서
- [API 엔드포인트 명세](../../01_architecture/02_da/api_endpoints.md)
- [사용자 로그인 플로우](../01_user_flow/flow_001_signin.md)
- [API 설계 체크리스트](../../00_roadmap/topic01_jwt-authentication/topic01-phase01-task01-api-design.md)

---

**이 문서는 백엔드 개발 시 DTO 클래스 작성과 프론트엔드 TypeScript 인터페이스 작성의 기준이 됩니다.** 