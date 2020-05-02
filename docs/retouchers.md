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

### Attackableness

| Function | Additional parameters | Description | Chains Modified |
| -------- | --------------------- | ----------- | --------------- |
| constant | attackableness        | Return `attackableness` when `Attackable.getAttackableness` is called | attackableness |

### Reorient
All the handlers reorient the entity to point in `action.direction` (marked as a star) after some event takes place.

| Function | Additional parameters | Description  | Chains Modified |
| -------- | --------------------- | ------------ | --------------- |
| onMove   | - | * on move                        | move |
| onDisplace | - | * on displace                  | displace |
| onActionSuccess | - | * on an action succeeding | succeedAction |
| onAttack | - | * on an attack                   | attack |
| onDig    | - | * on a dig                       | dig |

### Skip
All added handlers modify `event.propagate` based on some value.

| Function | Additional parameters | Description | Chains Modified |
| -------- | --------------------- | ----------- | --------------- |
| emptyAttack | - | stop the attack if list of targets is empty | getAttack |
| emptyDig | - | stop the dig if list of targets is empty | getDig |
| blockedMove | - | stop the move if the targeted direction is blocked | getMove | 
| noPlayer | - | stop if the list of targets doesn't have a player | getAttack |
