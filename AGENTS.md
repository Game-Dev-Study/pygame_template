# Project Instructions

이 저장소는 Pygame 기반 2D 게임을 시작하기 위한 확장형 템플릿이다. 파일을 읽고 쓸 때 한국어가 포함될 수 있으므로 UTF-8 인코딩을 우선 사용한다.

## Structure

- `.project/`는 게임 기획, 메모, 작업 초안을 둔다.
- `main.py`는 일반 실행 진입점이며 `Game`을 실행한다.
- `develop.py`는 개발 실행 진입점이며 `DebugGame`과 HUD를 실행한다.
- `settings.py`에는 사용자 설정, 입력 상수, 개발 설정을 둔다.
- `core/`는 Pygame 초기화, 메인 루프, 입력 수집, 화면 출력, 사운드를 담당한다.
- `scenes/`는 화면 상태와 흐름, overlay, scene 전환, 모델과 UI의 조정을 담당한다.
- `ui/`는 transform, renderer, UI 상호작용 기반 클래스와 화면별 UI를 담당한다.
- `game/`은 게임 규칙, 상태 모델, 여러 화면에 걸친 관리자와 독립 로직을 둔다.
- `assets/`는 이미지, 폰트, 사운드 리소스를 둔다.
- `data/`는 저장 데이터와 런타임 데이터 파일을 둔다.
- `utilities/`는 상태를 소유하지 않는 범용 보조 함수를 둔다.
- `docs/`는 아키텍처, 클래스 계층, UI 및 주요 기능 흐름을 기록한다.

## Architecture

```text
Core
  ↓
Scenes ──→ UI
  ├──────→ Game
  └──────→ Utilities

Game ──→ Utilities
Game ─X→ Scenes / UI
```

- scene은 화면 흐름과 모델·UI 사이의 조정을 맡는다.
- game은 scene과 UI를 import하지 않는다.
- 게임 규칙, 상태, 여러 화면에 걸친 관리 로직은 `game/`에 모은다.
- 상태를 소유하지 않는 범용 helper만 `utilities/`에 둔다.
- 타입 힌트만을 위한 역방향 참조는 `TYPE_CHECKING` 아래에 둔다.
- 여러 컬렉션이나 상태를 동시에 바꾸는 작업은 해당 상태를 소유한 game 객체의 메서드로 캡슐화한다.

## General Rules

- Pygame 이벤트 큐는 `Game.read_inputs()`에서만 읽는다. scene이나 UI 내부에서 `pygame.event.get()`을 다시 호출하지 않는다.
- 게임 로직과 렌더링 좌표는 기본적으로 `virtual_screen` 기준 좌표를 사용한다.
- 실제 화면 크기는 코드가 제공하는 해상도 값으로만 변경하며 `pygame.RESIZABLE`을 사용하지 않는다.
- 런타임에 바뀔 수 있는 설정은 값 직접 import보다 `import settings` 후 `settings.X` 또는 helper 함수로 읽는다.
- 객체를 화면에서 제거할 때 listener list를 직접 수정하지 말고 해당 객체의 `destroy()`를 호출한다.
- 미리보기와 실제 적용이 같은 계산 경로를 사용하도록 구성한다.
- 무작위 계산은 필요하면 외부 난수 객체를 받을 수 있게 하여 재현성과 테스트 가능성을 확보한다.
- package `__init__.py`에는 실제 외부 사용 요소만 공개한다. 다른 package 소유 타입을 편의상 재수출하지 않는다.

## Scene-Based Implementations

- `Renderer`, `UIElement` 같은 기반 클래스를 상속한 화면별 구현체는 관리 root 아래의 `{scene_name}/`에 둔다.
- 새 구현체를 만들기 전에 해당 root의 `global/`에서 재사용 가능한 요소를 먼저 찾는다.
- 파일 경로는 `{root_folder}/{scene_name}/{element_name}.py` 형태를 사용한다.
- 여러 scene에서 공유하는 구현체는 `{root_folder}/global/{element_name}.py`로 옮긴다.
- 공용 구현체는 특정 scene 상태를 암묵적으로 읽지 않고 필요한 데이터와 callback을 생성자로 받는다.
- 각 하위 폴더의 `__init__.py`에는 그 폴더에서 외부에 공개할 구현체만 모은다.

## Documentation and Verification

- 모듈 책임이나 의존 방향이 바뀌면 `docs/architecture.md`를 함께 갱신한다.
- 상속 구조가 바뀌면 `docs/class_hierarchy.md`, UI 또는 주요 기능 흐름이 바뀌면 대응 문서를 갱신한다.
- 기본 확인은 `.venv\Scripts\python.exe -m compileall core scenes ui game utilities settings.py main.py develop.py`를 사용한다.
- 데이터 파일을 수정했다면 UTF-8로 읽히는지와 JSON 문법이 유효한지 확인한다.
