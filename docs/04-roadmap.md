# 개발 로드맵

## 1. 원칙

개발 순서는 콘텐츠 양이 아니라 **핵심 재미 검증 순서**를 따른다.

가장 먼저 확인해야 할 것은 다음이다.

> 선임의 시야를 피해 움직이고, 발견됐을 때 도망가는 것이 재미있는가?

이 질문에 답하기 전에는 복잡한 콘텐츠나 서버 기능을 추가하지 않는다.

---

## 2. Phase 0 — 기획 고정

### 목표

구현 전에 MVP 범위와 핵심 규칙을 문서로 고정한다.

### 작업

- 게임 기획서 확정
- 기술 설계서 확정
- MVP 범위 확정
- 콘텐츠 톤 가이드 확정
- 초기 밸런스 값 정의

### 완료 조건

- 무엇을 만들지 설명할 수 있음
- 무엇을 만들지 않을지도 명확함
- Vertical Slice 범위가 확정됨

---

## 3. Phase 1 — 프로젝트 부트스트랩

### 목표

브라우저에서 Phaser 게임 루프가 동작하도록 한다.

### 작업

- Vite + TypeScript 초기화
- Phaser 설치
- 기본 Game config
- Boot / Menu / Game / Ending Scene 뼈대
- 개발 서버 실행
- production build 확인

### 완료 조건

- 브라우저에서 빈 GameScene 실행
- Scene 전환 정상
- build 성공

---

## 4. Phase 2 — 플레이어와 맵

### 목표

플레이어가 테스트 맵을 자연스럽게 돌아다닐 수 있게 한다.

### 작업

- placeholder Player
- WASD / 방향키 이동
- 달리기
- 대각선 속도 정규화
- Camera follow
- Tiled 테스트 맵
- Collision layer
- 벽 충돌

### 검증 포인트

- 이동감이 답답하지 않은가?
- 카메라가 어지럽지 않은가?
- 좁은 통로에서 충돌이 자연스러운가?

---

## 5. Phase 3 — 선임 순찰

### 목표

맵에 살아 움직이는 위협을 만든다.

### 작업

- Senior Entity
- Patrol point 로드
- Patrol FSM
- waypoint 이동
- 포인트 도착 후 랜덤 대기
- 방향 표시 또는 sprite facing 처리

### 완료 조건

- 선임이 지정된 경로를 반복적으로 순찰
- 벽에 쉽게 끼지 않음

---

## 6. Phase 4 — Detection + Chase Vertical Slice

### 목표

프로젝트의 가장 중요한 프로토타입을 완성한다.

### 작업

- Vision range
- Vision angle
- Line of Sight
- Vision cone debug visualization
- Detection meter
- PATROL → CHASE 전환
- Player 추적
- Capture distance
- 잡힌 후 Patrol 복귀

### 플레이 테스트 질문

- 위험이 언제 발생하는지 이해되는가?
- 선임 시야가 너무 넓거나 좁지 않은가?
- 추격 중 탈출이 가능한가?
- 맵에 우회 경로가 충분한가?
- 발각 자체가 긴장감을 만드는가?

### 의사결정 게이트

이 단계가 재미없으면 Phase 5로 넘어가지 않는다.

조정 대상:

- Player walk/run speed
- Senior patrol/chase speed
- Vision range
- Vision angle
- Detection delay
- Map geometry

---

## 7. Phase 5 — 멘탈과 실패 루프

### 목표

잡혔을 때의 결과와 게임오버 흐름을 만든다.

### 작업

- MentalSystem
- Capture damage
- 짧은 이동 제한
- 재포획 방지 invulnerability
- Mental HUD
- Mental 0 → EndingScene
- Retry

### 완료 조건

- 반복적으로 잡히면 실패
- 실패 후 빠르게 재도전 가능
- soft-lock 없음

---

## 8. Phase 6 — 증거와 목표 시스템

### 목표

맵 탐색에 명확한 목적을 부여한다.

### 작업

- Evidence Entity
- EvidenceSystem
- 증거 3개 배치
- E 상호작용
- 중복 수집 방지
- Evidence HUD
- ObjectiveSystem
- 현재 목표 표시

