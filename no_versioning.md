

# Arguments for and against versioning

*(This article requires a basic understanding of my vision for Untitled Mod Game. If you are unfamiliar, [have a quick skim](umg_tech_details.md).)*

In Untitled Mod Game, there is going to be no versioning of mods.<br>
This is a very bold call, and it sounds stupid at face value.
But please, hear me out! 
This lil article will explain my thought process.

---------------

In pretty much all software packaging systems, software has
version information.<br>
Versioning is great, because it allows introducing breaking changes to software without harming existing users.

And that's pretty much the main "point" of versioning, is dealing with breaking changes. 
If users don't want to deal with breaking changes, they can just stay on an older version.

Pros of versioning:
- Allows developers to break compatibility in favour of better features or removal of tech debt
- Users can use older versions if they want, granting a lot more freedom

Cons of versioning: (no major cons, really)
- Userbase can become fragmented

---------------

Now, before we start, I'd just like to note,
I'm **100% FOR** the use of versioning, and I think you'd have to be an idiot not to see the value in it for 99% of situations.

But UMG is a bit special in what it's trying to achieve.<br>
There's a very concerning situation where versioning could yield to a bit of a mess, I'll explain it below.

--------------------

Let's do a thought experiment, lets assume that mods ARE versioned in UMG.

As discussed in [my other article](umg_tech_details.md), a central goal of the UMG ecosystem is to ensure hyper-compatibility between mods.

I want to be able to load the `ridable` mod, and have it work fully with the `projectiles` mod. That way, I can ride my horse off into the sunset, weilding my minigun, and rocking a cowboy hat.

Ideally, the `projectiles` mod should not need to care about the `ridable` mod. In fact, both mods should not know about each other, since they are unrelated.<br>
However, both mods will still need to tag onto other mods.
Both the `projectiles` mod, and the `ridable` mod will need the `dimensions` mod to be loaded, so that they can fudge around with what dimensions steeds are in, and what dimensions projectiles are spawned in.

```mermaid
stateDiagram-v2
    riding --> dimensions_v0
    projectiles --> dimensions_v0
```

But lets assume that there was a breaking change in the `dimensions` mod, from version 0.0 to version 1.0, which overhauled the way entities are stored inside of dimensions, and changed a few things of the API.

Lets say that the `riding` mod updated to the latest version, but the author of the `projectiles` mod disagreed with the changes, so they stayed on the older version.<br>
Suddenly, we would have a setup like so:

```mermaid
stateDiagram-v2
    riding --> dimensions_v1
    projectiles --> dimensions_v0
```

Now, what fricken SUCKS, is that `riding` and `projectiles` are no longer compatible.<br>
Why? Because they use two different versions for the `dimensions` mod.
When entites are emplaced into a world, their `.dimension` component is going to be handled (and mangled) by two competing systems; one in `dimensions_v1`, and one in `dimensions_v0`.<br>
This is terrible.

Now, one could argue that this "issue" is the fault of whoever wrote the dimensions mod.<br>
And I would 100% agree. But that's kinda ignoring the real issue here. The real issue, is that this setup, (where `riding` and `projectiles` use different versions) is ALLOWED to exist.

The `riding` mod works fine on it's own.<br>
The `projectiles` mod works fine on it's own too!<br>

But what's bad, is that this setup will spread like a cancer. Any mods that build on top of `riding` will no longer be able to use the `projectiles` mod. Same vice versa. We have created a situation where our beautiful degree of hyper-compatibility is killed.

It would have been much better if whoever wrote `projectiles` was instead forced to use `dimensions_v1`; that way, compatibility between mods is guaranteed.


To hammer it home, this is a crude example of what the current `test` mod's dependency tree looks like:
(This isn't even the whole tree, I stopped coz it got too big)

```mermaid

stateDiagram-v2
    xy --> reducers
    riding --> reducers
    projectiles --> reducers
    inventory --> reducers
    sync --> reducers
    state --> reducers
    input --> reducers

    objects --> typecheck
    riding --> typecheck
    projectiles --> typecheck
    inventory --> typecheck
    sync --> typecheck
    state --> typecheck
    input --> typecheck

    xy --> objects
    riding --> objects
    projectiles --> objects
    inventory --> objects
    sync --> objects
    state --> objects
    input --> objects

    test --> sound

    inventory --> input
    projectiles --> input
    riding --> input

    inventory --> xy
    riding --> xy
    projectiles --> xy

```



```mermaid

stateDiagram-v2
    borders --> dimensions
    borders --> rendering
    borders --> scheduling
    borders --> objects
    borders --> typecheck
    base --> typecheck
    base --> sync
    sync --> typecheck
    sync --> reducers
    base --> objects
    objects --> typecheck
    base --> state
    state --> objects
    state --> typecheck
    state --> reducers
    base --> input
    input --> objects
    base --> rendering
    rendering --> objects
    rendering --> typecheck
    rendering --> reducers
    rendering --> state
    rendering --> dimensions
    dimensions --> sync
    dimensions --> objects
    rendering --> scheduling
    scheduling --> typecheck
    scheduling --> objects
    base --> reducers
    base --> physics
    physics --> state
    physics --> dimensions
    physics --> xy
    xy --> sync
    xy --> typecheck
    xy --> state
    xy --> reducers
    base --> control
    control --> input
    control --> rendering
    control --> xy
    control --> sync
    base --> xy
    base --> follow
    follow --> control
    follow --> rendering
    follow --> typecheck
    base --> juice
    juice --> state
    juice --> rendering
    juice --> reducers
    juice --> typecheck
    base --> sound
    sound --> objects
    sound --> typecheck
    base --> mortality
    mortality --> sync
    mortality --> typecheck
    mortality --> rendering
    base --> scheduling
    base --> dimensions
    base --> initializers
    initializers --> dimensions
    initializers --> xy
    base --> usables
    usables --> items
    items --> objects
    items --> sync
    items --> rendering
    items --> ui
    ui --> rendering
    ui --> input
    items --> input
    usables --> objects
    usables --> sync
    usables --> input
    borders --> dimensions
    borders --> rendering
    borders --> scheduling
    borders --> objects
    borders --> typecheck
    categories --> base
    categories --> chunks
    chunks --> base
    chat --> base
    commands --> chat
    commands --> base
    commands --> ui
    crafting --> base
    crafting --> items
    grids --> base
    grids --> categories
    grids --> typecheck
    light --> rendering
    light --> reducers
    modern --> base
    modern --> categories
    modern --> worldeditor
    worldeditor --> base
    worldeditor --> chat
    worldeditor --> ui
    worldeditor --> chunks
    modern --> chunks
    modern --> chat
    modern --> ui
    modern --> grids
    move_behaviour --> base
    placement --> base
    projectiles --> typecheck
    projectiles --> sync
    projectiles --> objects
    projectiles --> state
    projectiles --> input
    projectiles --> rendering
    projectiles --> reducers
    projectiles --> physics
    projectiles --> items
    projectiles --> control
    projectiles --> xy
    projectiles --> juice
    projectiles --> sound
    projectiles --> mortality
    projectiles --> usables
    terrain --> base
    test --> base
    test --> juice
    test --> chat
    test --> light
    test --> items
    test --> follow
    test --> placement
    test --> crafting
    test --> move_behaviour
    test --> terrain
    test --> worldeditor
    test --> commands
    test --> modern
    test --> weather
    weather --> base
    weather --> light
    weather --> chat
    test --> vignette
    vignette --> base
    test --> scheduling
    test --> projectiles
    test --> rendering
    test --> xy
    test --> borders

```
