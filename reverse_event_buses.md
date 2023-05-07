
# Reverse event buses in Untitled Mod Game

*Note: This reading requires an understanding of my project vision for Untitled Mod Game. See my other post!*

Event buses are an *awesome* tool in the context of Untitled Mod Game.

Whenever a significant action happens in a base mod, that base mod can simply
emit an event to tell other mods that something interesting happened.

What's beautiful, is that the base mod emitting the event doesn't care who's listening.
It just throws the event into the void; the systems that care about it will
tag into it. Perhaps no one is listening! Perhaps 10 other mods are listening.
The base mod doesn't care.

A good example of this is the `entityDeath` event.<br>
This event is called automatically when an entity dies, and the entity is
passed in as the first argument, like so:
```lua
umg.call("entityDeath", ent)
```
If other systems want to listen to this event, they can use `umg.on`:
```lua
-- Meanwhile, in a completely different mod:
umg.on("entityDeath", function(ent)
    -- plays a death sound when an entity dies.
    if ent.deathSound then
        playSound(ent.deathSound)
    end
end)
```

Now, this stuff is pretty basic, and this is nothing new. Event buses are a common
pattern, especially in game development.

I could give examples of the entity emitting blood particles if it has the `meat` component,
or spawning a full-health copy of itself if it has the `secondLife` component, but that's not the point
of this post.


# The problem space:

Okay, so event buses are great for dispatching information when we are within an unknown context.

But sometimes we don't want to dispatch information.<br>
Instead, we may want to *receive* information.

Behold, the holy *reverse event bus*!!!

(I think it's best explained if I give a problem statement, and an example.)

Lets imagine that we have a system for attacking entities.<br>
This system needs to know if an entity can attack or not.<br>
But it doesn't know what other mods are loaded! It also doesn't know the context of the game
outside of it's pure little abstract layer.

So, we turn to *reverse event buses.*

With reverse event buses, we have two functions: 
```lua
-- asks a question
umg.ask(question, reducer, ...)

-- answers a question
umg.answer(question, answerFunc)
```


`ask` is similar to `call`, in that it initiates the "interaction".
```lua
-- attack system
local reducer = operators.OR

local canAttack = umg.ask("isAttackBlocked", reducer, entity, targetEntity)

if canAttack then
    attack(entity, targetEntity)
end
```
The `operators.OR` is the reducer function; it takes all the results from the , and reduces
them to one value by repeatedly applying the function.
(you could also use logical AND, or even a sum function.)


`answer` is similar to `on`, in that it responds to a question.<br>
Instead of executing something, however, the `answer` should ideally be a pure
function that just returns a result. For example:
```lua
-- team handler system

umg.answer("isAttackBlocked", function(entity, targetEntity)
    if entity.team == targetEntity.team then
        -- the entities are in the same team, so the attack should be blocked.
        return true
    end
    return false
end)
```

Now, just quickly, I want to recap on why this pattern is actually useful.
*Why wouldn't you just check if the entities are on the same team in the `attack` code, instead of asking?*

With this setup, the `attack` system doesn't *know* about the concept of teams.
In fact, the concept of "teams" may not even exist, depending on what mods are loaded!

The attack system is asking a question in an abstract manner, and getting different results
depending on the what mods are loaded.

This is the exact same thing as event buses, except we are receiving information instead
of dispatching information.

# But is this a useful concept?
Surely this idea is too niche to be used everywhere, right?  Is this a useful concept?

YES, Absolutely. This pattern SHOULD NOT be used everywhere. Alike event buses, this
pattern is very easy to abuse, and should only be used when appropriate.

Still though, given my experience developing base mods for the platform so far,
I would argue that this pattern is still *very* useful in certain circumstances.

Here are some other useful questions that could be asked:
```lua
umg.ask("canOpenInventory", OR, inventory, ent) -- whether an inventory can be opened by `ent`
umg.ask("isHidden", OR, ent) -- whether `ent` is hidden
umg.ask("canUseItem", OR ent, itemEnt) -- whether `ent` can use `itemEnt`
umg.ask("canRide", OR, ent, steedEnt) -- whether `ent` can ride `steedEnt`
```


# To conclude:
Reverse event buses are a cool concept! :)

Although they can probably be abused very easily, (same as event buses,)
they still provide a very useful pattern for my project.

Thanks for reading this opinion piece! I hope I opened your mind a bit.

- Oli
