# GFX.Animation

GFX.Animation composes reusable timelines for 2D and 3D position, rotation,
scale and color. A timeline can be sequenced, run in parallel, looped or played
back and forth, then attached to an ECS entity through its `Playback` component.

```text
silex install GFX.Animation
```

```sx
use GFX.Animation
use GFX.Components
use GFX.ECS
use STD.Math

let route = Animation.sequence([
    Animation.move_to(Math.Vec2(10.0, 0.0), 1.0),
    Animation.wait(0.25),
    Animation.move_to(Math.Vec2(), 1.0),
])..loop()

world.spawn(ECS.EntityRecipe()
    ..with(Components.Transform2D())
    ..with(route.play())
)
```

The package owns its timeline vocabulary, easing functions, playback component,
application plugin, tests and examples. The plugin integrates public GFX
Application, ECS, Scene2D, Canvas and Scene3D capabilities without privileged
access.

GFX.Animation contributes `Plugins.Animation` to the `GFX.Plugins` catalog.
`Tests/Consumer` verifies the public package and catalog boundary from an
anonymous application workspace.

See [Docs/README.md](Docs/README.md) for composition and playback semantics.
