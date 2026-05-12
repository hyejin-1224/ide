# Manager 사용자 플로우

Manager 권한 사용자의 시스템 사용 흐름도 - 결재 승인 중심

```mermaid
flowchart TD
    Start([로그인]) --> Auth{인증 성공?}
    Auth -->|실패| LoginFail[로그인 실패<br/>5회 실패 시 30분 잠금]
    LoginFail --> Start
    
    Auth -->|성공| Session[세션 생성<br/>30분 유효<br/>Manager 권한 확인]
    Session --> Dashboard[📊 대시보드<br/>결재 대기 건 확인]
    
    Dashboard --> CheckNoti[🔔 알림 확인]
    CheckNoti --> NotiType{알림 유형}
    
    NotiType -->|Requested| ReqNoti[승인 요청 알림<br/>User가 제출]
    NotiType -->|Withdrawn| WithNoti[결재 취소 알림<br/>User가 취소]
    NotiType -->|Rollback| RollNoti[롤백 사후 승인 알림<br/>승인 필요]
    
    ReqNoti --> ReviewPolicy
    WithNoti --> Dashboard
    RollNoti --> PostApproval[사후 승인 모달<br/>취소/삭제 불가]
    
    PostApproval --> PostComment[코멘트 작성]
    PostComment --> PostApprove[승인 완료]
    PostApprove --> Dashboard
    
    %% Policy Review Flow
    ReviewPolicy[폴리시 검토] --> ViewDetail[상세 내용 확인<br/>Rules/Matrix/Settings]
    ViewDetail --> ReviewDecision{결재 결정}
    
    ReviewDecision -->|승인| ApproveFlow[Approve 처리]
    ReviewDecision -->|거절| RejectFlow[Reject 처리]
    
    %% Reject Flow
    RejectFlow --> RejectComment[거절 사유 작성<br/>코멘트 필수]
    RejectComment --> RejectSubmit[거절 처리]
    RejectSubmit --> SendRejectNoti[User에게 알림 발송<br/>Rejected 상태]
    SendRejectNoti --> RejectComplete[User 수정 대기]
    RejectComplete --> Dashboard
    
    %% Approve Flow
    ApproveFlow --> ApproveComment[승인 코멘트 작성]
    ApproveComment --> ApproveSubmit[승인 처리]
    ApproveSubmit --> SendApproveNoti[User에게 알림 발송<br/>Approved 상태]
    SendApproveNoti --> ApproveComplete[배포 가능 상태]
    
    %% Post-Approval Actions
    ApproveComplete --> PostActions{승인 후 작업}
    
    PostActions -->|배포| DeployOption{배포 옵션}
    DeployOption -->|즉시 운영| ActiveDeploy[Active 처리<br/>즉시 배포]
    DeployOption -->|예약 운영| ScheduleDeploy[Scheduled 처리<br/>예약 배포]
    
    ActiveDeploy --> Versioning[버저닝<br/>타임스탬프 기록]
    ScheduleDeploy --> WaitSchedule[예약 시간 대기]
    WaitSchedule --> Versioning
    
    Versioning --> MonitorActive[Active History<br/>운영 이력 조회]
    
    PostActions -->|승인 취소| RevokeAction[Revoke 실행]
    RevokeAction --> RevokeComment[취소 사유 작성]
    RevokeComment --> RevokeSubmit[승인 취소 처리]
    RevokeSubmit --> SendRevokeNoti[User에게 알림 발송<br/>Revoked 상태]
    SendRevokeNoti --> RevokeComplete[User 수정 필요]
    RevokeComplete --> Dashboard
    
    PostActions -->|운영 중단| TerminateAction[Terminate 실행]
    TerminateAction --> TermComment[중단 사유 작성]
    TermComment --> TermSubmit[운영 중단 처리]
    TermSubmit --> TermComplete[Canceled/Terminated 상태]
    TermComplete --> Dashboard
    
    PostActions -->|일정 변경| RescheduleAction[Reschedule 실행<br/>Scheduled 상태만 가능]
    RescheduleAction --> NewSchedule[새 예약 시간 설정]
    NewSchedule --> WaitSchedule
    
    %% Monitoring and Management
    MonitorActive --> MonitorActions{모니터링 작업}
    
    MonitorActions -->|이력 조회| ViewHistory[과거 운영 이력<br/>버전별 확인]
    ViewHistory --> Dashboard
    
    MonitorActions -->|계속 운영| ContinueOp[계속 모니터링]
    ContinueOp --> Dashboard
    
    %% Other Menu Access
    Dashboard --> OtherMenu{기타 메뉴}
    
    OtherMenu -->|프로젝트| ProjectView[프로젝트 조회<br/>Read/Update 권한]
    ProjectView --> ViewDetails[상세 내용 확인<br/>수정 가능]
    ViewDetails --> Dashboard
    
    OtherMenu -->|Test & Sim| TestMenu[Test & Simulation<br/>조회 및 실행]
    TestMenu --> Dashboard
    
    OtherMenu -->|AI Model| AIModelMenu[AI Model<br/>조회 및 설정]
    AIModelMenu --> Dashboard
    
    OtherMenu -->|Data Store| DataStoreMenu[Data Store<br/>조회]
    DataStoreMenu --> Dashboard
    
    %% Logout
    Dashboard -->|로그아웃| Logout([세션 종료])
    
    %% Styling
    classDef authStyle fill:#f9f,stroke:#333,stroke-width:2px
    classDef processStyle fill:#bbf,stroke:#333,stroke-width:2px
    classDef decisionStyle fill:#ffa,stroke:#333,stroke-width:2px
    classDef successStyle fill:#bfb,stroke:#333,stroke-width:2px
    classDef errorStyle fill:#fbb,stroke:#333,stroke-width:2px
    classDef notiStyle fill:#fcf,stroke:#333,stroke-width:2px
    classDef actionStyle fill:#cef,stroke:#333,stroke-width:2px
    
    class Start,Logout authStyle
    class Dashboard,ReviewPolicy,MonitorActive processStyle
    class Auth,NotiType,ReviewDecision,PostActions,MonitorActions,DeployOption,OtherMenu decisionStyle
    class ApproveComplete,ActiveDeploy,Versioning successStyle
    class LoginFail,RejectComplete,TermComplete errorStyle
    class CheckNoti,ReqNoti,WithNoti,RollNoti,SendApproveNoti,SendRejectNoti,SendRevokeNoti notiStyle
    class RevokeAction,TerminateAction,RescheduleAction actionStyle
```

