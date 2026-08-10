# Pygame 2D Game Template

고정 가상 해상도, scene 생명주기, listener 기반 렌더링과 UI를 제공하는 Pygame 2D 게임 템플릿이다. 작은 프로젝트로 바로 시작할 수 있고, 게임이 커지면 `game/`을 사용해 규칙과 상태를 화면 코드에서 분리할 수 있다.

## 시작하기

Python 가상 환경을 만든 뒤 의존성을 설치한다.

```powershell
python -m venv .venv
.venv\Scripts\python.exe -m pip install -r requirements.txt
```

일반 실행:

```powershell
.venv\Scripts\python.exe main.py
```

입력 상태 HUD를 포함한 개발 실행:

```powershell
.venv\Scripts\python.exe develop.py
```

## 프로젝트 구조

```text
project-root/
├─ .project/              # 기획, 메모, 작업 초안
├─ assets/
│  ├─ images/
│  ├─ fonts/
│  └─ sounds/
├─ core/                  # 초기화, 루프, 입력, 화면 출력, 사운드
├─ scenes/                # 화면 상태, overlay, 전환, 계층 간 조정
├─ ui/                    # transform, renderer, UI 상호작용
│  ├─ global/             # 여러 scene이 공유하는 UI(필요할 때 생성)
│  └─ {scene_name}/       # 특정 scene 전용 UI(필요할 때 생성)
├─ game/                  # 게임 규칙, 상태, 관리자와 독립 로직
├─ utilities/             # 상태를 소유하지 않는 범용 helper
├─ data/                  # 저장 및 런타임 데이터
├─ docs/                  # 아키텍처와 주요 흐름 문서
├─ settings.py            # 사용자, 입력, 개발 설정
├─ main.py                # 일반 실행 진입점
└─ develop.py             # 개발 실행 진입점
```

각 영역의 세부 규칙은 해당 폴더의 `AGENTS.md`에 있다. `game/`은 게임 고유 코드를 강제하지 않는 빈 확장점이며, 기능이 생길 때 필요한 파일부터 추가한다.

## 아키텍처

```text
Core
  ↓
Scenes ──→ UI
  ├──────→ Game
  └──────→ Utilities

Game ──→ Utilities
Game ─X→ Scenes / UI
```

- `core`는 Pygame 실행 환경을 소유한다.
- `scenes`는 화면 흐름을 관리하고 UI와 game 객체를 조정한다.
- `ui`는 표시와 상호작용만 담당하며 게임 상태를 소유하지 않는다.
- `game`은 게임 규칙, 상태, 여러 화면에 걸친 관리자와 독립 로직을 담당하고 scene/UI를 import하지 않는다.
- `utilities`에는 상태를 소유하지 않는 범용 helper만 둔다.

상세 내용은 [아키텍처](docs/architecture.md), [클래스 계층](docs/class_hierarchy.md), [UI 흐름](docs/ui_flow.md), [기능 흐름](docs/feature_flow.md) 문서를 참고한다.

## 핵심 실행 흐름

```text
Game.run()
├─ update()
│  ├─ read_inputs()
│  └─ Scene.update(...)
└─ draw()
   ├─ virtual_screen에 Scene.draw()
   └─ present()로 실제 창에 스케일 출력
```

Pygame 이벤트 큐는 `Game.read_inputs()`에서 프레임당 한 번만 읽는다. scene과 UI는 전달받은 `game_events`, 가상 화면 기준 `mouse_position`, `wheel_move`를 사용한다.

게임 로직과 렌더링 좌표는 `virtual_screen`을 기준으로 한다. 실제 창 크기는 `game.set_screen_size(width, height)`로 변경하며, 사용자 드래그 리사이즈와 `pygame.RESIZABLE`은 사용하지 않는다.

## Scene과 listener

새 화면은 `Scene`을 상속한다.

- `scene_initialize()`: 초기 객체와 상태 생성
- `scene_background_update(...)`: overlay가 있어도 실행할 로직
- `scene_update(...)`: overlay가 없을 때 실행할 로직
- `scene_draw()`: 화면별 직접 그리기

