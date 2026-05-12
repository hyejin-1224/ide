# CDE (Credit Decision Engine) 아키텍처 문서

신용 의사결정 엔진의 정보 구조(IA), 사용자 흐름(User Flow), 시스템 흐름(System Flow)을 정의한 문서입니다.

## 📋 목차

- [정보 구조 (Information Architecture)](#정보-구조-information-architecture)
- [사용자 흐름 (User Flows)](#사용자-흐름-user-flows)
- [시스템 흐름 (System Flows)](#시스템-흐름-system-flows)
- [문서 관리](#문서-관리)

---

## 정보 구조 (Information Architecture)

전체 시스템의 메뉴 구조와 권한별 접근 제어를 정의합니다.

📁 [Information Architecture](./architecture/information-architecture.md)

### 주요 메뉴
- 🔐 인증 (Login/Authentication)
- 📊 대시보드 (Dashboard)
- 📁 프로젝트 (Project)
  - Parameter (원본변수 + 파생변수)
  - Segment (세그먼트 매트릭스)
  - Scoring (Score Card / AI Model)
  - Rule (의사결정 규칙)
  - Policy (최종 정책)
  - Active History (운영 이력)
- 🧪 테스트 & 시뮬레이션
  - Policy Test (A/B 테스트)
  - Simulation (과거 데이터 분석)
- 🤖 AI Model (BentoML 연동)
- 💾 Data Store (외부 DB 연동)
- 👥 Management (Master 전용)
  - User 관리
  - Usage 통계
- 🔔 알림 (Notification)

### 권한별 접근 제어

| 메뉴 | Master | Manager | User |
|------|--------|---------|------|
| Dashboard | ✅ | ✅ | ✅ |
| Project | ✅ | ✅ (Read/Update) | ✅ (CRUD) |
| Test & Simulation | ✅ | ✅ | ✅ |
| AI Model | ✅ | ✅ | ✅ |
| Data Store | ✅ | ✅ | ✅ |
| Management | ✅ | ❌ | ❌ |
| Policy Approval | - | ✅ (승인/거절) | ✅ (요청) |

---

## 사용자 흐름 (User Flows)

각 권한별 사용자의 시스템 사용 흐름을 정의합니다.

### 📁 [User Flows 디렉토리](./architecture/user-flows/)

#### 1. [Master 사용자 플로우](./architecture/user-flows/01-master-flow.md)
- 전체 메뉴 접근 권한
- 사용자 생성/수정/삭제 (Master 전용)
- 프로젝트 관리 (CRUD)
- 폴리시 승인 요청 및 배포
- Usage 통계 조회 (Master 전용)

**주요 프로세스:**
- 사용자 생성 → 난수 비밀번호 Email 발송
- 폴리시 승인 요청 → Manager 선택 → 배포

#### 2. [Manager 사용자 플로우](./architecture/user-flows/02-manager-flow.md)
- 폴리시 결재 권한 (핵심 역할)
- 승인/거절 결정
- 승인 후 관리 (Revoke/Terminate/Reschedule)
- 사후 승인 처리 (롤백)
- 프로젝트 조회 (Read/Update)

**주요 프로세스:**
- 알림 수신 (Requested/Withdrawn)
- 폴리시 검토 → 승인/거절 → User 알림 발송
- 배포 옵션 선택 (Active/Scheduled)
- 롤백 사후 승인

#### 3. [User 사용자 플로우](./architecture/user-flows/03-user-flow.md)
- 프로젝트 전체 관리 (CRUD)
- Parameter/Segment/Scoring/Rule/Policy 생성
- 폴리시 승인 요청
- 테스트 & 분석
- 알림 수신 (Approved/Rejected/Revoked)

**주요 프로세스:**
- 첫 로그인 → 비밀번호 변경 필수
- 프로젝트 선택 → 구성 요소 생성 → 폴리시 완성
- 승인 요청 → Manager 선택 → 결과 대기
- Policy Test / Simulation 실행

---

## 시스템 흐름 (System Flows)

백엔드 시스템의 데이터 처리 흐름과 아키텍처를 정의합니다.

### 📁 [System Flows 디렉토리](./architecture/system-flows/)

#### 1. [전체 시스템 아키텍처](./architecture/system-flows/01-overall-architecture.md)
5개 레이어 구조:
- **인증/권한 레이어**: Authentication & Authorization
- **애플리케이션 레이어**: Project/Test/Management Module
- **데이터 처리 레이어**: Parameter/Decision Engine, Approval Workflow
- **통합 레이어**: Data Store, AI Engine, Notification Service
- **데이터 저장소**: Database, Cache, Logging

**실시간 의사결정 흐름:**
```
요청 수신 → Parameter 매핑 → 파생변수 계산 
→ Active Policy 실행 → Decision 응답
```

#### 2. [파라미터 처리 플로우](./architecture/system-flows/02-parameter-processing.md)
**원본변수 생성 프로세스:**
- 6가지 입력 소스: 직접입력/File/URL/Model/Project/Query
- 유효성 검증 → 중복 체크 → 저장 → 매핑

**파생변수 생성 프로세스:**
- 기반 변수 선택 → Condition 설정 → Result 정의
- Default 설정 → 테스트 → 저장

**In Use 상태 관리:**
- 수정 불가 (이름 제외)
- 복제만 가능

#### 3. [폴리시 결재 플로우](./architecture/system-flows/03-policy-approval.md)
**10단계 상태:**
1. Pending → 2. Requested → 3. Withdrawn
4. Approved → 5. Rejected → 6. Revoked
7. Scheduled → 8. Active → 9. Terminated → 10. Canceled

**주요 프로세스:**
- User 승인 요청 → Manager 검토 → 승인/거절
- 배포 옵션 (Active/Scheduled)
- 버저닝 및 Active History 기록
- 롤백 및 사후 승인

#### 4. [실시간 의사결정 플로우](./architecture/system-flows/04-realtime-decision.md)
**3가지 Policy 타입:**
1. **Rules Only**: Rule Priority 순 평가 → Else Decision
2. **Matrix Only**: Segment 매칭 → Scoring → Matrix Decision
3. **Rule + Matrix**: Rule 평가 → Approve만 Matrix 진행

**응답 구조:**
```json
{
  "transaction_id": "...",
  "decision": "Approve|Reject|Pending",
  "attributes": { ... },
  "segment": "...",
  "grade": "...",
  "score": 850
}
```

---

## 문서 관리

### 버전 관리
- 모든 다이어그램은 Mermaid 형식
- Git으로 버전 관리
- 변경 이력 추적 가능

### 다이어그램 렌더링
- GitHub/GitLab에서 자동 렌더링
- VS Code Mermaid 플러그인 사용 권장
- [Mermaid Live Editor](https://mermaid.live/) 활용 가능

### 기여 가이드
1. 변경 사항 발생 시 해당 `.md` 파일 수정
2. Mermaid 문법 확인
3. 커밋 및 Pull Request

---

## 📚 추가 리소스

### 정책 문서
- 로그인/회원 정책
- 권한 정책
- 리스트 페이지 정책
- 파라미터 정책
- 폴리시 결재 정책
- Data Store 정책

### 관련 링크
- [Mermaid 문법 가이드](https://mermaid.js.org/)
- [GitHub Mermaid 지원](https://github.blog/2022-02-14-include-diagrams-markdown-files-mermaid/)

---

## 📞 문의

문서 관련 문의 사항은 개발팀에 연락해주세요.

---

**Last Updated**: 2026-05-12

**Version**: 1.0.0

**Authors**: CDE Development Team
