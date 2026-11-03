# 서비스 연동 설계

**상태: 계약 초안, 실제 연결 검증 필요**

## 역할 경계
| 서비스 | 소유 책임 |
|---|---|
| AI Control Center | 서비스 등록·Health·설정·시작/중지·버전·운영 로그·권한 관리 |
| A/O (Hermes_equip) | 요청 해석, Agent/Task/Tool 위임, 승인 및 작업 결과 관리 |
| EquipmentRAG | 자료 등록·증분 색인, 코드·문서 검색, 출처 반환 |
| ContextManager | 짧은 대화 연속성, 요약, 토큰 예산, LLM 프록시 |
| llama.cpp | 언어 모델 추론 |
| DatasetHub/Equipment Tools(향후) | 원본 로그·이미지 수집, 구조화 데이터 및 통계 조회 |
| Vision AI(향후) | 유사도·군집화·불량 분류와 모델 관리 |

## 통합 정책
- 기존 A/O Web UI·Agent 관리 기능을 재구현하지 않고 가능하면 재사용합니다.
- ContextManager는 사내 PC의 소스 확인이 어려우므로, **A/O가 호출하는 실제 API 계약**을 우선 문서화하고 그 계약대로 재구현합니다.
- 통합 UI는 각 서비스의 Health/Ready, 버전, 오류, 연결 상태, 로그와 리소스 사용량을 표시합니다.
- 서비스 재시작, 설정 변경 및 Git 쓰기 작업은 인증·권한·명시적 승인·감사 기록을 요구합니다.
- API의 실제 주소, 토큰 및 내부 설정은 공개 Git에 올리지 않습니다.

## 추후 Tool 후보
- `search_equipment_logs`, `query_alarms`, `get_cycle_times`, `get_inspection_results`: 읽기 전용 정확한 수치·이벤트 검색.
- `find_similar_images`, `classify_defect`: 필요 시 Vision 서비스 호출. 초기 필수 기능 아님.
- `review_git_diff`, `draft_issue`, `create_report`: 엔지니어 생산성 향상. 변경 액션은 승인 게이트 적용.

## 통합 확인 항목
1. RAG 검색→출처 인용 응답 및 색인 반영.
2. A/O→RAG, A/O→C/M→llama.cpp 연결 검증.
3. C/M 세션 유지·요약·토큰 예산 Mock 및 사내 환경 계약 테스트.
4. 서비스 하나가 중단돼도 UI에서 오류를 파악하고 안전하게 복구할 수 있는지.
5. 표준 로그 한 건에서 관련 코드·문서·과거 이력을 연결한 E2E 분석.

**금지:** LLM/Agent의 직접 서보 동작, Safety 우회, 운영 설비 Recipe 자동 변경, 무승인 생산 배포.
