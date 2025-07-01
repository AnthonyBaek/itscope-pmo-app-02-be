# Topic01: JWT Authentication

## Topic 개요
- 목적: 사전 등록된 사용자만 로그인 가능, JWT 기반 인증으로 보안 강화
- 범위: 로그인, 토큰 발급, 인증된 사용자만 접근 가능한 메인 페이지
- 기술 스택: Spring Boot, Spring Security, JWT, React, Axios

## Phase/Task 구조
- Phase 01: 기본 인증 및 메인 진입
  - Task 01: API 명세 설계 (api-design)
  - Task 02: Spring Boot 프로젝트 초기화 (spring-initializr)
  - Task 03: 프론트엔드 로그인 UI 개발 (fe-login-ui)
  - Task 04: JWT 인증 가드 구현 (jwt-guard)
- Phase 02: 인증 고도화 및 보안 강화
  - Task 01: 비밀번호 변경 기능 (password-change)
  - Task 02: 인증 실패/예외 처리 강화 (auth-error-handling)

각 Task는 B/E, F/E 관점의 체크리스트와 테스트 분석, 통합 검증 방법을 포함합니다. 