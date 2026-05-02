
- 이벤트 기능 개발
    
    - 이벤트 기능은 현재 유저의 재화, 레시피 개수, 특정 요리 재료 획득 등을 추적하다 특정 값에 맞으면 트리거를 발동하는 시스템
    - 이와 연동한 퀘스트 시스템이 있음 해당 트리거에 맞는 퀘스트를 실행함
    - 퀘스트는 정해진 재화와 정해진 레시피 개수, 특정 요리 재료 획득, 특정 요리를 만들었을 때 오른쪽에서 NPC가 등장 (스프라이트가 등장하며 말풍선으로 대화 진행, 클릭으로 대화 넘김)
    - 따라서 NPC 대화 테이블을 저장해야함
    - 퀘스트는 다음과 같음 당사자가 원하는 특정 요리를 만들어서 팔아야 함 (NPC가 대화로 사연을 말하고 어떤 음식 or 몇 성 이상의 음식을 팔라고 말함) 보상은 특정 던전 해금, 새로운 요리 도구 보상, 새로운 요리 레시피 보상, 특정 요리 재료 보상, 새로운 용병 등장 등 다양하게 설정 가능
    - ex) 게임 시작 시 재화가 0원, 요리도구가 0, 레시피가 0이기 때문에 이벤트 기능으로 추적됨 이후 퀘스트에 트리거 되어 몇가지 재료와 도구를 주고 ~~요리를 만들라고 시킴 → 튜토리얼 가능
    - ex) 3성 이상의 재료 획득 시 왕의 부탁 이벤트 발생 3성 이상의 요리를 달라고 함 → 보상으로 새로운 요리 도구
    - 이벤트가 겹칠 수 있기에 우선순위와 대기열 시스템이 필요
    - 개발된 기능 리스트
    
    새 시스템 (이벤트 기반)
    
    조건(Criterion) 충족 → 퀘스트 트리거 → 액션(Action) 실행 → 완료 조건 충족 → 보상 액션 실행
    
    핵**심 3요소:**
    
    1. **Criterion (조건) - CriterionBase 상속
    
    - TriggerConditions: 퀘스트가 등장하는 조건
    - CompletionConditions: 퀘스트가 완료되는 조건
    - 두 가지 타입:**
        - **OnValueReached: 폴링으로 체크 (예: 골드 1000 보유)**
        - **OnActionPerformed: 이벤트로 체크 (예: 요리 판매 시)**
    
    2. **Action (액션) - QuestActionBase 상속
    
    - OnAcceptActions: 퀘스트 수락 시 실행
    - OnCompleteActions: 퀘스트 완료 시 실행 (보상)**
    
    3. **Priority (우선순위)
    
    - 대기열에서 처리 순서 결정**
    
    ---
    
    1. 퀘스트 생명주기
    
    상태 흐름 (QuestSystem.cs:11-18)
    
    Locked → Available → Active → Complete → Finished ↓ Failed (시간 초과)
    
    처리 흐름
    
    Phase 1: 초기화 (Awake)
    
    InitializeQuests() (QuestSystem.cs:148) → 새 시스템 퀘스트 → _eventTriggerQuests에 추가 → 레거시 퀘스트 → _pendingQuests에 추가 (시간순 정렬)
    
    Phase 2: 리스닝 시작 (Start)
    
    StartCriterionListening() (QuestSystem.cs:215) → OnActionPerformed 타입 Criterion들이 이벤트 구독 시작 → OnValueReached 타입은 Update()에서 폴링
    
    Phase 3: 트리거 감지 (Update)
    
    CheckQuestTriggers() (QuestSystem.cs:310) → 레거시: 시간 체크
    
    CheckCriterionTriggers() (QuestSystem.cs:329) → 새 시스템: OnValueReached 조건 폴링
    
    OnCriterionMet() (QuestSystem.cs:274) → 새 시스템: OnActionPerformed 이벤트 발생 시
    
    Phase 4: 대기열 처리
    
    EnqueueQuest() (QuestSystem.cs:383) → _priorityQueue에 추가
    
    ProcessPriorityQueue() (QuestSystem.cs:404) → 매 프레임마다 하나씩 Dequeue → TriggerQuest() 호출 → OnQuestAvailable 이벤트 발생
    
    Phase 5: 퀘스트 진행
    
    AcceptQuest() (QuestSystem.cs:436) → ActiveQuest 생성 → OnAcceptActions 실행 (새 시스템) → StartCompletionConditionListening() (새 시스템) → OnQuestAccepted 이벤트 발생
    
    Phase 6: 완료 체크
    
    레거시: UpdateObjective() (QuestSystem.cs:496) → AllObjectivesComplete 체크 → CompleteQuest()
    
    새 시스템: CheckQuestCompletion() (QuestSystem.cs:681) → CompletionConditions 모두 충족 시 → CompleteQuest()
    
    Phase 7: 보상 지급
    
    ClaimReward() (QuestSystem.cs:533) → 새 시스템: OnCompleteActions 실행 → 레거시: GiveReward() 호출 → _completedQuests에 추가 → _activeQuests에서 제거
    
    ---
    
    2. 주요 특징
    
    ✅ 장점
    
    3. 하이브리드 설계: 기존 코드 호환성 유지하면서 새 기능 추가
    4. 유연한 조건 시스템: Criterion으로 다양한 트리거 조건 구현 가능
    5. 우선순위 관리: 중요한 퀘스트를 먼저 처리
    6. 이벤트 기반 + 폴링: 두 가지 체크 방식 지원
    7. 상세한 로깅: 디버깅이 용이
