# Animate values and compose timelines with GFX.Animation

`GFX.Animation` evolves a value in an update loop with `Tween<T>`. It also
composes position, rotation, scale, and color changes into timelines attached
to ECS entities through `Playback`.

[Lire cette documentation en français.](../FR/README.md)

## Install the package

```text
silex install GFX.Animation
```

GFX.Animation requires Silex 0.43.0 or newer.

## Animate a value in an update loop

A `Tween<T>` retains its current value and playback progress. Build its steps
once, add it to the application resources, then call `advance` with the frame
delta. The following fragment shows the resource and its `update` system:

```sx
use GFX.Animation
use GFX.Application
use GFX.Resources

func update_drawing(
    time:@Resources.FrameTime,
    pulse:&Animation.Tween<float>
) {
    let amount = pulse.advance(time.delta)
    // Redraw here from `amount`.
}

Application()
    ..add_resource(Animation.tween(0.0, 1.0, 0.6,
        easing:Animation.Easing.sine_in_out)
        ..ping_pong()
        ..loop()
    )
    ..add_system(Application.Schedule.update, update_drawing)
```

`advance` does not rebuild the tween or access any ECS entity. It reuses the
current value and a cursor over the configured steps.

The same type covers a composed animation. `to` adds an interpolation and
`wait` holds the latest value:

```sx
var opacity:Animation.Tween<float> = Animation.tween(0.0)
    ..to(1.0, 0.4, easing:Animation.Easing.out)
    ..wait(0.2)
    ..to(0.25, 0.6, easing:Animation.Easing.in_out)
```

`value` reads the current value without advancing it. `seek` positions the
tween, `restart` returns it to the beginning, and `finished` reports whether a
non-looping tween reached its end. `ping_pong` plays the steps in both
directions and `loop` repeats the cycle.

The `float` type, 2D and 3D vectors, 4D vectors, colors, and quaternions have
built-in interpolation. For another type `T`, construct
`Animation.Tween<T>(initial, interpolate)` with a `func(T, T, float) T`
function; `Tween<T>` therefore remains independent from property reflection.

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

`play` creates an ECS `Playback` component. `Plugins.Animation` advances it from
`FrameTime` and applies its channels to Scene2D transforms, Scene2D Canvas
color, and Scene3D transforms. The first observed component value becomes the
channel's starting value.

In an application hosted by `Plugins.SceneManager`, keep
`Plugins.Animation()` on the Application for the time resource and add
`Plugins.AnimationContent()` to the scene. The same ordinary systems then
target components in its local `World`; no animated-object protocol is
required.

```sx
use GFX.Plugins

application
    ..add_plugin(Plugins.Animation())
    ..add_plugin(Plugins.SceneManager(scene))

scene.add_plugin(Plugins.AnimationContent())
```

`Plugins.AnimationContent()` requires `Plugins.Animation()` to be resolved on the
Application; an incomplete scene is rejected before mounting.

`loop` repeats the timeline. `ping_pong` alternates forward and reverse
directions and can be combined with `loop`. `Easing.constant` holds the source
value until the clip boundary, including in reverse playback.

The package contributes `Plugins.Animation` and `Plugins.AnimationContent`
directly to the `GFX.Plugins` catalog, so catalog completion exposes both
levels. It keeps clip storage private and deliberately targets GFX transforms
and colors instead of exposing a generic property-reflection system.

## See the demonstrations

Visual applications belong to Silex-Examples:

- [Easing gallery](https://github.com/Matanek/Silex-Examples/blob/main/Sources/EasingGallery.sx)
- [2D timeline](https://github.com/Matanek/Silex-Examples/blob/main/Sources/Timeline2D.sx)
- [3D timeline](https://github.com/Matanek/Silex-Examples/blob/main/Sources/Timeline3D.sx)
