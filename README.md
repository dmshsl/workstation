# 극한환경로봇연구팀 워크스테이션 사용 현황

팀원 누구나 링크 하나로 **지금 누가 워크스테이션(GPU)을 쓰고 있는지, 몇 시간 남았는지** 보고,
비어 있으면 바로 등록하는 페이지입니다. 서버 코드 없이 `index.html` 파일 하나 + Firebase(무료)로 동작합니다.

## 기능

- 비어 있으면 **"현재 사용자 없음"**, 이름 + 사용 시간 적고 등록하면 그 순간부터 시작
- 누가 쓰고 있으면 **"OOO 사용 중 · 2시간 13분 남음"** (실시간 카운트다운, 종료 예정 시각)
- 시간이 다 지나면 **자동 해제** → 다음 사람이 등록 가능
- **사용 종료** 버튼 (일찍 끝났을 때), **연장** 버튼 (+30분 / +1시간 / +2시간)
- **다음 예약(대기)**: 사용 중일 때 **이름만** 올려두는 명단. 자동으로 시작되지 않고, 비면 본인이 시간을 정해 **직접 시작**
- **최근 사용 기록** 20건
- 비면 알려주는 브라우저 알림 (탭이 열려 있는 동안), 탭 제목/파비콘에 상태 표시
- 동시에 두 명이 등록해도 한 명만 잡히고 나머지는 대기열로 (Firebase 트랜잭션)

---

## 설정 순서 (처음 한 번만, 약 10분)

### 1. Firebase 프로젝트 만들기

1. https://console.firebase.google.com 접속 (구글 계정 로그인)
2. **프로젝트 추가** → 이름 예: `gpu-board` → 계속
3. "Google 애널리틱스 사용 설정"은 **꺼도** 됩니다 → **프로젝트 만들기**

### 2. Realtime Database 만들기

1. 왼쪽 메뉴 **빌드 → Realtime Database** → **데이터베이스 만들기**
2. 위치: **싱가포르 (asia-southeast1)** 추천 (한국에서 가장 가까움)
3. 보안 규칙: 아무거나 골라도 됩니다 (다음 단계에서 교체) → **사용 설정**
4. 화면 위쪽에 보이는 주소를 메모해 두세요. 이런 모양입니다:
   `https://gpu-board-xxxxx-default-rtdb.asia-southeast1.firebasedatabase.app`

### 3. 보안 규칙 붙여넣기

1. Realtime Database 화면의 **규칙** 탭
2. 내용을 전부 지우고 이 저장소의 `database.rules.json` 내용을 **통째로 붙여넣기** → **게시**

> "테스트 모드"로 만들면 30일 뒤에 자동으로 잠겨서 페이지가 멈춥니다.
> 위 규칙을 직접 게시해 두면 만료 없이 계속 동작하고, `board` 경로 외에는 아무것도 열리지 않습니다.

### 4. 웹 앱 등록하고 설정값 복사

1. 왼쪽 위 **프로젝트 개요** 옆 톱니바퀴 ⚙ → **프로젝트 설정**
2. 아래 **내 앱** → 웹 아이콘 **`</>`** 클릭
3. 앱 닉네임 아무거나 (예: `board`) → "Firebase 호스팅 설정"은 체크 **안 함** → **앱 등록**
4. 화면에 나오는 코드 중 아래 부분만 복사:
   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "gpu-board-xxxxx.firebaseapp.com",
     databaseURL: "https://gpu-board-xxxxx-default-rtdb.asia-southeast1.firebasedatabase.app",
     projectId: "gpu-board-xxxxx",
     storageBucket: "gpu-board-xxxxx.firebasestorage.app",
     messagingSenderId: "1234567890",
     appId: "1:1234567890:web:abcdef"
   };
   ```
5. `index.html` 을 메모장/VS Code로 열어 `▼▼▼ 1) Firebase 설정` 표시 아래의
   `const firebaseConfig = { ... };` 블록을 **복사한 것으로 통째로 교체** → 저장
6. ⚠️ 복사한 코드에 **`databaseURL` 줄이 없으면** 2-4에서 메모한 주소를 직접 한 줄 추가하세요.
   (데이터베이스를 나중에 만들면 이 줄이 빠진 채로 나올 때가 있습니다.)

여기까지 하고 `index.html` 을 더블클릭해 열었을 때 위쪽의 **"데모 모드" 배너가 사라지고**
오른쪽 위에 **"실시간 연결됨"** 이 뜨면 성공입니다.

### 5. GitHub Pages 에 올리기

1. https://github.com/new → 저장소 이름 예: `gpu-board` → **Public** → **Create repository**
   (Pages 무료 사용은 Public 저장소여야 합니다. 설정값이 공개돼도 괜찮은 이유는 아래 FAQ 참고)
2. **Add file → Upload files** → `index.html`, `database.rules.json`, `README.md` 끌어다 놓기 → **Commit changes**
3. 저장소 **Settings → Pages** → Build and deployment의 Source: **Deploy from a branch**
   → Branch: **main** / **(root)** → **Save**
4. 1~2분 뒤 `https://<깃허브아이디>.github.io/gpu-board/` 로 접속 → 이 링크를 팀에 공유

