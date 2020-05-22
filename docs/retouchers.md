---
id: retouchers
title: Retouchers
---

`Retouchers` are functions that add handlers to the chain template of an entity class. 
This can be done manually too, but the goal of retouchers is to provide an easier interface and attempt to hide internal details as much as possible, as well as eliminate duplicate code.

Not to be confused with `Tinkers`, which add handlers to the chains of an instance of an entity.

## Example usage

Here's an excerpt from the player script
```lua
-- ... --
local Skip = require 'logic.retouchers.skip'
Skip.emptyAttack(Player) -- apply a retoucher, or simply retouch
Skip.emptyDig(Player)

local Reorient = require 'logic.retouchers.reorient'
Reorient.onActionSuccess(Player)
-- ... --
```

`Skip.emptyAttack` marks the attack component of the action chain unsuccessful if there are no entities targeted by the weapon. In contrast, the default behavior is to attack empty cells. Similarly, `Skip.emptyDig` does the same for the dig action component.

`Reorient.onActionSuccess` reorients the player to the direction of the action once the action succeeds.

## List of retouchers

### Algos
Apply `GeneralAlgo` or `SimpleAlgo`. See [algos](algos.md).

| Function | Description | Chains Modified |
| -------- | ----------- | --------------- |
| general  | applies the general algo | action |
| simple   | applies the simple algo  | action |

### Attackableness

| Function | Additional parameters | Description | Chains Modified |
| -------- | --------------------- | ----------- | --------------- |
| constant | attackableness        | Return `attackableness` when `Attackable.getAttackableness` is called | attackableness |

### Bump
Fires `Changes.Bump` event for the view-model after a displace call results in no avail.

| Chains Modified |
| --------------- |
| displace |

Apply it like so:
```lua
Retouchers.Bump(EntityClass)
```

### Equip
Equips items. See [items](items.md). 

| Function | Description | Chains Modified |
| -------- | ----------- | --------------- |
| onDisplace  | equips the items and drops the overlow when the entity displaces onto a dropped item | displace |

### Invincibility
Makes the `invincibility` status effect meaningful.

| Function | Additional parameters | Description | Chains Modified |
| -------- | --------------------- | ----------- | --------------- |
| preventsDamage | - | Prevents taking damage by being attacked if the current invincibility level is greater than 0 | defence |
| setAfterHit | amount | Sets the invincibility stat to `amount` after getting hit | beHit |
| decreases | - | Decrease the invincibility stat each tick | tick |

### NoAction
Nullifies the next action after some event. Works by setting `entity.didAction` to `true`, so would not last between beats.

| Function | Description                      | Chains Modified |
| -------- | -------------------------------- | --------------- |
| ifHit    | Skip the next time the entity would act if it were hit | beHit   |
| ifPushed | Skip the next time the entity would act if it were pushed | bePushed |
| byAttacking | Make the attacked entities skip the next time they would act if the retouched entity were to attack them | attack |  

### Reorient
All the handlers reorient the entity to point in `action.direction` (marked as a star) after some event takes place.

| Function | Description  | Chains Modified |
| -------- | ------------ | --------------- |
| onMove   | * on move                        | move |
| onDisplace | * on displace                  | displace |
| onActionSuccess | * on an action succeeding | succeedAction |
| onAttack | * on an attack                   | attack |
| onDig    | * on a dig                       | dig |

### Skip
All added handlers modify `event.propagate` based on some value.

| Function | Description | Chains Modified |
| -------- | ----------- | --------------- |
| emptyAttack | stop the attack if list of targets is empty | getAttack |
| emptyDig | stop the dig if list of targets is empty | getDig |
| blockedMove | stop the move if the targeted direction is blocked | getMove | 
| noPlayer | stop if the list of targets doesn't have a player | getAttack |

## Mods

Mods may add retouchers too. Read more on mods [here](mods.md).