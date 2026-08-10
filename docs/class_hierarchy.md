# Class Hierarchy

현재 템플릿의 주요 상속 구조다.

```text
Game
└─ DebugGame

Scene
├─ TitleScene
└─ Hud

Transform
├─ Renderer
│  ├─ AnimatedRenderer
│  └─ ShiftRenderer
└─ UIElement
```

게임별 scene, renderer, UI element는 대응하는 기반 클래스를 상속해 확장한다.
