# UI Flow

## Scene UI Overview

새 UI의 재사용 가능성을 검토할 때 가장 먼저 확인하는 목록이다. 여기서 역할이 비슷한 UI가 있는 scene을 고른 뒤, 해당 `ui/{scene_name}/__init__.py`와 관련 구현 파일만 자세히 확인한다.

| 위치 | UI 개요 |
|---|---|
| `ui/global/` | 현재 공용 UI 구현 없음 |
| `ui/title_scene/` | 현재 화면 전용 UI 구현 없음 |

UI를 추가하거나 이동할 때 이 표에 구현 이름을 모두 나열하기보다, 해당 scene에 어떤 목적의 UI가 있는지 빠르게 선별할 수 있는 수준으로 요약한다. 각 UI의 구체적인 역할은 해당 scene의 `__init__.py`에 기록한다.

## Frame

```text
Game.read_inputs()
→ Scene.update(...)
  → background listeners
  → UI focus / hover / click
  → foreground listeners
→ Scene.draw()
→ Game.present()
```

입력 이벤트는 프레임마다 `Game.read_inputs()`에서 한 번만 소비한다. scene과 UI는 전달받은 입력을 사용한다.

## Rendering

모든 게임 요소는 고정 크기 `virtual_screen` 좌표로 그린다. `Game.present()`가 이를 실제 창 크기에 맞게 스케일하고 letterbox를 적용한다.

## Focus and Lifetime

`ui_listener`는 뒤에서부터 검사하므로 나중에 생성된 UI가 먼저 focus를 얻는다. 화면에서 제거할 때는 listener 목록을 직접 수정하지 않고 `destroy()`를 호출한다.
