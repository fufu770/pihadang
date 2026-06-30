# 당피하기 (자일로큐브 게임) — 이어가기 메모

> 다음 세션에서 이 파일을 먼저 읽고 이어서 작업할 것. (작업 디렉터리: `C:\project\game`)

## 한 줄 요약
영당이(자일로큐브) 캐릭터로 **설탕 과자를 피하고 자일로큐브를 먹는 모바일 세로형 시간 어택 회피 게임**. 단일 `index.html`(캔버스) + 음원/이미지 에셋. GitHub Pages로 영구 배포됨.

## 라이브 / 배포
- **현재 사용 주소(이것만 공유)**: https://fufu770.github.io/pihadang/score.html
  - 옛 주소들(`/`, `play.html`, `go.html`, `clean.html`, `game.html`)은 브라우저 캐시 회피용으로 만든 동일 복사본. 모두 같은 index.html.
  - 캐시 문제로 코드 바꿀 때마다 새 진입 파일을 만들어 왔음. 다음에 코드 바꾸면 dist의 index/play/go/clean/game/score.html 전부 동일 복사 후 push (또는 새 파일 1개 추가).
- **랭킹 = 온라인 전체 공유(Firebase Realtime DB)**: 모두 같은 랭킹 공유, 게임오버 시 즉시 등록. dreamlo+프록시는 다 죽어서 폐기 → Firebase로 교체(2026-06-29).
  - DB URL: `https://yongdang-default-rtdb.firebaseio.com` (프로젝트명 YONGDANG, 계정 for3015@gmail.com). 프록시 불필요(HTTPS+CORS 직접 fetch).
  - 코드: `FB.base` + `fbSubmit(name,score,level)`(**닉네임당 최고기록 1줄만 PUT**, key=`b64e(name)`, 기존보다 높을 때만 갱신) + `fbTop(n)`(GET /scores.json → 닉네임별 최고 집계 후 점수 내림차순 top n=20). DB=고유 닉네임 수만큼만. 랭킹 1줄에 닉네임·`Lv.N`·점수.
  - ⚠️ 과거 버그: 매 판 POST+100개 prune 방식은 한 명이 슬롯 독차지→타인 밀려나 랭킹이 줄어듦. → keyed PUT(이름당 1줄)로 복귀해 해결(2026-06-30). DB도 best-per-name으로 1회 정리함. 시작화면 `refreshStartBoard()`(↻새로고침 버튼), 게임오버 시 async 등록 후 `renderRanking`(이번 판 name+score 하이라이트). 실패 시 `loadCache()`(localStorage RANK_KEY) 폴백.
  - **닉네임 규칙**: 빈 닉네임이면 시작 차단(빨간 테두리+흔들기, `nameError`/`syncNameValid`/`#nameMsg`). 기본값 "나" 없앰. **온라인 랭킹에 이미 있는 닉네임이면 중복으로 시작 차단**(`isNameTaken`), 단 같은 세션 재시작은 `claimedName`으로 면제. iOS 소리 위해 `Sound.init()/unlockAudio()`는 중복검사 await 전에 동기 호출, `Bgm.start()`는 통과 후.
  - **보안 규칙(영구)**: `/scores`만 read/write public, 루트는 false. 콘솔 Realtime Database→규칙 탭에 붙여넣고 게시(2026-06-29 적용 확인). (테스트 모드는 30일 뒤 잠기니 영구 규칙 필수.)
  - 옛 dreamlo 함수(`LB`/`lbFetchText`/`lbSubmit`/`lbDelete`/`lbTop`)는 제거됨. `b64e`/`b64d`는 남아있으나 미사용.
  - 랭킹 비우려면: `curl -X DELETE 'https://yongdang-default-rtdb.firebaseio.com/scores.json'` (전체 삭제).
