---
id: decorators
title: Decorators
---


## Overview

Decorators are used extensively throughout the project. They act pretty much like individual components that are used to augment your entity **classes**. They are **not dynamic**, that is, they *cannot* be added on already instantiated objects. Nevertheless, the chains they add *are* dynamic and can be modified at any time.

### Some new terminology on methods:

1. `be<Something>` methods refer to applying that something on the entity after a series of checks that are able to interfere with that, not allowing the effects. For example, `beAttacked` does the attack after going through a series of handlers, e.g. blocking some of the damage, or stopping the event from propagating completely, e.g. by being invincible. 
2. `do<Something>` methods refer to applying the action on a previously unknown object. For example, `doAttack` method on `World` calculates which entity is going to be targeted and calls `beAttacked` on those entities.
3. `apply<Something>` methods refer to those methods which call `do<Something>` on `World`.
4. `execute<Something>` is the same as the first one, except it is that object that does the action, that is, it is not the subject of the action. Foe example, `executeAttack` would make the entity try to do an attack.


### How to use the decorators

Assume you have an entity class you'd like to decorate. For that, just do the following:
```lua
Decorators.Start(MyEntity)                -- create the chain template on that object
decorate(MyEntity, MyDecorator)           -- add stuff to that template, add stuff to your entity
decorate(MyEntity, Decorators.Attackable) -- apply a predefined decorator
```
You do not need to import anything, since the basic decorators and the decorate function are available globally.

> NOTE: You cannot decorate instances! You can apply the decorators only to classes!
> So, in order to, e.g., stop your entities from taking damage while they are in some phase of their lifecycle, use some other logic, e.g. adding handlers to chains. 
> In our example, this would mean stopping event from propagating in `myEntity.chains.defence` chain by adding a nullifier handler to it 


### What do decorators do?

In short, Decorators add particular behaviour to your classes.

They all share these 3 basic stages


#### *Decoration*

This is the moment the `Decorator.decorate` method is called. It works by taking the list of `affectedChains` from the decorator class and adding them to the `chainTemplate` of your entity class. 

For example, such chains would create a chain named `myCheck` on the entity class's `chainTemplate` and add the handlers `checkHandler1` and `checkHandler2` onto it. After that, it would create the chain `myExecute`, adding just the `executeHandler1` onto it and, finally, it would just create the new `myEmpty` chain on the `chainTemplate`, without adding any handler onto it:
```lua
MyDecorator.affectedChains = {
    { 'myCheck',   { checkHandler1, checkHandler2 } },
    { 'myExecute', { executeHandler1 }              },
    { 'myEmpty',   {}                               }
}
```  

By default, *sorted* chains and chain templates are used, which means one can do the following:
```lua
MyDecorator.affectedChains = {
    { 'myCheck',   
        { 
            { checkHandler1, Ranks.LOW }, -- using priority rank
            { checkHandler2, 600000    }, -- using priority directly 
            checkHandler3                 -- defaults to MEDIUM
        } 
    },
}
``` 

The decorator also gets pushed to the `myEntityClass.decoratorsList` for the further initialization stage.

#### *Initialization*

The initialization stage takes place when your entity class is being instantiated. At this time, all Decorators saved on `myEntityClass.decoratorsList` are getting instantiated and their `myDecorator:__construct()` methods called. This is the moment they should instantiate things on the instance, if they need to. The decorator instances are saved on `entity.decorators` (`entity` here being your instantiated `myEntityClass`), which is a table where the keys are the names of the applied decorator classes. For example, to access the `Attackable` decorator instance, one would do:
```lua
entity.decorators.Attackable
```
This is useful for checking whether a specific decorator has been applied, although you are encouraged to use `entity:isDecorated(decorator)` if you are just checking.
```lua
decorate(EntityClass, Decorators.Attackable)
local entity = EntityClass()
print(entity:isDecorated(Decorators.Attackable)) -- true
print(class.name(entity.decorators.Attackable)) -- Attackable
```


