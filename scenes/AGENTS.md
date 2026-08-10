# Scene Instructions

`scenes/`는 화면 단위 상태와 흐름, overlay scene, scene 전환, listener 흐름, 모델과 UI 사이의 조정을 담당한다.

## Scene Lifecycle

- 새 화면은 `Scene`을 상속하고 보통 별도 `scene_name.py` 파일로 만든다.
- 초기 객체 생성과 상태값 설정은 `scene_initialize()`에 둔다.
- overlay 여부와 상관없이 돌아야 하는 로직은 `scene_background_update(...)`에 둔다.
- overlay가 없을 때만 돌아야 하는 로직은 `scene_update(...)`에 둔다.
- 직접 그릴 것이 있으면 `scene_draw()`를 오버라이드하고, listener도 그려야 하면 원하는 위치에서 `super().scene_draw()`를 호출한다.

## Listeners

- `background_listeners` 객체는 `background_update(...)`를 가져야 한다.
- `update_listeners` 객체는 `update(...)`를 가져야 한다.
- `draw_listeners` 객체는 `draw(screen)`을 가져야 한다.
- `ui_listener` 객체는 `pos_check(...)`, focus hook, click hook을 가져야 한다.
- 객체 제거는 직접 list 조작 대신 각 객체의 `destroy()`를 호출한다.

## UI Focus

- `operate_ui(...)`는 `ui_listener`를 뒤에서부터 검사한다.
- 나중에 생성된 UI가 먼저 focus를 잡는다.
- 클릭 우선순위는 왼클릭, 오른클릭, 휠클릭 순서다.

## Exports

- `scenes/__init__.py`에는 실제 외부에서 바로 사용할 scene만 공개한다.
- 개발용 HUD처럼 `DebugGame`에서만 직접 쓰는 요소는 export하지 않는다.

## Dependencies

- 재사용 가치가 있는 시각·상호작용 구현은 `ui/`로 분리하고 scene은 데이터와 callback을 전달한다.
- 게임 규칙, 상태 계산, 여러 화면에 걸친 관리자와 독립 로직은 `game/`에 둔다.
- 여러 컬렉션을 동시에 변경할 때 scene에서 직접 조작하지 않고 상태 소유 객체의 메서드를 호출한다.