- GitHub repo: `fufu770/pihadang` (public), 배포 소스 = `dist/` 폴더(이게 git 저장소). gh CLI 영구 로그인됨(토큰 불필요).
- **gh CLI 영구 로그인 완료(keyring)** → 토큰 없이 재배포 가능
- 재배포 방법:
  1. `C:\project\game\index.html` 수정
  2. `index.html`을 `deploy\`와 `dist\`에 복사, 변경된 에셋도 `dist\`에 복사
  3. `git -C C:\project\game\dist add -A; git -C ... commit -m "..."; git -C ... push origin main`
  4. 1~2분 뒤 Pages 반영 (확인 시 강력 새로고침/주소 끝 `?n`)

## 파일 구조 (`C:\project\game`)
- `index.html` — 게임 본체(단일 파일, 약 67KB). CONFIG 객체에 모든 밸런스 모여 있음.
- `bgm.mp3` — 배경음악(자일로큐브 브링커)
- `voice1~7.mp3` — 아이템 먹을 때 맛 이름 음성(첨부 영상에서 ffmpeg로 추출, 7개 단일 단어)
- `ogimage.png` — 카톡 미리보기 썸네일
- `ending.png` — 레벨20 클리어 엔딩에서 내려오는 자일로큐브 캔 이미지(NO SUGAR CANDY 라벨)
- `무설탕캔디는 자일로큐브.m4a` — 엔딩에서 캔을 받아먹는 순간 재생되는 음성
- `dist/` — **배포본 = git 저장소(origin = GitHub)**. 라이브와 동일.
- `deploy/` — 배포용 사본(드래그 배포용, 보조)
- `character/` — 원본 캐릭터 이미지(영당이 포스터, 무지개 오리지널)
- `server.js` / `cloudflared.exe` / `voicetest.html` / `pihadang_deploy.zip` — 개발·임시 도구(지금은 불필요)
- ffmpeg 경로: `C:\Users\for30\AppData\Local\Microsoft\WinGet\Packages\Gyan.FFmpeg_*\ffmpeg-*-full_build\bin\ffmpeg.exe`

## 게임 규칙 (현재)
- **시간 어택**: 시간바 30초로 시작, 계속 감소(레벨↑ 가속), **0이면 사망**
- **장애물**(도넛·콜라·과자봉지·초콜릿): 맞으면 **즉사**(방패 중엔 안전)
- **자일로큐브 아이템**: 먹으면 시간 **+2.7초** — star=점수, clock=슬로우, shield=방패(5초 무적)
- **🌈 무지개 오리지널**: 먹으면 시간 **+6.25초**(대량)+200점, 놓치면 **−4초**(즉사 아님)
- 10초마다 레벨업(최대 Lv.20) → 장애물 속도·수량 증가(레벨 구간마다 동시 낙하 개수↑)
- **엔딩(Lv.20 클리어)**: elapsed≥200초(maxLevel×levelInterval) 생존 시 `winGame()` 발동 → gameState "ending". 빵빠레(`Sound.fanfare`)+폭죽/꽃가루(`endParts`)+`ending.png` 캔이 빠르게 하강(vy 110, 플레이어 쪽 호밍)→받아먹는 순간 `endVoice`(m4a) 재생→`#endText`("— THE END — 무설탕 캔디는 자일로큐브") 등장+다시하기. **미리보기 치트: 플레이 중 `E`키**. 루프(`loop`)는 "playing"||"ending"에서 동작, 입력은 `canControl()` 게이트.
- **화면 비율 고정**: 고정 논리해상도 `GAME_W×GAME_H`=450×800(9:16) `#stage` 박스+레터박스 → 기기(노트북/폰/폴더블) 무관 동일 난이도. 입력은 `view`(scale/left/top)로 화면→게임좌표 변환.
- 조작: 모바일 **상대 드래그**(touchSensitivity), PC 마우스/키보드(←→·A/D), Enter/Space 시작
- 캐릭터: 영당이 4색 스킨, 방향성 달리기 모션, 피격 시 아파하는 표정
- 사운드: BGM(mp3), 효과음(WebAudio), 아이템 먹을 때 맛이름 음성(셔플·단일채널), 사망 시 "살찐당"(TTS)
- 랭킹: 온라인 **TOP20**(게임오버+시작화면 표시), 카톡 공유 + OG 카드
- **닉네임 규칙**: 빈/욕설(`hasProfanity`+`BANNED_WORDS` 블록리스트)/중복(`isNameTaken`, 내 이름은 `myNames`/`claimedName` 면제) 차단. 입력창 타이핑 중엔 게임 단축키(A/D/E/스페이스) 무시(`isField`로 e.target+activeElement 검사), Enter는 시작 유지.
- 캐시 회피용 진입 파일에 **fix.html, latest.html** 추가됨(현재 index/play/go/clean/game/score/fix/latest 8개 동일 복사). + `<head>`에 no-cache 메타 3종 넣어 앞으로 새로고침만으로 최신 반영.
- **게임오버 공유 = 카카오톡 직접 공유(도전장)**: 노란 펄스 버튼 "🥊 친구에게 도전장 보내기". Kakao JS SDK 2.7.5 로드 + `Kakao.init("9d3818b7ccfc3a16ab00e1b498b03a98")`(JS키). `shareKakao()` = `Kakao.Share.sendDefault`(feed: ogimage.png 카드 + "🏃 도전하기" 버튼, link=score.html). 폴백: navigator.share→링크복사.
  - 카카오 개발자 앱: **당피하기, 앱 ID 1499964**, 계정 for3015@gmail.com(은영). 공유 작동에 도메인 **2곳** 등록 필수: ① 제품 링크 관리>웹 도메인 ② 플랫폼키>JavaScript키>JavaScript SDK 도메인 — 둘 다 `https://fufu770.github.io`. (안 하면 sharer.kakao.com 4019 오류)

