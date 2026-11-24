# 시트 편집 가이드

이 폴더가 기획 데이터의 원본이다. 서버와 클라이언트는 여기서 받아 쓰고, 각자 복사본을 고치지 않는다.

스테이지별 웨이브는 `Waves/` 폴더에 스테이지마다 파일 하나씩 둔다. 시트는 UTF-8(BOM) CSV, CRLF 줄바꿈이고 **1줄이 영문 열 이름**이다. 새 시트도 같은 형식으로 만든다.

클라이언트는 서버 연결 전까지 이 시트를 임시 ScriptableObject 에셋으로 옮겨 쓴다. 클라이언트 프로젝트에는 CSV 복사본도 임포트 메뉴도 없고, 클라이언트 저장소의 `sheets-to-so` 스킬이 시트를 읽어 에셋 값을 직접 고친다. 클라이언트에서 숫자를 인스펙터로 고치면 다음 반영 때 덮어써진다.

## ID 규칙

모든 데이터의 ID는 종류와 상관없이 겹치면 안 된다. 한 번 정한 ID는 바꾸지 않는다.

| 종류 | 규칙 |
|---|---|
| 등급 | 1~4 |
| 타워 | 101~ (3자리) |
| 스킬 | 타워ID×100+순번 (궁수 10101~, 일병 10301~). 추가는 그 타워의 다음 빈 번호 |
| 뽑기 팩 | 1001~ (4자리) |
| 스테이지 | 2자리. 앞자리 = 모드(`GameMode` 값), 뒷자리 = 난이도(`Difficulty` 값). 11 싱글 보통, 12 싱글 어려움, 13 싱글 지옥, 21 멀티 보통, 22 멀티 어려움, 23 멀티 지옥 |
| 슬라임 | 6자리. 일반 100001~, 보스 100101~ (등장 순서대로 번호를 붙인다) |
| 아이템 | 7자리. 재화 1000001~, 특정 헌터 카드 2000000+타워ID (예정, 궁수 101 → 2000101). 2100000~ 대역(헌터 카드 팩 아이템)은 2026-11-24에 없앴다. 번호는 다시 쓰지 않는다 |

`Enums.csv`의 `value`는 ID가 아니다. 이넘 안에서만 겹치지 않으면 된다.

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
| Stages | name | client | 화면 표시용 |
| Enums | description | client | 설명용 |
| Items | name | client | 화면 표시용 |
| NewAccount | (전체) | server | 새 계정 생성은 서버만 한다 |

프리팹, 썸네일, 머티리얼 같은 Unity 에셋 연결은 시트에 넣지 않는다. 타워·슬라임 프리팹은 클라이언트가 `codeName`과 같은 이름의 프리팹을 로드하고, 썸네일 같은 나머지는 인스펙터에서 연결한다.

## Enums.csv

이넘 정의. 이넘은 **코드가 값에 따라 다르게 동작하는 종류**이고, 새 값을 추가하면 서버·클라이언트 코드도 같이 바꿔야 한다. 행을 늘리는 목록(타워, 아이템, 재화, 스테이지 등)은 이넘이 아니라 시트(ID)로 둔다. 재화가 이넘이 아닌 이유도 같다 (`Items.csv`).

| 열 | 설명 |
|---|---|
| enumName | 이넘 이름 |
| name | 값 이름. **다른 시트의 선택지 열에는 이 이름을 적는다** |
| value | 정수 값. DB와 코드에 숫자로 저장되므로 **한 번 정하면 바꾸지 않는다**. 새 값은 맨 뒤 번호로 추가한다 |
| description | 설명 |

- 새 이넘은 1부터 시작하고 0은 "없음"으로 비워 둔다
- 클라이언트에 이미 있는 이넘(`SkillTrigger`, `SkillEffectType`, `VfxAnchor`, `SlimeRenderGroup`, `TowerAttackType`)은 에셋·프리팹에 숫자로 저장돼 있을 수 있어 **클라이언트 값(0부터)을 그대로** 쓴다

| 시트.열 | 이넘 |
|---|---|
| Stages.mode | GameMode |
| Stages.difficulty | Difficulty |
| Towers.attackType | TowerAttackType |
| Skills.trigger | SkillTrigger |
| Skills.effectType | SkillEffectType |
| Skills.vfxAnchor | VfxAnchor |
| Slimes.kind | SlimeKind |
| Slimes.renderGroup | SlimeRenderGroup |
| Items.itemType | ItemType |
| Items.openMode | OpenMode |
| Items.scope | ItemScope |

## Grades.csv

