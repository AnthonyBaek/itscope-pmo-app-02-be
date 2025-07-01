# 향상된 도메인 모델 설계 (Enhanced Domain Models)

## 개요
이 문서는 실제 PMO 애플리케이션에 적합한 현실적이고 복잡한 도메인 모델을 정의합니다.  
단순한 로그인용 User 모델을 넘어서 조직, 프로젝트, 권한 관리가 가능한 엔터프라이즈급 도메인 구조를 제안합니다.

---

## 핵심 도메인 엔티티

### User (사용자)

| 항목 | 상세 내용 |
|------|-----------|
| **테이블명** | `users` |
| **용도** | 시스템 사용자 정보 관리 |
| **관계** | 1:N (UserProfile), N:M (Project, Role) |
| **패키지** | `com.itscope.pmo.domain.user` |

#### 필드 명세

| 필드명 | 타입 | 제약조건 | 설명 | DB 컬럼 | 인덱스 |
|--------|------|----------|------|---------|--------|
| `id` | Long | PK, Auto Increment | 사용자 고유 ID | `user_id` | PRIMARY |
| `email` | String | UNIQUE, NOT NULL | 로그인 이메일 | `email` | UNIQUE |
| `password` | String | NOT NULL | 암호화된 비밀번호 | `password_hash` | - |
| `employeeId` | String | UNIQUE | 사번 | `employee_id` | UNIQUE |
| `status` | Enum | NOT NULL | 계정 상태 | `status` | INDEX |
| `createdAt` | LocalDateTime | NOT NULL | 계정 생성일 | `created_at` | INDEX |
| `updatedAt` | LocalDateTime | NOT NULL | 최종 수정일 | `updated_at` | - |
| `lastLoginAt` | LocalDateTime | NULL | 마지막 로그인 | `last_login_at` | INDEX |

#### 사용자 상태 (UserStatus)

| 상태값 | 설명 | 로그인 가능 | 비고 |
|--------|------|-------------|------|
| `ACTIVE` | 활성 사용자 | ✅ | 정상 사용 |
| `INACTIVE` | 비활성 사용자 | ❌ | 일시적 비활성 |
| `SUSPENDED` | 정지된 사용자 | ❌ | 관리자에 의한 정지 |
| `PENDING` | 승인 대기 | ❌ | 신규 가입 승인 대기 |
| `DELETED` | 삭제된 사용자 | ❌ | 소프트 삭제 |

### UserProfile (사용자 프로필)

| 항목 | 상세 내용 |
|------|-----------|
| **테이블명** | `user_profiles` |
| **용도** | 사용자 상세 프로필 정보 |
| **관계** | 1:1 (User), N:1 (Department, Position) |
| **패키지** | `com.itscope.pmo.domain.user` |

#### 필드 명세

| 필드명 | 타입 | 제약조건 | 설명 | DB 컬럼 | 예시 값 |
|--------|------|----------|------|---------|---------|
| `id` | Long | PK, Auto Increment | 프로필 ID | `profile_id` | - |
| `userId` | Long | FK, UNIQUE | 사용자 ID | `user_id` | - |
| `firstName` | String | NOT NULL | 이름 | `first_name` | `철수` |
| `lastName` | String | NOT NULL | 성 | `last_name` | `김` |
| `displayName` | String | NULL | 표시명 | `display_name` | `김철수` |
| `phoneNumber` | String | NULL | 전화번호 | `phone_number` | `010-1234-5678` |
| `birthDate` | LocalDate | NULL | 생년월일 | `birth_date` | `1990-01-15` |
| `hireDate` | LocalDate | NULL | 입사일 | `hire_date` | `2020-03-01` |
| `departmentId` | Long | FK | 부서 ID | `department_id` | - |
| `positionId` | Long | FK | 직급 ID | `position_id` | - |
| `profileImageUrl` | String | NULL | 프로필 이미지 URL | `profile_image_url` | `/images/profiles/user1.jpg` |

### Department (부서)

| 항목 | 상세 내용 |
|------|-----------|
| **테이블명** | `departments` |
| **용도** | 조직 부서 정보 관리 |
| **관계** | 1:N (UserProfile), 자기참조 (상위부서) |
| **패키지** | `com.itscope.pmo.domain.organization` |

#### 필드 명세

