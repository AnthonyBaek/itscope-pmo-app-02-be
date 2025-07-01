# Topic01 Phase01 Task02: 프로젝트 초기화 (BE + FE 프로젝트 세팅)

## 개요
- 목적: 인증 시스템 개발을 위한 Spring Boot(백엔드)와 React(프론트엔드) 프로젝트 기본 구조 및 환경 세팅
- 범위: 백엔드 Spring Boot 프로젝트 생성, 프론트엔드 React + TypeScript + Vite 프로젝트 생성, 개발 환경 구축

<!-- > 체크리스트 ID는 topic-번호, phase-번호, task-번호, BE/FE/INT, 순번으로 구성됩니다. (예: 01-01-02-BE-01)
> 상세화 시 하위 체크리스트는 01-01-02-BE-01-01, 01-01-02-BE-01-02 등으로 추가합니다. -->

## 체크리스트

### 백엔드(B/E)
1. [ ] **01-01-02-BE-01**: Spring Initializr에서 프로젝트 생성 (Gradle/Maven, Java 17+)
2. [ ] **01-01-02-BE-02**: 필수 의존성(JPA, Security, Web, Validation, PostgreSQL, JWT 등) 추가
3. [ ] **01-01-02-BE-03**: 기본 패키지 구조 설계 및 생성
4. [ ] **01-01-02-BE-04**: application.yml 기본 환경설정 작성
5. [ ] **01-01-02-BE-05**: 프로젝트 빌드 및 실행 확인

### 프론트엔드(F/E)
1. [ ] **01-01-02-FE-01**: React + TypeScript + Vite 프로젝트 생성
   - [ ] **01-01-02-FE-01-01**: `npm create vite@latest itscope-pmo-fe -- --template react-ts` 실행
   - [ ] **01-01-02-FE-01-02**: 프로젝트 생성 확인 및 기본 실행 테스트
   - [ ] **01-01-02-FE-01-03**: Git 초기화 및 .gitignore 설정

2. [ ] **01-01-02-FE-02**: 필수 의존성 설치
   - [ ] **01-01-02-FE-02-01**: **라우팅 & 상태관리**: `react-router-dom`, `zustand`
   - [ ] **01-01-02-FE-02-02**: **서버 상태관리**: `@tanstack/react-query`, `axios`
   - [ ] **01-01-02-FE-02-03**: **폼 & 검증**: `react-hook-form`, `zod`, `@hookform/resolvers`
   - [ ] **01-01-02-FE-02-04**: **UI 컴포넌트**: `@radix-ui/react-*` (shadcn/ui 의존성)
   - [ ] **01-01-02-FE-02-05**: **스타일링**: `tailwindcss`, `autoprefixer`, `postcss`
   - [ ] **01-01-02-FE-02-06**: **토스트 알림**: `sonner`
   - [ ] **01-01-02-FE-02-07**: **개발도구**: `@types/node`

3. [ ] **01-01-02-FE-03**: 테스팅 환경 설정
   - [ ] **01-01-02-FE-03-01**: **단위 테스트**: `vitest`, `@testing-library/react`, `jsdom`
   - [ ] **01-01-02-FE-03-02**: **E2E 테스트**: `@playwright/test` 설치
   - [ ] **01-01-02-FE-03-03**: **테스트 설정 파일** 생성 (vitest.config.ts, playwright.config.ts)

4. [ ] **01-01-02-FE-04**: 개발 도구 설정
   - [ ] **01-01-02-FE-04-01**: **ESLint 설정**: TypeScript, React 규칙 추가
   - [ ] **01-01-02-FE-04-02**: **Prettier 설정**: .prettierrc 생성 및 규칙 정의
   - [ ] **01-01-02-FE-04-03**: **TypeScript 설정**: tsconfig.json 최적화 (path aliases 포함)
   - [ ] **01-01-02-FE-04-04**: **VS Code 설정**: .vscode/settings.json 생성

5. [ ] **01-01-02-FE-05**: 프로젝트 구조 설계 및 폴더 생성
   - [ ] **01-01-02-FE-05-01**: **타입 폴더**: `src/types/` (api.ts, auth.ts, user.ts, common.ts)
   - [ ] **01-01-02-FE-05-02**: **스키마 폴더**: `src/schemas/` (validation schemas)
   - [ ] **01-01-02-FE-05-03**: **상태 폴더**: `src/stores/` (Zustand stores)
   - [ ] **01-01-02-FE-05-04**: **훅 폴더**: `src/hooks/` (custom hooks)
   - [ ] **01-01-02-FE-05-05**: **서비스 폴더**: `src/services/` (API clients)
   - [ ] **01-01-02-FE-05-06**: **컴포넌트 폴더**: `src/components/` (UI components)
   - [ ] **01-01-02-FE-05-07**: **페이지 폴더**: `src/pages/` (route pages)

6. [ ] **01-01-02-FE-06**: Tailwind CSS 및 shadcn/ui 초기 설정
   - [ ] **01-01-02-FE-06-01**: **Tailwind CSS 설정**: tailwind.config.js, postcss.config.js 생성
   - [ ] **01-01-02-FE-06-02**: **shadcn/ui 초기화**: `npx shadcn-ui@latest init`
   - [ ] **01-01-02-FE-06-03**: **기본 컴포넌트 설치**: Button, Input, Form, Card, Toast
   - [ ] **01-01-02-FE-06-04**: **글로벌 CSS 설정**: globals.css에 Tailwind directives 추가

