# Composer des timelines avec GFX.Animation

`GFX.Animation` décrit des changements de position, rotation, échelle et
couleur au fil du temps sans exposer de runner d’animation ni de renderer. Une
timeline peut être séquencée, exécutée en parallèle, bouclée ou jouée en
aller-retour, puis attachée à une entité ECS avec son composant `Playback`.

[Read this documentation in English.](../EN/README.md)

## Installer le package

```text
silex install GFX.Animation
```

GFX.Animation demande Silex 0.39.0 ou une version plus récente.

## Construire une timeline

`move_to`, `rotate_to`, `scale_to` et `color_to` créent des timelines pour un
seul canal. `sequence` et `parallel` les composent :

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

Chaque clip possède une durée positive et un `Easing`. Une composition en
parallèle rejette les clips qui contrôlent le même canal pendant une période
commune. Un enfant déjà marqué `loop` ou `ping_pong` ne peut pas être composé :
les options de lecture appartiennent à la timeline finale.

## Lire une timeline dans l’ECS

`play` crée un composant ECS `Playback`. `Animation.Plugin` l’avance à partir
de `FrameTime` et applique ses canaux aux transformations Scene2D, à la couleur
du Canvas Scene2D et aux transformations Scene3D. La première valeur observée
sur le composant devient la valeur initiale du canal.

`loop` répète la timeline. `ping_pong` alterne les directions avant et arrière
et peut se combiner avec `loop`. `Easing.constant` maintient la valeur initiale
jusqu’à la frontière du clip, y compris en lecture inverse.

Le package contribue `Plugins.Animation` au catalogue `GFX.Plugins`. Il garde
son stockage de clips privé et cible volontairement les transformations et
couleurs de GFX plutôt qu’un système générique de réflexion sur les propriétés.

## Voir les démonstrations

Les applications visuelles appartiennent à Silex-Examples :

- [Galerie des courbes d’easing](https://github.com/Matanek/Silex-Examples/blob/main/Sources/EasingGallery.sx)
- [Timeline 2D](https://github.com/Matanek/Silex-Examples/blob/main/Sources/Timeline2D.sx)
- [Timeline 3D](https://github.com/Matanek/Silex-Examples/blob/main/Sources/Timeline3D.sx)
