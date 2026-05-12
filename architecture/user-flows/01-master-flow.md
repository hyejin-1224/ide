# Master 사용자 플로우

Master 권한 사용자의 시스템 사용 흐름도

```mermaid
flowchart TD
    Start([로그인]) --> Auth{인증 성공?}
    Auth -->|실패| LoginFail[로그인 실패<br/>5회 실패 시 30분 잠금]
    LoginFail --> Start
    
    Auth -->|성공| Session[세션 생성<br/>30분 유효]
    Session --> Dashboard[📊 대시보드<br/>통계 및 주요 지표 확인]
    
    Dashboard --> MenuSelect{메뉴 선택}
    
    %% Project Management Branch
    MenuSelect -->|프로젝트 관리| ProjectMgmt[📁 프로젝트 관리]
    ProjectMgmt --> ProjectSelect[프로젝트 선택/생성]
    ProjectSelect --> Components{구성 요소}
    
    Components -->|Parameter| ParamMgmt[원본/파생변수 생성<br/>Import/직접입력]
    ParamMgmt --> ParamMapping[Parameter Mapping]
    
    Components -->|Segment| SegmentMgmt[세그먼트 생성<br/>매트릭스 구성]
    
    Components -->|Scoring| ScoringMgmt[스코어링 생성<br/>Score Card/AI Model]
    ScoringMgmt --> ModelMapping[Model Mapping]
    
    Components -->|Rule| RuleMgmt[룰 생성<br/>Condition/Decision]
    
    Components -->|Policy| PolicyMgmt[폴리시 생성<br/>Rules/Matrix 조합]
    
    ParamMapping --> PolicyFlow
    SegmentMgmt --> PolicyFlow
    ModelMapping --> PolicyFlow
    RuleMgmt --> PolicyFlow
    PolicyMgmt --> PolicyFlow[폴리시 완성]
    
    %% Policy Approval Flow
    PolicyFlow --> ApprovalReq[승인 요청<br/>Manager 선택]
    ApprovalReq --> Requested[Requested<br/>승인 대기]
    
    Requested -->|결재 취소| Withdrawn[Withdrawn<br/>수정 가능]
    Withdrawn --> ApprovalReq
    
    Requested --> ManagerReview[Manager 검토]
    ManagerReview -->|거절| Rejected[Rejected<br/>수정 후 재요청]
    Rejected --> PolicyFlow
    
    ManagerReview -->|승인| Approved[Approved<br/>배포 가능]
    
    Approved -->|승인 취소| Revoked[Revoked<br/>수정 필요]
    Revoked --> PolicyFlow
    
    Approved --> DeployOption{배포 옵션}
    DeployOption -->|즉시| Active[Active<br/>운영 중]
    DeployOption -->|예약| Scheduled[Scheduled<br/>운영 예약]
    
    Scheduled -->|예약 시간 도래| Active
    Active --> Versioning[버저닝<br/>Active History 기록]
    
    Active -->|운영 중단| Terminated[Terminated<br/>운영 종료]
    
    Versioning --> Monitoring[모니터링<br/>Active History 조회]
    Monitoring -->|롤백| RollbackFlow[롤백 실행<br/>사후 승인]
    
    %% User Management Branch
    MenuSelect -->|사용자 관리| UserMgmt[👥 사용자 관리<br/>Master 전용]
    UserMgmt --> UserActions{관리 작업}
    
    UserActions -->|생성| CreateUser[사용자 생성<br/>ID/Name/Email/Role]
    CreateUser --> SendPw[난수 비밀번호 생성<br/>Email 발송]
    
    UserActions -->|수정| EditUser[권한 변경<br/>Manager/User]
    
    UserActions -->|삭제| DeleteUser[사용자 삭제<br/>90일 보관]
    
    UserActions -->|PW 초기화| ResetPw[비밀번호 초기화<br/>Email 발송]
    
    SendPw --> UserList[사용자 리스트 조회]
    EditUser --> UserList
    DeleteUser --> UserList
    ResetPw --> UserList
    
    %% Management Branch
    MenuSelect -->|Management| MgmtMenu[📊 Management<br/>Master 전용]
    MgmtMenu --> UsageStats{사용량 통계}
    
    UsageStats --> TxStats[트랜잭션 통계<br/>전날 23:59:59까지]
    UsageStats --> MonthStats[이번 달 누적 사용량]
    UsageStats --> YearStats[연간 누적 사용량]
    
    %% Additional Features
    MenuSelect -->|Test & Simulation| TestMenu[🧪 테스트 & 분석]
    TestMenu --> PolicyTest[Policy Test<br/>챔피언/챌린저]
    TestMenu --> Simulation[Simulation<br/>과거 데이터 분석]
    
    MenuSelect -->|AI Model| AIModelMenu[🤖 AI Model<br/>BentoML 연동]
    
    MenuSelect -->|Data Store| DataStoreMenu[💾 Data Store<br/>외부 DB 연동]
    
    %% Notification
    Dashboard -.알림.-> Notification[🔔 알림<br/>결재 상태]
    Notification --> Dashboard
    
    %% Logout
    UserList --> Dashboard
    TxStats --> Dashboard
    MonthStats --> Dashboard
    YearStats --> Dashboard
    PolicyTest --> Dashboard
    Simulation --> Dashboard
    AIModelMenu --> Dashboard
    DataStoreMenu --> Dashboard
    Monitoring --> Dashboard
    Terminated --> Dashboard
    
    Dashboard -->|로그아웃| Logout([세션 종료])
    
    %% Styling
    classDef authStyle fill:#f9f,stroke:#333,stroke-width:2px
    classDef processStyle fill:#bbf,stroke:#333,stroke-width:2px
    classDef decisionStyle fill:#ffa,stroke:#333,stroke-width:2px
    classDef successStyle fill:#bfb,stroke:#333,stroke-width:2px
    classDef errorStyle fill:#fbb,stroke:#333,stroke-width:2px
    classDef mgmtStyle fill:#fcb,stroke:#333,stroke-width:2px
    
    class Start,Logout authStyle
    class Dashboard,ProjectMgmt,UserMgmt,MgmtMenu processStyle
    class Auth,MenuSelect,Components,UserActions,UsageStats,DeployOption decisionStyle
    class Active,Approved,Versioning successStyle
    class LoginFail,Rejected,Terminated errorStyle
    class SendPw,ResetPw,CreateUser mgmtStyle
```

