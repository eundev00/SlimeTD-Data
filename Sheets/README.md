# 시트 편집 가이드

이 폴더가 기획 데이터의 원본이다. 서버와 클라이언트는 여기서 받아 쓰고, 각자 복사본을 고치지 않는다.

시트는 UTF-8(BOM) CSV, CRLF 줄바꿈이고 **1줄이 영문 열 이름**이다. 새 시트도 같은 형식으로 만든다.

클라이언트가 아직 이 저장소를 직접 읽지 않으므로, 고친 시트는 클라이언트 `Assets/Datas/Sheets/`에 복사한 뒤 아래 메뉴로 임포트한다. 클라이언트에서 숫자를 인스펙터로 고치면 다음 임포트 때 덮어써진다.

- 타워/등급/레벨업 비용: `SlimeTD > Data > Import Tower Sheets`
- 스킬: `SlimeTD > Data > Import Skill Sheets`
- 헌터 카드 뽑기: `SlimeTD > Data > Import Hunter Card Gacha Sheets` (Grades.csv의 `gachaWeight`도 여기서 읽는다)

## ID 규칙

모든 데이터의 ID는 종류와 상관없이 겹치면 안 된다. 한 번 정한 ID는 바꾸지 않는다.

| 종류 | 규칙 |
|---|---|
| 등급 | 1~4 |
| 타워 | 101~ (3자리) |
| 스킬 | 타워ID×100+순번 (궁수 10101~, 일병 10301~). 추가는 그 타워의 다음 빈 번호 |
| 뽑기 팩 | 1001~ (4자리) |
| 웨이브 테이블 | 11~ (2자리) |
| 슬라임 | 6자리. 일반 100001~, 보스 100101~ (같은 구간의 일반과 보스는 끝 두 자리를 맞춘다) |
| 아이템 | 7자리. 재화 1000001~, 헌터 카드 팩 2100000+팩ID (1장 팩 1001 → 2101001), 특정 헌터 카드 2000000+타워ID (예정, 궁수 101 → 2000101) |

## 열 용도

변환 도구는 열 용도에 따라 서버용과 클라이언트용 데이터를 나눠 내보낸다. **아래 표에 없는 열은 `both`**(서버와 클라이언트 모두 사용)다. 전투 결과에 영향을 주는 수치는 서버가 계산하므로 `both`다.

| 시트 | 열 | 용도 | 이유 |
|---|---|---|---|
| Grades | name | client | 화면 표시용 |
| Towers | name | client | 화면 표시용 |
| Skills | name, description | client | 화면 표시용 |
| Skills | vfxKey, vfxAnchor | client | 연출용 |
| Skills | animTrigger | client | 연출용. 단, 공격 타격 시점은 애니메이션 클립이 정하므로 서버용 타이밍 데이터가 따로 필요하다 (미정) |
| Slimes | renderGroup | client | 렌더링 순서용 |
| Items | name | client | 화면 표시용 |
| NewAccount | (전체) | server | 새 계정 생성은 서버만 한다 |

프리팹, 썸네일, 머티리얼 같은 Unity 에셋 연결은 시트에 넣지 않는다. 클라이언트가 `codeName`으로 찾아 인스펙터에서 연결한다.

## Grades.csv

| 열 | 설명 |
|---|---|
| gradeId | 등급 ID |
| name | 등급 이름. 헌터 상세 팝업의 등급 배지와 태그에 표시된다 |
| baseAttack | 기준 공격력 |
| attackGrowthRate | 레벨당 공격력 증가율(%) |
| gachaWeight | 뽑기 등급 가중치. 보유 헌터가 없는 등급은 빼고 남은 가중치끼리 비율을 다시 계산한다 |

## LevelUpCosts.csv

등급별 레벨업 비용. 한 행이 "그 등급의 한 레벨" 비용이다.

| 열 | 설명 |
|---|---|
| gradeId | 등급 ID |
| level | 올리기 전 레벨. 등급마다 1부터 14까지 빠짐없이 있어야 한다 (1이면 1 -> 2) |
| cards | 필요한 그 헌터의 카드 수 |
| gold | 필요 골드 |

## Towers.csv

