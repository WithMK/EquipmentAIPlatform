# Equipment AI Platform — Ecosystem Architecture
> 버전 v0.2 · 2026-11-05 · 설계 방향 합의, 실제 API/배포 세부사항 미확정
>
> 이 파일은 EquipmentAIPlatform · DatasetHub · EquipmentAIStudio 간 **공통 아키텍처 기준 문서(SSOT)**입니다. 상세 스냅샷은 개인 Library `/설비제어SW/Equipment_AI_Platform_Integrated_Architecture_v0.1.md`에 보관합니다.

## 목적과 책임

1. **DatasetHub (Data Collecting PC, WPF/Desktop/Background)**: 설비 PC의 로그·검사 이미지를 SMB 경로로 주기적 PULL; 실패 복구/이력 관리; 로그 구조화(DB 등록), 이미지 인덱싱 및 LOT/ProductID/검사 실행 단위 매칭. **매칭은 SAIGE와 무관**한 설비 생산 데이터 처리 기능.
2. **EquipmentAIStudio (독립 웹 저장소)**: 여러 설비의 LOT→Product 계층형 Inspection DataGrid(기간/설비/검사 결과 검색), 제품 클릭 후 고해상도 이미지 Viewer·비교·개발자 검토/재분류, Dataset/AI Model 버전·성능·이력 관리, SAIGE 실행 연결. **DatasetHub 상태 조회와 권한 기반 제한적 수동 재수집 요청도 제공**. 단, 수집·매칭 엔진을 웹에서 재구현하지 않음.
3. **Equipment AI Platform Web (상위 범용 포털)**: AI Agent/RAG와 다양한 설비용 웹의 통합 진입점, 공통 메뉴/인증/라우팅. EquipmentAIStudio는 먼저 독립 개발하고 이후 `/studio/`에 연결 가능하도록 설계.
4. **SAIGE AI Tool**: 학습용 PC에서 담당자 1명이 직접 실행하는 독립 학습 프로그램. 라벨링/학습은 SAIGE 내부에서 수행. DatasetHub나 웹이 이를 재구현하지 않음.

## 흐름

```text
[설비 9대 / 제어 + 검사 PC 총 27대 예시]
         | 원본 .log / 이미지 / 검사 메타데이터
         v   SMB 주기적 PULL
[DatasetHub Desktop/Background]
  Collection -> Processing -> Matching (DB)
         |                           |
         +--> 중앙 원본 파일 저장소   +--> 제품/검사/이미지 메타데이터
                                     |
                                  조회 API
                                     v
[EquipmentAIStudio Web] <--- 상위 포털 /studio 메뉴·라우팅 ---> [Equipment AI Platform Web]
  조회 / Dataset 선택 / Model Registry                         Agent / RAG / 통합 UI
         |
         +--> [학습용 PC SAIGE: 로컬 별도 프로그램 직접 학습]
```

## 운영과 검사 분석의 역할 분리

- **별도 설비 관제·모니터링**: 실시간 가동률, 알람, 생산 상태. EquipmentAIStudio와 기능 중복 금지.
- **DatasetHub Desktop (필수)**: 설비/PC 등록·설정, SMB 공유폴더 연결 검사, 수집 모니터링·장애 이력, 복구 및 수집 제어의 최종 권한. 웹·AI 플랫폼 장애와 무관하게 계속 수집.
- **EquipmentAIStudio DatasetHub Management (원격 보조)**: Hub 프로세스/Heartbeat, PC별 수집 성공·실패·지연·누락, 장애·복구 상태 조회. 검증·인가된 수동 수집/재시도 요청에 한정; 수집 경로·설비 직접 동작·프로세스 강제 종료는 초기 제외.
- Hub 자체가 죽을 때는 동일 Hub API의 생존 응답을 받을 수 없으므로 독립적인 웹 또는 다른 PC에서 상태 확인. Hub와 웹이 동일 호스트에서 함께 종료되는 상황에는 별도 외부 감시 필요.
- 원본 검사 판정은 유지하고 개발자 검토/분류와 변경 이력은 별도 보관.

## 장기 AI 분석 연계

