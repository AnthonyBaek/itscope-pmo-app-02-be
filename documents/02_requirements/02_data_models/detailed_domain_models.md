# 상세 도메인 모델 명세서 (Domain Models)

## 개요
이 문서는 PMO 애플리케이션의 모든 도메인 모델을 표 형식으로 상세하게 정리합니다.

---

## 핵심 도메인 모델

### User

| 항목 | 상세 내용 |
|------|-----------|
| **클래스명** | `User` |
| **용도** | 사용자 정보 표현 |
| **사용처** | MainResponse의 user 필드, JWT 클레임 |
| **패키지** | `com.itscope.pmo.domain.user` |
| **테이블 매핑** | `users` (향후 확장 시) |

#### 필드 명세

| 필드명 | 타입 | 필수여부 | 설명 | 제약조건 | 예시 값 |
|--------|------|----------|------|----------|---------|
| `email` | String | Yes | 사용자 이메일 주소 | 유니크, RFC 5322 형식 | `test@test.com` |
| `name` | String | Yes | 사용자 이름 | 1~50자 | `테스트 사용자` |

#### JSON 표현

| 시나리오 | JSON |
|----------|------|
| **기본 사용자** | `{"email": "test@test.com", "name": "테스트 사용자"}` |
| **관리자 사용자** | `{"email": "admin@itscope.com", "name": "관리자"}` |
| **긴 이름 사용자** | `{"email": "long@test.com", "name": "매우 긴 이름을 가진 사용자"}` |

#### 비즈니스 규칙

| 규칙 | 설명 | 예외 상황 |
|------|------|-----------|
| **이메일 유니크** | 동일한 이메일로 중복 등록 불가 | 대소문자 구분 없음 |
| **이름 필수** | 사용자 이름은 반드시 입력 | 공백만으로 구성 불가 |
| **이메일 형식** | 유효한 이메일 형식만 허용 | 국제화 도메인 지원 |

#### Entity vs DTO 매핑

| Entity 필드 | DTO 필드 | 변환 규칙 | 비고 |
|-------------|----------|-----------|------|
| `email` | `email` | 직접 매핑 | 소문자 변환 |
| `name` | `name` | 직접 매핑 | Trim 처리 |
| `id` | - | DTO에서 제외 | 내부 식별자 |
| `password` | - | DTO에서 제외 | 보안 정보 |
| `createdAt` | - | DTO에서 제외 | 메타 정보 |
| `updatedAt` | - | DTO에서 제외 | 메타 정보 |

---

## 향후 확장 도메인 모델 (Phase 2)

### UserProfile

| 항목 | 상세 내용 |
|------|-----------|
| **클래스명** | `UserProfile` |
| **용도** | 사용자 상세 프로필 정보 |
| **사용처** | 프로필 조회/수정 API |
| **패키지** | `com.itscope.pmo.domain.user` |

#### 필드 명세 (계획)

| 필드명 | 타입 | 필수여부 | 설명 | 제약조건 | 예시 값 |
|--------|------|----------|------|----------|---------|
| `email` | String | Yes | 사용자 이메일 | User와 동일 | `test@test.com` |
| `name` | String | Yes | 사용자 이름 | User와 동일 | `테스트 사용자` |
| `role` | String | Yes | 사용자 역할 | ADMIN, USER | `USER` |
| `department` | String | No | 소속 부서 | 1~100자 | `개발팀` |
| `position` | String | No | 직급 | 1~50자 | `시니어 개발자` |
| `phoneNumber` | String | No | 전화번호 | 국제 형식 | `+82-10-1234-5678` |
| `createdAt` | LocalDateTime | Yes | 계정 생성일 | ISO 8601 | `2024-01-15T10:30:00Z` |
| `lastLoginAt` | LocalDateTime | No | 마지막 로그인 | ISO 8601 | `2024-01-15T10:30:00Z` |

### Role

| 항목 | 상세 내용 |
|------|-----------|
| **클래스명** | `Role` |
| **용도** | 사용자 권한 관리 |
| **사용처** | 권한 기반 접근 제어 |
| **패키지** | `com.itscope.pmo.domain.auth` |

#### 필드 명세 (계획)

| 필드명 | 타입 | 필수여부 | 설명 | 제약조건 | 예시 값 |
|--------|------|----------|------|----------|---------|
| `id` | Long | Yes | 역할 ID | 자동 증가 | `1` |
| `name` | String | Yes | 역할 이름 | 유니크, 대문자 | `ADMIN` |
| `description` | String | No | 역할 설명 | 1~200자 | `시스템 관리자` |
| `permissions` | Set<Permission> | No | 권한 목록 | - | `[READ_USER, WRITE_USER]` |

#### 기본 역할 정의

