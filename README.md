# eigentopic

근거와 변화 이력을 보존하며, 누적 자료에서 발전하는 주제 지도.

새 자료를 기존 주제와 비교하고, 필요하면 과거 근거까지 다시 읽어 주제의 이름과 경계를 고친다. 결과에는 현재의 지도와 함께 **무엇이, 어떤 근거로 달라졌는지**가 남는다.

`eigen`은 자료에서 드러나는 고유한 방향을 떠올리게 하는 이름이다. 고유벡터 계산이나 특정 수치 알고리즘의 채택을 뜻하지 않는다.

## 현재 상태

**설계 초안이다. 실행 가능한 수집기, 주제 추론 엔진, UI는 아직 없다.**

- [DESIGN.md](DESIGN.md): 범위, 기록 모델, 갱신 순서, 검증 기준과 미정 사항
- [examples/synthetic-batch.json](examples/synthetic-batch.json): 전부 가상으로 만든 기록 예시
- [.gitignore](.gitignore): 실제 입력, 생성 산출물, 개인별 설정과 비밀정보의 추적 제외 규칙

## 첫 범위

기존 외부 수집기가 전달한 자료를 받아 주제 지도를 천천히 갱신하는 **slow layer**부터 설계한다.

1. 입력을 정규화하고 중복을 식별한다.
2. 출처에 연결된 관찰을 만든다.
3. 비슷한 관찰을 묶고 기존 주제와 비교한다.
4. 새 근거에 비추어 관련 과거 자료와 주제의 해석을 다시 검토한다.
5. 주제 버전, 이전 지도, 의미 있는 변화와 그 근거를 함께 남긴다.

주제는 미리 정한 분류표에만 맞추지 않고 자료에서 생겨나도록 한다. 자동 생성·이름 수정·분리·병합의 구체적인 판단 방법과 품질 기준은 아직 정하지 않았다.

## 개인별 관점

같은 근거를 두고도 사용자마다 보고 싶은 순서와 필요한 설명이 다를 수 있다. 개인화는 우선 **표시 비중과 설명 방식**에 적용한다. 사용자 취향이 원문, 출처, 반대 근거를 바꾸는 설계는 피한다.

주제 숨기기는 되돌릴 수 있어야 하며 기록은 유지한다. 앞으로 비슷한 주제를 생성하지 않도록 하는 억제 정책은 별도의 결정이다. 여러 사용자가 같은 자료 집합을 쓸지, 각자 자료를 가질지와 공유 방식은 미정이다.

## 공개 저장소와 실제 자료

이 저장소에는 설계와 검토된 합성 예제만 넣는다. 실제 원문, 관찰, 주제 지도, 스냅샷, 변화 기록, 사용자 프로필, 실행 로그와 생성 산출물은 로컬의 제외 경로에 둔다.

권장 실행 경로는 `runtime/` 아래다. `.gitignore`는 추적하지 않은 파일을 Git에서 제외할 뿐, **백업·암호화·접근 제어를 제공하지 않으며 이미 커밋된 내용을 지우지 않는다.** 공개 전에 `git status`와 스테이징된 변경을 직접 확인해야 한다. 실제 자료가 커밋 이력에 들어가지 않도록 관리한다.

별도 비공개 백업의 위치와 보존·복구 정책은 구현 단계에서 정한다. 라이선스는 아직 선택하지 않았다.

## 설계의 출발점

- Blei & Lafferty (2006)의 [Dynamic Topic Models](https://www.cs.columbia.edu/~blei/papers/BleiLafferty2006a.pdf): 시간에 따른 주제 표현의 변화
- Pirolli & Card (2005)의 [sensemaking 연구](https://www.researchgate.net/profile/Peter-Pirolli/publication/215439203_The_sensemaking_process_and_leverage_points_for_analyst_technology_as_identified_through_cognitive_task_analysis/links/02bfe50f09ca94efc0000000/The-sensemaking-process-and-leverage-points-for-analyst-technology-as-identified-through-cognitive-task-analysis.pdf): 자료와 해석을 오가며 표현을 고치는 과정
- [epistemic-protocols](https://github.com/jongwony/epistemic-protocols/tree/10abcb37573c187b3fc23d5dc834519475b920b2): 원자료 보존, 재해석, 사례에서 추상화하기, 미정인 결정을 드러내기
- [explain](https://github.com/jongwony/cc-plugin/blob/15584bc0cf1f4e50e040ae167d5edb1c0397ccce/explain/skills/explain/SKILL.md): 읽는 사람의 이해에 맞춰 구성 요소와 변화의 이유를 설명하기

각 자료가 뒷받침하는 범위와 eigentopic에 적용하면서 추가한 설계 판단은 [DESIGN.md](DESIGN.md#참고-자료와-적용-범위)에 구분했다.