| 열 | 설명 |
|---|---|
| towerId | 타워 ID (101~) |
| codeName | 영문 코드명. 에셋 이름 `TowerData_<코드명>`에 쓰인다 |
| name | 화면에 보이는 이름 |
| gradeId | 등급 ID |
| enabled | 활성 (1/0) |
| attackRange | 사거리 |
| attackCooldown | 공격 쿨다운(초) |
| critChance | 치명타 확률(%) |
| critBonus | 치명타 보너스(%) |
| startUnlocked | 시작 해금 (1/0). 타워마다 지정한다. 비우면 잠금 |
| unlockGemPrice | 해금 보석 가격 (0 = 구매 불가) |
| inGacha | 뽑기 대상 (1/0). 비우면 1. 잠긴 헌터는 잠긴 카드로만 나오고, 해금되면 등급 풀에 들어간다 |

## HunterCardGachaPacks.csv

| 열 | 설명 |
|---|---|
| packId | 팩 ID (1001~) |
| cardCount | 카드 장수 |
| price | 골드 가격 |
| guaranteeGradeId | 이 등급 이상 1장 보장 (0 = 보장 없음). 팩 안에 없으면 마지막 장을 이 등급 이상에서 다시 뽑는다 |

## HunterCardGachaSettings.csv

`key,value` 두 열이다.

| key | 설명 |
|---|---|
| lockedCardChance | 잠긴 카드 확률(%). 잠긴 헌터가 없으면 판정하지 않는다 |
| pityCount | 이 장수 동안 천장 등급이 안 나오면 다음 장은 천장 등급 |
| pityGradeId | 천장 등급 ID |
| maxLevelRefundGold | 최대 레벨 헌터의 카드가 나오면 대신 주는 골드 |

## Skills.csv

스킬 하나에 효과는 한 개다.

| 열 | 설명 |
|---|---|
| skillId | 스킬 ID |
| name, description | 화면에 보이는 이름과 설명 |
| towerId | 귀속된 타워 |
| unlockLevel | 해금 레벨 (비우면 1) |
| upgradeOf | 대체하는 스킬 ID. 업그레이드일 때만. 같은 타워의 더 낮은 해금 레벨 스킬이어야 한다 |
| trigger | 발동 시점: `Always` / `OnAttack` / `OnHit` / `OnKill` / `EveryNthAttack` |
| nth | `EveryNthAttack`일 때 N |
| chance | 발동 확률(%). 비우면 확률 판정 없음 |
| cooldown | 쿨타임(초). 비우면 없음 |
| effectType | 효과 종류 (아래) |
| value | 수치. `%`를 붙이면 퍼센트, 없으면 고정값 |
| duration | 지속 시간(초). 비우면 상시 |
| vfxKey | 스킬이 발동할 때 재생할 연출 프리팹 이름 (PrefabKeys). 비우면 연출 없음 |
| vfxAnchor | 연출을 붙일 타워 앵커: `Root`(발밑) / `Center` / `Head` / `Front`. 비우면 Root. 타워 프리팹의 `TowerVfxAnchors`에 지정하지 않은 앵커는 Root로 대신한다 |
| animTrigger | 스킬이 발동한 공격에 쓸 애니메이터 트리거 (예: `AttackJump`). 비우면 기본 `Attack`. 타워 애니메이터에 그 트리거와 State가 있어야 하고, 클립에 `OnAttackHit`/`OnAttackEnd`(연출을 휘두르는 순간에 띄우려면 `OnAttackCast`도) 이벤트가 있어야 한다 |

스킬 발동과 치명타는 공격 시작 시점에 판정한다. 연출은 클립의 `OnAttackCast`(없으면 `OnAttackHit`)에서, 데미지는 `OnAttackHit`에서 들어간다.

스킬 아이콘은 시트에 적지 않는다. `Assets/Art/UI/SkillIcon/SkillIcon_{스킬ID}.png`(128x128)를 두고 스킬 아이콘 아틀라스로 묶어 로드한다. 스프라이트 이름은 파일 이름이고 코드에서는 `TowerSkillData.IconName`으로 얻는다.

### effectType과 수치 해석