### 목표 흐름

```text
COLLECT_EVIDENCE
  ↓ 3 / 3
WRITE_LETTER
```

---

## 9. Phase 7 — 마음의 편지

### 목표

게임의 주제를 실제 플레이 메커니즘과 연결한다.

### 작업

- Writing spot
- 증거 완료 조건 검사
- Writing 상태
- 이동 잠금
- Progress bar
- 작성 중 발각 처리
- 작성 완료 이벤트

### 목표 흐름

```text
WRITE_LETTER
  ↓ complete
REACH_MAILBOX
```

---

## 10. Phase 8 — 최종 러시와 클리어

### 목표

게임의 클라이맥스와 완결된 성공 루프를 만든다.

### 작업

- Mailbox
- 편지 보유 조건
- 제출 상호작용
- 최종 목표 HUD
- 필요 시 AI 소폭 강화
- Success Ending
- 결과 화면
- 플레이 시간 기록

### 완료 조건

처음부터 끝까지 한 판 플레이 가능.

---

## 11. Phase 9 — 콘텐츠 및 피드백 강화

### 목표

프로토타입 느낌을 줄이고 플레이 경험을 읽기 쉽게 만든다.

### 작업

- Dialogue box
- 잡힘 대사
- 증거 설명
- Alert icon
- 사운드
- 군화 접근음
- 화면 전환
- 간단한 screen shake
- UI 정리

---

## 12. Phase 10 — 아트 적용

### 목표

검증된 게임플레이 위에 최종 비주얼을 적용한다.

### 작업

- Pixel art tile
- Player sprite
- Senior sprite
- Evidence object
- Mailbox
- 기본 animation
- UI assets

아트 제작 전에 게임플레이가 충분히 검증되어 있어야 한다.

---

## 13. Phase 11 — QA 및 밸런싱

### 필수 테스트

- 최신 Chrome
- 최신 Firefox
- 최신 Edge
- Safari 가능 범위 확인

### 체크 항목

- 맵 밖 이동 불가
- 벽 끼임 없음
- 증거 3개 모두 획득 가능
- 편지 작성 조건 정상
- 작성 중 capture 정상
- 제출 조건 정상
- Game Over 정상
- Retry 후 상태 초기화 정상
- 탭 전환 시 타이머/AI 이상 여부

### 밸런스 목표

첫 플레이 클리어율 약 50~70%에서 시작해 조정한다.

---

## 14. Phase 12 — 웹 배포

### 작업

- production build
- hosting 연결
- HTTPS 확인
- asset path 검증
- direct URL 접속 테스트
- 모바일 화면에서 최소한의 오류 여부 확인

### 후보

1. Cloudflare Pages
2. Vercel
3. GitHub Pages

### 완료 조건

누구나 public URL 하나로 게임을 플레이할 수 있다.

---

## 15. MVP 이후 로드맵 후보

MVP가 재미있다는 결론이 나온 경우에만 진행한다.

### v0.2 — Variety

- 다양한 부조리 이벤트
- 추가 대사
- 랜덤 순찰 변화
- 의심도 시스템

### v0.3 — Replayability

- 여러 엔딩
- 추가 증거
- 랜덤 이벤트
- 난이도 옵션

### v0.4 — Platform

- 모바일 조작
- 반응형 UI
- 터치 UX

### v0.5 — Sharing

- 결과 카드
- 기록 공유
- 로컬 최고 기록

### Future

- 온라인 랭킹
- 추가 스테이지
- 여러 AI 타입

---

## 16. 구현 우선순위 요약

```text
1. Project bootstrap
2. Player movement
3. Map + collision
4. Senior patrol
5. Detection
6. Chase
7. Capture
8. Mental
9. Evidence
10. Objective
11. Letter
12. Mailbox
13. Ending
14. Audio / UX feedback
15. Art
16. Deploy
```

항상 새로운 기능을 추가하기 전에 다음 질문을 확인한다.

> 이 기능이 핵심 스텔스 루프를 더 재미있거나 더 이해하기 쉽게 만드는가?