## 주요 코드 위치 (index.html 내 CONFIG/모듈)
- `CONFIG` — player/obstacle/difficulty/item/must/timeAttack 전부
- `OBSTACLE_THEMES.sugar` — 과자 4종 그리기
- `SKINS` — 영당이 4색
- `ITEMS` — star/clock/shield, `드로우`: `drawCandyCube`(rainbow 지원)
- `Sound`(효과음) / `Speak`(TTS) / `Bgm`(mp3) / `Music`(미사용 신스, 비활성) 모듈
- `FLAVOR_VOICES` + `playFlavor()` — 맛이름 음성(셔플 가방)
- 랭킹: `LB`(dreamlo pub/priv + proxies), `lbTop`/`lbSubmit`/`renderRanking`
- `update()` 안에 시간감소·스폰·충돌·아이템·무지개 로직

## 랭킹 백엔드
- dreamlo 무료 리더보드(계정 불필요). HTTP만 지원 → 브라우저에서 **CORS 프록시**(corsproxy.io 우선, allorigins 폴백) 경유.
- pub(읽기)/priv(쓰기) 코드는 index.html `LB` 객체에 있음. 닉네임은 base64로 인코딩 저장(한글 안전).
- ⚠️ 공개 프록시라 가끔 끊김 → "온라인 연결 실패" 폴백(로컬 캐시). 안정화하려면 더 튼튼한 무료 백엔드로 교체 검토.

## 알려진 이슈 / 다음 작업 후보
1. **랭킹 안정성** — 프록시 의존, 가끔 실패. 더 견고한 백엔드 검토.
2. **보안** — 루트에 `ghp_*.txt`(0바이트, 파일명이 노출된 토큰). 삭제 + GitHub에서 토큰 폐기 권장. (사용자가 "나중에"로 보류함)
3. **정리** — 미사용 파일(cloudflared.exe 약 52MB, zip, voicetest.html) 제거 여부.
4. **밸런스/신규 기능** — 사용자 요청 대기.
5. iOS: 웹 진동 미지원(햅틱 없음), 소리는 무음스위치 해제+첫 터치 필요.

## 참고 (사용자 선호)
- 한국어로 소통. 사용자는 코딩 비전문가 → 단계가 단순해야 함. 계정/토큰 작업을 어려워함.
- 게임 톤: 귀엽고 밝게. 강조 글귀는 두껍게+색다르게(Black Han Sans/핑크). 본문 글꼴 Jua.
