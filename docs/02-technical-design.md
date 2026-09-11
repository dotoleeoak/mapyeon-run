# 기술 설계서

## 1. 목적

본 문서는 `mapyeon-run` MVP 구현을 위한 기술 구조와 주요 설계 결정을 정의한다.

목표는 다음과 같다.

- 브라우저에서 즉시 실행 가능
- 정적 호스팅 가능
- 5~10분짜리 싱글플레이 게임에 적합한 단순한 구조
- 게임 로직과 화면 표현을 가능한 한 분리
- 향후 콘텐츠 확장이 쉬운 구조

---

## 2. 초기 기술 스택

| 영역 | 선택 |
| --- | --- |
| Language | TypeScript |
| Game Framework | Phaser 3 |
| Build Tool | Vite |
| Physics | Phaser Arcade Physics |
| Map Authoring | Tiled Map Editor |
| Hosting | Cloudflare Pages / Vercel / GitHub Pages |

### 선택 이유

**TypeScript**

- 상태와 시스템 간 인터페이스 정의가 쉽다.
- 리팩터링과 규모 확장에 유리하다.

**Phaser 3**

- 브라우저 기반 2D 게임에 적합하다.
- Scene, Input, Camera, Physics, Audio 등 필요한 기능을 대부분 제공한다.

**Vite**

- 개발 서버와 빌드 구성이 단순하다.
- TypeScript 프로젝트와 궁합이 좋다.

**Arcade Physics**

- 복잡한 물리 시뮬레이션이 필요하지 않은 MVP에 적합하다.

---

## 3. 런타임 구조

```text
Browser
  ↓
Phaser Game
  ↓
Scenes
  ↓
Entities / Systems / AI
  ↓
Rendering / Input / Physics / Audio
```

Scene은 전체 흐름을 조율하고, 개별 게임 규칙은 작은 시스템 또는 Entity가 담당한다.

---

## 4. Scene 구조

### BootScene

- 리소스 preload
- 로딩 UI
- 완료 후 MenuScene 이동

### MenuScene

- 타이틀
- 게임 시작
- 조작법
- 크레딧

### GameScene

- 맵 생성
- Entity 생성
- 시스템 연결
- 충돌 설정
- HUD 생성

### EndingScene

- 성공 / 실패 결과 출력
- 플레이 시간, 남은 멘탈 등 결과 표시
- 다시 하기 / 메인 메뉴

---

## 5. 권장 프로젝트 구조

```text
src/
├── main.ts
├── game/
│   ├── config.ts
│   ├── constants.ts
│   ├── events.ts
│   ├── scenes/
│   │   ├── BootScene.ts
│   │   ├── MenuScene.ts
│   │   ├── GameScene.ts
│   │   └── EndingScene.ts
│   ├── entities/
│   │   ├── Player.ts
│   │   ├── Senior.ts
│   │   └── Evidence.ts
│   ├── systems/
│   │   ├── DetectionSystem.ts
│   │   ├── EvidenceSystem.ts
│   │   ├── MentalSystem.ts
│   │   └── ObjectiveSystem.ts
│   ├── ai/
│   │   ├── SeniorState.ts
│   │   └── SeniorStateMachine.ts
│   ├── ui/
│   │   ├── HUD.ts
│   │   ├── DialogueBox.ts
│   │   └── Notification.ts
│   ├── data/
│   │   ├── evidence.ts
│   │   ├── dialogue.ts
│   │   └── balance.ts
│   └── types/
│       └── game.ts
└── assets/
    ├── sprites/
    ├── maps/
    ├── sounds/
    └── ui/
```

이 구조는 구현 시작 시점의 가이드이며, Vertical Slice 결과에 따라 단순화할 수 있다.

---

## 6. GameScene 책임

GameScene에 모든 로직을 몰아넣지 않는다.

GameScene의 책임:

- 월드 초기화
- Entity 생성
- System 생성 및 연결
- Physics collider 연결
- Scene lifecycle 관리
- 게임 종료 트리거

세부 AI, 증거 수집, 멘탈 계산 등은 별도 객체로 분리한다.

