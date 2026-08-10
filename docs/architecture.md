# Architecture

이 템플릿은 실행 환경, 화면 조정, UI, 게임 코드, 범용 helper의 책임을 분리한다.

```text
core → scenes → ui
          ├──→ game → utilities
          └──→ utilities
```

- `core`: Pygame 초기화, 입력 수집, 메인 루프, 가상 화면 출력, 사운드
- `scenes`: 화면 생명주기, overlay, 전환, 모델과 UI 조정
- `ui`: transform, renderer, 상호작용 요소와 화면별 구현
- `game`: 게임별 정의, 런타임 상태, 규칙, 관리자와 독립 로직을 위한 확장 지점
- `utilities`: 상태를 소유하지 않는 범용 helper

`game`은 `scenes`와 `ui`를 알지 않는다. `utilities`는 게임 상태를 소유하지 않는다.
