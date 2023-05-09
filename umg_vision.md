

# Technical implementation details and vision for UMG

"Untitled Mod Game" (or "UMG" in short) is a multiplayer
game that is based on mods.

I've been developing it for many months at this point, and it's been really
fun!

It's setup is similar to that of Garrys Mod or Roblox;
where most playable content is User-generated.

------------------------------

However, UMG seeks to take things a bit further, and address
a few issues that exist with the traditional traditional modding approach.

With UMG, we have two "types" of mods: "Base" and "Playable" mods.
- Base mods
    - Base mods provides standardized protocols and tools for modders to use.
    - They are not able to be "played", they are just APIs for other modders to tag into.
- Playable mods
    - Playable mods provide gameplay and game content.

To explain the point of this, we need to understand what I call
"The Riding problem".

# The riding problem:

Lets imagine that we have 2 modders, "John" and "Mary".

Mary likes elephants, so she is making an elephant riding mod.
John likes horses, so he is making a mod where you can ride horses.

Both Mary and John go about their business, and create their mods.
Since there is no way for John and Mary to communicate, they both code
the riding behaviour independently.

This is *terrible*.<br>
Why?  Well, the code for riding animals has been written twice independently.
Which is a big waste of time!

It would be much better if John and Mary's mods both "extended" a common mod,
e.g, the "Ridable Animals Mod".
This way, code is only written once in a generic fashion, and time is saved.

But actually, there's a bigger problem than "duplicate code": *Compatibility.*<br>
Imagine if someone loads the ridable elephants mod, and the ridable horses mod at the same time.<br>
Now imagine the player jumps on a horse, and then goes over to an elephant,
and tries to ride the elephant *whilst riding* the horse.

At best, nothing happens.<br>
At worst, the game crashes, or they get glitched across the world in an unpredictable fashion.

Without John and Mary following a standard protocol, there is no way for them
to know if they are breaking each others work.

-----------------------------

Ideally, in UMG, the "ridable" behaviour would be extrapolated to a "Base" mod.
The "Playable" mods, (ridable horses and ridable elephants) could then extend the "ridable" mod.

# Technical implementation:
So this setup is cool and all, but how would this work in a technical sense?<br>
How do mods know about each other in this way?<br>
Also, what stops other base mods from being incompatible with each other, causing the same class of problems?

To understand this, lets do a quick overview of UMG architecture:

## The UMG core Entity Component System:
*If you have never heard of ECSes in a gamedev context, I recommend looking it up real quick.*

In UMG, everything in the world is an entity.
Players, bullets, enemies, trees, grass, are all entities.<br>
Entities exist on both the server and the client; however only the server
has the authority to create and delete them.

A `Group` is like an array that holds entities.
Entities are automatically added to groups if they have the required components for that group.

```lua
-- Here's a group with components (x, y, image).
-- All entities with these components are added to myGroup automatically.
local myGroup = umg.group("x", "y", "image")
```

We can then imagine our "Systems" iterating over `group`s of entities,
executing code and changing the state of entities as they go.

So, back to the example from before. With the ridable horses and elephants.
With our setup, we could have both horses and elephants contain a `ridable`
component, and have a system act on all entities with `x, y, ridable` components.

```lua
local ridableGroup = umg.group("ridable", "x", "y")

local function update()
    for ent in ridableGroup do
        local riderEnt = ent.ridable.rider
        -- set the rider's position to the steed ent.
        riderEnt.x = ent.x
        riderEnt.y = ent.y
        riderEnt.z = ent.z + ent.ridable.rideHeight
    end
end

... -- more code here, etc
```
We would also have more code that sets up the rider, and mounts / unmounts
the riding entity, etc.

The main point is that now both the horse and the elephant can follow a
standard protocol for riding:
```lua
-- Horse entity
return {
  image = "horse",
  ridable = {
    rideHeight = 10
  },
  speed = 45
}
```

```lua
-- Elephant entity
return {
  image = "elephant",
  ridable = {
    rideHeight = 28
  },
  speed = 10
}
```

Awesome!  Now, John and Mary's mods work together just fine. 

-------------

Unfortunetely, Mary and John still have some problems that need to be addressed.

In John's horse riding mod, he wants to limit horse riding to the "knight" class.<br>
In Mary's elephant riding mod, she wants the elephants to flap their ears when the player mounts.

But... how can this be solved?<br>
Remember, the `riding` mod is a *base mod.*
Which means it knows NOTHING about the current game context; all it cares about is the `ridable` component.

So, the `riding` mod knows NOTHING about elephant ears.<br>
It also knows NOTHING about "knights" either. "Knights" may not even exist, depending on what mods are loaded!

To give Mary and John the tools to solve this problem,
we can use *event-bus abstraction.*
(This was addressed in the `reverse_event_buses` article, you should check that out)

Specifically, John and Mary need two things:
- Mary needs an event to be emitted whenever a ridable entity is mounted
- 
```lua


```


**TODO: Explain the following:**
(Even give code examples if possible.)

- `ridable` component, for the steed
- `rider` component, for the steed, pointing to the rider
- events emitted so other mods can tag onto the behaviour:
  - `mount(riderEnt, steedEnt)`   `unmount(riderEnt, steedEnt)`
- asking protocol, so other mods can determine what can/can't be ridden.
  - `canRide(riderEnt, steedEnt)`
  - (For example, only the blue team can ride blue horses)