| 필드명 | 타입 | 제약조건 | 설명 | DB 컬럼 | 예시 값 |
|--------|------|----------|------|---------|---------|
| `id` | Long | PK, Auto Increment | 부서 ID | `department_id` | - |
| `code` | String | UNIQUE, NOT NULL | 부서 코드 | `dept_code` | `DEV001` |
| `name` | String | NOT NULL | 부서명 | `dept_name` | `개발팀` |
| `description` | String | NULL | 부서 설명 | `description` | `소프트웨어 개발 전담 부서` |
| `parentDepartmentId` | Long | FK, NULL | 상위 부서 ID | `parent_dept_id` | - |
| `managerId` | Long | FK, NULL | 부서장 사용자 ID | `manager_id` | - |
| `status` | Enum | NOT NULL | 부서 상태 | `status` | `ACTIVE` |
| `createdAt` | LocalDateTime | NOT NULL | 생성일 | `created_at` | - |

### Position (직급)

| 항목 | 상세 내용 |
|------|-----------|
| **테이블명** | `positions` |
| **용도** | 직급 정보 관리 |
| **관계** | 1:N (UserProfile) |
| **패키지** | `com.itscope.pmo.domain.organization` |

#### 필드 명세

| 필드명 | 타입 | 제약조건 | 설명 | DB 컬럼 | 예시 값 |
|--------|------|----------|------|---------|---------|
| `id` | Long | PK, Auto Increment | 직급 ID | `position_id` | - |
| `code` | String | UNIQUE, NOT NULL | 직급 코드 | `position_code` | `MGR` |
| `name` | String | NOT NULL | 직급명 | `position_name` | `매니저` |
| `level` | Integer | NOT NULL | 직급 레벨 | `position_level` | `3` |
| `description` | String | NULL | 직급 설명 | `description` | `팀 관리 책임자` |
| `status` | Enum | NOT NULL | 직급 상태 | `status` | `ACTIVE` |

---

## 권한 관리 도메인

### Role (역할)

| 항목 | 상세 내용 |
|------|-----------|
| **테이블명** | `roles` |
| **용도** | 시스템 역할 정의 |
| **관계** | N:M (User, Permission) |
| **패키지** | `com.itscope.pmo.domain.auth` |

#### 필드 명세

| 필드명 | 타입 | 제약조건 | 설명 | DB 컬럼 | 예시 값 |
|--------|------|----------|------|---------|---------|
| `id` | Long | PK, Auto Increment | 역할 ID | `role_id` | - |
| `code` | String | UNIQUE, NOT NULL | 역할 코드 | `role_code` | `PROJECT_MANAGER` |
| `name` | String | NOT NULL | 역할명 | `role_name` | `프로젝트 매니저` |
| `description` | String | NULL | 역할 설명 | `description` | `프로젝트 관리 권한` |
| `isSystemRole` | Boolean | NOT NULL | 시스템 기본 역할 여부 | `is_system_role` | `false` |
| `status` | Enum | NOT NULL | 역할 상태 | `status` | `ACTIVE` |

#### 기본 시스템 역할

| 역할 코드 | 역할명 | 설명 | 권한 범위 |
|-----------|--------|------|-----------|
| `SUPER_ADMIN` | 슈퍼 관리자 | 시스템 전체 관리 | 모든 권한 |
| `ADMIN` | 관리자 | 조직 및 사용자 관리 | 사용자/조직 관리 |
| `PMO_MANAGER` | PMO 매니저 | PMO 전체 관리 | 프로젝트 포트폴리오 관리 |
| `PROJECT_MANAGER` | 프로젝트 매니저 | 프로젝트 관리 | 담당 프로젝트 관리 |
| `TEAM_LEAD` | 팀 리더 | 팀 관리 | 팀 멤버 관리 |
| `DEVELOPER` | 개발자 | 개발 업무 | 프로젝트 참여 |
| `VIEWER` | 조회자 | 읽기 전용 | 조회 권한만 |

### Permission (권한)

| 항목 | 상세 내용 |
|------|-----------|
| **테이블명** | `permissions` |
| **용도** | 세부 권한 정의 |
| **관계** | N:M (Role) |
| **패키지** | `com.itscope.pmo.domain.auth` |

#### 필드 명세