객체는 역할에 따라 `background_listeners`, `update_listeners`, `draw_listeners`, `ui_listener`에 등록된다. 제거할 때 목록을 직접 수정하지 말고 객체의 `destroy()`를 호출한다.

## UI 확장

```text
Transform
├─ Renderer
│  ├─ AnimatedRenderer
│  └─ ShiftRenderer
└─ UIElement
```

- `Transform`은 중심 좌표 기반 위치와 크기를 관리한다.
- `Renderer` 계열은 시각 출력과 draw listener 등록을 담당한다.
- `UIElement`는 focus, hover, click 영역을 담당하고 출력은 renderer에 위임한다.
- 생성 순서는 draw/update 순서와 UI focus 우선순위에 영향을 준다.
- UI가 직접 만든 renderer는 UI가 제거하고, 외부에서 받은 renderer는 만든 쪽이 제거한다.

화면 전용 UI는 `ui/{scene_name}/` 아래에서 UI 하나마다 별도의 `.py` 파일로 나눈다. `ui/{scene_name}/__init__.py`에는 공개할 UI import와 함께 각 UI의 역할을 설명하는 주석을 작성한다.

새 UI를 만들기 전에는 먼저 `docs/ui_flow.md`의 scene별 UI 개요에서 역할이 비슷한 UI가 있는지 크게 확인한다. 후보 scene이 있을 때만 해당 `ui/{scene_name}/__init__.py`의 설명과 관련 구현 파일을 자세히 검토한다. 모든 scene의 파일을 처음부터 전부 읽을 필요는 없다.

다른 scene에서도 재사용할 수 있다면 코드를 복제하지 않고 `ui/global/{element_name}.py`로 옮겨 함께 사용한다. 공용 UI는 특정 scene 내부 상태를 직접 읽지 않고 데이터와 callback을 입력받는다. UI를 추가하거나 이동하면 `docs/ui_flow.md`의 개요도 함께 갱신한다.

## 게임 코드 확장

게임 규칙이 단순한 동안에는 scene만으로 시작해도 된다. 규모가 커질 때 다음 기준으로 분리한다.

- 정의 데이터, 런타임 인스턴스, 상태 및 계산 모델 → `game/`
- 여러 화면에 걸친 상태 관리자, 절차적 생성, 경로·가시성·시간 진행 같은 독립 로직 → `game/`
- 상태를 소유하지 않는 범용 함수 → `utilities/`

여러 상태를 함께 바꾸는 작업은 해당 상태 소유자의 메서드로 캡슐화한다. 미리보기와 실제 적용은 같은 계산 경로를 사용하고, 무작위 계산은 난수 객체를 주입할 수 있게 만들면 테스트하기 쉽다.

처음에는 `game/player.py`, `game/inventory.py`처럼 필요한 파일만 추가한다. 코드가 충분히 많아졌을 때만 `models/`, `states/`, `managers/`, `systems/` 같은 하위 package로 나눈다.

## 설정과 공개 API

런타임에 바뀔 수 있는 설정은 값을 직접 import하지 않고 다음처럼 모듈에서 읽는다.

```python
import settings

volume = settings.SFX
screen_size = settings.get_screen_size()
```

각 package의 `__init__.py`에는 실제 외부에서 사용할 요소만 공개한다. 상속용 기반 클래스는 필요한 모듈에서 직접 import하고, 다른 package가 소유한 타입을 편의를 위해 재수출하지 않는다. 기본 `Renderer`는 화면 구성에서 자주 사용하므로 `ui` package에서 공개한다.

## 검증

```powershell
.venv\Scripts\python.exe -m compileall core scenes ui game utilities settings.py main.py develop.py
```

JSON 데이터를 수정했다면 UTF-8로 읽히는지와 JSON 문법이 유효한지도 확인한다. 구조나 책임을 바꿨다면 `docs/`의 관련 문서를 함께 갱신한다.