---

## 7. 플레이어 모델

예상 상태:

```text
NORMAL
HIDING
WRITING
PUNISHED
GAME_OVER
```

주요 데이터:

```text
mental
isInvulnerable
currentState
movementSpeed
```

이동은 velocity 기반으로 처리하고, 대각선 이동 시 normalize하여 속도 증가를 방지한다.

---

## 8. 선임 AI

Finite State Machine을 사용한다.

```text
PATROL
  ↓ 발견
CHASE
  ↓ 잡음
PUNISH
  ↓
RETURN
  ↓
PATROL
```

확장 시:

```text
CHASE → SUSPICIOUS → PATROL
```

상태별 책임을 명확히 분리해 AI 로직이 한 함수 안에 복잡하게 얽히지 않게 한다.

---

## 9. 탐지 시스템

탐지 조건:

```text
거리 조건
AND 시야각 조건
AND 벽 차폐 없음
```

### 거리

선임과 플레이어 간 거리가 `visionRange` 이하인지 검사한다.

### 시야각

선임이 바라보는 방향과 플레이어 방향 벡터 간 각도를 비교한다.

### Line of Sight

선임과 플레이어 사이에 충돌 타일이 존재하면 탐지하지 않는다.

### 탐지 지연

즉시 CHASE로 전환하지 않고 짧은 detection meter를 사용한다.

초기 목표값: 약 0.6초.

---

## 10. 추적

MVP 맵은 단순하게 설계해 초기에는 직접 추적을 사용한다.

```text
Senior → Player position
```

복잡한 벽 구조 때문에 선임이 자주 걸리는 문제가 발생하면 그때 A* pathfinding 도입을 검토한다.

처음부터 pathfinding을 넣지 않는다.

---

## 11. 월드와 Tilemap

초기 기준:

- 논리 해상도: 960×540
- 타일 크기: 32×32
- 월드는 화면보다 크게 구성
- 카메라는 플레이어 follow

Tiled 레이어 예시:

```text
Floor
Walls
Decoration
Collision
Objects
```

Object Layer 예시:

```text
player_spawn
senior_spawn
patrol_point
Evidence
writing_desk
mailbox
```

좌표를 코드에 직접 하드코딩하기보다 맵 데이터에서 읽는다.

---

## 12. 상호작용

플레이어 주변 일정 거리 안에 상호작용 가능한 객체가 있으면 E 입력을 허용한다.

초기 상호작용 거리: 약 48px.

대상:

- Evidence
- Writing Spot
- Mailbox

---

## 13. 주요 시스템

### EvidenceSystem

책임:

- 증거 획득
- 중복 획득 방지
- 현재 수량 관리
- 목표 전환 조건 검사

### MentalSystem

책임:

- 현재 멘탈 관리
- 피해 적용
- 최소값 0 보장
- 게임 오버 이벤트 발생

### ObjectiveSystem

상태:

```text
COLLECT_EVIDENCE
WRITE_LETTER
REACH_MAILBOX
COMPLETED
```

### DetectionSystem

책임:

- 거리 검사
- 시야각 검사
- Line of Sight 검사
- detection progress 관리

---

## 14. 이벤트 기반 통신

Scene, System, UI 간 결합을 줄이기 위해 EventEmitter 사용을 고려한다.

예시 이벤트:

```text
evidence:collected
mental:changed
player:caught
letter:started
letter:completed
objective:changed
game:completed
```

HUD는 매 프레임 값을 읽기보다 상태 변경 이벤트를 구독해 갱신한다.

---

## 15. 게임 상태

한 번의 플레이 결과를 표현하는 데이터 모델 예시:

```ts
interface GameState {
  mental: number;
  evidenceIds: string[];
  letterWritten: boolean;
  elapsedTime: number;
  caughtCount: number;
}
```

향후 공유 카드나 랭킹 기능이 추가될 경우에도 재사용할 수 있다.

---

## 16. 밸런스 데이터 분리

자주 변경되는 수치는 별도 데이터 파일에 둔다.

예상 항목:

```text
player.walkSpeed
player.runSpeed
senior.patrolSpeed
senior.chaseSpeed
senior.visionRange
senior.visionAngle
captureDistance
captureDamage
letterWriteDuration
interactionRange
invulnerabilityDuration
```

초기 테스트 값:

| 항목 | 값 |
| --- | ---: |
| Player Walk | 120 px/s |
| Player Run | 180 px/s |
| Senior Patrol | 70 px/s |
| Senior Chase | 130 px/s |
| Vision Range | 220 px |
| Vision Angle | 70° |
| Capture Distance | 30 px |
| Mental | 100 |
| Capture Damage | 20 |
| Letter Write | 5 s |
| Invulnerability | 3 s |

수치는 플레이 테스트를 통해 조정한다.

---

## 17. 마음의 편지 작성

조건:

- 필수 증거 확보
- 작성 장소와 상호작용

작성 중:

- 플레이어 이동 불가
- 선임 AI는 계속 동작
- Progress UI 표시
- 잡히면 작성 취소

작성 완료 후 `REACH_MAILBOX` 상태로 전환한다.

---

## 18. 편지함 및 클리어

`letterWritten === true` 상태에서 Mailbox와 상호작용하면 성공 처리한다.

EndingScene으로 전달할 데이터:

```text
success
elapsedMs
mental
evidenceCount
caughtCount
```

---

## 19. UI 설계

HUD는 월드 카메라에 영향을 받지 않도록 고정한다.

기본 정보:

- Mental
- Evidence count
- Current objective
- Interaction prompt

디버그 빌드에서는 추가로 다음 정보를 표시할 수 있다.

- FPS
- Player 좌표
- Senior state
- Vision cone
- Patrol points
- Collision bounds

---

## 20. Audio

MVP 최소 사운드:

```text
footstep
alert
caught
evidence
letter
success
```

선임과 플레이어 사이 거리에 따라 발소리 볼륨을 조절해 화면 밖 위험을 전달한다.

---

## 21. 테스트 전략

렌더링보다 순수 게임 로직을 우선 단위 테스트한다.

우선 대상:

- MentalSystem
- EvidenceSystem
- ObjectiveSystem
- StateMachine

필수 조건:

- 증거 중복 획득 불가
- 멘탈이 0 미만으로 내려가지 않음
- 증거 부족 시 편지 작성 불가
- 편지 작성 전 제출 불가
- Game Over 이후 플레이 입력 차단
- 종료 이후 AI 업데이트 차단

테스트 도구는 구현 시 Vitest를 우선 검토한다.

---

## 22. 디버깅 기능

개발 빌드에서 다음 시각화를 제공하는 것이 좋다.

- Vision cone
- Senior state text
- Patrol path
- Collider
- Detection meter

개발용 치트 후보:

```text
1: 증거 전부 획득
2: 멘탈 회복
3: 편지 작성 완료
4: Senior AI 정지
```

Production에서는 제거한다.

---

## 23. 배포

프로젝트는 정적 웹 애플리케이션으로 빌드한다.

```text
GitHub
  ↓ push
CI / Hosting provider
  ↓ build
Static assets
  ↓ CDN
Browser
```

우선 후보:

1. Cloudflare Pages
2. Vercel
3. GitHub Pages

백엔드는 MVP에 포함하지 않는다.

---

## 24. 백엔드 도입 기준

다음 기능이 필요해질 때 서버 도입을 검토한다.

- 온라인 랭킹
- 계정
- 클라우드 세이브
- 멀티플레이
- 서버 검증이 필요한 기록

MVP에서는 모든 게임 상태를 브라우저 메모리에서 처리한다.

---

## 25. 핵심 기술 검증 순서

가장 먼저 만들 Vertical Slice:

```text
작은 방/복도
+
Player
+
Senior
+
Wall collision
+
Vision
+
Chase
```

이 단계의 질문은 하나다.

> 선임의 시야를 피해 이동하고, 발견됐을 때 도망가는 경험 자체가 재미있는가?

이 질문이 해결되기 전에는 복잡한 콘텐츠, 여러 NPC, 서버 기능을 추가하지 않는다.
