# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Gravedraw** — Unity 2022.3.62f3 기반 모바일 2D 덱빌딩 게임. Android/iOS 타겟.
로그라이크 구조의 턴제 전투 + 카드 컬렉션 진행 방식.

- **Unity Project Root:** `Gravedraw/` (리포지토리 하위 폴더)
- **Remote:** https://github.com/SongJiUk/Gravedraw.git

## Build & Run

- **프로젝트 열기:** Unity Hub → Open → `Gravedraw/` 폴더 선택
- **에디터 실행:** Unity Editor Play 버튼
- **빌드:** File → Build Settings → Android 또는 iOS 선택 → Build

## Architecture

### 폴더 구조

```
Assets/
├── Scripts/
│   ├── Card/           # 카드 데이터, 효과, 덱 관리
│   ├── Battle/         # 전투 흐름, 턴 상태머신, 적 AI
│   ├── UI/             # 손패 뷰, 카드 뷰, 전투 HUD
│   ├── Run/            # 런 진행, 맵 노드, 보상 시스템
│   ├── Manager/        # Managers.cs 싱글톤 허브
│   └── Data/           # GameData, 세이브/로드
├── Cards/              # CardData ScriptableObject 에셋
├── Scenes/
│   ├── TitleScene      # 타이틀/로딩
│   ├── MapScene        # 맵 탐색
│   └── BattleScene     # 전투
└── Tests/
    ├── EditMode/       # 순수 로직 단위 테스트
    └── PlayMode/       # 런타임 통합 테스트
```

### Manager 싱글톤 허브

`Managers.cs`를 중앙 진입점으로 사용. 모든 매니저는 정적 프로퍼티로 접근:

```csharp
Managers.Battle    // BattleManager — 턴 진행, 승패 판정
Managers.Deck      // DeckManager — 드로우/핸드/버리기 더미
Managers.Run       // RunManager — 런 상태(층, 보상, 카드 컬렉션)
Managers.UI        // UIManager — UI 생명주기
Managers.Resource  // ResourceManager — 에셋 로딩/캐싱
Managers.Data      // DataManager — 카드/적 데이터 로드
Managers.Save      // SaveManager — 세이브/로드
```

### 카드 시스템

- `CardData` (ScriptableObject) — 카드 정의: 이름, 코스트, 효과 목록, 스프라이트
- `CardEffect` — 인터페이스 기반 효과 단위 (`IDamageEffect`, `IDrawEffect`, `IBuffEffect` 등)
- `DeckManager` — 드로우 더미 / 핸드 / 버리기 더미 상태 관리 및 셔플 처리

```csharp
// 카드 효과는 조합 가능한 단위로 설계
public interface ICardEffect
{
    void Execute(BattleContext ctx);
}
```

### 전투 상태머신 (BattleStateMachine)

```
StartTurn → PlayerTurn → (카드 사용) → EndPlayerTurn
         → EnemyTurn  → (적 행동)   → EndEnemyTurn
         → Victory / Defeat
```

- 상태는 enum + `IBattleState` 인터페이스로 관리
- 각 상태 진입/종료 시 이벤트 발행 (`OnTurnStart`, `OnTurnEnd`)

### UI 시스템

- `UI_Base.cs` — enum 기반 바인딩 (idleGame 구조 유지):
  ```csharp
  Bind<Button>(typeof(Buttons));
  Get<Button>(Buttons.EndTurn);
  ```
- `HandView` — 카드 손패 표시, 터치 드래그로 카드 사용
- `CardView` — 개별 카드 프리팹 (CardData 기반 동적 세팅)
- `EnemyIntentView` — 적 다음 행동 의도 표시

### 런 진행 (RunManager)

- 맵은 노드 기반 (전투 / 이벤트 / 상점 / 보스)
- 런 중 획득 카드 목록 + 유물(Relic) 목록 보관
- 런 종료 시 세이브 삭제, 영구 진행도(해금 등)만 보존

### 데이터 레이어

