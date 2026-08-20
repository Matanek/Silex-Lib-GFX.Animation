# Compose animation timelines

`GFX.Animation` describes changes over time without exposing an animation
runner or renderer. `move_to`, `rotate_to`, `scale_to` and `color_to` create
single-channel timelines; `sequence` and `parallel` compose them.

Each clip has a positive duration and an `Easing`. Parallel timelines reject
overlapping clips that control the same channel. A composed child cannot
already carry `loop` or `ping_pong`; playback options belong to the final
timeline.

Calling `play` creates an ECS `Playback` component. `Animation.Plugin` advances
that component from `FrameTime` and applies its channels to Scene2D transforms,
Scene2D canvas color and Scene3D transforms. The first observed component value
becomes the starting value of its channel.

`loop` repeats the timeline. `ping_pong` alternates forward and reverse
directions; it can be combined with `loop`. `Easing.constant` holds the source
value until the clip boundary, including in the reverse direction.

The package deliberately models GFX transform and color animation. It does not
expose its clip storage or pretend to be a generic property-reflection system.
