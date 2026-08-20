# 삼국지전략판 테크트리 체크리스트 — 배포 가이드

맹원 200명+ 이 로그인 없이 링크만 열면 쓸 수 있는 온라인 체크리스트입니다.

- 호스팅: **GitHub Pages** (무료)
- 데이터 저장: **Firebase Realtime Database** (무료 티어, 이 규모면 충분)
- 파일은 `index.html` 하나뿐입니다.

---

## 1단계. Firebase 프로젝트 만들기 (약 5분)

1. https://console.firebase.google.com 접속 → Google 계정으로 로그인
2. **프로젝트 추가** → 이름 아무거나 (예: `chunbae-techtree`) → 계속
   - Google 애널리틱스는 **꺼도 됩니다**
3. 프로젝트가 만들어지면 왼쪽 메뉴 **빌드 → Realtime Database** → **데이터베이스 만들기**
   - 위치: `asia-southeast1 (싱가포르)` 추천
   - 보안 규칙: 일단 **잠금 모드**로 시작 (다음 단계에서 바꿉니다)
4. Realtime Database 화면 상단의 **규칙** 탭 → 아래 내용으로 교체 → **게시**

```json
{
  "rules": {
    "users": {
      ".read": true,
      "$uid": {
        ".write": "!data.exists() || newData.child('acct').val() === data.child('acct').val()"
      }
    },
    "prog": {
      ".read": true,
      ".write": true
    }
  }
}
```

> 의미: 누구나 읽기 가능. `users`는 신규 등록은 자유지만, 이미 등록된 닉네임은
> 계정번호가 같아야만 수정 가능(닉네임 뺏기 방지). 동맹 내부용 수준의 보안입니다.

## 2단계. 웹 앱 등록하고 설정값 복사

1. Firebase 콘솔 좌측 상단 **⚙️ 프로젝트 설정** → 아래 **내 앱** → **웹(</>) 아이콘** 클릭
2. 앱 닉네임 아무거나 → **앱 등록** (호스팅 체크 불필요)
3. 화면에 나오는 `firebaseConfig = { ... }` 블록을 복사
4. `index.html`을 열어 맨 위 **① Firebase 설정** 부분에 그대로 붙여넣기
   - ⚠️ `databaseURL` 항목이 없으면 직접 추가하세요.
     Realtime Database 화면 상단에 표시된 주소입니다.
     예) `https://chunbae-techtree-default-rtdb.asia-southeast1.firebasedatabase.app`
5. 바로 아래 `ADMIN_CODE = "2580"` 을 원하는 관리자 코드로 변경

## 3단계. GitHub Pages에 올리기 (약 5분)

1. https://github.com 로그인 → **New repository**
   - 이름: 예) `techtree` / Public / README 없이 생성
2. 저장소 페이지에서 **uploading an existing file** 클릭 → `index.html` 드래그 → **Commit changes**
3. 저장소 **Settings → Pages** 메뉴
   - Source: `Deploy from a branch`
   - Branch: `main` / 폴더 `/ (root)` → **Save**
4. 1~2분 뒤 상단에 주소가 나옵니다:
   `https://아이디.github.io/techtree/`

이 주소를 카톡방에 뿌리면 끝입니다. **맹원은 어떤 가입/로그인도 필요 없습니다.**

## 사용 방법

| 구분 | 방법 |
|---|---|
| 맹원 | 링크 접속 → 닉네임 + 계정번호 입력 → 체크 (자동 저장) |
| 재접속 | 같은 닉네임 + 같은 계정번호 입력 |
| 관리자 | 로그인 화면 하단에 관리자 코드 입력 → 전체 현황, 맹원별 상세, 기록 삭제 |
| 백업 | 관리자 화면 우측 상단 **JSON 백업** → 파일 다운로드 |

## 자주 묻는 것

**Q. 비용이 나가나요?**
GitHub Pages 무료, Firebase 무료 티어(Spark)로 충분합니다. 200명이 매일 써도
읽기/쓰기량이 무료 한도에 한참 못 미칩니다. 카드 등록도 필요 없습니다.

**Q. 계정번호가 진짜 비밀번호인가요?**
간이 잠금장치입니다(남이 내 닉네임으로 못 들어오게 하는 정도). 민감한 번호 대신
기억할 수 있는 임의 숫자를 쓰라고 안내하세요.

**Q. 관리자 코드가 코드에 들어있는데 괜찮나요?**
페이지 소스를 열어보면 알 수 있는 수준이라, "악의적인 맹원"까지 막지는 못합니다.
동맹 내부 도구로는 보통 충분하지만, 더 단단하게 하려면 Firebase Authentication을
붙여야 합니다 (필요하면 요청 주세요).

**Q. 테크트리 항목을 수정하고 싶어요.**
`index.html`의 `SECTIONS` 배열을 수정한 뒤 GitHub에서 파일을 다시 업로드하면
즉시 반영됩니다. 항목 id는 바꾸지 마세요(체크 기록이 id 기준입니다).

**Q. 시즌이 바뀌어서 전원 초기화하고 싶어요.**
Firebase 콘솔 → Realtime Database → `prog` 노드 옆 ✕ 를 눌러 삭제하면
전원의 체크가 초기화됩니다 (`users`는 남겨두면 명단 유지).