- 카드/적 정의: `CardData`, `EnemyData` (ScriptableObject)
- 런 상태: `RunData.cs` — JSON 직렬화, 로컬 저장 (PlayerPrefs 또는 파일)
- 영구 데이터: `PermanentData.cs` — 해금 카드, 업적 등

### 주요 패키지

| 패키지 | 용도 |
|---|---|
| com.unity.2d.animation 9.2.0 | 캐릭터/카드 스켈레탈 애니메이션 |
| com.unity.2d.aseprite 1.1.9 | 픽셀아트 스프라이트 직접 임포트 |
| com.unity.2d.psd-importer 8.1.0 | 레이어드 카드 아트 임포트 |
| com.unity.2d.pixel-perfect 5.1.0 | 픽셀아트 렌더링 |
| com.unity.timeline 1.7.7 | 전투 연출/컷씬 |
| com.unity.burst 1.8.21 | 고성능 로직 컴파일 |
| com.unity.collections 1.2.4 | Burst/Jobs용 네이티브 컬렉션 |

### 모바일 규칙

- UI는 **세로 화면(Portrait)** 기준으로 설계
- 카드 조작은 `IPointerDownHandler` / `IDragHandler` 기반 터치 입력
- `CardView`는 오브젝트 풀링으로 재사용 (PoolManager 활용)
- `CardData` ScriptableObject는 공유 참조 — 런타임에 복사 금지
- Update() 직접 사용 금지 → `UpdateManager`의 `ITickable` 등록 방식 사용

## IDE Setup

- **권장:** Visual Studio 또는 JetBrains Rider
- **VS Code:** `.vscode/` 설정 존재 — `visualstudiotoolsforunity` 확장 설치 필요
- **디버거:** `.vscode/launch.json`의 "Attach to Unity" 사용

---

## 언어 규칙

- 모든 답변과 코드 주석은 반드시 한국어로 작성할 것
- 추측하지 말고, 애매한 게 있으면 질문할 것

## 학습 원칙

- 코드를 바로 짜주지 말고, 접근 방법과 힌트를 먼저 제시할 것
- 사용자가 직접 시도한 코드가 있으면 그걸 기반으로 개선 방향만 알려줄 것
- 왜 이렇게 구현하는지 이유를 항상 설명할 것
- 모르는 개념이 나오면 관련 Unity 공식 문서 링크 함께 제공할 것

## 코드 요청 시 대응 방식

- 1단계: 문제 접근 방법 설명
- 2단계: 사용자가 직접 구현 시도
- 3단계: 요청 시에만 전체 코드 제공
- 단, 반복 작업/자동화/데이터 변환은 바로 코드 제공 가능

## 코드 리뷰 방식

- 잘된 부분 먼저 언급
- 문제점은 왜 문제인지 이유 설명
- 개선 방법은 힌트로 먼저 제시, 직접 수정은 요청 시에만

## 코드 리뷰 스타일

- 변경 사항 요약을 먼저 설명
- 성능 영향도 반드시 언급
- 모바일 메모리/배터리 영향 고려해서 리뷰

## Git 컨벤션

- 커밋 메시지는 한국어로 작성
- 형식: `[타입] 내용` (예: `[feat] 카드 드로우 기능 추가`)
- 타입: `feat` / `fix` / `refactor` / `chore` / `docs`

## 프로젝트 세팅 체크리스트

- [ ] `Assets/Tests/EditMode/` + `.asmdef` 생성 (순수 로직 단위 테스트)
- [ ] `Assets/Tests/PlayMode/` + `.asmdef` 생성 (런타임 통합 테스트)
- [ ] `run_tests.sh` — Unity CLI 자동 실행 + 한국어 리포트
- [ ] 테스트 대상: 카드 효과 수식, 덱 셔플 로직, 턴 진행 흐름
- [ ] 테스트 메서드명은 한국어로 작성 가능
- [ ] 싱글톤 의존성 최소화 → `BattleContext` 주입 방식으로 테스트 가능하게 설계