| 열 | 설명 |
|---|---|
| gradeId | 등급 ID |
| name | 등급 이름. 헌터 상세 팝업의 등급 배지와 태그에 표시된다 |
| baseAttack | 기준 공격력 |
| attackGrowthRate | 레벨당 공격력 증가율(%) |
| gachaWeight | 뽑기 등급 가중치. 보유 헌터가 없는 등급은 빼고 남은 가중치끼리 비율을 다시 계산한다. 0이면 뽑기에서 나오지 않는다 (전설은 0: 다이아 구매·이벤트로만 얻는다) |

## LevelUpCosts.csv

등급별 레벨업 비용. 한 행이 "그 등급의 한 레벨" 비용이다.

| 열 | 설명 |
|---|---|
| gradeId | 등급 ID |
| level | 올리기 전 레벨. 등급마다 1부터 14까지 빠짐없이 있어야 한다 (1이면 1 -> 2) |
| cards | 필요한 그 헌터의 카드 수. 0이면 카드 없이 코인만 쓴다 (전설은 뽑기에서 나오지 않으므로 0) |
| coin | 필요 코인 |

값은 클라이언트 저장소 `Doc/슬라임TD_메타재화밸런스.md` 6장(2026-11-24 확정)을 옮긴 것이다. 레벨 15가 최대이고, 최대 레벨 헌터 카드는 뽑기에서 코인으로 환급한다 (`HunterCardGachaSettings.maxLevelRefundCoin`).

## Towers.csv

| 열 | 설명 |
|---|---|
| towerId | 타워 ID (101~) |
| codeName | 영문 코드명. 에셋 이름 `TowerData_<코드명>`에 쓰인다 |
| name | 화면에 보이는 이름 |
| gradeId | 등급 ID |
| attackType | 공격 방식 `TowerAttackType`: `Melee`(근접) / `Projectile`(투사체) / `Beam`(즉시 타격 빔). `Projectile`이면 `TowerProjectiles.csv`에 행이 있어야 한다. 투사체가 어떻게 생겼는지(프리팹)는 클라이언트가 정한다 |
| enabled | 활성 (1/0) |
| attackRange | 사거리 |
| attackCooldown | 공격 쿨다운(초) |
| critChance | 치명타 확률(%) |
| critBonus | 치명타 보너스(%) |
| startUnlocked | 시작 해금 (1/0). 타워마다 지정한다. 비우면 잠금. **기본 정책: 일반·희귀·영웅 헌터는 모두 1(레벨 1로 보유한 채 시작). 전설은 0 (다이아 구매 또는 이벤트 지급)** |
| unlockDiamondPrice | 해금 다이아 가격 (0 = 구매 불가). 전설 가격은 미정 |
| inGacha | 뽑기 대상 (1/0). 비우면 1. 뽑기에서는 해금(보유)한 헌터 카드만 나온다 (잠긴 헌터는 나오지 않는다) |

## TowerProjectiles.csv

투사체를 쓰는 타워(`attackType=Projectile`)만 한 행씩 둔다. 키는 `towerId`이고, 타워와 1:1이라 별도 ID가 없다. 투사체 관련 값이 늘어나면 `Towers.csv`가 아니라 여기에 열을 추가한다. 투사체는 발사 시점의 목표 위치로 직진하며 추적하지 않는다 (경로상 다른 슬라임과 부딪히면 그쪽을 맞힌다).

| 열 | 설명 |
|---|---|
| towerId | 타워 ID (`Towers.csv`) |
| speed | 비행 속도 (초당 거리) |
| hitRadius | 충돌 판정 반경. 비행 경로에서 이 반경 안에 들어온 슬라임을 맞힌다 |
| lifetime | 수명(초). 이 시간 안에 아무것도 못 맞히면 사라진다 |

- 값은 클라이언트 투사체 프리팹(`ArcherArrow`, `CrossbowBolt`, `GoodSlimeBall`)의 `Projectile` 컴포넌트에서 옮겼다. 모노리스(301)는 빔처럼 보이는 투사체이고 값은 착한 슬라임과 같다. 클라이언트는 이 시트 값을 `TowerData`로 받아 투사체에 적용하고, 프리팹은 모양만 정한다
- 빔 전용 값이 생기면 같은 방식으로 `TowerBeams.csv`를 만든다

## HunterCardGachaPacks.csv

| 열 | 설명 |
|---|---|
| packId | 팩 ID (1001~) |
| cardCount | 카드 장수 |
| price | 팩 티켓(`PackTicket`) 가격. 카드 1장 = 티켓 10 (1장 팩 10, 10장 팩 100) |
| guaranteeGradeId | 이 등급 이상 1장 보장 (0 = 보장 없음). 팩 안에 없으면 마지막 장을 이 등급 이상에서 다시 뽑는다 (그 등급 보유 헌터가 없으면 그대로) |

