

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

But there's another bigger problem looming on the horizon: *Compatibility.*<br>
Imagine if someone loads the ridable elephants mod, and the ridable horses mod at the same time.<br>
Lets imagine the player jumps on a horse, and then goes over to an elephant,
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
Also, what stops other base mods from interacting with each other, causing the same class of problems?

**Quick note: This section assumes a very basic understanding of gamedev design patterns, namely Entity Component Systems and event buses.**

**TODO: Explain the following:**
(Even give code examples if possible.)

- `ridable` component, for the steed
- `rider` component, for the steed, pointing to the rider
- events emitted so other mods can tag onto the behaviour:
  - `mount(riderEnt, steedEnt)`   `unmount(riderEnt, steedEnt)`
- asking protocol, so other mods can determine what can/can't be ridden.
  - `canRide(riderEnt, steedEnt)`
  - (For example, only the blue team can ride blue horses)