| 필드명 | 타입 | 제약조건 | 설명 | DB 컬럼 | 예시 값 |
|--------|------|----------|------|---------|---------|
| `id` | Long | PK, Auto Increment | 권한 ID | `permission_id` | - |
| `code` | String | UNIQUE, NOT NULL | 권한 코드 | `permission_code` | `PROJECT_READ` |
| `name` | String | NOT NULL | 권한명 | `permission_name` | `프로젝트 조회` |
| `resource` | String | NOT NULL | 리소스 | `resource` | `PROJECT` |
| `action` | String | NOT NULL | 액션 | `action` | `READ` |
| `description` | String | NULL | 권한 설명 | `description` | `프로젝트 정보 조회 권한` |

#### 권한 체계

| 리소스 | 액션 | 권한 코드 | 설명 |
|--------|------|-----------|------|
| **USER** | CREATE | `USER_CREATE` | 사용자 생성 |
| **USER** | READ | `USER_READ` | 사용자 조회 |
| **USER** | UPDATE | `USER_UPDATE` | 사용자 수정 |
| **USER** | DELETE | `USER_DELETE` | 사용자 삭제 |
| **PROJECT** | CREATE | `PROJECT_CREATE` | 프로젝트 생성 |
| **PROJECT** | READ | `PROJECT_READ` | 프로젝트 조회 |
| **PROJECT** | UPDATE | `PROJECT_UPDATE` | 프로젝트 수정 |
| **PROJECT** | DELETE | `PROJECT_DELETE` | 프로젝트 삭제 |
| **TASK** | CREATE | `TASK_CREATE` | 태스크 생성 |
| **TASK** | READ | `TASK_READ` | 태스크 조회 |
| **TASK** | UPDATE | `TASK_UPDATE` | 태스크 수정 |
| **TASK** | DELETE | `TASK_DELETE` | 태스크 삭제 |

---

## 프로젝트 관리 도메인

### Project (프로젝트)

| 항목 | 상세 내용 |
|------|-----------|
| **테이블명** | `projects` |
| **용도** | 프로젝트 정보 관리 |
| **관계** | N:M (User), 1:N (Task) |
| **패키지** | `com.itscope.pmo.domain.project` |

#### 필드 명세

| 필드명 | 타입 | 제약조건 | 설명 | DB 컬럼 | 예시 값 |
|--------|------|----------|------|---------|---------|
| `id` | Long | PK, Auto Increment | 프로젝트 ID | `project_id` | - |
| `code` | String | UNIQUE, NOT NULL | 프로젝트 코드 | `project_code` | `PRJ2024001` |
| `name` | String | NOT NULL | 프로젝트명 | `project_name` | `PMO 시스템 개발` |
| `description` | Text | NULL | 프로젝트 설명 | `description` | `프로젝트 관리 시스템 구축` |
| `status` | Enum | NOT NULL | 프로젝트 상태 | `status` | `IN_PROGRESS` |
| `priority` | Enum | NOT NULL | 우선순위 | `priority` | `HIGH` |
| `startDate` | LocalDate | NULL | 시작일 | `start_date` | `2024-01-01` |
| `endDate` | LocalDate | NULL | 종료일 | `end_date` | `2024-12-31` |
| `budget` | BigDecimal | NULL | 예산 | `budget` | `1000000000` |
| `managerId` | Long | FK | 프로젝트 매니저 ID | `manager_id` | - |
| `departmentId` | Long | FK | 담당 부서 ID | `department_id` | - |

### ProjectMember (프로젝트 멤버)

| 항목 | 상세 내용 |
|------|-----------|
| **테이블명** | `project_members` |
| **용도** | 프로젝트 참여자 관리 |
| **관계** | N:1 (Project, User) |
| **패키지** | `com.itscope.pmo.domain.project` |

#### 필드 명세

| 필드명 | 타입 | 제약조건 | 설명 | DB 컬럼 | 예시 값 |
|--------|------|----------|------|---------|---------|
| `id` | Long | PK, Auto Increment | 멤버 ID | `member_id` | - |
| `projectId` | Long | FK, NOT NULL | 프로젝트 ID | `project_id` | - |
| `userId` | Long | FK, NOT NULL | 사용자 ID | `user_id` | - |
| `role` | Enum | NOT NULL | 프로젝트 내 역할 | `role` | `DEVELOPER` |
| `joinDate` | LocalDate | NOT NULL | 참여 시작일 | `join_date` | `2024-01-15` |
| `leaveDate` | LocalDate | NULL | 참여 종료일 | `leave_date` | `2024-06-30` |
| `allocation` | Integer | NOT NULL | 투입률 (%) | `allocation_percent` | `80` |
| `status` | Enum | NOT NULL | 참여 상태 | `status` | `ACTIVE` |

