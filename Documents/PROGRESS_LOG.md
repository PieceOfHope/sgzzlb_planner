# 진행 기록 (Progress Log)

다른 세션에서 이어서 작업할 수 있도록, 지금까지 한 일과 그 배경을 시간순으로 정리한 문서입니다. 새 세션을 시작하면 이 문서부터 읽으면 됩니다.

> 마지막 갱신: 2026-08-20

---

## 0. 두 프로젝트의 관계

- **`newslg-resource-planner`** (`D:\02_SourceCode\newslg-resource-planner`) — "천하결전" 게임용 자원 계산기. 원본이자 계속 별도로 운영 중인 프로젝트.
- **`sgzzlb_tech_tree`** (`D:\02_SourceCode\sgzzlb_tech_tree`, 이 프로젝트) — 위 계산기를 "삼국지 전략판" 게임에 맞게 이식하는 신규 프로젝트. GitHub: `https://github.com/PieceOfHope/sgzzlb_planner.git` (브랜치 `main`).

---

## 1. `newslg-resource-planner`에서 있었던 일 (이식 작업 이전)

이식을 시작하기 직전, 원본 프로젝트에서 버그 두 건을 고쳤습니다. 이 교훈은 이식 작업에도 그대로 적용해야 해서 [PORTING_GUIDE.md](./PORTING_GUIDE.md)·[CODE_STRUCTURE.md](./CODE_STRUCTURE.md)에도 반영해 두었습니다.

1. **1차 수정** (커밋 `d179d6b`, 이전 세션): 탭4(주요 건물 최적화) 표가 키 입력마다 `innerHTML`로 통째로 다시 그려지면서, 입력 중이던 input의 포커스가 끊겨 두 자리 이상 숫자를 입력해도 한 자리만 반영되는 문제. `activeId`/`activeVal`을 저장해뒀다가 재렌더 후 재포커스하는 방식으로 1차 수정.
2. **2차 수정** (커밋 `7f0556f`, 이번 세션 시작 직후): 1차 수정 후에도 같은 증상이 재발한다는 신고를 받고 재확인. 원인은 "새로 생성된 `<input type=number>`에 `.focus()`를 호출하면 값 전체가 선택 상태가 되어, 바로 다음 키 입력이 이어붙지 않고 전체를 덮어써버림"이었음. `restoreCaretToEnd()` 헬퍼(타입을 잠깐 `text`로 바꿔 `setSelectionRange`로 커서를 끝으로 이동 후 원복)를 추가해 해결. 실제 브라우저에서 키 입력 시뮬레이션으로 재현·검증까지 완료.

