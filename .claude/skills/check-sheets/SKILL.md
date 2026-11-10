---
name: check-sheets
description: SlimeTD-Data의 Sheets/*.csv 기획 데이터를 검사한다. 시트를 고친 뒤 커밋 전, 또는 "시트 검사", "데이터 검사", "CSV 확인" 요청 시 사용. 형식, ID 규칙, 참조 무결성, 게임 규칙을 아래 목록대로 빠짐없이 확인하고 결과를 표로 보고한다.
---

# 시트 검사

`Sheets/*.csv` 전체를 검사한다. 열 설명과 규칙의 근거는 `Sheets/README.md`다. 규칙이 README와 다르면 README를 따르고, 이 스킬과 README가 어긋난다고 보고한다.

**검사만 한다. 시트를 고치지 않는다.** 고칠 내용은 제안으로만 보고한다.

## 진행 순서

1. `Sheets/` 아래 CSV를 전부 읽는다. 아래 "시트 목록"에 없는 시트가 있으면 보고한다
2. 형식 검사는 PowerShell로 바이트를 확인한다 (눈으로 판단하지 않는다)
3. 아래 검사 항목을 **번호 순서대로 하나도 빼지 않고** 확인한다. 시트가 많아도 행을 건너뛰지 않는다
4. 결과를 "보고 형식"대로 정리한다

## 시트 목록

Grades, LevelUpCosts, Towers, Skills, HunterCardGachaPacks, HunterCardGachaSettings, Slimes, WaveTables, WaveZones, GameConstants, Items, NewAccount

`HunterCardGachaSettings`, `GameConstants`는 `key,value` 두 열이고, 나머지는 표 형식이다.

## 검사 항목

### A. 형식
1. 파일이 UTF-8 BOM(EF BB BF)으로 시작한다
2. 줄바꿈이 모두 CRLF다
3. 1줄이 영문 열 이름이고, `Sheets/README.md`에 적힌 열이 모두 있다. 없는 열, README에 없는 열, 중복된 열 이름을 보고한다
4. 모든 행의 칸 수가 열 수와 같다
5. 필수 열이 비어 있지 않다 (README에서 "비우면 …"이라고 한 열만 비울 수 있다)
6. 숫자 열은 숫자, 정수 열은 정수, 1/0 열은 1 또는 0, 선택지 열은 허용된 값만 있다

### B. ID
7. 각 시트의 키가 중복되지 않는다 (`LevelUpCosts`는 gradeId+level, `WaveZones`는 tableId+zone, key,value 시트는 key)
8. ID 자릿수: 등급 1, 웨이브 테이블 2, 타워 3, 뽑기 팩 4, 스킬 5, 슬라임 6, 아이템 7
9. 모든 시트의 ID를 통틀어 같은 값이 없다
10. 스킬 ID = 타워ID×100+순번 (`skillId / 100 == towerId`)
11. 아이템 ID: 재화 1000001~1999999, 헌터 카드 팩 = 2100000+팩ID, 특정 헌터 카드(예정) = 2000000+타워ID
12. `codeName`이 시트 안에서 중복되지 않는다 (Towers, Slimes, WaveTables, Items)

### C. 참조
13. `Towers.gradeId`, `LevelUpCosts.gradeId`, `HunterCardGachaSettings.pityGradeId`, `GameConstants.summonGradeId` → Grades
14. `HunterCardGachaPacks.guaranteeGradeId` → Grades (0은 보장 없음)
15. `Skills.towerId` → Towers, `Skills.upgradeOf` → Skills
16. `WaveZones.tableId` → WaveTables, `WaveZones.slimeId`/`bossId` → Slimes
17. `Items.targetId` → HunterCardPack이면 HunterCardGachaPacks.packId
18. `NewAccount.itemId` → Items

### D. 게임 규칙
19. **Skills**
    - `unlockLevel`(비우면 1)은 1~15
    - `upgradeOf`가 있으면 같은 타워의 스킬이고, 대체 대상의 해금 레벨이 더 낮다. 자기 자신은 안 된다
    - `trigger=EveryNthAttack`이면 `nth`가 1 이상, 아니면 `nth`는 비운다
    - `chance`는 0 초과 100 이하, `cooldown`은 0 이상, `duration`은 0보다 크다
    - `value` 단위: AttackSpeed/DamageMultiplier/Slow는 퍼센트(%)만, Attack은 둘 다, 나머지(CritChance, CritBonus, MultiShot, Pierce, JellyGain, Splash, Stun)는 고정값만
    - Slow는 `duration` 필수, 1등급 타워의 Slow는 최대 5%
20. **LevelUpCosts**: Grades의 모든 등급에 레벨 1~14가 빠짐없이 있다. `cards`, `gold`는 0 이상
21. **Towers**: attackRange, attackCooldown > 0, critChance 0~100, critBonus/unlockGemPrice ≥ 0. 잠긴 헌터(`startUnlocked`가 1이 아님)인데 `unlockGemPrice`가 0이면 경고 (구매 불가)
22. **Grades**: baseAttack > 0, attackGrowthRate/gachaWeight ≥ 0
23. **뽑기**: 팩 cardCount ≥ 1, price ≥ 0. 설정 lockedCardChance 0~100, pityCount ≥ 1, maxLevelRefundGold ≥ 0
24. **Slimes**: Boss는 `instantGameOver` 필수, Normal은 비운다(경고). `renderGroup`이 `kind`와 다르면 경고. baseHealth, baseSpeed > 0, lifeCost, jellyReward ≥ 0
25. **WaveTables**: spawnInterval > 0, interWaveDelay/normalHpGrowth ≥ 0
26. **WaveZones**: 테이블마다 구간이 1부터 빠짐없이 이어진다. `slimeId`는 kind=Normal, `bossId`는 kind=Boss. count ≥ 1, bossExtraStartDelay ≥ 0
27. **Items**
    - Currency: targetId, openMode 비움
    - HunterCardPack: targetId 필수, openMode=OnAcquire
    - openMode=OnAcquire면 scope 비움, openMode가 비면 scope(Account/Battle) 필수
28. **NewAccount**: amount ≥ 1, scope=Battle 아이템(젤리 등)은 지급할 수 없다
29. **GameConstants**: README의 키가 모두 있다. startingLife ≥ 1, startingJelly/summonBaseCost/summonCostIncrease ≥ 0, slowMinSpeedRatio·bossSlowScale 0~1

### E. 문서
30. 새로 생긴 시트, 열, 키, 선택지 값이 `Sheets/README.md`에 설명돼 있다. 없으면 보고한다

## 보고 형식

1. 맨 위에 한 줄 요약: `시트 N개 검사: 오류 N개, 경고 N개`
2. 문제가 있으면 표로 보고한다

| 구분 | 위치 | 항목 | 내용 | 제안 |
|---|---|---|---|---|
| 오류 | Skills.csv 12줄 `towerId` | 15 | Towers에 towerId=999가 없다 | 타워 ID 확인 |

   - **오류**: 서버나 클라이언트가 잘못 동작하는 문제. **경고**: 동작은 하지만 의도와 다를 수 있는 것
   - 위치는 `파일:줄 [열]`. 시트 전체 문제는 줄을 생략한다
3. 문제가 없으면 "검사 항목 1~30 모두 통과"라고만 쓴다
4. 확인하지 못한 항목이 있으면 숨기지 말고 따로 적는다