뽑기 규칙 (2026-11-24): 장마다 등급을 `Grades.gachaWeight`로 정하고, 그 등급의 보유(해금) 헌터 중 하나가 나온다. **천장과 잠긴 카드 규칙은 없다.**

## HunterCardGachaSettings.csv

`key,value` 두 열이다.

| key | 설명 |
|---|---|
| maxLevelRefundCoin | 최대 레벨 헌터의 카드가 나오면 대신 주는 코인 (장당) |

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
| GoldGain | 고정만 | 골드 +N (인게임). 공격 1회마다 확률 판정 후 바로 얻는다 (OnAttack) |
| Stun | 고정만 | 맞은 적을 `duration`(초) 동안 기절시킨다. **value는 쓰지 않는다.** 비우지 않고 1을 넣어 둔다 |

**스킬이 발동한 공격은 치명타 판정을 하지 않는다.** 데미지에 영향을 주는 OnAttack 스킬(MultiShot, DamageMultiplier, Splash)이 터지면 그 공격의 화살 전부가 치명타 없이 같은 데미지가 된다. Always 패시브와 GoldGain은 해당하지 않는다. chance와 cooldown은 함께 쓸 수 있고, 쿨타임은 공격속도와 무관하게 발동했을 때부터 흐른다.

허용되지 않는 단위를 쓰거나 해석할 수 없는 값이 있는 행은 클라이언트에 반영하지 않고 보고한다.

## Slimes.csv

클라이언트는 행마다 `SlimeData_<codeName>` 에셋으로 옮겨 쓴다.

| 열 | 설명 |
|---|---|
| slimeId | 슬라임 ID (6자리) |
| codeName | 영문 코드명. 클라이언트 슬라임 프리팹 이름(`Slime_01`, `Boss_01`)과 같고, 클라이언트는 이 이름으로 프리팹을 로드한다 |
| kind | `Normal` / `Boss` |
| baseHealth | 일반: 웨이브 성장률 적용 전 기본 HP / 보스: 이 값 그대로 |
| baseSpeed | 이동 속도 |
| lifeCost | 경로 끝 도달 시 차감 라이프 |
| goldReward | 처치 골드 (인게임) |
| instantGameOver | 보스만. 1이면 경로 끝 도달 시 즉시 게임오버, 0이면 `lifeCost`만큼 차감. 일반은 비운다. **지금은 모든 보스가 0(라이프 10 차감)**이다 |
| renderGroup | 렌더링 버킷 `Normal` / `Boss` |

## Stages.csv

스테이지 = 플레이어가 고르는 게임 하나 (모드 × 난이도). 웨이브는 `Waves/<codeName>.csv`에 있다. 지금은 싱글 보통(11)과 멀티 보통(21)이 있고, 나머지는 만들 때 ID 규칙대로 추가한다. 멀티는 두 사람이 맵 하나를 같이 막는 협동이다. 맵 입구가 여럿이면 무리마다 슬라임을 입구에 번갈아 내보내 입구마다 같은 수가 나온다 (멀티 보통은 싱글 보통 웨이브를 복사해 시작했다)

| 열 | 설명 |
|---|---|
| stageId | 스테이지 ID (2자리, ID 규칙 참고) |
| codeName | 영문 코드명. 웨이브 파일 이름이 된다 (`SingleNormal` → `Waves/SingleNormal.csv`) |
| mode | `GameMode` (`Single` / `Multi`) |
| difficulty | `Difficulty` (`Normal` / `Hard` / `Hell`) |
| name | 화면에 보이는 이름 |
| enabled | 1/0. 0이면 고를 수 없다 |
| startingLife | 시작 라이프 |
| startingGold | 시작 골드 (인게임) |
| spawnInterval | 한 무리 안에서 슬라임이 나오는 간격(초) |
| interWaveDelay | 웨이브 사이 기본 대기(초) |
| normalHpGrowth | 웨이브당 HP 증가율(복리, 0.06 = 6%). 일반 슬라임에만 적용 |
| mapKey | 맵 이름. 클라이언트는 같은 이름의 맵 프리팹(`Assets/Prefabs/Game/Map/<mapKey>.prefab`)을 로드한다. 맵의 그리드와 경로는 아직 클라이언트에만 있고, 서버가 쓰게 되면 이 키로 맵 데이터를 찾는다 |

## StageRewards.csv