> 이식 작업 중 새 입력 표를 만들 때는 이 두 가지(포커스 보존 + 캐럿 위치 복원)를 처음부터 챙길 것 — [CODE_STRUCTURE.md §6](./CODE_STRUCTURE.md#6-탭-4--주요-건물-최적화-2684~3251행)의 `restoreCaretToEnd` 항목 참고.

---

## 2. 이식 프로젝트 착수 (`sgzzlb_tech_tree`)

### 2.1 스캐폴드 생성
- `newslg-resource-planner/resource-planner.html`을 그대로 복사해 `sgzzlb_tech_tree/resource-planner.html`로 가져옴 (아직 삼국지 전략판 데이터 미적용 상태 — 원본 그대로).
- [PORTING_GUIDE.md](./PORTING_GUIDE.md): 무엇을 그대로 재사용하고(아키텍처 패턴, 포커스 보존 기법 등) 무엇을 새로 설계해야 하는지(자원 종류, 건물/기술 데이터, 비용 공식, 자원상자/둔전 같은 특수 시스템), 진행 순서 8단계, 지금 결정 필요한 항목을 정리.
- [CODE_STRUCTURE.md](./CODE_STRUCTURE.md): 복사한 HTML을 줄 단위로 훑으며 각 함수에 🟢(그대로 재사용)/🟡(구조만 재사용)/🔴(전면 재작성) 등급을 매긴 코드 워크스루.

### 2.2 `yoonsb/` 서브 프로젝트 발견
사용자가 `sgzzlb_tech_tree/yoonsb/`에 별도 도구를 넣어둠 — 확인해보니:
- **개념이 전혀 다른 도구**: 자원 계산기가 아니라 "동맹원 200명+이 공유하는 테크트리 체크리스트"(정해진 업그레이드 순서를 각자 체크, Firebase Realtime Database로 실시간 공유·저장).
- 구성: `yoonsb/index.html`(단일 파일 앱) + `yoonsb/README.md`(Firebase 프로젝트 생성 → GitHub Pages 배포 가이드).
- 데이터(`SECTIONS` 배열)는 "아조씨 테크트리 Ver.02"라는 커뮤니티 공유 빌드오더로 보임 — 군왕전 레벨 구간별 업그레이드 순서 + 메모.

### 2.3 허브 페이지 + Git/GitHub 연결
- `index.html` (루트) 생성: 카드 2개로 `resource-planner.html`(자원 계산기)과 `yoonsb/index.html`(테크트리 체크리스트)을 각각 링크.
- `git init` → 커밋 → GitHub 원격 저장소(`https://github.com/PieceOfHope/sgzzlb_planner.git`) 연결 및 push.
- 커밋 히스토리:
  1. `86d6330` — 초기 커밋 (`resource-planner.html` 복사본 + `PORTING_GUIDE.md`/`CODE_STRUCTURE.md`)
  2. `6342728` — `index.html` 허브 페이지 + `yoonsb/` 추가
  3. `fb96cb5` — **협업자 `Kbloodhound`(`yunsb33@gmail.com`)가 직접 push**: `yoonsb/index.html`의 Firebase 설정값(`firebaseConfig`) 기입. (이미 push 권한이 있는 상태였음 — 별도 초대 조치 불필요했음)
  4. `fe0e3fd` — `sgzzlb.xlsx` 파싱 결과 추가 (아래 §3)

### 2.4 알아둘 것 — Firebase 설정값 공개
`fb96cb5`로 `yoonsb/index.html`의 `firebaseConfig`(및 `ADMIN_CODE`)가 실제 값으로 채워져 **공개(public) 저장소에 그대로 올라가 있음**. `apiKey`는 Firebase 설계상 원래 공개되는 값이라 그 자체로 위험하지는 않지만(보안은 Realtime Database 규칙이 담당), `ADMIN_CODE`를 바꿨다면 그것도 공개돼 있다는 점은 인지하고 있을 것.

---

## 3. `sgzzlb.xlsx` 파싱 (이번 세션 마지막 작업)

사용자가 `sgzzlb_tech_tree/sgzzlb.xlsx`를 추가. 시트 2개:

### 3.1 시트 "건물자원" → [`Documents/data/buildings.json`](./data/buildings.json)
- 원본 구조: 건물마다 4~5열짜리 그룹(건물명 헤더 + 목재/철광/석재 + 선택적 효과)이 가로로 27개 나열된 "와이드 포맷" 표. 그룹 폭이 건물마다 달라(효과 열 유무) 위치 기반 파싱 필요했음.
- 파싱 결과: **27개 건물**, 각 레벨별 `{cost:{w,i,s}, effect?}`. 예: 군왕전(maxLv10), 창고(maxLv20), 농장/채석장/벌목장/제련소(maxLv30), 좌민부(maxLv3), 민가/군영(maxLv20), 사령단(maxLv3), 징병소/협력/병사전×4(maxLv10), 병영×5(maxLv10), 구궁도/팔괘진/사기단/원구단/숙소/열병대(maxLv5).
- **원본(천하결전)과 달리 자원이 4종(목재/철광/석재/식량) 이상으로 보임** — "효과" 텍스트에 "식량 생산량", "동전 생산량", "장수 휘하 병력수" 등이 등장. 아직 `w/i/s` 3종 비용만 확인됐고, **식량/동전이 건물 업그레이드 "비용"으로도 쓰이는지는 이 시트만으론 불확실** (다음 세션에서 확인 필요, §4 참고).

### 3.2 시트 "임무표" → [`Documents/data/missions.json`](./data/missions.json)
- **원본에는 없던 완전히 새로운 개념**: 시즌 진행 미션(퀘스트) 목록. 챕터("장") 1~10 각각에 여러 임무가 있고, 임무마다 명성(reputation) + 자원 보상(목재/철광/석재/식량/동전/금화)이 있으며, 챕터 끝에 "N장 완료" 임무와 "N장 소계"(챕터 합계) 행이 있음.
- 레이아웃 함정: 챕터 1~9는 왼쪽(B~J열)에 세로로 쌓여 있는데, **챕터 10만 오른쪽(V~AD열)에 똑같은 스키마로 따로 배치**되어 있었음 — 파싱 스크립트가 이를 별도 블록으로 처리 후 하나의 리스트로 합침.
- 자유 텍스트 메모 2개도 `annotations`에 보존: "2열병대 신규 임무!..." (L2), "7군왕 창고폭발! 석재 다 모으면 4.5만!" (V2).
- **이건 "자원 계산기"의 소재라기보다는 "임무 진행 가이드/체크리스트"에 가까운 데이터** — `yoonsb/`의 체크리스트 개념과 성격이 더 가까울 수 있음 (§4 참고).

파싱 스크립트 자체는 저장하지 않고 1회성으로 실행 후 결과 JSON만 커밋했습니다. 재파싱이 필요하면 이 로그의 접근 방식(openpyxl로 헤더 행 스캔 → 그룹/챕터 경계 탐지)을 참고해 다시 작성하면 됩니다.

---

## 4. 다음 세션에서 이어갈 것 (미해결 항목)

1. **`buildings.json`/`missions.json`을 실제 `resource-planner.html`에 연결하기** — [PORTING_GUIDE.md §4](./PORTING_GUIDE.md#4-진행-순서-제안) 3~4단계(DATA_SPEC 정의, 전역 상수 교체)에 해당. 아직 손 안 댐.
2. **자원 종류 확정** — `건물자원` 시트는 목재/철광/석재만 "비용"으로 쓰고, 식량/동전은 "효과"(생산량 보너스)로만 등장. `임무표` 시트는 목재/철광/석재/식량/동전/금화 6종을 전부 "보상"으로 씀. 건물 업그레이드에 식량/동전이 비용으로도 드는지, `resource-planner.html`의 `RES` 상수를 몇 종으로 정의해야 하는지 확인 필요.
3. **"임무표"가 이 프로젝트(`tech_tree`)의 핵심 소재인지 재검토** — 애초 목표였던 "자원 계산기 이식"과는 결이 다른 데이터(시즌 진행 체크리스트)라, 오히려 `yoonsb/`의 체크리스트 컨셉과 합치는 게 맞을 수도 있음. 사용자와 이 부분 방향 확인 필요.
4. **"기술 트리"가 단순 레벨업인지 실제 분기형 트리인지** — 지금까지 확인된 `건물자원` 시트는 원본(천하결전)과 동일하게 단순 "레벨 1→N 순차 업그레이드" 구조였음(분기/선행조건 없음). 이 질문은 사실상 해소된 것으로 보이나, 최종 확인은 사용자에게 다시 물어볼 것.
5. **자원 상자·둔전 같은 특수 획득 시스템의 삼국지 전략판 대응 여부** — 아직 미확인.
6. **`yoonsb/index.html`의 Firebase 프로젝트가 실제로 살아있는지 / GitHub Pages가 켜졌는지** — 마지막으로 확인했을 때는 안내만 했고 사용자가 실행 완료했는지 확인 안 됨.

---

## 5. 프로젝트 현재 상태 스냅샷

```
sgzzlb_tech_tree/
├── index.html                  # 허브 랜딩 페이지 (2개 링크)
├── resource-planner.html       # 원본 그대로 복사 (미이식 상태)
├── sgzzlb.xlsx                 # 사용자가 추가한 원본 게임 데이터
├── Documents/
│   ├── PORTING_GUIDE.md
│   ├── CODE_STRUCTURE.md
│   ├── PROGRESS_LOG.md         # 이 문서
│   └── data/
│       ├── buildings.json      # sgzzlb.xlsx "건물자원" 파싱 결과
│       └── missions.json       # sgzzlb.xlsx "임무표" 파싱 결과
└── yoonsb/
    ├── index.html               # Firebase 기반 테크트리 체크리스트 (별개 도구)
    └── README.md
```

- Git: `main` 브랜치, origin에 최신 상태로 push 완료 (커밋 `fe0e3fd`까지).
- 협업자 `Kbloodhound`(`yunsb33@gmail.com`)가 이미 push 권한 보유·활동 중.
