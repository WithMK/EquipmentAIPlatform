# Equipment AI Platform — Ecosystem Architecture
> 버전 v0.1 · 2026-11-03 · 설계 방향 합의, 실제 API/배포 세부사항 미확정
>
> 이 파일은 EquipmentAIPlatform · DatasetHub · 향후 Trimming AI Web 간 **공통 아키텍처 기준 문서(SSOT)**입니다. 상세 스냅샷은 개인 Library `/설비제어SW/Equipment_AI_Platform_Integrated_Architecture_v0.1.md`에 보관합니다.

## 목적과 책임

1. **DatasetHub (Data Collecting PC, WPF/Desktop/Background)**: 설비 PC의 로그·검사 이미지를 SMB 경로로 주기적 PULL; 실패 복구/이력 관리; 로그 구조화(DB 등록), 이미지 인덱싱 및 LOT/ProductID/검사 실행 단위 매칭. **매칭은 SAIGE와 무관**한 설비 생산 데이터 처리 기능.
2. **Trimming AI Web (신규 별도 저장소 예정)**: 매칭된 데이터 조회·검색, 제품/검사 이미지 Viewer, 학습 데이터셋 선택/내보내기, 모델 목록·버전·성능·파일 경로 관리, 학습용 PC SAIGE 실행 연결. **수집 및 매칭을 웹에서 다시 구현하지 않음**.
3. **Equipment AI Platform Web (상위 범용 포털)**: AI Agent/RAG와 다양한 설비용 웹의 통합 진입점, 공통 메뉴/인증/라우팅. 트리밍 웹은 먼저 독립 개발하고 이후 `/trimming/`에 연결 가능하도록 설계.
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
[Trimming AI Web] <--- 상위 포털 /trimming 메뉴·라우팅 ---> [Equipment AI Platform Web]
  조회 / Dataset 선택 / Model Registry                         Agent / RAG / 통합 UI
         |
         +--> [학습용 PC SAIGE: 로컬 별도 프로그램 직접 학습]
```

## 데이터 계약

- 데이터 원본은 설비 PC에 보관, DatasetHub는 읽기 위주 접근. 장애가 설비 제어 동작에 영향을 주지 않아야 함.
- 이미지 권장 원본 경로 `D:\Log\Image\YYYYMMDD\...`. 기존 수집 경로 및 파일명은 보존.
- 로그(표준 11필드), 이미지 파일, 검사 메타데이터를 별도로 인덱싱한 후 MachineId/LOT/ProductID/InspectionRunId/검사 위치 등 합의한 식별자로 연결. 시간만으로 무조건 매칭하지 않으며 미매칭·충돌 기록 필요.
- 이미지 바이트는 파일 저장소에 보관, DB는 식별자·관계·경로·검사 결과·이력 저장.
- 웹은 DB 파일 공유 직접 접근 대신 읽기 API로 조회. 이미지 제공도 권한 관리된 서버 경로/API 사용.
- 학습 Dataset Export는 원본 보존. SAIGE 실제 import/export 계약은 추후 확인.

## 독립 웹 개발 → 단일 외부 접속점 통합

개발 초기에는 AI Platform Web과 Trimming AI Web을 각기 다른 포트에서 실행할 수 있다(예시: 8000, 8100).
추후 Reverse Proxy(Nginx 또는 YARP)로 **하나의 외부 주소·포트**만 공개하고 경로를 구분한다.
예: `http://server:8080/ai/` → 범용 웹, `http://server:8080/trimming/` → 트리밍 웹.
포트·경로는 예시이며 확정 설정이 아님. 내부 서버 프로세스는 별도 유지할 수 있다.
트리밍 웹은 base path, 정적 에셋, 라우팅, API endpoint, 인증을 설정 가능하게 설계한다. 프록시가 공통 레이아웃/인증을 자동 제공하지는 않는다.

## SAIGE 실행 제약

웹 브라우저는 학습용 PC의 EXE를 임의 실행하지 못한다. 등록된 Custom URI Protocol 또는 허가된 Local Launcher 연계를 검토한다. 임의 명령 실행, 원격 무인 학습 실행은 범위에 포함하지 않는다.

## 상시 실행 정책

DatasetHub는 현재 Desktop 상시 실행을 전제로 한다. 장기 운영 중 로그아웃/앱 종료가 수집 중단으로 이어지는 문제를 피하려면 Collector/Processing/Matching 엔진을 향후 Windows Worker Service로 분리하고 WPF는 운영 UI로 유지하는 것을 검토한다. 엔진의 서비스 계약은 UI 독립적으로 유지한다.

## 저장소 간 동기화

- **EquipmentAIPlatform**: 이 파일을 기준으로 관리하고 버전/결정 변경 이력을 기록.
- **DatasetHub**: `docs/ECOSYSTEM_INTEGRATION.md`에서 이 문서를 링크하며 Collector/Matching 소유 범위를 준수.
- **Trimming AI Web (저장소 신설 후)**: 동일한 `docs/ECOSYSTEM_INTEGRATION.md` 파일을 추가, 공통 문서를 링크하고 전용 웹의 조회/API/SAIGE 연계 범위를 준수.
- **개인 Library**: 동일한 합의사항의 상세 설계 스냅샷을 별도로 보관하며 Git 커밋만으로 자동 동기화되지는 않음.

## 미결정

Trimming AI Web 저장소명·기술 스택, 검사 실행 ID 및 메타데이터 계약, 웹 조회 API, DB 확장 여부, 인증/SSO, SAIGE Import·런처 방식, 배포/운영 포트. 미결정 사항은 확정 전 임의 구현하지 않는다.

## 구현 순서

DatasetHub 수집 안정화 → 로그 DB 등록/이미지 인덱싱/제품 매칭 → Trimming AI Web 독립 구현 → 상시 Worker 분리 검토 → 상위 Platform 웹 통합.
