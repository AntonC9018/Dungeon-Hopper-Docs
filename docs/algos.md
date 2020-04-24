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

4. If another real is blocking their way of e.g. moving / attacking, they will pass them the turn. They will call `executeAction()` on that real. If it has already executed their action, though, the current `mov` will fail.

5. If all the `movs` have failed, the `failAction` chain is traversed on the actor entity.


## The Player Algo

1. There is an assumption about the action that this algorithm makes. It's that the `direction` field (if needed) is already set on the action before `executeAction()` is called. This is in contrast with the `GeneralAlgo`, where the direction for the action is chosen inside `executeAction()`. In case of player, the next action and the direction are set before the new iteration of the game loop, since the players don't have their `computeAction()` being called.

2. The `World` calls `executeAction` on player, causing it to loop through the chain of the `nextAction`, trying the action components one after the other.

3. If the action fails, the `failAction` chain is traversed (TODO)