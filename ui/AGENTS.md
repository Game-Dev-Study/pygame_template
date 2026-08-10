# UI Instructions

`ui/`는 `Transform`, `Renderer`, `AnimatedRenderer`, `ShiftRenderer`, `UIElement`를 관리한다.

## Transform

- 위치 기준은 center 좌표다.
- transform 변경 반응은 `set_transform()`을 바꾸기보다 `on_transform_updated()` 훅을 사용한다.

## Renderer

- renderer 계열은 `Transform`을 상속하고 생성 시 스스로 scene listener에 등록된다.
- 모든 renderer는 `draw_listeners`에 등록된다.
- `AnimatedRenderer`와 `ShiftRenderer`는 `background=True`면 `background_listeners`, 아니면 `update_listeners`에 추가 등록된다.
- 이미지 경로 또는 `pygame.Surface`를 받을 수 있다.
- 크기가 바뀌면 원본 이미지를 현재 transform 크기에 맞춰 다시 스케일한다.

## UIElement

- `UIElement`는 상호작용 영역을 담당하고 시각 출력은 연결된 `Renderer`가 담당한다.
- renderer를 받지 않으면 반드시 scene, pos, size를 받아 기본 `Renderer`를 직접 만든다.
- renderer를 받으면 renderer의 transform과 scene을 기준으로 초기화한다.
- `background=True`면 `background_listeners`, 아니면 `update_listeners`에 등록된다.
- UI 제거는 반드시 `destroy()`를 호출한다.
- 자신이 만든 renderer는 `destroy()` 때 함께 제거한다. 외부에서 받은 renderer는 만든 쪽이 제거 책임을 가진다.

## Sub UI

- 부모 UI를 먼저 만들고 자식 UI를 나중에 만든 뒤 `add_sub_ui(...)`로 연결한다.
- scene은 `ui_listener`를 뒤에서부터 검사하므로 생성 순서가 focus 순서에 영향을 준다.

## Exports

- `Renderer`, `AnimatedRenderer`, `ShiftRenderer`, `UIElement`처럼 상속해서 쓰는 클래스는 루트 `AGENTS.md`의 scene별 구현체 관리 규칙을 따른다.
- 화면별 UI는 반드시 `ui/{scene_name}/` 폴더 아래에 두고, UI 하나마다 별도의 `.py` 파일로 나눈다.
- 파일 경로는 `ui/{scene_name}/{ui_name}.py` 형식을 사용한다.
- `ui/{scene_name}/__init__.py`에는 외부에 공개할 UI를 import하고, 각 UI가 어떤 역할을 하는지 설명하는 주석을 함께 작성한다.
- `Renderer`는 직접 사용 빈도가 높으므로 package export 대상이다.

## Reuse and Dependencies

- 새 UI를 만들기 전에는 모든 scene의 `__init__.py`나 구현 파일을 무작정 읽지 않는다.
- 먼저 `docs/ui_flow.md`의 scene별 UI 개요를 확인해 기존 UI의 역할과 재사용 후보를 큰 범위에서 찾는다.
- 재사용 가능성이 있는 scene이 확인되면 그 scene의 `ui/{scene_name}/__init__.py`에서 UI별 설명을 읽고, 관련 구현 파일만 세부적으로 검토한다.
- `docs/ui_flow.md`만으로 후보를 판단하기 어려울 때에만 검색 범위를 다른 scene으로 넓힌다.
- 다른 scene에서도 재사용할 수 있는 UI라면 복제하지 않고 `ui/global/`로 옮긴 뒤 필요한 scene에서 함께 사용한다.
- 공용 UI는 특정 scene 속성을 직접 읽지 않고 필요한 데이터와 callback을 생성자 입력으로 받는다.
- UI는 game 상태를 소유하거나 게임 규칙을 계산하지 않는다.
- 화면별 export는 해당 하위 폴더의 `__init__.py`에 모은다.
- UI를 추가하거나 다른 scene 또는 `global/`로 이동했다면 `docs/ui_flow.md`의 scene별 UI 개요도 함께 갱신한다.