스테이지를 한 판 끝낼 때 계정에 주는 보상. 서버가 전투 결과를 확인(검증 일치)했을 때만 준다. 한 스테이지에 여러 줄을 둘 수 있다. 줄이 없는 스테이지는 보상이 없다

| 열 | 설명 |
|---|---|
| stageId | `Stages.csv` stageId |
| rewardType | `Clear`: 클리어 보상 (실패하면 줄여서 준다) / `FirstClear`: 그 스테이지를 처음 클리어할 때 한 번 |
| itemId | `Items.csv` itemId. 계정 재화(`scope=Account`, `itemType=Currency`)만 |
| amount | 지급 수량 (1 이상) |

- 클리어(마지막 웨이브까지 다 막음): `Clear` 보상 전부. 처음이면 `FirstClear` 보상도
- 실패: `Clear` 보상 × (통과 구간 ÷ 전체 구간) × `GameConstants.reward.failRate`, 버림. 통과 구간 = 도달 웨이브 ÷ `reward.segmentWaves`, 전체 구간 = 최종 웨이브 ÷ `reward.segmentWaves` (둘 다 버림). 첫 구간을 못 넘기면 보상이 없다
- 예) 싱글 보통(50웨이브, 10웨이브 구간) 30웨이브 실패 → 3/5 × 0.5 = 0.3 → 팩 티켓 117, 코인 210, 경험치 30
- 경험치는 받는 즉시 유저 레벨업에 쓰인다 (`GameConstants.userLevel.*`)

지금 값 (2026-11-24, `Doc/슬라임TD_메타재화밸런스.md` 5장): 싱글 보통(11) 클리어 = 팩 티켓 390, 코인 700, 경험치 100 / 최초 클리어 = 다이아 10

## Waves/<codeName>.csv

한 스테이지의 웨이브. 파일 하나가 스테이지 하나이고, 파일 이름이 `Stages.codeName`이다. **한 행이 한 웨이브에 나오는 몬스터 한 무리**다.

| 열 | 설명 |
|---|---|
| wave | 웨이브 번호. 1부터 빠짐없이 이어진다. 마지막 번호가 그 스테이지의 최종 웨이브다 (스테이지마다 다를 수 있다) |
| order | 웨이브 안에서 나오는 순서. 웨이브마다 1부터 빠짐없이 |
| slimeId | 나오는 슬라임 (`Slimes.csv`) |
| count | 마릿수 |
| delay | 이 무리가 나오기 전 추가 대기(초). 비우면 0 |

- 진행: 웨이브 시작 → 무리를 `order` 순서대로 내보낸다 (무리 앞에 `delay`, 무리 안 슬라임은 `spawnInterval` 간격) → 다 나오면 `interWaveDelay` 후 다음 웨이브. 이전 웨이브 슬라임이 남아 있어도 넘어가고, 마지막 웨이브만 슬라임이 모두 처리돼야 클리어다
- 보스(`kind=Boss`)가 있는 웨이브가 보스 웨이브다. 따로 표시하지 않는다. 보스 웨이브도 일반 슬라임이 먼저 나오고, 보스 무리의 `delay`만큼 기다린 뒤 보스가 나온다
- HP: 일반 = `baseHealth × (1 + normalHpGrowth)^(웨이브 - 1)`, 보스 = `baseHealth`
- `SingleNormal`은 10웨이브 구간 5개다. 구간마다 웨이브 1~10에 그 구간의 일반 슬라임이 나오고, 10번째 웨이브는 일반 슬라임 뒤에 보스가 추가 대기(1·2구간 0.5초, 3~5구간 1초) 후 나온다. 슬라임 종류, 마릿수, 대기는 처음에 클라이언트 `WaveEasyTable` 에셋 값을 옮겨 만들었다. 클라이언트는 이 시트를 `StageData` 에셋으로 옮겨 그대로 쓴다
- 이벤트가 생기면 열을 추가하거나 짝 파일(`Waves/<codeName>.Events.csv`)을 둔다

## Items.csv

게임 안에서 주고받는 모든 것(지금은 재화)의 목록이다. 보상, 우편, 상점, 새 계정 지급은 모두 `itemId + 수량`으로 표현한다. 아이템 테이블은 "무엇이 있는가"만 정의하고, 내용물이나 효과는 종류별 시트가 정한다. 헌터 카드 팩은 아이템이 아니다: 상점에서 팩 티켓(`PackTicket`)으로 바로 연다 (`HunterCardGachaPacks.csv`). 팩을 보상으로 주고 싶으면 팩 티켓을 준다.

