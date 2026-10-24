---
name: firebase-세팅
description: 내 컴퓨터에 Firebase를 세팅한다. Firebase 도구 설치, 구글 로그인, 프로젝트 만들기, 웹 앱 등록, Firestore 데이터베이스, 기본 보안 규칙, 첫 화면 배포(https://프로젝트.web.app)까지 순서대로 하고, 필요하면 Google 로그인과 Blaze 요금제를 안내한다. "파이어베이스 세팅해 줘", "내 사이트 올릴 곳 만들어 줘"에 쓴다.
---

# Firebase 세팅

다 끝나면 셀러는 아래 세 가지를 갖게 된다.
- 자기 Firebase 프로젝트
- `https://프로젝트아이디.web.app` 주소에 뜨는 첫 화면
- 앱이 쓸 Firestore 데이터베이스

작업 폴더는 `~/firebase-sites/<프로젝트아이디>`다.

## 꼭 지킬 것

- **로그인·카드 등록은 셀러가 직접:** 구글 로그인과 결제 카드 등록은 셀러가 한다. 비밀번호나 카드 번호를 채팅으로 받지 않는다.
- **이미 있는 프로젝트는 건드리지 않는다:** 셀러가 고르지 않은 프로젝트에는 배포하지 않는다. 명령마다 `--project <아이디>`를 붙인다.
- **기본 보안 규칙은 '전부 막기':** 데이터베이스를 처음 만들 때는 아무도 못 읽고 못 쓰게 시작한다. 앱을 만들 때 필요한 것만 연다.
- **요금:** 무료(Spark)로 시작한다. Cloud Functions, 비밀값 보관(Secret Manager), 예약 실행이 필요할 때만 Blaze로 바꾸자고 하고, 셀러 확인을 받는다.

## 1. 도구 설치

1. `node -v`로 Node.js 18 이상인지 본다. 없으면 https://nodejs.org 에서 LTS를 설치하라고 안내하고 기다린다.
2. Firebase 도구를 설치한다.
   ```
   npm install -g firebase-tools
   ```
   - 맥에서 권한 오류(EACCES)가 나면 sudo를 쓰지 않는다. 대신 이렇게 한 뒤 다시 설치한다.
     ```
     mkdir -p ~/.npm-global && npm config set prefix ~/.npm-global && echo 'export PATH=$HOME/.npm-global/bin:$PATH' >> ~/.zshrc && export PATH=$HOME/.npm-global/bin:$PATH
     ```
3. `firebase --version`이 숫자를 내면 끝이다.

## 2. 구글 로그인 (셀러가 직접)

`firebase login`은 사람이 직접 답해야 하는 명령이라 내가 대신 돌릴 수 없다. 셀러 화면에 터미널을 열어 준다.

- 맥: `osascript -e 'tell application "Terminal" to do script "firebase login"'`
- 윈도우: `start cmd /k firebase login`

셀러에게 이렇게 안내한다.
1. 터미널이 묻는 질문(사용 통계 보내기 등)에는 Y나 N 아무거나 답해도 된다.
2. 브라우저가 열리면 Firebase를 쓸 구글 계정으로 로그인하고 '허용'을 누른다.
3. 터미널에 `Success! Logged in as …`가 보이면 알려 달라고 한다.

다 되면 내가 확인한다.
```
firebase login:list
```
계정이 여러 개면, 어느 계정으로 할지 셀러에게 묻는다.

## 3. 프로젝트 만들기

1. **아이디 정하기:** 셀러와 프로젝트 아이디를 정한다. 영문 소문자·숫자·하이픈만 쓰고 6~30자로 한다. 예: `myacademy-2026`
   - 이 아이디가 주소가 된다: `https://myacademy-2026.web.app`
   - 전 세계에서 하나뿐이어야 해서 이미 쓰인 아이디면 만들기가 실패한다. 그러면 뒤에 숫자를 붙인다.
2. **만들기:**
   ```
   firebase projects:create <아이디> --display-name "<보이는 이름>"
   ```
3. **이미 있는 프로젝트를 쓸 때:** `firebase projects:list`로 목록을 보여 주고 셀러가 고르게 한다.

## 4. 작업 폴더와 첫 화면

`~/firebase-sites/<아이디>` 폴더를 만들고 아래 네 파일을 쓴다(`<아이디>`는 실제 값으로 바꾼다).

`.firebaserc`
```json
{ "projects": { "default": "<아이디>" } }
```

`firebase.json`
```json
{
  "hosting": { "public": "public", "ignore": ["firebase.json", "**/.*", "**/node_modules/**"] },
  "firestore": { "rules": "firestore.rules" }
}
```

`firestore.rules` (처음엔 전부 막는다)
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} { allow read, write: if false; }
  }
}
```

`public/index.html`: 셀러 이름이나 학원 이름을 넣은 간단한 첫 화면이다. 모바일에서 읽기 좋게, 한 장으로 만든다.

## 5. 웹 앱 등록과 설정값

```
firebase apps:create web "<보이는 이름> web" --project <아이디>
firebase apps:sdkconfig web --project <아이디> --out public/firebase-config.json
```
- 이 파일은 JSON이다. 화면 코드에서 `fetch("/firebase-config.json")`로 읽어 `initializeApp(설정)`에 넣는다(또는 그 값을 화면 코드에 그대로 옮긴다).
- 이 설정값(apiKey 등)은 비밀이 아니라 화면에 들어가는 공개값이다. 그래도 채팅에 길게 붙이지 않는다.
- 앱이 여러 개라 `apps:sdkconfig`가 앱을 고르라고 하면, `firebase apps:list --project <아이디>`로 웹 앱 ID를 찾아서 뒤에 붙인다.

## 6. Firestore 데이터베이스

```
firebase firestore:databases:create "(default)" --location asia-northeast3 --project <아이디>
```
- `asia-northeast3`은 서울이다. 한 번 정하면 바꿀 수 없다.
- API를 켜는 중이라는 오류가 나면 1~2분 뒤 같은 명령을 다시 실행한다.

## 7. 첫 배포

```
cd ~/firebase-sites/<아이디>
firebase deploy --only hosting,firestore:rules --project <아이디>
```
- 다 되면 `https://<아이디>.web.app`을 열어서 첫 화면이 보이는지 확인한다(`curl -sI https://<아이디>.web.app`이 200).
- 셀러에게는 주소와 '데이터베이스는 아직 전부 막혀 있다'는 것만 알린다.

