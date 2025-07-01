# Flow 001: 사용자 로그인 플로우

## 개요
사용자가 로그인 페이지에서 인증을 거쳐 메인 페이지에 진입하는 전체 플로우를 정의합니다.

---

## 전체 사용자 플로우

```mermaid
flowchart TD
    Start([사용자 접속]) --> LoginPage[로그인 페이지]
    LoginPage --> InputForm[이메일/비밀번호 입력]
    InputForm --> Validation{클라이언트 검증}
    
    Validation -->|실패| ShowError1[입력 오류 메시지 표시]
    ShowError1 --> InputForm
    
    Validation -->|성공| LoginAPI[POST /api/auth/login 호출]
    LoginAPI --> ServerAuth{서버 인증 처리}
    
    ServerAuth -->|실패| ShowError2[로그인 실패 메시지 표시]
    ShowError2 --> InputForm
    
    ServerAuth -->|성공| TokenReceived[JWT 토큰 수신]
    TokenReceived --> StoreToken[토큰 로컬 저장]
    StoreToken --> RedirectMain[메인 페이지로 리다이렉트]
    
    RedirectMain --> MainPage[메인 페이지 로드]
    MainPage --> MainAPI[GET /api/main 호출]
    MainAPI --> TokenCheck{토큰 유효성 검증}
    
    TokenCheck -->|실패| Logout[자동 로그아웃]
    Logout --> LoginPage
    
    TokenCheck -->|성공| UserData[사용자 데이터 수신]
    UserData --> RenderMain[메인 페이지 렌더링]
    RenderMain --> End([로그인 완료])

    style Start fill:#e1f5fe
    style End fill:#c8e6c9
    style ShowError1 fill:#ffcdd2
    style ShowError2 fill:#ffcdd2
    style Logout fill:#ffcdd2
```

---

## 세부 시퀀스 다이어그램

### 1. 로그인 인증 시퀀스

```mermaid
sequenceDiagram
    participant User as 사용자
    participant UI as 로그인 UI
    participant Client as 클라이언트
    participant API as 백엔드 API
    participant Auth as 인증 서비스
    participant DB as 데이터베이스
    
    User->>UI: 이메일/비밀번호 입력
    UI->>Client: 폼 데이터 검증
    
    alt 클라이언트 검증 실패
        Client-->>UI: 입력 오류 메시지
        UI-->>User: 오류 표시
    else 클라이언트 검증 성공
        Client->>API: POST /api/auth/login
        API->>Auth: 사용자 인증 요청
        Auth->>DB: 사용자 조회
        DB-->>Auth: 사용자 정보 반환
        
        alt 사용자 없음 또는 비밀번호 불일치
            Auth-->>API: 인증 실패
            API-->>Client: 401 Unauthorized
            Client-->>UI: 로그인 실패 메시지
            UI-->>User: 오류 표시
        else 인증 성공
            Auth->>Auth: JWT 토큰 생성
            Auth-->>API: JWT 토큰
            API-->>Client: 200 OK + JWT
            Client->>Client: 토큰 로컬 저장
            Client->>UI: 메인 페이지로 리다이렉트
        end
    end
```

### 2. 메인 페이지 접근 시퀀스

```mermaid
sequenceDiagram
    participant User as 사용자
    participant UI as 메인 UI
    participant Client as 클라이언트
    participant API as 백엔드 API
    participant Auth as 인증 서비스
    
    User->>UI: 메인 페이지 접근
    UI->>Client: 페이지 로드 요청
    Client->>Client: 저장된 토큰 확인
    
    alt 토큰 없음
        Client->>UI: 로그인 페이지로 리다이렉트
    else 토큰 있음
        Client->>API: GET /api/main + JWT 토큰
        API->>Auth: JWT 토큰 검증
        
        alt 토큰 유효하지 않음
            Auth-->>API: 토큰 검증 실패
            API-->>Client: 401 Unauthorized
            Client->>Client: 토큰 삭제
            Client->>UI: 로그인 페이지로 리다이렉트
        else 토큰 유효
            Auth-->>API: 사용자 정보
            API-->>Client: 200 OK + 사용자 데이터
            Client->>UI: 메인 페이지 렌더링
            UI-->>User: 환영 메시지 및 사용자 정보 표시
        end
    end
```

---

## 상태 다이어그램

```mermaid
stateDiagram-v2
    [*] --> Unauthenticated: 초기 상태
    
    Unauthenticated --> LoginForm: 로그인 페이지 접근
    LoginForm --> LoginForm: 입력 검증 실패
    LoginForm --> Authenticating: 로그인 API 호출
    
    Authenticating --> LoginForm: 인증 실패 (401)
    Authenticating --> Authenticated: 인증 성공 + JWT 저장
    
    Authenticated --> MainPage: 메인 페이지 접근
    MainPage --> MainPage: API 호출 성공
    MainPage --> Unauthenticated: 토큰 만료/무효
    
    Authenticated --> Unauthenticated: 로그아웃
    
    note right of Authenticated
        JWT 토큰이 로컬에 저장된 상태
        모든 API 호출 시 토큰 포함
    end note
    
    note right of MainPage
        사용자 데이터 표시
        인증이 필요한 기능 사용 가능
    end note
```

---

## 에러 처리 플로우

```mermaid
flowchart TD
    ApiCall[API 호출] --> Response{응답 상태}
    
    Response -->|200 OK| Success[성공 처리]
    Response -->|400 Bad Request| ValidationError[입력 검증 오류]
    Response -->|401 Unauthorized| AuthError[인증 오류]
    Response -->|500 Server Error| ServerError[서버 오류]
    Response -->|Network Error| NetworkError[네트워크 오류]
    
    ValidationError --> ShowFieldError[필드별 오류 메시지 표시]
    AuthError --> ClearToken[토큰 삭제]
    ClearToken --> RedirectLogin[로그인 페이지로 이동]
    ServerError --> ShowToast[일반 오류 토스트 표시]
    NetworkError --> ShowRetry[재시도 버튼 표시]
    
    ShowFieldError --> UserAction[사용자 수정 후 재시도]
    ShowToast --> UserAction
    ShowRetry --> UserAction
    UserAction --> ApiCall
    
    style Success fill:#c8e6c9
    style ValidationError fill:#fff3e0
    style AuthError fill:#ffcdd2
    style ServerError fill:#ffcdd2
    style NetworkError fill:#ffcdd2
```

---

## 플로우 체크포인트

### 성공 시나리오
1. ✅ 사용자가 올바른 이메일/비밀번호 입력
2. ✅ 클라이언트 측 검증 통과
3. ✅ 서버 인증 성공
4. ✅ JWT 토큰 발급 및 저장
5. ✅ 메인 페이지 리다이렉트
6. ✅ 토큰을 사용한 API 호출 성공
7. ✅ 사용자 데이터 표시

### 실패 시나리오
1. ❌ 잘못된 이메일 형식 → 클라이언트 검증 실패
2. ❌ 존재하지 않는 사용자 → 401 Unauthorized
3. ❌ 잘못된 비밀번호 → 401 Unauthorized  
4. ❌ 만료된 토큰 → 자동 로그아웃
5. ❌ 네트워크 오류 → 재시도 옵션 제공

---

## 관련 문서
- [API 엔드포인트 명세](../../01_architecture/02_da/api_endpoints.md)
- [백엔드 기술 아키텍처](../../01_architecture/01_ta/backend-architecture-kor.md)
- [Topic01 API 설계](../../00_roadmap/topic01_jwt-authentication/topic01-phase01-task01-api-design.md)