| 열 | 설명 |
|---|---|
| itemId | 아이템 ID (7자리, 대역은 ID 규칙 참고) |
| codeName | 영문 코드명. 클라이언트 아이콘·표시 연결 키. 재화는 클라이언트 `CurrencyType`과 같은 이름이다. 메타 재화는 `Coin`(헌터 레벨업, 상점), `Diamond`(전설 헌터 구매, 충전 등), `PackTicket`(헌터 카드 팩), `Exp`(유저 레벨, 계정에는 지금 레벨 안에서 쌓인 양), `Energy`(스테이지 입장, 시간 회복), 인게임 재화는 `Gold`(소환), `Jelly`(강화, 지급처는 추후 작업)다. 서버는 재화를 이 codeName으로 찾는다 |
| name | 화면에 보이는 이름 |
| itemType | `Currency`(재화). 예정: `HunterCard`(특정 헌터 카드) |
| targetId | 연결 대상. 지금 쓰는 아이템은 없다 (예정: `HunterCard`는 타워 ID) |
| openMode | `OnAcquire`: 인벤토리에 들어오는 순간(우편 받기 등) 판정해서 결과로 바꿔 지급한다. 비우면 그대로 보관 |
| scope | 보관 위치. `Account`: 계정에 저장 / `Battle`: 스테이지 안에서만 쓰고 사라짐. `OnAcquire` 아이템은 보관되지 않으므로 비운다 |

- **특정 헌터 카드**(`HunterCard`, 예: 궁수 카드)는 예정이다. 추가할 때 ID는 2000000+타워ID, `targetId`는 타워 ID로 한다

## NewAccount.csv

새 계정을 만들 때 지급하는 아이템. 한 행이 한 아이템이다.

| 열 | 설명 |
|---|---|
| itemId | `Items.csv`의 아이템 ID. `scope`가 `Account`인 아이템만 쓸 수 있다 |
| amount | 지급 수량 |

코인·다이아 10000은 개발용 임시 값이다. 에너지는 최대치(30)로 시작하고, 팩 티켓·경험치는 0으로 시작한다. **헌터 카드(레벨업 재료)는 0장으로 시작한다.** 헌터 보유 여부는 이 시트가 아니라 `Towers.csv`의 `startUnlocked`가 정한다. 클라이언트 `GameConfig`의 헌터별 시작 카드 100장은 레벨업 테스트용 임시 값이라 이 시트에 넣지 않는다.

## GameConstants.csv

게임 전역 상수. 행이 하나뿐인 값은 모두 여기에 둔다 (인게임 전투 규칙과 계정 성장 규칙). `key,value` 두 열이다. 값이 늘면 키에 묶음 접두어(`summon.` 등)를 붙인다. 처음 값은 클라이언트 `GameConfig` 에셋에서 옮겼다 (에셋에 저장되지 않은 값은 코드 기본값을 옮겼다). 스테이지마다 다를 수 있는 값(시작 라이프, 시작 골드)은 `Stages.csv`에 둔다. 디버그용 값(`ignoreGoldCost`, `autoStartWave`, `testTower`, `debugTowerLevel`)은 클라이언트 개발 설정이라 넣지 않는다.

| key | 설명 |
|---|---|
| summonBaseCost | 소환 비용 기본값. 비용 = 기본 + 증가 × 소환 횟수 |
| summonCostIncrease | 소환 1회당 비용 증가 |
| summonGradeId | 소환 등급 |
| slowMinSpeedRatio | 감속 하한. 기본 속도의 이 비율 밑으로 내려가지 않는다 |
| bossSlowScale | 보스에게 걸리는 감속 비율 |
| energy.max | 에너지 시간 회복 최대치. 보상·이벤트로는 넘겨 가질 수 있다 |
| energy.costPerBattle | 스테이지 1판 에너지 (싱글 시작 요청 때 쓴다. 모자라면 시작할 수 없다) |
| energy.recoverMinutes | 에너지 1 회복에 걸리는 분 (최대치까지) |
| userLevel.expBase | 유저 레벨업 필요 경험치 = expBase + expPerLevel × 지금 레벨 |
| userLevel.expPerLevel | 〃 |
| reward.segmentWaves | 결과 보상 구간 크기(웨이브). 통과 구간 = 도달 웨이브 ÷ 이 값 (버림) |
| reward.failRate | 실패 보상 비율. 실패 보상 = 클리어 보상 × (통과 구간 ÷ 전체 구간) × 이 값 (버림) |

계정 성장 값은 클라이언트 저장소 `Doc/슬라임TD_메타재화밸런스.md`(2026-11-24 확정)를 옮긴 것이다. 레벨업 보상은 없다.