| 역할명 | 설명 | 권한 | 비고 |
|--------|------|------|------|
| **ADMIN** | 시스템 관리자 | 모든 권한 | 시스템 전체 관리 |
| **USER** | 일반 사용자 | 기본 권한 | 메인 페이지 접근 |
| **GUEST** | 게스트 사용자 | 읽기 권한만 | 로그인 전 상태 |

---

## 데이터 변환 매트릭스

### Entity → DTO 변환

| Source (Entity) | Target (DTO) | 변환 메서드 | 변환 규칙 |
|-----------------|--------------|-------------|-----------|
| `UserEntity` | `User` | `UserMapper.entityToDto()` | id, password 제외 |
| `UserEntity` | `UserProfileResponse` | `UserMapper.entityToProfileDto()` | 모든 필드 포함 |

### DTO → Entity 변환

| Source (DTO) | Target (Entity) | 변환 메서드 | 변환 규칙 |
|--------------|-----------------|-------------|-----------|
| `LoginRequest` | `UserEntity` | `UserMapper.loginToEntity()` | 인증용 임시 객체 |
| `User` | `UserEntity` | `UserMapper.dtoToEntity()` | 기본 필드만 |

### MapStruct 매퍼 인터페이스

| 매퍼 인터페이스 | 용도 | 주요 메서드 |
|-----------------|------|-------------|
| `UserMapper` | User 도메인 변환 | `entityToDto()`, `dtoToEntity()` |
| `AuthMapper` | 인증 관련 변환 | `loginRequestToUser()` |
| `ProfileMapper` | 프로필 관련 변환 | `entityToProfileDto()` |

---

## 도메인 비즈니스 로직

### User 도메인 메서드

| 메서드명 | 반환 타입 | 설명 | 비즈니스 규칙 |
|----------|-----------|------|----------------|
| `getDisplayName()` | String | 화면 표시용 이름 | 이름이 없으면 이메일 사용 |
| `isValidEmail()` | boolean | 이메일 유효성 검증 | RFC 5322 표준 |
| `maskEmail()` | String | 이메일 마스킹 | 개인정보 보호용 |
| `getDomainFromEmail()` | String | 이메일 도메인 추출 | @뒤의 도메인만 |

#### 예시 구현

| 메서드 | 입력 | 출력 | 비고 |
|--------|------|------|------|
| `getDisplayName()` | `name="테스트"` | `"테스트"` | 이름 우선 |
| `getDisplayName()` | `name=null, email="test@test.com"` | `"test@test.com"` | 이메일 대체 |
| `maskEmail()` | `"test@test.com"` | `"te***@test.com"` | 앞 2자리만 표시 |
| `getDomainFromEmail()` | `"test@itscope.com"` | `"itscope.com"` | 도메인만 추출 |

---

## 도메인 이벤트 (향후 확장)

### UserEvent

| 이벤트명 | 발생 시점 | 페이로드 | 처리 방식 |
|----------|-----------|----------|-----------|
| `UserRegistered` | 사용자 등록 시 | `User` 객체 | 환영 이메일 발송 |
| `UserLoggedIn` | 로그인 성공 시 | `User` 객체, 로그인 시간 | 로그인 이력 저장 |
| `UserProfileUpdated` | 프로필 수정 시 | 변경 전후 `User` 객체 | 변경 이력 로깅 |
| `UserDeactivated` | 계정 비활성화 시 | `User` 객체 | 관련 세션 정리 |

---

## 도메인 제약조건

### 비즈니스 무결성 규칙

| 제약조건 | 설명 | 구현 위치 | 예외 처리 |
|----------|------|-----------|-----------|
| **이메일 유니크** | 동일 이메일 중복 불가 | 데이터베이스 제약 | `UserAlreadyExistsException` |
| **이름 필수** | 사용자 이름 반드시 입력 | Entity 검증 | `InvalidUserDataException` |
| **이메일 형식** | 유효한 이메일만 허용 | DTO 검증 | `ValidationException` |

### 데이터 정합성 검증

| 검증 항목 | 규칙 | 구현 방법 | 실패 시 동작 |
|-----------|------|-----------|---------------|
| **중복 이메일** | 신규 사용자 등록 시 | DB 제약 + 서비스 검증 | 409 Conflict 응답 |
| **이메일 형식** | 모든 이메일 입력 시 | Bean Validation | 400 Bad Request 응답 |
| **이름 길이** | 이름 입력/수정 시 | Bean Validation | 400 Bad Request 응답 |

---

## 관련 문서
- [상세 DTO 명세](./detailed_dtos.md)
- [JWT 토큰 상세 명세](./detailed_jwt_token.md)
- [API 엔드포인트 명세](../../01_architecture/02_da/api_endpoints.md) 