### 6. 확인

- 휴대폰과 PC 두 곳에서 열고 한쪽에서 등록 → 다른 쪽이 새로고침 없이 바로 바뀌면 끝.

---

## 자주 묻는 것

**"데모 모드" 배너가 계속 떠요**
`index.html` 의 `firebaseConfig` 가 아직 예시 값(`여기에_붙여넣기`)입니다. 4번 단계를 다시 확인하세요.
GitHub에 올린 뒤라면 수정한 파일을 다시 업로드해야 합니다.

**"데이터베이스 권한 오류" 가 떠요**
3번 규칙을 **게시**하지 않았거나, `databaseURL` 이 다른 프로젝트 주소입니다.

**설정값(apiKey)이 공개돼도 괜찮나요?**
Firebase 웹 apiKey는 비밀번호가 아니라 프로젝트 식별자라서 공개 저장소에 두는 게 정상입니다.
접근 제어는 3번 규칙이 담당합니다. 다만 이 규칙은 **주소를 아는 사람은 누구나 현황판을 고칠 수 있게**
열어 둔 것이므로, 링크는 팀 안에서만 공유하세요.

**대기 예약은 언제 시작되나요?**
**자동으로 시작되지 않습니다.** 대기는 시간 없이 이름만 올려두는 명단입니다.
앞 사람이 끝나면 현황판이 "사용 가능 · 대기자 N명"으로 바뀌고, 대기자 줄마다
**사용 시작** / **대기 취소** 버튼이 나타납니다. 카톡 등으로 서로 협의한 뒤
실제로 쓸 사람이 그때 시간을 정해 **사용 시작**을 누르면 됩니다.

새벽에 앞 사람이 끝나도 타이머가 혼자 돌지 않기 때문에, 자는 동안 사용 시간이
날아가는 일이 없습니다.

**아무도 페이지를 안 열어도 시간이 지나면 해제되나요?**
네. 종료 시각이 데이터에 저장돼 있어서, 누가 언제 열든 그 시각을 넘겼으면 "사용자 없음"으로 보입니다.

**최대 시간, 기록 개수 바꾸기**
`index.html` 위쪽 `SETTINGS` 에서 `minHours`, `maxHours`, `historyLimit` 를 바꾸면 됩니다.
팀 이름/제목은 `<title>`, `.eyebrow`, `<h1>` 부분의 글자를 바꾸면 됩니다.

**비용**
Firebase 무료(Spark) 요금제로 충분합니다. 이 페이지는 데이터가 몇 KB 수준이라 한도의 1%도 쓰지 않습니다.

**미리 만져보기**
Firebase 설정 전에도 `index.html` 을 열면 데모 모드로 모든 기능을 써볼 수 있습니다 (그 화면 안에서만 동작, 새로고침하면 초기화).

## 파일

| 파일 | 설명 |
|---|---|
| `index.html` | 페이지 전체 (화면 + 동작). Firebase 설정값만 넣으면 됨 |
| `database.rules.json` | Firebase Realtime Database 보안 규칙 (규칙 탭에 붙여넣기) |
| `test/` | 로직 검증 스크립트 (`node test/logic.test.mjs`). 업로드 안 해도 됨 |