#### *Activation*

*Activation* as such implies the decorator's method `myDecorator:activate()` getting called. Typically, it would have the instance as the first parameter. 

A fair amount of predefined decorators use the **checkApplyCycle** as their activation. The idea is straightforward: they would do a pass over their `check` chain and, if it were successful, that is, if it went through all its handlers without getting interrupted, the `do` chain is going to be passed too. Otherwise, it wouldn't. See [logic.decorators.utils](https://github.com/AntonC9018/Dungeon-Hopper/blob/master/logic/decorators/utils.lua).

For example, take `Decorators.Attacking`. It adds two chains: `getAttack`, which is the `check` (or `get`) chain, and `attack`, which is the `do` chain. 


## List of basic decorators

### `Start`

Adds the *chainTemplate* and *decoratorsList* to your Entity class. 

> This is technically not a decorator, but just an ordinary function.

> You must call this first if you want your next decorators to work at all.


### `Acting`

Enables the entity to apply action that were saved as `Entity.nextAction` during the `computeAction` beat stage.

| Added chain     | Automatically added handlers | Description |
|-----------------|------------------------------| ----------- |
| `checkAction`   | -                            | TODO        |
| `action`        | -                            | contains the action algorithm(s)|
| `failAction`    | -                            | traversed if no action succeeded|
| `succeedAction` | -                            | traversed if an action succeeded|

**Shorthand activation**: `Entity:executeAction()`


### `Attackable`

This decorator enables the entity to take normal hits.

| Added chain | Automatically added handlers | Description |
|-------------|------------------------------| ----------- |
| `defence`   | Reduction by base armor      | This chain is traversed when your entity is about to take damage. These methods mild or amplify effects of the attack. |
| `beHit`     | `takeHit`, `die`             | Handlers of this chain are traversed after a hit has been assured to come through by the `defence` chain.  |
| `attackableness` | | |

`takeHit` does damage to you (without applying status effects and pushing, see `Pushable` and `Statused` for that). 
> `Entity.takeHit()` is the shorthand for `Entity.decorators.WithHP.activate()` 

`die` checks if the health is 0 and calls the `Entity.die()` if it is.
> `Entity.die()` is the shorthand for `Entity.decorators.Killable.activate()`


**Shorthand activation**: `Entity:beAttacked(action)`

Also has a function, `Attackable.getAttackableness(actor, attacker)`, which traverses the `attackableness` chain and return the Attackableness of this entity, which can be NO, YES, IF_CLOSE or SKIP (think ghosts). Default return value: `Attackableness.YES`. 

This function also has a shorthand activation, `Entity:getAttackableness(attacker)`, which returns `Attackableness.NO` if the entity has not been decorated with `Attackable`.

### `Attacking`

This decorator enables the entity to do normal hits.

| Added chain | Automatically added handlers | Description |
|-------------|------------------------------| ----------- |
| `getAttack` | `setBase`, `getTargets`      | Used for creating the Attack object and modifying it with e.g. more damage |
| `attack`    | `applyAttack`, `applyPush`, `applyStatus` | Used for doing the Attack and applying push and the related status effects |

You may pass the list of entities you want to attack as a parameter to the activation. This would skip the process of getting targets.

**Shorthand activation**: `Entity:executeAttack(action)`

### `Interacting`

Adds the ability to interact with objects decorated with `Interactable`.

| Added chain   | Automatically added handlers | Description |
|---------------|------------------------------| ----------- |
| `checkInteract` | `getTarget`, `checkIsInteractable` | |
| `interact`      | `interact` | |

### `Interactable`

| Added chain   | Automatically added handlers | Description |
|---------------|------------------------------| ----------- |
| `checkInteracted` | - | |
| `beInteracted`    | - | |

### `Inventory`

See [items](items.md).

### `Diggable`
Enables the entity to be dug. The wall would take damage on dig equal to dig damage.

