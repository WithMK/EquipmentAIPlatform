# Equipment AI Platform

설비 개발·양산·유지보수 지식을 통합하여 **근거 기반 질의응답, 트러블슈팅 및 승인형 업무 지원**을 제공하기 위한 사내 폐쇄망 AI 플랫폼입니다.

> **현재 상태:** 설계 문서만 관리. 아직 본 저장소에서 플랫폼 코드를 개발하지 않습니다.

## 핵심 구조
- **AI Control Center (신규):** 여러 서비스의 상태, 설정, 로그, 실행·배포를 한곳에서 통합 관리
- **[EquipmentRAG](https://github.com/WithMK/EquipmentRAG):** 코드, 기술 문서, 회의록 및 의사결정 검색·색인
- **[Hermes_equip](https://github.com/WithMK/Hermes_equip):** Agent Orchestrator(A/O), 작업·도구 호출·승인·산출물 관리
- **ContextManager (C/M):** 대화 컨텍스트·요약·토큰 예산·LLM 프록시. 사내 구현과 호환되는 계약을 기준으로 재구현 검토
- **Local LLM Runtime:** llama.cpp 기반 추론
- **[DatasetHub](https://github.com/WithMK/DatasetHub):** 설비 로그·이미지 중앙 수집과 데이터 관리. 향후 Tool 연동

기존 서비스와 저장소는 독립적으로 유지합니다. Control Center는 **운영 관리(Control Plane)**, A/O는 **사용자 작업 조율(Execution Plane)**을 담당합니다.

## 원칙
1. Windows·사내 폐쇄망에서 로컬 실행을 우선합니다.
2. 설비 제어·Safety·모션은 LLM/Agent와 철저하게 분리합니다.
3. 소스코드·문서는 RAG, 알람·사이클 시간·검사 결과는 DB/정량 분석 Tool로 조회합니다.
4. 초기에는 읽기 전용 분석·근거 제시를 중심으로 하며 변경 작업은 승인 기반입니다.
5. 외부 공개 저장소에 사내 코드, 실제 로그·이미지, 내부망 주소·자격정보를 넣지 않습니다.
6. 설계 방향과 실제 구현·검증 완료 상태를 구분합니다.

## 문서
- [전체 아키텍처](docs/ARCHITECTURE.md)
- [서비스 연동](docs/SERVICE_INTEGRATION.md)
- [개발 로드맵](docs/DEVELOPMENT_ROADMAP.md)
- [의사결정 기록](docs/DESIGN_DECISIONS.md)
