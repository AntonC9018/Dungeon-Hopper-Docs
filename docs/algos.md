---
id: algos
title: Algorithms
---

## Overview

Here described the algorithms of **non-player action execution process** and **player action execution process**.


## The General Algo

Here are the steps involved in that:

1. The `World` loops through all non-player reals and makes them `executeAction()`. 

2. Each real generates a list of possible `movs`, which are just directions they would like to do stuff into.

3. An `action` with that mov's `direction` set on it, walks this action's `chain`, which has the corresponding handlers.
For example, an `AttackMoveAction` is going to try attacking (after checking if it should via the `shouldAttack` chain) and then going to try moving (same, after checking).

4. If another real is blocking their way of moving / attacking, they will pass them the turn. They will call `executeAction()` on that real. If it has already executed their action, though, the current mov will fail.

5. If all the `movs` have failed, the `failAction` chain is traversed on the actor entity.


## The Player Algo

1. The next action is set before the new iteration of the game loop.

2. The `World` calls `executeAction` on player, causing it to loop through the chain of the `nextAction`, trying the action components one after the other.

3. If the attack fails, the `failAction` chain is traversed (TODO)