EquipmentAIStudio의 선택 Product/Inspection 맥락을 EquipmentAIPlatform Agent와 EquipmentRAG·Source Code/Git·SW/Recipe/Model 버전에 연결한다. 자연어 검색은 검증된 읽기 전용 조회 Tool을 거쳐 기존 DataGrid에 결과를 표시한다. 이미지 유사도 검색은 별도 Vision 임베딩 서비스가 필요하다.

## 데이터 계약

- 데이터 원본은 설비 PC에 보관, DatasetHub는 읽기 위주 접근. 장애가 설비 제어 동작에 영향을 주지 않아야 함.
- 이미지 권장 원본 경로 `D:\Log\Image\YYYYMMDD\...`. 기존 수집 경로 및 파일명은 보존.
- 로그(표준 11필드), 이미지 파일, 검사 메타데이터를 별도로 인덱싱한 후 MachineId/LOT/ProductID/InspectionRunId/검사 위치 등 합의한 식별자로 연결. 시간만으로 무조건 매칭하지 않으며 미매칭·충돌 기록 필요.
- 이미지 바이트는 파일 저장소에 보관, DB는 식별자·관계·경로·검사 결과·이력 저장.
- 웹은 DB 파일 공유 직접 접근 대신 읽기 API로 조회. 이미지 제공도 권한 관리된 서버 경로/API 사용.
- 학습 Dataset Export는 원본 보존. SAIGE 실제 import/export 계약은 추후 확인.

## 독립 웹 개발 → 단일 외부 접속점 통합

개발 초기에는 AI Platform Web과 EquipmentAIStudio를 각기 다른 포트에서 실행할 수 있다(예시: 8000, 8100).
추후 Reverse Proxy(Nginx 또는 YARP)로 **하나의 외부 주소·포트**만 공개하고 경로를 구분한다.
예: `http://server:8080/ai/` → 범용 웹, `http://server:8080/studio/` → EquipmentAIStudio.
포트·경로는 예시이며 확정 설정이 아님. 내부 서버 프로세스는 별도 유지할 수 있다.
EquipmentAIStudio은 base path, 정적 에셋, 라우팅, API endpoint, 인증을 설정 가능하게 설계한다. 프록시가 공통 레이아웃/인증을 자동 제공하지는 않는다.

## SAIGE 실행 제약

웹 브라우저는 학습용 PC의 EXE를 임의 실행하지 못한다. 등록된 Custom URI Protocol 또는 허가된 Local Launcher 연계를 검토한다. 임의 명령 실행, 원격 무인 학습 실행은 범위에 포함하지 않는다.

## 상시 실행 정책

DatasetHub는 현재 Desktop 상시 실행을 전제로 한다. 장기 운영 중 로그아웃/앱 종료가 수집 중단으로 이어지는 문제를 피하려면 Collector/Processing/Matching 엔진을 향후 Windows Worker Service로 분리하고 WPF는 운영 UI로 유지하는 것을 검토한다. 엔진의 서비스 계약은 UI 독립적으로 유지한다.

## 저장소 간 동기화

- **EquipmentAIPlatform**: 이 파일을 기준으로 관리하고 버전/결정 변경 이력을 기록.
- **DatasetHub**: `docs/ECOSYSTEM_INTEGRATION.md`에서 이 문서를 링크하며 Collector/Matching 소유 범위를 준수.
- **EquipmentAIStudio**: `README.md`, `docs/ARCHITECTURE.md`, `docs/DEVELOPMENT_ROADMAP.md`에 화면/서비스 세부 설계를 유지하고 이 기준 문서를 참조.
- **개인 Library**: 동일한 합의사항의 상세 설계 스냅샷을 별도로 보관하며 Git 커밋만으로 자동 동기화되지는 않음.

## 미결정

EquipmentAIStudio 기술 스택, 검사 실행 ID 및 메타데이터 계약, 웹 조회 API, DB 확장 여부, 인증/SSO, SAIGE Import·런처 방식, 배포/운영 포트. 미결정 사항은 확정 전 임의 구현하지 않는다.

## 구현 순서

DatasetHub 수집 안정화와 EquipmentAIStudio Mock 프론트엔드 선행 개발을 병행 → 로그 DB 등록/이미지 인덱싱/제품 매칭 → 조회·관리 API 연계 → Studio DataGrid/Image Review·Dataset·Model 관리 고도화 → 상위 Platform 웹 통합. Windows Worker 분리는 선택적 안정화 과제.
