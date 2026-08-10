# Utilities Instructions

`utilities/`는 상태를 소유하지 않고 여러 모듈에서 재사용하는 범용 보조 함수를 둔다.

## Rules

- 특정 scene, UI, renderer에만 필요한 helper는 해당 영역 가까이에 둔다.
- 두 개 이상의 영역에서 재사용되는 순수 helper만 `utilities/`로 옮긴다.
- Pygame 전역 상태나 scene listener를 직접 건드리는 helper는 이름과 책임을 명확히 한다.
- 여러 화면에 걸친 상태나 게임 규칙을 소유한다면 `utilities/`가 아니라 `game/`에 둔다.