7. [ ] **01-01-02-FE-07**: 환경 변수 및 설정 파일 생성
   - [ ] **01-01-02-FE-07-01**: **.env 파일들** 생성 (.env.local, .env.production)
   - [ ] **01-01-02-FE-07-02**: **API 환경변수** 정의 (VITE_API_BASE_URL 등)
   - [ ] **01-01-02-FE-07-03**: **환경별 설정** config 파일 생성
   - [ ] **01-01-02-FE-07-04**: **타입 안전한 환경변수** 접근 방법 구현

8. [ ] **01-01-02-FE-08**: 전역 프로바이더 설정
   - [ ] **01-01-02-FE-08-01**: **TanStack Query 프로바이더** 설정 (QueryClient, QueryClientProvider)
   - [ ] **01-01-02-FE-08-02**: **React Router 설정** (BrowserRouter, 기본 라우트)
   - [ ] **01-01-02-FE-08-03**: **Sonner Toast 프로바이더** 설정
   - [ ] **01-01-02-FE-08-04**: **App.tsx 전역 설정** 통합

9. [ ] **01-01-02-FE-09**: 프로젝트 빌드 및 실행 확인
   - [ ] **01-01-02-FE-09-01**: **개발 서버 실행** (`npm run dev`) 확인
   - [ ] **01-01-02-FE-09-02**: **프로덕션 빌드** (`npm run build`) 성공 확인
   - [ ] **01-01-02-FE-09-03**: **린터 & 타입 체크** (`npm run lint`, `tsc --noEmit`) 통과 확인
   - [ ] **01-01-02-FE-09-04**: **테스트 실행** (`npm run test`) 기본 테스트 통과 확인

10. [ ] **01-01-02-FE-10**: 백엔드 연동 환경 준비
    - [ ] **01-01-02-FE-10-01**: **API 클라이언트 기본 구조** 생성 (axios 인스턴스)
    - [ ] **01-01-02-FE-10-02**: **CORS 정책 확인** 및 프록시 설정 (vite.config.ts)
    - [ ] **01-01-02-FE-10-03**: **백엔드 API 명세서 확인** 및 타입 정의 준비
    - [ ] **01-01-02-FE-10-04**: **개발/운영 환경별 API URL** 설정 검증

## 테스트 분석

### 백엔드 단독 테스트
- **01-01-02-BE-T01**: 프로젝트가 정상적으로 빌드/실행되는지 확인
- **01-01-02-BE-T02**: 의존성 누락/충돌 여부 점검

### 프론트엔드 단독 테스트
- **01-01-02-FE-T01**: **프로젝트 생성 및 기본 실행** 테스트 (npm run dev 정상 동작)
- **01-01-02-FE-T02**: **의존성 설치 및 충돌** 확인 (package.json, lock 파일 검증)
- **01-01-02-FE-T03**: **개발 도구 설정** 동작 확인 (ESLint, Prettier, TypeScript)
- **01-01-02-FE-T04**: **빌드 및 테스트** 실행 확인 (build, lint, test 명령어)
- **01-01-02-FE-T05**: **shadcn/ui 컴포넌트** 렌더링 테스트 (기본 컴포넌트 설치 확인)
- **01-01-02-FE-T06**: **환경 변수 및 설정** 적용 확인 (API URL, 환경별 config)
- **01-01-02-FE-T07**: **전역 프로바이더** 설정 확인 (TanStack Query, Router, Toast)

### 통합(BE+FE) 테스트
- **01-01-02-INT-T01**: **BE/FE 환경변수** 및 API 연동 설정이 일치하는지 검증
- **01-01-02-INT-T02**: **CORS 정책** 확인 (프론트엔드에서 백엔드 API 호출 가능 여부)
- **01-01-02-INT-T03**: **개발 환경 동시 실행** 확인 (BE: 8080, FE: 5173 포트 충돌 없음)

## 결과 검증 방법

### 백엔드 검증
- **프로젝트 실행 스크린샷**: Spring Boot 애플리케이션 정상 시작 로그
- **환경설정 파일 캡처**: application.yml, 의존성 목록 (build.gradle/pom.xml)
- **API 엔드포인트 확인**: Swagger UI 또는 기본 Health Check 엔드포인트 접근

### 프론트엔드 검증
- **프로젝트 실행 스크린샷**: `npm run dev` 실행 후 브라우저 화면 (Vite + React 기본 페이지)
- **의존성 확인**: package.json 파일 및 설치된 패키지 목록 캡처
- **빌드 성공 로그**: `npm run build` 명령어 실행 결과 및 dist 폴더 생성 확인
- **개발 도구 동작**: ESLint, Prettier 실행 결과 스크린샷
- **테스트 실행 결과**: `npm run test` 기본 테스트 통과 확인
- **shadcn/ui 컴포넌트**: 설치된 기본 컴포넌트 렌더링 스크린샷
- **환경 변수 설정**: .env 파일들 및 config 설정 캡처

### 통합 검증
- **BE/FE 동시 실행**: 백엔드(8080), 프론트엔드(5173) 포트에서 정상 실행 확인
- **CORS 테스트**: 프론트엔드에서 백엔드 API 호출 가능 여부 확인
- **환경변수 일치성**: API URL 등 설정이 BE/FE 간 일치하는지 문서화
- **개발 환경 구축 완료**: 두 프로젝트가 모두 정상 동작하는 통합 스크린샷 