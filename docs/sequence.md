---
id: sequence
title: Sequence
---


## Overview

*Sequences* represent series of consequent *steps*, that is, actions, an entity does.

Such entities are called *sequential* and are decorated with `Decorators.Sequential`.

A `Sequence` is basically an array of such steps.

## Step

Step objects describe the action and the next step to do very precicely.

### Properties

| Property name (in a config) | Type     | Description                            |
| ----------------------------|--------- | -----------------------------------    |
| action                      | Action   | The action to be used for this step    |
| movs                        | Function | The [movs](movs.md) algorithm         | 
| success                     | Number   | The step number if the action succeeds |
| fail                        | Number   | The step if the action fails           |
| checkSuccess                | Chain*   | The chain passed to check if the action is considered to have succeeded. |
| enter                       | Chain*   | Executed when the step is selected as the next one |
| exit                        | Chain*   | Executed when this step stops being the current step |
| repet                       | Number   | Now many times to repeat this step until starting to check success |

Out of these fields, only the action field must be specified in all steps.

#### Chain*
This can either be a chain, a handler or a list of handlers. If you are providing a list of handlers, however, the chain used is the normal chain, not the sorted one (schain). This is mainly done for optimization purposes.

#### `checkSuccess`
This chain is traversed once the entity is `ticked`, which is the reason why the event passed down the chain has `triggerEvent` being the tick event. The event actually has the actor set correctly, so you can access the result event for checks as `event.actor.enclosingEvent.algoEvent.resultEvent`.

If your handler decides to mark the event successful, set `event.success` to `true`. Optionally, you may provide the next step index as `event.index`. If not specified, the `success` parameter from config is used instead.

#### `enter` and `exit`
These do not receive their own event, but are passed the tick event directly. It is not recommended that you modify it.

Similarly, the result event can be accessed via `event.actor.enclosingEvent.algoEvent.resultEvent`.


### Example
For example, take a simple skeleton.

It has 2 steps in its sequence:
1. Attack or Move
2. Stay idle

So the steps would look something like this:

```lua

step1 = 
{
    action = AttackMove, -- try to attack, then to move
    movs = basic,        -- require this predefined movs algorithm
    fail = 1,            -- in case all those fail, go back to step < 1 >
    success = 2          -- this option can be omitted, as steps are considered in sequence by default
}

step2 =
{
    action = None
}

```


Now take the case of, e.g., an armadillo:
1. Turn to player, stand still
2. Attack / Dig / Move
3. Don't do anything at all for 2 loops

The first step is the more complex one

``` lua

step1 = {
    action =
        -- create a new action from the list of handlers 
        Action.fromHandlers(
            -- the name of the action
            "TurnToPlayer", 
              -- use a handler, predefined or coded on your own
              turnToPlayer
        ) 
    checkSuccess = 
        -- we've gotta chuck a checkOrthogonal function here
        -- to check whether the armadillo and the player
        -- are on one line / column
        -- for this, we create a custom chain on which we hang that function
        -- create a chain that consists of one handler
        chain: checkOrthogonal,
     -- the next step index
    success = 2,
    -- in case this fails, e.g. we're frozen, remain at the 1st step
    fail = 1
}

step2 = {
    -- this one is simpler
    action = AttackDigMove,
    -- for success we again need a custom chain
    checkSuccess = checkNotMove,
    -- in case of moving, do the next step
    success = 3
    -- in case frozen, keep rolling
    fail = 2,
    -- also, we're invincible while rolling
    enter = addInfiniteArmor,
    -- and we shouldn't be while not rolling
    exit = removeInfiniteArmor
    -- and add the movs function
    movs = followOrientation
}

step3 = {
    action = None,
    repet = 2, -- repeat this step 2 times before going to the next one
    success = 1 -- this and the fail can be omitted, as the sequence loops by default
}
```

For a whole code example, see [this](https://github.com/AntonC9018/Dungeon-Hopper/blob/master/logic/sequence/EXAMPLE.lua).