| effectType | 허용 단위 | 의미 |
|---|---|---|
| Attack | 고정 / 퍼센트 | 공격력 +N, 또는 레벨 반영 기본 공격력의 +N% |
| AttackSpeed | 퍼센트만 | 공격속도 +N% |
| CritChance | 고정만 | 치명타 확률 +N%p |
| CritBonus | 고정만 | 치명타 보너스 +N%p |
| MultiShot | 고정만 | **추가로** 쏘는 발수 (2연발이면 1) |
| DamageMultiplier | 퍼센트만 | 공격력의 **총** 배율 (300%는 3배). 발동 시 치명타 판정을 하지 않는다 |
| Pierce | 고정만 | **추가로** 관통하는 대상 수 (1이면 총 2마리). 맞은 모든 대상에게 같은 데미지 |
| Slow | 퍼센트만 | 맞은 적의 이동속도 N% 감소. `duration`(초) 필수. 1등급 타워는 최대 5%. 중첩 규칙은 Doc/슬라임TD_타워스킬설계.md 5장 |
| Splash | 고정만 | 맞은 대상 중심 반경 N 안의 다른 슬라임에게 **같은 데미지**를 준다 (중심 대상 제외, 화살은 첫 타격 때 한 번만) |
| JellyGain | 고정만 | 젤리 +N. 공격 1회마다 확률 판정 후 바로 얻는다 (OnAttack) |
| Stun | 고정만 | 맞은 적을 `duration`(초) 동안 기절시킨다. **value는 쓰지 않는다.** 클라이언트 임포터가 해석할 수 없는 값이 있는 행을 건너뛰므로 비우지 않고 1을 넣어 둔다 |

**스킬이 발동한 공격은 치명타 판정을 하지 않는다.** 데미지에 영향을 주는 OnAttack 스킬(MultiShot, DamageMultiplier, Splash)이 터지면 그 공격의 화살 전부가 치명타 없이 같은 데미지가 된다. Always 패시브와 JellyGain은 해당하지 않는다. chance와 cooldown은 함께 쓸 수 있고, 쿨타임은 공격속도와 무관하게 발동했을 때부터 흐른다.

허용되지 않는 단위를 쓰거나 해석할 수 없는 값이 있는 행은 임포트 로그에 이유를 남기고 건너뛴다.

## Slimes.csv

클라이언트에서는 아직 `Assets/Datas/Slimes/*.asset`(ScriptableObject)이 원본이다. 이 시트는 그 값을 옮긴 것이고, 클라이언트 임포터가 생기기 전까지 양쪽 값을 같이 맞춘다.

| 열 | 설명 |
|---|---|
| slimeId | 슬라임 ID (6자리) |
| codeName | 영문 코드명. 클라이언트 에셋 이름(`Slime_01`, `Boss_01`)과 같다. 프리팹 연결 키 |
| kind | `Normal` / `Boss` |
| baseHealth | 일반: 웨이브 성장률 적용 전 기본 HP / 보스: 이 값 그대로 |
| baseSpeed | 이동 속도 |
| lifeCost | 경로 끝 도달 시 차감 라이프 |
| jellyReward | 처치 젤리 |
| instantGameOver | 보스만. 1이면 경로 끝 도달 시 즉시 게임오버, 0이면 `lifeCost`만큼 차감. 일반은 비운다. **지금은 모든 보스가 0(라이프 10 차감)**이다 |
| renderGroup | 렌더링 버킷 `Normal` / `Boss` |

## WaveTables.csv

스테이지(난이도)마다 한 행. 원본은 클라이언트 `WaveTableData` 에셋이다.

| 열 | 설명 |
|---|---|
| tableId | 웨이브 테이블 ID (2자리) |
| codeName | 영문 코드명 (`Easy` → 클라이언트 `WaveEasyTable`) |
| spawnInterval | 슬라임 간 스폰 간격(초). 전 웨이브 공통 |
| interWaveDelay | 웨이브 간 대기(초) |
| normalHpGrowth | 웨이브당 HP 증가율(복리, 0.06 = 6%). 일반 슬라임에만 적용 |

## WaveZones.csv

테이블의 구간 목록. 한 구간은 10웨이브이고, 1~9번째는 일반 슬라임, 10번째는 보스 단독이다. 최종 웨이브 = 구간 수 × 10.

