---
name: extract-combat-values
description: 클라이언트(C:\Projects\SlimeTD) 프리팹과 애니메이션에서 전투 값을 읽어 시트에 옮긴다. 슬라임 반지름(Slimes.radius)과 헌터·스킬 공격 시점 처음 값(attackHitDelay, attackDuration). "슬라임 반지름 다시 뽑기", "공격 시점 뽑기", 슬라임 충돌체·모델 배율을 바꿨을 때, 새 헌터나 전용 모션 스킬을 추가할 때 사용.
---

# 프리팹·애니메이션에서 전투 값 뽑기

클라이언트 프로젝트 `C:\Projects\SlimeTD`의 프리팹(`.prefab`), 애니메이터(`.controller`), 클립(`.anim`)을 YAML 텍스트로 읽어 값을 구한다. **클라이언트 파일은 읽기만 하고 고치지 않는다.** 코드(에디터 스크립트 등)를 만들지 않는다.

단위와 자릿수는 `Sheets/README.md` "거리 단위"를 따른다: 거리는 월드 단위(1 = 한 칸) 소수 첫째 자리, 시간(초)은 소수 둘째 자리 반올림.

## 진행 순서

1. 저장소 `CLAUDE.md`의 커밋 순서대로 작업 전에 `fetch` / `pull --ff-only` 한다
2. 아래 "슬라임 반지름" 또는 "공격 시점"을 구한다. 계산에 쓴 원래 값(충돌체 반지름, 배율, 이벤트 시각, exitTime, 클립 길이)을 표로 남긴다
3. 시트와 다른 값만 고친다 (UTF-8 BOM + CRLF 유지, CLAUDE.md의 PowerShell 쓰기 방법)
4. README의 값 설명(예: `Slimes.radius`의 "2026-11-27 값")이 바뀌면 README도 고친다
5. `check-sheets`로 검사하고, 바뀐 값과 계산 표를 보고한다. 커밋은 사용자가 요청할 때 한다

## 슬라임 반지름 → `Slimes.csv` `radius`

충돌체나 모델 배율을 바꿨을 때 돌린다. 모든 슬라임 행을 다시 계산한다.

1. 프리팹: `Assets/Prefabs/Game/Slime/<codeName>.prefab` (시트 `codeName`과 같은 이름). 없으면 보고하고 건너뛴다
2. 프리팹에서 `SphereCollider`(`--- !u!135`)를 찾아 `m_Radius`를 읽는다. 충돌체가 둘 이상이거나 다른 모양이면 보고한다
3. 충돌체가 붙은 GameObject의 Transform부터 `m_Father`를 따라 루트까지 올라가며 `m_LocalScale`을 모두 곱한다 (충돌체 오브젝트 자신 포함). x·y·z가 다르면 가장 큰 값을 쓰고 보고한다
4. 프리팹 안에 다른 프리팹(모델 FBX 등)이 중첩돼 있으면 `PrefabInstance`의 `m_Modifications`에 있는 `m_LocalScale` 덮어쓰기 값을 먼저 쓰고, 없으면 원본 에셋의 값을 쓴다
5. `radius` = `m_Radius` × 배율, 소수 첫째 자리 반올림

## 공격 시점 → `Towers.csv` / `Skills.csv` `attackHitDelay`, `attackDuration`

**새 헌터나 새 전용 모션 스킬을 추가할 때 처음 값만 뽑는다.** 시트가 기준이라 애니메이션을 고쳤다고 다시 돌리지 않는다 (클라이언트가 재생 속도를 맞춘다).

1. 헌터 프리팹 `Assets/Prefabs/Game/Characters/<codeName>.prefab`의 `Animator` `m_Controller` guid → 같은 guid의 `.controller.meta`를 찾아 컨트롤러를 연다 (`AnimatorOverrideController`면 바뀐 클립을 따른다)
2. 트리거: 헌터는 `Attack`, 스킬은 그 스킬의 `animTrigger`. 컨트롤러에서 `m_ConditionEvent`가 그 트리거인 전이(`AnimatorStateTransition`)의 `m_DstState`가 공격 첫 상태다
3. 상태의 `m_Motion` guid → `.anim` 클립. 클립의 `m_Events`에서 `functionName`이 `OnAttackHit`, `OnAttackEnd`인 항목의 `time`(초)을 읽는다
4. 첫 상태 클립에 이벤트가 없고 다음 상태로 이어지면 (예: 궁수 Draw → Release): 첫 상태에서 나가는 전이의 `m_ExitTime` × 첫 클립 길이(클립의 `m_StopTime`)를 다음 클립 이벤트 시각에 더한다. 상태가 더 이어지면 같은 방법으로 계속 더한다
   - 예) 궁수: Draw 1.333초 × 0.8125 = 1.083, Release의 `OnAttackHit` 0.033 → 1.12, `OnAttackEnd` 1.333 → 2.42
5. 상태의 `m_Speed`가 1이 아니면 시각을 그 값으로 나눈다. 속도 파라미터(`AttackSpeed` 등)는 공격속도 배율이라 1로 본다
6. `attackHitDelay` = 타격 시각, `attackDuration` = 끝 시각 (둘째 자리 반올림)
7. `GameConstants`의 `hunter.attackHitDelay`, `hunter.attackDuration`과 같으면 비운다. 스킬은 그 타워 값과 같으면 비운다
   - 기본값의 기준 모션인 칼 계열 기본 공격(`SwordAnimator`의 `Attack` 상태, `Sword_Attack` 클립)을 쓰는 헌터는 값이 조금 달라도(모션 1.07초 vs 기본값 1초) 비운다. 모션 길이는 연출용이고 공격 간격은 `attackInterval`이 정한다
8. `OnAttackHit`이나 `OnAttackEnd`가 없으면 값을 넣지 않고 보고한다 (클라이언트는 둘 다 있어야 공격이 끝난다)
