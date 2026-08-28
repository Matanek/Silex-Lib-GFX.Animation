# Compose timelines with GFX.Animation

`GFX.Animation` describes position, rotation, scale, and color changes over
time without exposing an animation runner or renderer. A timeline can be
sequenced, run in parallel, looped, or played back and forth, then attached to
an ECS entity through its `Playback` component.

[Lire cette documentation en français.](../FR/README.md)

## Install the package

```text
silex install GFX.Animation
```

GFX.Animation requires Silex 0.39.0 or newer.

## Build a timeline

`move_to`, `rotate_to`, `scale_to`, and `color_to` create single-channel
timelines. `sequence` and `parallel` compose them:

```sx
use GFX.Animation
use GFX.Components
use GFX.ECS
use STD.Math

func main() {
    let route = Animation.sequence([
        Animation.move_to(Math.Vec2(10.0, 0.0), 1.0),
        Animation.wait(0.25),
        Animation.move_to(Math.Vec2(), 1.0),
    ])..loop()

    var world = ECS.World()
    world.spawn(ECS.EntityRecipe()
        ..with(Components.Transform2D())
        ..with(route.play())
    )
}
```

Each clip has a positive duration and an `Easing`. Parallel composition rejects
clips that control the same channel over an overlapping period. A child already
marked with `loop` or `ping_pong` cannot be composed: playback options belong
to the final timeline.

## Play a timeline in the ECS

`play` creates an ECS `Playback` component. `Animation.Plugin` advances it from
`FrameTime` and applies its channels to Scene2D transforms, Scene2D Canvas
color, and Scene3D transforms. The first observed component value becomes the
channel's starting value.

`loop` repeats the timeline. `ping_pong` alternates forward and reverse
directions and can be combined with `loop`. `Easing.constant` holds the source
value until the clip boundary, including in reverse playback.

The package contributes `Plugins.Animation` to the `GFX.Plugins` catalog. It
keeps clip storage private and deliberately targets GFX transforms and colors
instead of exposing a generic property-reflection system.

## See the demonstrations

Visual applications belong to Silex-Examples:

- [Easing gallery](https://github.com/Matanek/Silex-Examples/blob/main/Sources/EasingGallery.sx)
- [2D timeline](https://github.com/Matanek/Silex-Examples/blob/main/Sources/Timeline2D.sx)
- [3D timeline](https://github.com/Matanek/Silex-Examples/blob/main/Sources/Timeline3D.sx)