## Manager 사용자 주요 권한

### 1. 폴리시 결재 권한 (핵심 역할)
- **승인 요청 검토**: User가 제출한 폴리시 검토
- **승인/거절 결정**: Approve 또는 Reject 처리
- **코멘트 작성**: 결재 사유 필수 작성
- **알림 수신**: Requested, Withdrawn 상태 알림

### 2. 승인 후 관리 권한
- **승인 취소 (Revoke)**: 승인한 폴리시 취소
- **운영 중단 (Terminate)**: 운영 중인 폴리시 중단
- **일정 변경 (Reschedule)**: 예약된 폴리시 일정 수정
- **배포 실행**: Active 또는 Scheduled 처리

### 3. 사후 승인 처리
- 롤백 시 사후 승인 모달 강제 출력
- 모달 취소/삭제 불가
- 코멘트 작성 및 승인만 가능

### 4. 프로젝트 조회 권한
- 모든 프로젝트 Read 권한
- 일부 Update 권한 (이름 변경 등)
- CRUD 권한 없음 (생성/삭제 불가)

### 5. 기타 메뉴 접근
- Test & Simulation 조회 및 실행
- AI Model 조회
- Data Store 조회
- **Management 메뉴 접근 불가**

## 주요 프로세스

### 승인 프로세스
1. Dashboard에서 결재 대기 건 확인
2. 알림 클릭 또는 직접 조회
3. 폴리시 상세 내용 확인
   - Rules 구성
   - Matrix 설정
   - Segment/Scoring
4. 결재 결정:
   - **승인**: 코멘트 작성 → Approve → User 알림
   - **거절**: 사유 작성 → Reject → User 알림
5. User는 알림 수신 후 조치

### 거절 프로세스
1. 거절 사유 코멘트 작성 (필수)
2. Reject 처리
3. User에게 알림 발송
4. User는 폴리시 수정 후 재요청 가능

### 배포 프로세스 (승인 후)
1. Approved 상태 폴리시 선택
2. 배포 옵션 선택:
   - **Active**: 즉시 운영 시작
   - **Scheduled**: 예약 시간 설정
3. 버저닝 (타임스탬프 기록)
4. Active History에 기록
5. 모니터링

### 승인 취소 (Revoke) 프로세스
1. Approved 상태 폴리시 선택
2. Revoke 버튼 클릭
3. 취소 사유 코멘트 작성
4. Revoke 처리
5. User에게 알림 발송
6. User는 수정 후 재요청 가능

### 운영 중단 (Terminate) 프로세스
1. Active 또는 Scheduled 상태 폴리시 선택
2. Terminate 버튼 클릭
3. 중단 사유 코멘트 작성
4. Terminate 처리
5. Canceled/Terminated 상태로 변경

### 사후 승인 프로세스 (롤백)
1. User가 과거 폴리시로 롤백 실행
2. 즉시 배포 (승인 전)
3. Manager 로그인 시 사후 승인 모달 강제 출력
4. 코멘트 작성
5. 승인 완료 (모달 취소 불가)

## 알림 시스템

### Manager가 받는 알림
- **Requested**: User가 승인 요청
- **Withdrawn**: User가 결재 취소
- **Rollback**: 롤백 사후 승인 필요

### Manager가 보내는 알림 (User에게)
- **Approved**: 승인 완료
- **Rejected**: 거절
- **Revoked**: 승인 취소

### 알림 보관 정책
- 14일 보관
- 자동 삭제

## 특이사항

### 결재 권한
- 한 번에 한 명의 Manager만 지정 가능
- 승인권자로 지정된 Manager만 알림 수신
- 다른 Manager는 조회만 가능

### 코멘트 작성
- 승인/거절/취소 시 코멘트 작성 필수
- 코멘트는 User에게 전달
- 이력으로 기록

### 모니터링
- Active History에서 운영 이력 조회
- 버전별 폴리시 내용 확인
- 롤백 내역 확인

### Management 메뉴 제한
- User 생성/삭제 불가
- 사용량 통계 조회 불가
- Master 전용 기능