---

## 관계형 테이블 (Many-to-Many)

### user_roles (사용자-역할 매핑)

| 필드명 | 타입 | 제약조건 | 설명 |
|--------|------|----------|------|
| `user_id` | Long | FK, NOT NULL | 사용자 ID |
| `role_id` | Long | FK, NOT NULL | 역할 ID |
| `assigned_at` | LocalDateTime | NOT NULL | 할당일시 |
| `assigned_by` | Long | FK | 할당자 ID |

### role_permissions (역할-권한 매핑)

| 필드명 | 타입 | 제약조건 | 설명 |
|--------|------|----------|------|
| `role_id` | Long | FK, NOT NULL | 역할 ID |
| `permission_id` | Long | FK, NOT NULL | 권한 ID |
| `created_at` | LocalDateTime | NOT NULL | 생성일시 |

---

## 도메인 서비스

### 사용자 관리 서비스

| 서비스 | 메서드 | 설명 | 권한 필요 |
|--------|--------|------|-----------|
| `UserService` | `createUser()` | 사용자 생성 | `USER_CREATE` |
| `UserService` | `updateUserProfile()` | 프로필 수정 | `USER_UPDATE` |
| `UserService` | `deactivateUser()` | 사용자 비활성화 | `USER_DELETE` |
| `UserService` | `findByDepartment()` | 부서별 사용자 조회 | `USER_READ` |

### 권한 관리 서비스

| 서비스 | 메서드 | 설명 | 권한 필요 |
|--------|--------|------|-----------|
| `AuthorizationService` | `assignRole()` | 역할 할당 | `ROLE_MANAGE` |
| `AuthorizationService` | `checkPermission()` | 권한 검증 | - |
| `AuthorizationService` | `getUserPermissions()` | 사용자 권한 조회 | `USER_READ` |

### 프로젝트 관리 서비스

| 서비스 | 메서드 | 설명 | 권한 필요 |
|--------|--------|------|-----------|
| `ProjectService` | `createProject()` | 프로젝트 생성 | `PROJECT_CREATE` |
| `ProjectService` | `addProjectMember()` | 멤버 추가 | `PROJECT_UPDATE` |
| `ProjectService` | `getProjectsByManager()` | 매니저별 프로젝트 조회 | `PROJECT_READ` |

---

## 데이터베이스 인덱스 전략

### 성능 최적화 인덱스

| 테이블 | 인덱스 | 용도 | 타입 |
|--------|--------|------|------|
| `users` | `idx_users_email` | 로그인 조회 | UNIQUE |
| `users` | `idx_users_employee_id` | 사번 조회 | UNIQUE |
| `users` | `idx_users_status_created` | 상태별 생성일 조회 | COMPOSITE |
| `user_profiles` | `idx_profiles_dept_position` | 부서+직급 조회 | COMPOSITE |
| `projects` | `idx_projects_manager_status` | 매니저별 상태 조회 | COMPOSITE |
| `project_members` | `idx_members_project_user` | 프로젝트-사용자 조회 | COMPOSITE |

---

## 확장 고려사항

### Phase 2 확장 계획

| 도메인 | 확장 내용 | 우선순위 |
|--------|-----------|----------|
| **감사 로그** | 모든 변경 이력 추적 | 높음 |
| **첨부파일** | 프로젝트 문서 관리 | 중간 |
| **알림** | 실시간 알림 시스템 | 중간 |
| **대시보드** | 개인별 맞춤 대시보드 | 낮음 |
| **리포트** | 프로젝트 진행 리포트 | 낮음 |

### 성능 고려사항

| 항목 | 고려사항 | 해결방안 |
|------|----------|-----------|
| **사용자 권한 조회** | 복잡한 JOIN 쿼리 | 권한 캐싱 |
| **프로젝트 목록** | 대용량 데이터 | 페이징 + 인덱스 |
| **조직도 조회** | 재귀 쿼리 | 계층 경로 캐싱 |

---

## 관련 문서
- [기본 도메인 모델](./detailed_domain_models.md)
- [상세 DTO 명세](./detailed_dtos.md)
- [JWT 토큰 상세 명세](./detailed_jwt_token.md)
- [API 엔드포인트 명세](../../01_architecture/02_da/api_endpoints.md) 