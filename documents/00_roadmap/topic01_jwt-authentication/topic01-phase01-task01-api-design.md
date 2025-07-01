# Topic01 Phase01 Task01: API 명세 설계 (api-design)

## 개요
- 목적: 인증/로그인/메인 진입에 필요한 API 명세 설계 및 문서화

> 체크리스트 ID는 topic-번호, phase-번호, task-번호, BE/FE/INT, 순번으로 구성됩니다. (예: 01-01-01-BE-01)
> 상세화 시 하위 체크리스트는 01-01-01-BE-01-01, 01-01-01-BE-01-02 등으로 추가합니다.

---

## API Specification

### 1. 로그인 API
```
POST /api/auth/login
Content-Type: application/json

Request Body:
{
  "email": "test@test.com",
  "password": "password123"
}

Success Response (200 OK):
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "tokenType": "Bearer",
  "expiresIn": 3600
}

Error Response (401 Unauthorized):
{
  "message": "이메일 또는 비밀번호가 일치하지 않습니다.",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

### 2. 메인 페이지 데이터 API
```
GET /api/main
Authorization: Bearer {accessToken}

Success Response (200 OK):
{
  "message": "안녕하세요, test@test.com 님!",
  "user": {
    "email": "test@test.com",
    "name": "테스트 사용자"
  }
}

Error Response (401 Unauthorized):
{
  "message": "인증이 필요합니다.",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

---

## 테스트 시나리오
1. 정상 로그인: test@test.com / password123 → JWT 토큰 발급
2. 잘못된 비밀번호: test@test.com / wrong → 401 에러
3. 존재하지 않는 이메일: none@test.com / password123 → 401 에러
4. 토큰 없이 메인 접근: Authorization 헤더 없음 → 401 에러
5. 유효한 토큰으로 메인 접근: 올바른 JWT → 사용자 정보 반환

---

## 체크리스트

### 백엔드(B/E)
1. [ ] **01-01-01-BE-01**: 요구사항 분석 및 인증 관련 엔드포인트 목록 도출
2. [ ] **01-01-01-BE-02**: 요청/응답 스키마 정의 (예시 JSON 포함)
3. [ ] **01-01-01-BE-03**: 에러/예외 응답 정책 수립
4. [ ] **01-01-01-BE-04**: Swagger/OpenAPI 문서 초안 작성
5. [ ] **01-01-01-BE-05**: API 명세서 팀 내 리뷰 및 피드백 반영

### 프론트엔드(F/E)
1. [ ] **01-01-01-FE-01**: 화면/기능 흐름에 맞는 API 호출 시나리오 정리
2. [ ] **01-01-01-FE-02**: Mock 데이터/Mock 서버 준비
3. [ ] **01-01-01-FE-03**: API 명세서 기반 프론트엔드 통신 모듈 설계
4. [ ] **01-01-01-FE-04**: API 명세서 리뷰 및 BE와 상호 검증

---

## 테스트 분석

### 백엔드 단독 테스트
- **01-01-01-BE-T01**: Postman 등으로 API 명세대로 요청/응답 검증
- **01-01-01-BE-T02**: 에러/예외 상황별 응답 검증

### 프론트엔드 단독 테스트
- **01-01-01-FE-T01**: Mock API로 화면 연동 및 예외 처리 동작 확인

### 통합(BE+FE) 테스트
- **01-01-01-INT-T01**: 실제 API 연동 후, 화면에서 정상/에러 응답 처리 확인
- **01-01-01-INT-T02**: E2E 시나리오(로그인 → 메인 진입) 전체 흐름 검증

## 결과 검증 방법
- API 명세서와 실제 동작이 일치하는지 양팀이 상호 리뷰
- 통합 테스트 결과를 스크린샷/캡처 등으로 기록 