## Master 사용자 주요 권한

### 1. 전체 메뉴 접근
- Dashboard, Project, Test & Simulation, AI Model, Data Store
- **Management 메뉴 독점 접근**

### 2. 프로젝트 관리
- Parameter, Segment, Scoring, Rule, Policy 생성/수정/삭제
- 폴리시 승인 요청 및 배포
- 운영 이력 조회 및 롤백

### 3. 사용자 관리 (Master 전용)
- **사용자 생성**: ID/Name/Email/Role 지정
- **권한 할당**: Manager/User 권한 부여
- **비밀번호 관리**: 초기화 및 난수 생성
- **사용자 삭제**: 90일 보관 후 완전 삭제

### 4. 통계 조회 (Master 전용)
- 트랜잭션 통계 (전날까지)
- 월간/연간 누적 사용량

### 5. 폴리시 결재
- 승인 요청 → Manager 선택
- 결재 취소 (Withdrawn)
- 승인 후 배포 (Active/Scheduled)
- 운영 중단 권한

## 주요 프로세스

### 사용자 생성 프로세스
1. ID/Name/Email 입력
2. Role 선택 (Manager/User)
3. 시스템이 난수 비밀번호 생성
4. Email로 비밀번호 발송
5. 사용자 첫 로그인 시 비밀번호 변경 필수

### 폴리시 배포 프로세스
1. 폴리시 생성 (Rules/Matrix 조합)
2. 승인 요청 (Manager 선택)
3. Manager 승인
4. 배포 옵션 선택:
   - **Active**: 즉시 운영
   - **Scheduled**: 예약 운영
5. 버저닝 및 Active History 기록
6. 모니터링 및 롤백 가능

### 알림 수신
- 폴리시 결재 상태 변경 시 알림
- Requested, Withdrawn, Approved, Rejected, Revoked, Scheduled, Canceled
- 알림 14일 보관

## 특이사항

### 동시 접속 제한
- 동일 아이디로 동시 접속 불가
- 신규 로그인 시 기존 세션 로그아웃

### 세션 관리
- 세션 타임아웃: 30분
- 자동 로그인 기능 없음

### 로그인 보안
- 로그인 실패 5회 시 30분 계정 잠금
- 30분 후 자동으로 잠금 해제
- 비밀번호 틀린 횟수 초기화
