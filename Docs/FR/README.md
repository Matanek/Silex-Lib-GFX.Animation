# Animer des valeurs et composer des timelines avec GFX.Animation

`GFX.Animation` fait évoluer une valeur dans une boucle de mise à jour avec
`Tween<T>`. Il compose aussi des changements de position, rotation, échelle et
couleur dans des timelines attachées aux entités ECS avec `Playback`.

[Read this documentation in English.](../EN/README.md)

## Installer le package

```text
silex install GFX.Animation
```

GFX.Animation demande Silex 0.43.0 ou une version plus récente.

## Animer une valeur dans une mise à jour

Un `Tween<T>` conserve sa valeur courante et son avancement. Construisez ses
étapes une seule fois, ajoutez-le aux ressources de l’application, puis appelez
`advance` avec le delta de la frame. Le fragment suivant montre la ressource et
son système `update` :

```sx
use GFX.Animation
use GFX.Application
use GFX.Resources

func update_drawing(
    time:@Resources.FrameTime,
    pulse:&Animation.Tween<float>
) {
    let amount = pulse.advance(time.delta)
    // Redessiner ici à partir de `amount`.
}

Application()
    ..add_resource(Animation.tween(0.0, 1.0, 0.6,
        easing:Animation.Easing.sine_in_out)
        ..ping_pong()
        ..loop()
    )
    ..add_system(Application.Schedule.update, update_drawing)
```

`advance` ne reconstruit pas le tween et n’accède à aucune entité ECS. Il
réutilise la valeur courante et un curseur sur les étapes configurées.

Le même type couvre une animation composée. `to` ajoute une interpolation et
`wait` maintient la dernière valeur :

```sx
var opacity:Animation.Tween<float> = Animation.tween(0.0)
    ..to(1.0, 0.4, easing:Animation.Easing.out)
    ..wait(0.2)
    ..to(0.25, 0.6, easing:Animation.Easing.in_out)
```

`value` lit la valeur courante sans avancer. `seek` positionne le tween,
`restart` le ramène au début et `finished` indique qu’un tween non bouclé a
atteint sa fin. `ping_pong` joue les étapes dans les deux directions et `loop`
répète le cycle.

Le type `float`, les vecteurs 2D et 3D, les vecteurs 4D, les couleurs et les
quaternions possèdent une interpolation intégrée. Pour un autre type `T`, construisez
`Animation.Tween<T>(initial, interpolate)` avec une fonction
`func(T, T, float) T` ; `Tween<T>` reste ainsi indépendant d’un système de
réflexion sur les propriétés.

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

Dans une application hébergée par `Application.Scenes`, gardez
`Animation.Plugin()` sur l’Application pour la ressource temporelle et ajoutez
`Animation.Content()` à la scène. Les mêmes systèmes ordinaires ciblent alors
les composants de son `World` local ; aucun protocole d’objet animé n’est
nécessaire.

```sx
application
    ..add_plugin(Animation.Plugin())
    ..add_plugin(Application.Scenes(scene))

scene.add_plugin(Animation.Content())
```

`Animation.Content()` exige que `Animation.Plugin()` soit résolu sur
l’Application ; une scène incomplète est refusée avant son montage.

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
