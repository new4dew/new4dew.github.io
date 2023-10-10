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
   - 본연 업무 완료되는 핵심 프로세스가 Pivot (**Pivot인 경우 보상트랜잭션 안 쓰고 Local Transaction 처리로 Rollback이라고 기재, 또는 명시적 상태 변경을 위한 추가라고 적기**)
   - Pivot 이전에 실패 시 보상이 필요한 경우 Compensation 
   - 실패시에도 본연의 업무 기능 Pivot에 영향 없거나 후행 재처리가 가능하면 Retry
4. 업무 순서 조정
   - 보상 최소화
   - Compensation -> Pivot -> Retry 순으로 프로세스 배치
     
# 비격리 대책
## 문제
### Dirty Read
Saga가 진행 중에 다른 트랜잭션이나 Saga가 아직 커밋되지 않은 변경분을 읽어가는 경우
### Non-Repeatable Read
한 트랜잭션 내에서 같은 key 가진 row를 두 번 읽었는데 그 사이에 값이 변경/삭제된 경우
### Lost Update
한 Saga의 변경분을 다른 Saga가 덮어쓰는 경우

## 대책
### Semantic Lock
application 수준의 락, 플래그 셋팅
### Optimistic Lock
값을 다시 읽어서 Lost Update 방지
### Pessimistic Lock
DB에서 주로 처리하는 행단위 lock (select for update)

# 성능 개선
## MSA 도입으로 인한 개선
1. 더 신속한 업데이트
- Service별 독립적 배포
2. 독립적인 확장성
- scale in/out
3. 장애전파 방지
- 장애 격리
4. 성능과 유저 경험
- 재기동 및 scale 시간 단축

# Cloud vs On-Premise
https://www.ridge.co/blog/cloud-vs-on-premise/
https://www.geeksforgeeks.org/on-premises-vs-on-cloud/
