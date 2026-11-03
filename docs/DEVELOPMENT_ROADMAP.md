# 개발 로드맵

**현재는 설계 문서만 등록하며 코드 구현은 시작하지 않습니다.** 단계별 일정과 최종 기술 스택은 미정입니다.

## Phase 0 — 설계 (현재)
- 시스템·서비스 책임 경계, 데이터 흐름, API 계약 및 의사결정 문서화.
- 기존 EquipmentRAG·Hermes_equip의 실제 기능/버전과 연동 요구사항 확인.
- C/M에서 재현해야 할 기존 API·세션·요약 동작 정의.

## Phase 1 — 통합 운영 UI
- 서비스 상태·버전·Health/Ready·설정·로그를 통합 조회.
- A/O 기존 관리 UI를 활용하여 중복 개발 최소화.
- Windows 폐쇄망에서 서비스 실행·재시작·오류 격리·관리자 인증 검증.

## Phase 2 — C/M 재구현 및 서비스 통합
- C/M API 계약 테스트, LLM 프록시, 세션/요약/토큰 예산 구현.
- A/O ↔ EquipmentRAG 및 A/O ↔ C/M ↔ llama.cpp E2E 검증.
- 재시작/타임아웃·오류 복구 정책 검증.

## Phase 3 — 설비 지식·데이터 연결
- DatasetHub·로그 파서·Inspection DB를 읽기 전용 Tool로 연결.
- 알람, 시퀀스, Cycle/Step Time, Recipe, 검사 결과, Lot 관련 통계.
- 알람 1건을 로그→소스코드→과거 조치 이력까지 추적하는 PoC.

## Phase 4 — 업무 액션과 품질 분석 확장
- 보고서/이슈 초안, Git 리뷰 및 승인 기반 변경 워크플로.
- 필요 시 Vision AI 유사도 검색·불량 분류와 MLOps 연계.
- Monitoring/Remote Inspection PC 이용 검증 및 운영 정책 정착.

## 미결정
- UI/Backend 기술 스택 및 API 통합 방식.
- AI Server 물리 구성과 Data Collection 배치.
- LLM과 이미지 학습의 GPU 자원 할당.
- 서비스 자동 복구·운영 권한·내부 인증 방식.
- 폐쇄망 업데이트·버전 호환·롤백 절차.
