# 전체 시스템 아키텍처

**상태: 설계 초안, 미구현**

```text
[Engineer / Monitoring PC / Remote Inspection PC]
                        |
                  AI Control Center
           (통합 UI·서비스 상태·설정·로그)
                        |
       +----------------+----------------+
       |                |                |
 Agent Orchestrator  EquipmentRAG   ContextManager
 (작업·Agent·Tool)  (코드/문서 검색)  (대화/요약/프록시)
       |                |                |
       +----------------+----------------+
                        |
                llama.cpp / Local LLM
                        |
          향후 Equipment Data Tools
                        |
         DatasetHub / 중앙 로그·이미지 저장
                        ^
                   주기적 PULL
                        |
    적층 트리밍 설비 9대 × (Control PC 1 + Inspection PC 2)
```

> 논리적 개념도입니다. 실제 서비스 호출 방향과 배치/포트는 구현 단계에서 확정합니다.

## 배치 방향
- 설비 PC 총 27대(제어 9대, 검사 18대). 기존 실시간 제어와 이미지 취득·검사 실행은 독립 유지.
- 중앙 측 AI Server(기존 이미지 학습 PC 활용 검토), Monitoring PC, Remote Inspection PC를 분리된 역할로 유지.
- AI Server는 Local LLM과 Agent, RAG, 향후 Vision AI를 운영할 수 있으나 GPU 메모리와 학습 시 부하 충돌을 고려해 작업 스케줄링 필요.
- 중앙 Data Collection 기능의 실제 PC 배치는 I/O·가용성·저장 용량 검토 후 확정. 논리 서비스 위치와 물리 배치는 별개.

## 데이터 접근
| 데이터 | 관리 방식 |
|---|---|
| 소스코드, 사양, 회의록, 의사결정, 대응 사례 | EquipmentRAG (원본·버전·출처 보존) |
| 알람, 표준 로그, T/T·S/T, 검사 결과 | 중앙 원본 보존 + 구조화 DB + 조회 Tool |
| 검사 이미지 | 파일 저장소 + DB 메타데이터 |
| NG 패턴·유사도·학습 모델 | 향후 별도 Vision AI/MLOps 서비스 |
| Agent 작업·의사결정·승인 | A/O의 작업 상태 저장소 |
| 대화 이력·요약·토큰 예산 | ContextManager |

## 우선 목표
1. '이 함수가 왜 이렇게 구현됐나?' → 코드와 의사결정 이력 연결.
2. '특정 알람의 원인은?' → 전후 로그, 코드, 과거 사례를 근거로 분석.
3. 'T/T가 증가한 이유는?' → 정확한 통계 도구로 산출 후 LLM이 변경·이력과 연계 해석.

설비 데이터는 중앙에서 주기적으로 PULL 수집합니다. PULL도 설비 네트워크/디스크에 읽기 부하를 주므로 동시성·전송량 제한 및 검증이 필요합니다. LLM 장애가 생산 설비를 정지시키면 안 됩니다.