| 열 | 설명 |
|---|---|
| tableId | 소속 웨이브 테이블 |
| zone | 구간 순번 (1부터, 빠짐없이) |
| slimeId | 구간 1~9번째 웨이브 슬라임 (`kind` = Normal) |
| count | 일반 웨이브 마릿수 |
| bossId | 구간 10번째 웨이브 보스 (`kind` = Boss) |
| bossExtraStartDelay | 보스 웨이브 시작 전 추가 대기(초) |

계산식: 일반 HP = `baseHealth × (1 + normalHpGrowth)^(웨이브 - 1)`, 보스 HP = `baseHealth`.

## Items.csv

게임 안에서 주고받는 모든 것(재화, 헌터 카드 팩 등)의 목록이다. 보상, 우편, 상점, 새 계정 지급은 모두 `itemId + 수량`으로 표현한다. 아이템 테이블은 "무엇이 있는가"만 정의하고, 내용물이나 효과는 종류별 시트가 정한다 (헌터 카드 팩 → `HunterCardGachaPacks.csv`).

| 열 | 설명 |
|---|---|
| itemId | 아이템 ID (7자리, 대역은 ID 규칙 참고) |
| codeName | 영문 코드명. 클라이언트 아이콘·표시 연결 키. 재화는 클라이언트 `CurrencyType`(`Gold`, `Gem`)과 같은 이름이다 |
| name | 화면에 보이는 이름 |
| itemType | `Currency`(재화) / `HunterCardPack`(헌터 카드 팩). 예정: `HunterCard`(특정 헌터 카드) |
| targetId | 연결 대상. `HunterCardPack`은 `HunterCardGachaPacks.csv`의 팩 ID |
| openMode | `OnAcquire`: 인벤토리에 들어오는 순간(우편 받기 등) 판정해서 결과로 바꿔 지급한다. 비우면 그대로 보관 |
| scope | 보관 위치. `Account`: 계정에 저장 / `Battle`: 스테이지 안에서만 쓰고 사라짐. `OnAcquire` 아이템은 보관되지 않으므로 비운다 |

- **헌터 카드 팩 판정**: 팩의 장수와 보장 등급(`HunterCardGachaPacks.csv`)과 뽑기 설정(`HunterCardGachaSettings.csv`)으로 판정한다
- **천장**: 상점에서 골드로 산 팩만 천장 카운트에 포함한다. 우편·선물·보상으로 받은 팩은 천장과 무관하게 판정한다
- **특정 헌터 카드**(`HunterCard`, 예: 궁수 카드)는 예정이다. 추가할 때 ID는 2000000+타워ID, `targetId`는 타워 ID로 한다

## NewAccount.csv

새 계정을 만들 때 지급하는 아이템. 한 행이 한 아이템이다.

| 열 | 설명 |
|---|---|
| itemId | `Items.csv`의 아이템 ID. `scope`가 `Account`인 아이템만 쓸 수 있다 |
| amount | 지급 수량 |

개발용 임시 값이다 (골드 10000, 보석 10000). 클라이언트 `GameConfig`의 헌터별 시작 카드 100장은 특정 헌터 카드 아이템이 생기면 행으로 추가한다.

## GameConstants.csv

게임 전역 상수. 행이 하나뿐인 값은 모두 여기에 둔다 (지금은 인게임 전투 규칙만 있다). `key,value` 두 열이다. 값이 늘면 키에 묶음 접두어(`summon.` 등)를 붙인다. 원본은 클라이언트 `GameConfig` 에셋이다 (에셋에 저장되지 않은 값은 코드 기본값을 옮겼다). 디버그용 값(`ignoreJellyCost`, `autoStartWave`, `testTower`, `debugTowerLevel`)은 클라이언트 개발 설정이라 넣지 않는다.

| key | 설명 |
|---|---|
| startingLife | 스테이지 시작 라이프 |
| startingJelly | 스테이지 시작 젤리 |
| summonBaseCost | 소환 비용 기본값. 비용 = 기본 + 증가 × 소환 횟수 |
| summonCostIncrease | 소환 1회당 비용 증가 |
| summonGradeId | 소환 등급 |
| slowMinSpeedRatio | 감속 하한. 기본 속도의 이 비율 밑으로 내려가지 않는다 |
| bossSlowScale | 보스에게 걸리는 감속 비율 |
