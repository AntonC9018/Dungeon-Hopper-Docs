---
id: algos
title: Algorithms
---

## Overview

Here described the algorithms of **non-player action execution process** and **player action execution process**.


## The General Algo

> `Acting` that use this algo are assumed to also be decorated with the `Sequential` decorator

Overview:

1. The assumption is made that before this next step the `entity.nextAction` field has been set. This generally happens right before `executeAction()`, in the function `calculateAction()`, called by the `World` on the `Entity`. 

1. The `World` loops through all non-player reals and calls their `executeAction()`. 

2. Each real generates a list of possible `movs`, which are just directions they would like to do stuff into. The function that does this is set individually for every step sequence but the `movs` themselves are acquired by calling `entity.sequence:getMovs()`.

3. An `action` with that mov's `direction` set on it, walks this action's `chain`, which has the corresponding handlers.
For example, with an `AttackMoveAction` as the action, the entity is going to try attacking and then, if it fails, moving.

4. *This is the most important difference between this algorithm and the player algorithm*: if another real is blocking their way of e.g. moving / attacking, they will pass them the turn.  They will call `executeAction()` on that real. If it has already executed their action, though, the current `mov` will fail.

5. If all the `movs` have failed, the `failAction` chain is traversed on the actor entity.


## The Player Algo

1. There is an assumption about the action that this algorithm makes. It's that the `direction` field (if needed) is already set on the action before `executeAction()` is called. This is in contrast with the `GeneralAlgo`, where the direction for the action is chosen inside `executeAction()`. In case of player, the next action and the direction are set before the new iteration of the game loop, since the players don't have their `computeAction()` being called.

2. The `World` calls `executeAction` on player, causing it to loop through the chain of the `nextAction`, trying the action components one after the other.

3. If the action fails, the `failAction` chain is traversed.


### Is the Player Algo just for the player?

Note that the Player Algo is useful not just for the player, but can be used for other things too. In fact, the general algo applies pretty much to just enemies, while the player algo is used for traps, special tiles, etc. This is because their actions are pretty homogenous, that is, they do the same thing every beat. For example, bounce traps test if there is something on top of them to do the action of bouncing on that something in the direction that trey're looking; the special tiles see if there is something on top of them and do e.g. submerging on that something. This lets us return the action (+, optionally, direction) directly in the overriden `calculateAction` method, which also simplifies the algorithm quite a bit.

Here's an excerpt from the code for traps as an example usage of the player algo not for the player. Note that the code for the decorator has not been included.
```lua
-- ...
-- add the action algorithm
Trap.chainTemplate:addHandler(
    'action', 
    -- use the player algo, as it just does the action, which is what we need
    require 'logic.algos.player'
)
-- ...
-- define our custom action that calls the new decorator's activation
local BounceAction = Action.fromHandlers(
    'BounceAction',
    handlerUtils.applyHandler('executeBounce')
)

-- define a new method that calls the new decorator
function Trap:executeBounce(action)
    return self.decorators.Bouncing:activate(self, action)
end

-- override calculateAction. Return our custom action
function Trap:calculateAction()
    local action = BounceAction()
    -- set the orientation right away since it won't change
    action.direction = self.orientation
    self.nextAction = action
end
-- ...
```