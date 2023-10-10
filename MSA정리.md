# 비동기 설계/개발 가이드
- 보상이 필요한 업무 서비스는 프로세스 관리와 모니터링 용이한 Saga Orchestration 패턴으로 전환 개발
- Orchestration 방식에 필요한 command topic은 참여자(participant)에서 등록/관리
- Saga 적용으로 발생하는 격리(Isopation) 미지원 문제는 업무 프로세스 조정 등으로 회피 선 고려, 필요한 경우 Semantic Lock 적용
## Orchestration (Command/Reply 방식으로 전환 필요)
- 여러개의 Step 중 하나가 실패하면 내 업무를 실패로 처리하고 이전 단계를 보상해야하는 경우
- 내 업무가 여러 타 MSA에 저장하는데 순서가 보장돼야 하는 경우
- 내 업무에서 타 MSA로 저장 요청 후 그 리턴을 받아 다음 Step 진행하는 경우
## Choregraphy (Domain Event 방식으로 처리 가능)
- 내 업무가 여러 MSA에 저장하는데 그 순서 상관 없는 경우
- 타 MSA에 저장하는데 리턴이 없거나 있어도 실시간성 아닌 경우
- 타 MSA에 저장하는 업무의 실패여부는 내 주요 업무처리에 영향 없는 경우 (재처리 가능)
## 보상업무 식별 및 연계협의 과정
1. 업무 프로세스 분석/자료 작성
2. 타 MSA 연계기능 및 리턴 식별
3. 핵심기능 분류 (Compensation Pivot Retry)
   - 본연 업무 완료되는 핵심 프로세스가 Pivot
   - Pivot 이전에 실패 시 보상이 필요한 경우 Compensation
   - 실패시에도 본연의 업무 기능 Pivot에 영향 없거나 후행 재처리가 가능하면 Retry
4. 업무 순서 조정
   - 보상 최소화
   - Compensation -> Pivot -> Retry 순으로 프로세스 배치
     