| Added chain | Automatically added handlers | Description |
|-------------|------------------------------| ----------- |
| `checkDig`  | `checkPower`                 |             |
| `beDug`     | `takeDigDamage`, `die`       |             |

### `Digging`
This decorator enables the entity to dig.

| Added chain | Automatically added handlers | Description |
|-------------|------------------------------| ----------- |
| `getDig`    | `setBase`, `getTargets`      |             |
| `dig`       | `applyDig`                   |             |

> Currently, the logic by which `getTargets` works is very similar to that of normal attacking. Thus, these might be merged in some way in the future. This would mean shovels will be of a subclass of Weapon.


### `DynamicStats`
This decorator adds the possibility to easily initialize with fallbacks to default values, retrieve and modify any stats of your entity.

This decorator doesn't add any chains, but has a number of useful methods. See [Dynamic Stats](dynamicstats.md).

### `Killable`

| Added chain | Automatically added handlers | Description |
|-------------|------------------------------| ----------- |
| `checkDie`  | -                            |             |
| `die`       | `die` | sets `entity.dead` to `true` and calls `world:removeDead()`|


**Shorthand activation**: `Entity:die()`


### `Moving`
Enables entity to displace.

| Added chain   | Automatically added handlers | Description |
|---------------|------------------------------| ----------- |
| `getMove`     | `getBaseMove`                |             |
| `move`        | `displace`                   |             |

**Shorthand activation**: `Entity:executeMove(action)`


### `PlayerControl`
Converts direction to an action for the player

**Shorthand activation**: `Player:generateAction()`, just for players


### `Pushable`

Enables the entity to be pushed

| Added chain   | Automatically added handlers | Description |
|---------------|------------------------------| ----------- |
| `checkPush`   | `checkPush`                  |             |
| `executePush` | `executePush`                |             |

**Shorthand activation**: `Entity:bePushed(action)`


### `Sequential`

Enables the entity to calculate their next action. Uses a `Sequence` object to keep track of the current step.

The `Sequence` is instantiated and set on the entity as `Entity.sequence`.

**Shorthand activation**: `Entity:calculateAction()`


### `Statused`

Makes the entity vulnerable to status effects. Status effects are being frozen, stunned, on fire, poisoned and so on.

| Added chain   | Automatically added handlers | Description |
|---------------|------------------------------| ----------- |
| `status`      | `status`                     | applies statuses |

| Modified chain | Automatically added handlers | Description |
|----------------|------------------------------| ----------- |
| `tick`         | `tick`                       | decreases all statuses |
| `checkAction`  | `free`                       | calls `free()` on statuses |

You may optionally pass a configuration parameter to the activation. See [stats](stats.md). 

**Shorthand activation**: `Entity:beStatused(action)`


### `Ticking`

Allows the entity to reset some fields at the `tick` phase.

| Added chain   | Automatically added handlers | Description |
|---------------|------------------------------| ----------- |
| `tick`        | -                            |             |


This:
```lua
actor.didAction = false
actor.doingAction = false
actor.nextAction = nil
actor.enclosingEvent = nil
```
is done directly by world.


**Shorthand activation**: `Entity:tick()`


### `WithHP`

Adds an `hp` object to the player. Makes them `takeDamage` on activation.

**Shorthand activation**: `Entity:takeDamage(damage)`


## Copying decorators

### `copyChains(from, to)`

Subclasses of subclasses of `Entity` can use this function to reapply all previously applied decorators and retouchers of the superclass.

```lua
local MyEntitySubclass = require 'wherever.your.subclass.is'
local MySubclass = class('MySubclass', MyEntitySubclass)
copyChains(MyEntitySubclass, MySubclass)
-- now you can apply other decorators without affecting the superclass
decorate(MySubclass, Decorators.Whatever)
```

### `Entity.reapplyDecorators(from, to)`

This function works the same way but ignores the retouchers.