- 레시피 북 UI
    
    - 레시피 북 UI는 Core 기능만 개발하고 UI디자인이나 효과는 나중에 작업할거야
    - 현재 보유중인 레시피를 보여주는 UI야
    - 하단 버튼을 누르면 화면 가운데 팝업 UI가 떠 (책 모양으로 변경할거야)
    - 해당 UI 왼쪽 위에 책갈피 같이 요리도구만큼 책갈피가 있고 해당 버튼으로 해당 요리 도구에 대한 레시피를 볼 수 있어
    - 초기에는 가장 먼저 첫 요리도구로 열어지고 화면에는 해당 요리 도구에 맞는 레시피가 나열되어 있어 보유중인 레시피는 해당 레시피에 맞는 아이콘 이미지가 있고 마우스를 위에 올리면 팝업으로 해당 레시피가 나와 (재료 개수와 종류) 미보유중인 레시피는 해당 요리가 회색 이미지로 가려져 있고 호버해도 아무런 UI안나와
- 0~60 시간 흐름에 맞춰 QA 개발 진행
    
    - 0 ~ 5분 : 게임 튜토리얼 및 기본 조작법 안내
        - 기본 요리 (잼, 샐러드)등에 맞는 레시피 카드, 재료 카드, 요리 도구 카드 제공 -> NPC가 제공해줌
        - 이후에 용병 퀘스트를 통해 재료 수급
    - 5 ~ 15분 : 초반 재료 수급 및 요리 판매
        - 대부분 1성 요리 카드 판매, 레시피 북 랜덤 구입
    - 15 ~ 30분 : 새로운 요리 도구 및 퀘스트 발생 **(새로운 퀘스트)**
        - 특정 요리 개수*()를 가져오면 암시장 오픈, 기존 섞기 요리 도구 + 새로운 요리 도구 획득 (굽기)
        - 암시장에서는 더 비싼 값에 어려운 레시피 오픈 (4개의 재료를 섞고, 해당 요리 + 재료를 한번 더 요리해야 하는 등) + 비싼 값
    - 30 ~ 45분 : 새로운 지역 오픈 및 재료 수급
        - 새로운 지역 오픈 (던전, 숲, 해변, 마왕성 등) -> 용병 퀘스트로 재료 수급, 해금은 순차적으로
        - 새로운 지역에서만 얻을 수 있는 재료로 요리 판매
    - 45 ~ 60분 : 새로운 퀘스트 발생, 용병이 새로 오픈된 지역 2회 이상 탐사시 발생?
        - 새로운 주민 NPC 등장 -> 새로운 요리 도구 및 레시피 북 판매
        - 새로운
    - 60 ~ 90분 : 중반부 플레이 -> 용병 업그레이드 기능 추가하여 더 좋은 재료 수급
        - 새로운 요리 도구 및 레시피 북 획득
        - 용병 퀘스트로 재료 수급
        - 요리 판매
- 코인은 생성되고 실제 코인UI로 빨려들어가는 연출
    
    - 실제 빨려 들어가고 코인 증가
- 용병 시스템 좀 더 구체화 필요함
    

- 카드 생성을 현재 어떤 식으로 하는지
    
    - 이펙트 처리
    - 영역 처리
- 용병, 던전 데이터 테이블화 시켜야 함