## 필요할 때만

### Google 로그인 켜기 (앱에 로그인이 필요할 때)

이 설정은 명령으로 할 수 없어서 콘솔에서 한다.
1. 브라우저에 `https://console.firebase.google.com/project/<아이디>/authentication/providers`를 연다.
2. 셀러가 「시작하기」 → 「Google」 → 사용 설정 → 지원 이메일을 고르고 저장한다.
3. 로그인 화면 주소가 `<아이디>.web.app`이 아니면, 같은 화면의 「설정 → 승인된 도메인」에 그 주소를 더한다.

### Blaze 요금제 (Functions·비밀값·예약 실행이 필요할 때)

1. 먼저 셀러에게 왜 필요한지 한 줄로 말하고 확인받는다. 예: "문자 발송 함수를 돌리려면 Blaze가 필요해요."
2. `https://console.firebase.google.com/project/<아이디>/usage/details`를 열고, 셀러가 직접 Blaze로 바꾸고 카드를 등록한다.
3. 작은 사이트는 보통 무료 한도 안에서 끝난다는 것도 알린다. 원하면 같은 화면에서 예산 알림(예: 월 1만 원)을 걸게 안내한다.

## 막힐 때

- **`Cannot run login in non-interactive mode`:** 로그인은 '2. 구글 로그인'처럼 셀러 터미널에서 해야 한다.
- **`Project ID already exists` / `already in use`:** 다른 아이디로 다시 만든다.
- **`Permission denied` / `403`:** 로그인한 계정이 그 프로젝트의 소유자가 아니다. `firebase login:list`로 계정을 확인한다.
- **배포 뒤 옛 화면이 보일 때:** 브라우저 캐시 때문이다. 주소 뒤에 `?v=2`를 붙여 다시 연다.
