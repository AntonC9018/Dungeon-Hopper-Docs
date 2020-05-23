---
id: lifecycle
title: The lifecycle of the game
---
The game goes through these 3 phases in its lifecycle:
1. Setup. Here all the classes and the global objects and functions are defined, mod content is added, item pools and stats are set up etc.
2. Initialization. Here, the world is generated, entities are instantiated, items are drawn from item pools and dropped in the world or placed in chests etc.
3. The game itself. This is where the gameplay happens. Eventually, when a level is conquered, some parts of the world may be reinitialized, like the map, the entities, the tiles etc.

The key thing to note is that the classes themself never change during the latter 2 phases. However, they are allowed to be morphed during the setup phase. This is mainly done by use of [decorators](decorators.md) and [retouchers](retouchers.md) and concerns the entity classes in particular.


## Globals

Some core functions, variables and classes were made available globally throughout the project so as to make the imports slimmer. To see all of them, scroll through [this file](https://github.com/AntonC9018/Dungeon-Hopper/blob/master/game/setup.lua). Most notable ones among them:

| Variable name         | Imported from            | Decription |
| --------------------- | ------------------------ | ---------- |
| `ins`                 | `lib.inspect`            | see [inspect](https://github.com/kikito/inspect.lua) |
| `class`               | `lib.Luaoop`             | see [Luaoop](https://github.com/ImagicTheCat/Luaoop)|
| `Vec`                 | `lib.vec`                | 2d vector. see [vec](https://github.com/AntonC9018/Dungeon-Hopper/blob/master/lib/vec.lua) |
| `Chain` + `Event`, `Ranks`, `Numbers` | `lib.chains.schain`      | see [chains](chains.md) |
| `Decorators`, `decorate` | `@decorators.decorators`, `@decorators.decorate` | see [decorators](decorators.md) |
| `Entity`              | `@base.entity` | the entity base class |
| `copyChains`          | `Entity.copyChains` | see [copyChains](decorators.md#copying-decorators) |
| `Item`                | `@items.item` | the item base class. see [items](items.md) |
| `GameObject`              | `@base.gameobject` | the game object base class |
| `Retouchers`              | `@retouchers.all` | all core retouchers. see [retouchers](retouchers.md) |


| Variable name         | Decription                         |
| --------------------- | ---------------------------------- |
| `Items`               | A mapping id -> item. see [items](items.md) |
| `Entities`            | A mapping id -> entity class.      |


There are some others, which were omitted.

### Mods

The content added by mods is also available globally by `Mods.MOD_NAME`. See [mods](mods.md).


## How is stuff rendered

This bulk section is reserved for later, but just to give you an overall idea:

* By doing things in the world, entities push the so-called `changes` in the world's changes list. These indicate what they did during the beat. They also report the `event` that took place, as well as the immediate position, state and the orientation of the entity that caused the `change`. For example, *doing attack* would be considered a valid `event`, as well as *being pushed*, *being displaced*, *being attacked*, *taking damage* etc.

* The **view-model** (currently, the **renderer**), figures out which animations to play on which object, how much time should the animation take etc. It would sync the animations to the beats in game. By the end of every beat, it would receive a list of all changes that happened during that beat and process that in some way so as to prepare all the necessary information for displaying what happened.

* The **view** (currently, the **graphics**), serves an abstraction of the API provided by the render engine. Its functions, like setting the position of a sprite, creating a new sprite, selecting a particular time into the animation etc. are called by the **view-model**. They are the ones calling the render engine API, e.g *Corona* (currently used) or *LOVE 2d*.
