# Gravedraw 구현 계획

> 확정일: 2026-03-26
> 개발 기간: 3주 (솔로 개발)

## 진행 상황

- [ ] Phase 1 — 데이터 구조 (1~2일) ← **현재 여기**
- [ ] Phase 2 — 덱 시스템 (3~4일)
- [ ] Phase 3 — 전투 상태머신 (5~7일)
- [ ] Phase 4 — 카드 콘텐츠 (8~9일)
- [ ] Phase 5 — 적 콘텐츠 (10~11일)
- [ ] Phase 6 — 맵 / 런 시스템 (12~13일)
- [ ] Phase 7 — 유물 시스템 (14일)
- [ ] Phase 8 — UI (15~17일)
- [ ] Phase 9 — 세이브 / 로드 (18일)
- [ ] Phase 10 — 폴리싱 + 빌드 (19~21일)

---

## Phase 1 — 데이터 구조 ← 현재 진행 중

**목표:** 런타임 코드 없이 데이터 뼈대만 잡기

### 세션 메모 (2026-03-26)
- 기획서(`GAMEDESIGN.md`) 작성 완료
- 구현 계획 확정
- Phase 1 첫 과제 설명까지 완료
- **다음 할 일:** `CardData.cs` 뼈대 작성 (ScriptableObject 상속)
  - Assets/Scripts/Card/CardData.cs 생성
  - 카드 이름, 비용, 설명, 타입, 희귀도, 스프라이트, 효과 목록 필드 포함

### 구현 대상
- `CardData` ScriptableObject (이름, 비용, 효과 목록, 희귀도, 타입)
- `ICardEffect` 인터페이스 + `BattleContext` 주입 구조
- `EnemyData` ScriptableObject (HP, 패턴 목록)
- `RelicData` ScriptableObject (이름, 패시브 효과)
- `StatusEffect` 열거형 (출혈, 약화, 취약)

**완료 기준:** Unity Inspector에서 카드/적 데이터 에셋 생성 가능

---

## Phase 2 — 덱 시스템

**목표:** 카드 드로우/사용/버리기/셔플 로직

### 구현 대상
- `DeckManager` — 드로우 더미, 손패, 버리기 더미 상태 관리
- 드로우 시 드로우 더미 소진 → 버리기 더미 자동 셔플
- `ITickable` 등록 방식으로 Update 직접 사용 금지

**완료 기준:** 에디터 테스트에서 10장 드로우/버리기/셔플 정상 동작

---

## Phase 3 — 전투 상태머신

**목표:** 턴 흐름 완성

```
StartTurn → PlayerTurn → EndPlayerTurn
                       → EnemyTurn → EndEnemyTurn
                                   → Victory / Defeat
```

### 구현 대상
- `IBattleState` 인터페이스 + 각 상태 클래스
- `BattleManager` — 상태 전환, 에너지 관리, HP/방어도 계산
- 적 패턴 실행 로직
- 상태이상 스택 처리

**완료 기준:** 코드로만 전투 1회 시뮬레이션 가능 (UI 없이)

---

## Phase 4 — 카드 콘텐츠

**목표:** 카드 20장 효과 전부 구현

### 구현 대상
- 공격 카드 12장
- 스킬 카드 8장
- 초기 덱 10장 설정

**완료 기준:** EditMode 테스트에서 각 카드 효과 단위 테스트 통과

---

## Phase 5 — 적 콘텐츠

**목표:** 적 데이터 에셋 생성 + 패턴 동작 확인

### 구현 대상
- 일반 적 5종 ScriptableObject
- 보스 2종 ScriptableObject
- 적 의도(Intent) 계산 로직

**완료 기준:** 각 적이 패턴대로 순환하며 행동

---

## Phase 6 — 맵 / 런 시스템

**목표:** 2층 맵 생성 + 런 흐름

### 구현 대상
- `RunManager`
- 맵 노드 생성 로직
- 이벤트 5개
- 상점

**완료 기준:** 타이틀 → 맵 → 전투 → 보상 → 맵 흐름 연결

---

## Phase 7 — 유물 시스템

**목표:** 패시브 효과 5개

### 구현 대상
- `RelicManager` — 이벤트 훅 방식
- 유물 5개 구현

**완료 기준:** 유물 장착 시 효과 정상 발동

---

## Phase 8 — UI

**목표:** 플레이 가능한 화면

### 구현 대상
- `HandView`, `CardView` (오브젝트 풀링)
- `BattleHUD`
- `MapView`
- `RewardView`

**완료 기준:** 터치로 카드 사용 → 전투 진행 → 보상 선택 가능

---

## Phase 9 — 세이브 / 로드

**목표:** 런 중 앱 종료 후 복귀 가능

### 구현 대상
- `RunData` JSON 직렬화
- `SaveManager`

**완료 기준:** 앱 재실행 시 런 이어하기 가능

---

## Phase 10 — 폴리싱 + 빌드

- 밸런싱
- 버그 수정
- Android 빌드 + 실기기 테스트
