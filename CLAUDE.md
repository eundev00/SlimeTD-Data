# SlimeTD-Data

슬라임 TD의 기획(정적) 데이터 원본 저장소. 서버(`C:\Projects\SlimeTD-Server`)와 클라이언트(`C:\Projects\SlimeTD`)는 이 저장소의 데이터를 받아 쓴다.

## 시트 규칙

- 원본은 `Sheets/*.csv`다. 사람이 CSV를 직접 편집한다 (엑셀 전환은 보류)
- 형식: UTF-8 **BOM**, **CRLF** 줄바꿈, 1줄이 영문 열 이름. 새 시트도 같은 형식으로 만든다. Write 도구는 BOM 없이 LF로 쓰므로 CSV는 PowerShell(`UTF8Encoding($true)`, `` `r`n ``)로 쓴다
- 열 설명, ID 대역, 열 용도(both/server/client)는 `Sheets/README.md`에 있다. **새 시트, 열, 키, 선택지 값을 추가하면 README도 같이 고친다**
- 파일 이름에 접두어를 붙이지 않는다 (`Towers.csv`). 나중에 코드로 만들 때는 `Table` 접미어를 쓴다 (`TowerTable`)
- 행이 하나뿐인 전역 수치는 `GameConstants.csv`(`key,value`)에 둔다. 값이 늘면 키에 묶음 접두어(`summon.` 등)를 붙인다
- **이넘**(코드가 값에 따라 다르게 동작하는 종류: 모드, 난이도, 아이템 종류 등)은 `Enums.csv`에 정의하고, 다른 시트에는 이넘 이름을 적는다. `value`는 한 번 정하면 바꾸지 않는다. 행을 늘리는 목록(타워, 아이템, 재화, 스테이지)은 이넘이 아니라 시트로 둔다
- **스테이지**(모드 × 난이도)는 `Stages.csv`에 두고, 웨이브는 스테이지마다 `Sheets/Waves/<codeName>.csv` 파일 하나씩 둔다
- 보상, 우편, 상점, 새 계정 지급은 모두 `Items.csv`의 `itemId + 수량`으로 표현한다

## 시트 검사

시트를 고친 뒤 **커밋 전에** `check-sheets` 스킬(`.claude/skills/check-sheets/SKILL.md`)로 검사한다.

- 검사 항목 30개를 순서대로 모두 확인하고, 오류/경고를 표로 보고한다
- **검사만 한다.** 시트를 고치지 않고, 고칠 내용은 제안으로만 보고한다
- 오류가 있으면 커밋하지 않고 사용자에게 먼저 보고한다
- 검사 규칙을 바꾸거나 새 시트를 추가하면 스킬의 검사 항목도 같이 고친다
- 다른 저장소(서버 등)의 세션에서는 스킬이 자동으로 잡히지 않는다. 그때는 스킬 파일을 직접 읽고 따른다

## 클라이언트와의 관계

- 클라이언트는 서버 연결 전까지 이 저장소의 시트를 임시 ScriptableObject 에셋으로 옮겨 쓴다. 클라이언트에는 CSV 복사본이 없고, 클라이언트 세션이 `sheets-to-so` 스킬로 반영한다
- 이 저장소가 원본이다. 수치를 고치면 클라이언트에 반영해야 하며, 그 전까지 값이 갈라질 수 있다
- **클라이언트 코드와 에셋은 직접 수정하지 않는다.** 반영할 내용은 정리해서 사용자에게 주고, 사용자가 클라이언트 채팅방에 전달한다

## 커밋 / 푸시

**이 저장소는 서버 세션과 클라이언트 세션이 같이 커밋한다.** 그래서 순서를 지킨다.

1. **작업 전**: `git fetch` 후 원격에만 있는 커밋(`git log HEAD..origin/main`)과 로컬 미커밋 변경(`git status`)을 확인하고, `git pull --ff-only`로 받은 다음 수정을 시작한다. 받은 변경 중 지금 작업과 겹치는 파일이 있으면 내용을 먼저 읽는다
2. **푸시 전**: 다시 `git fetch`로 원격에 새 커밋이 없는지 확인한다. 있으면 받아서(`git pull --rebase`) 겹치는 부분을 확인하고 시트 검사를 다시 한 뒤 푸시한다
3. 다른 세션의 커밋을 덮어쓰거나 되돌리지 않는다. 충돌이 나면 사용자에게 먼저 보고한다

PC에 git이 PATH에 없다. 서버 저장소와 같은 방법을 쓴다.

- git 실행 파일: `C:\Users\eunv0\AppData\Local\GitHubDesktop\app-3.6.6\resources\app\git\cmd\git.exe`
- 토큰: Windows 자격 증명 관리자의 `GitHub - https://api.github.com/eundev00` 항목을 `advapi32.dll` `CredRead`로 읽고 UTF-8로 디코딩한다. **토큰은 절대 출력하거나 파일에 쓰지 않는다**
- 푸시: `git -c "http.https://github.com/.extraheader=AUTHORIZATION: basic <base64('x-access-token:'+토큰)>" push origin main`
- 저장소: `eundev00/SlimeTD-Data`. 사용자가 외부에서 확인하려고 지금은 **공개**다. 확인이 끝나면 비공개로 바꿀 예정이다
- `.gitattributes`가 CSV의 줄바꿈 변환을 막는다 (`*.csv -text`)
