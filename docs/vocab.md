---
id: vocab
title: Calling Conventions
---

## GameObjects vs Entities 

1. *Every single game object that exists in the game world and may be found in a cell is a `game object`.*
    
These include: floor tiles, walls, player, enemies, gold, traps, dropped items, torches, items for sale, projectiles, and explosions. 

Everything that is not a game object and is found in world but not in a particular cell, is a `particle`.

2. *Every game object that is assumed to be able to do actions is an `entity`.* Also, all entities are in some way decorated with `Decorators`.
Entities may be addressed in a certain way depending on the properties they have:
    * *`Attackable`*, that is entities that can take damage from normal attacks and, for that matter, can be attacked e.g. player, enemies, environmental objects like barrels
    * *`Explodable`*, e.g. walls, traps and all *Attackable* entities, dropped items (?), gold (?)
    * *`Attacking`*, e.g. player, enemies, some traps, some walls, some environmental objects
    * *`Moving`*, that is entities that may change their location in the grid, e.g. player, enemies, environmental objects
    * *`Sized`*, that is entities that may occupy multiple cells at once
    * *`Statused`*, that is entities that are vulnerable to status effects, e.g. being frozen, on fire etc. and being pushed, e.g. player and enemies
    * *`Pickuppable`*, whether the thing can be picked up by player. *Picking-up* means destroying and converting to some other information, e.g. dropped item -> picked-up item
    * *`Real`* if the entity is a player, an enemy or an environmental object, such as a barrel etc. and *`Non-Real`* otherwise.

And others. Entities are given these properties by means of *decoration*. See ![Decorators](decorators.md) for a complete list of predefined decorators.


## Stats

`Stats` is a wrapped string-integer dictionary. The `Stats` wrapper class has the following methods: set, get, setIfHigher, mingle, clone, decrement.


## Modifiers

A `Modifier` is an object, the exact type of which is specified by objects that would use it. 
The most relevant place where `Modifiers` are used is the table `Entity.baseModifiers`, that has fields for attack, armor, push, status and possibly resistances (not yet implemented fully). Secondly, `Modifiers` are used and, in fact, defined by the `DynamicStats` decorator. It lets one get or modify a specific stat dynamically. More on this decorator in the ![Decorators section](decorators.md).


## Effects

`Effects` are objects that describe properties of an effect such as an attack, push, move etc.

They are propagated downwards with the event and are used to e.g. keep track of the amount of damage that would be taken from an attack.

These effects are generated and returned by the `DynamicStats` decorator, or can be created separately.