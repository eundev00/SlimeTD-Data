---
name: check-sheets
description: SlimeTD-Data의 Sheets/*.csv와 Sheets/Waves/*.csv 기획 데이터를 검사한다. 시트를 고친 뒤 커밋 전, 또는 "시트 검사", "데이터 검사", "CSV 확인" 요청 시 사용. 형식, 이넘, ID 규칙, 참조 무결성, 게임 규칙을 아래 목록대로 빠짐없이 확인하고 결과를 표로 보고한다.
---

# 시트 검사

`Sheets/*.csv`와 `Sheets/Waves/*.csv` 전체를 검사한다. 열 설명과 규칙의 근거는 `Sheets/README.md`다. 규칙이 README와 다르면 README를 따르고, 이 스킬과 README가 어긋난다고 보고한다.

**검사만 한다. 시트를 고치지 않는다.** 고칠 내용은 제안으로만 보고한다.

## 진행 순서

1. `Sheets/`와 `Sheets/Waves/` 아래 CSV를 전부 읽는다. 아래 "시트 목록"에 없는 시트가 있으면 보고한다
2. 형식 검사는 PowerShell로 바이트를 확인한다 (눈으로 판단하지 않는다)
3. 아래 검사 항목을 **번호 순서대로 하나도 빼지 않고** 확인한다. 시트가 많아도 행을 건너뛰지 않는다
4. 결과를 "보고 형식"대로 정리한다

## 시트 목록

- 표 형식: Enums, Grades, LevelUpCosts, Towers, TowerProjectiles, Skills, HunterCardGachaPacks, Slimes, CoopStages, SingleStages, StageRewards, Items, NewAccount
- `key,value` 형식: HunterCardGachaSettings, GameConstants
- 스테이지 웨이브: `Waves/<codeName>.csv` (CoopStages·SingleStages의 스테이지마다 하나)

## 검사 항목

### A. 형식
1. 파일이 UTF-8 BOM(EF BB BF)으로 시작한다
2. 줄바꿈이 모두 CRLF다
3. 1줄이 영문 열 이름이고, `Sheets/README.md`에 적힌 열이 모두 있다. 없는 열, README에 없는 열, 중복된 열 이름을 보고한다
4. 모든 행의 칸 수가 열 수와 같다
5. 필수 열이 비어 있지 않다 (README에서 "비우면 …"이라고 한 열만 비울 수 있다)
6. 숫자 열은 숫자, 정수 열은 정수, 1/0 열은 1 또는 0이다

### B. 이넘
7. `Enums.csv`: (enumName, name)이 중복되지 않고, 같은 이넘 안에서 value가 중복되지 않는다. value는 정수다
8. 새 이넘은 1부터 시작한다. 단, 클라이언트 이넘(`SkillTrigger`, `SkillEffectType`, `VfxAnchor`, `SlimeRenderGroup`, `TowerAttackType`)은 클라이언트 값(0부터)을 쓴다
9. 선택지 열의 값이 `Enums.csv`의 해당 이넘 이름에 있다 (README `Enums.csv`의 "시트.열 → 이넘" 표 기준)

### C. ID
10. 각 시트의 키가 중복되지 않는다 (`LevelUpCosts`는 gradeId+level, `TowerProjectiles`는 towerId, 웨이브 파일은 wave+order, key,value 시트는 key)
11. ID 자릿수: 등급 1, 스테이지 2, 타워 3, 뽑기 팩 4, 스킬 5, 슬라임 6, 아이템 7
12. 모든 시트의 ID를 통틀어 같은 값이 없다 (이넘 value는 ID가 아니므로 제외)
13. 스킬 ID = 타워ID×100+순번 (`skillId / 100 == towerId`)
14. 스테이지 ID: `CoopStages`는 11~19, `SingleStages`는 21 이상. 두 시트를 합쳐 중복이 없다
15. 아이템 ID: 재화 1000001~1999999, 특정 헌터 카드(예정) = 2000000+타워ID. 2100000~ 대역(없앤 헌터 카드 팩 아이템)을 쓰면 오류
16. `codeName`이 시트 안에서 중복되지 않는다 (Towers, Slimes, Items). 스테이지는 CoopStages·SingleStages를 합쳐 중복되지 않는다 (웨이브 파일 이름이므로)

### D. 참조
17. `Towers.gradeId`, `LevelUpCosts.gradeId`, `GameConstants.summonGradeId` → Grades
18. `HunterCardGachaPacks.guaranteeGradeId` → Grades (0은 보장 없음)
19. `Skills.towerId`, `TowerProjectiles.towerId` → Towers, `Skills.upgradeOf` → Skills
20. 웨이브 파일의 `slimeId` → Slimes
21. `Items.targetId` → `HunterCard`(예정)면 Towers.towerId. 다른 타입은 비운다
22. `NewAccount.itemId`, `StageRewards.itemId` → Items, `StageRewards.stageId` → CoopStages 또는 SingleStages
23. `CoopStages`·`SingleStages`의 행마다 `Waves/<codeName>.csv`가 있고, `Waves/` 안의 파일마다 두 시트 중 하나에 같은 codeName의 행이 있다

### E. 게임 규칙
24. **Skills**
    - `unlockLevel`(비우면 1)은 1~15
    - `upgradeOf`가 있으면 같은 타워의 스킬이고, 대체 대상의 해금 레벨이 더 낮다. 자기 자신은 안 된다
    - `trigger=EveryNthAttack`이면 `nth`가 1 이상, 아니면 `nth`는 비운다
    - `chance`는 0 초과 100 이하, `cooldown`은 0 이상, `duration`은 0보다 크다
    - `value` 단위: AttackSpeed/DamageMultiplier/Slow는 퍼센트(%)만, Attack은 둘 다, 나머지(CritChance, CritBonus, MultiShot, Pierce, GoldGain, Splash, Stun, Cross)는 고정값만
    - Slow는 `duration` 필수, 1등급 타워의 Slow는 최대 5%
    - Stun은 `duration` 필수. `value`는 쓰지 않지만 클라이언트 임포터 때문에 비우지 않는다
25. **LevelUpCosts**: Grades의 모든 등급에 레벨 1~14가 빠짐없이 있다. `cards`, `coin`은 0 이상. `Grades.gachaWeight`가 0인 등급(뽑기에서 안 나옴)은 `cards`가 모두 0이어야 한다 (아니면 카드를 얻을 길이 없어 오류). 같은 등급 안에서 레벨이 오를수록 cards·coin이 줄어들면 경고
26. **Towers**: attackRange, attackInterval > 0, 타격 시점(`attackHitDelay`, 비면 `hunter.attackHitDelay`) < `attackInterval` (전용 모션 스킬의 `attackHitDelay`도 그 타워 간격보다 짧다), critChance 0~100, critBonus/unlockDiamondPrice ≥ 0. 일반·희귀·영웅(`gachaWeight` > 0인 등급) 헌터의 `startUnlocked`가 1이 아니면 경고 (정책: 시작부터 보유). 잠긴 헌터인데 `unlockDiamondPrice`가 0이면 경고 (구매 불가, 전설 가격 미정이면 이벤트 지급만 가능)
    - **TowerProjectiles**: `attackType=Projectile`인 타워는 행이 반드시 있고, 다른 타워는 행이 없다. speed, hitRadius, lifetime > 0
27. **Grades**: baseAttack > 0, attackGrowthRate/gachaWeight ≥ 0
28. **뽑기**: 팩 cardCount ≥ 1, price ≥ 0 (팩 티켓). 설정은 `maxLevelRefundCoin` ≥ 0 하나뿐이다. 천장·잠긴 카드 키(`lockedCardChance`, `pityCount`, `pityGradeId`)가 남아 있으면 경고 (2026-11-24 삭제됨)
29. **Slimes**: Boss는 `instantGameOver` 필수, Normal은 비운다(경고). `renderGroup`이 `kind`와 다르면 경고. baseHealth, baseSpeed > 0, lifeCost, goldReward ≥ 0
30. **CoopStages, SingleStages**: CoopStages.difficulty는 `Difficulty` 이넘 값. startingLife ≥ 1, startingGold ≥ 0, spawnInterval > 0, interWaveDelay ≥ 0, normalHpGrowth ≥ 0. `enabled`가 1인 스테이지가 하나 이상 있다
31. **웨이브 파일**: `wave`가 1부터 빠짐없이 이어지고, 웨이브마다 `order`가 1부터 빠짐없이 이어진다. count ≥ 1, delay ≥ 0. 마지막 웨이브 번호(최종 웨이브)를 보고에 적는다
32. **Items**
    - Currency: targetId, openMode 비움
    - itemType은 `Currency`만 쓴다 (헌터 카드 팩 아이템은 2026-11-24에 없앴다: 팩은 상점에서 팩 티켓으로만 연다)
    - openMode=OnAcquire면 scope 비움, openMode가 비면 scope(Account/Battle) 필수
33. **NewAccount**: amount ≥ 1, scope=Battle 아이템(골드, 젤리)은 지급할 수 없다
    - **Items 재화**: 서버가 codeName으로 찾는 계정 재화 `Coin`, `Diamond`, `PackTicket`, `Energy`가 모두 있다 (없으면 서버가 시작하지 않으므로 오류). 없앤 번호(1000005, 2100000~)를 쓰면 오류
34. **GameConstants**: README의 키가 모두 있다. summonBaseCost/summonCostIncrease ≥ 0, slowMinSpeedRatio·bossSlowScale 0~1. energy.max ≥ 1, energy.costPerBattle ≥ 0 (최대치보다 크면 오류: 시간 회복만으로는 시작할 수 없음), energy.recoverMinutes > 0, reward.segmentWaves ≥ 1, reward.failRate 0~1
    - **StageRewards**: rewardType은 `Clear`/`FirstClear`, itemId는 계정 재화(scope=Account, itemType=Currency), amount ≥ 1, (stageId, rewardType, itemId)가 중복되지 않는다. `enabled`인 스테이지에 `Clear` 줄이 없으면 경고. 각 스테이지의 최종 웨이브가 `reward.segmentWaves`로 나누어떨어지지 않으면 경고 (마지막 구간이 짧아진다)

### F. 문서
35. 새로 생긴 시트, 열, 키, 이넘 값이 `Sheets/README.md`에 설명돼 있다. 없으면 보고한다

## 보고 형식

1. 맨 위에 한 줄 요약: `시트 N개 검사: 오류 N개, 경고 N개`
2. 문제가 있으면 표로 보고한다

| 구분 | 위치 | 항목 | 내용 | 제안 |
|---|---|---|---|---|
| 오류 | Skills.csv 12줄 `towerId` | 19 | Towers에 towerId=999가 없다 | 타워 ID 확인 |

   - **오류**: 서버나 클라이언트가 잘못 동작하는 문제. **경고**: 동작은 하지만 의도와 다를 수 있는 것
   - 위치는 `파일:줄 [열]`. 시트 전체 문제는 줄을 생략한다
3. 문제가 없으면 "검사 항목 1~35 모두 통과"라고만 쓰고, 스테이지별 최종 웨이브를 함께 적는다
4. 확인하지 못한 항목이 있으면 숨기지 말고 따로 적는다
