---
id: dynamicstats
title: Dynamic Stats
---

This is a decorator which adds the possibility to easily initialize with fallbacks to default values, retrieve and modify any stats of your entity.


## Methods

This decorator doesn't add any chains, but has a number of useful methods.


### `getStat(statIndex)`

Used to retrieve a stat. **Shorthand function:** `Entity:getStat(statIndex)`.
This function may return an `Effect`, a `Stats` object or a `Number`, depending on the stat.
Here's a list of all predefined stats (may be extended by modules):

| Name       | Modifier looked up | Returned as     | Fields: Name(defaultValue)              |
| ---------- | ------------------ | --------------- | --------------------------------------- |
| Attack     | attack             | Effect (Attack) | damage(0), pierce(0), source(`'normal'`), power(1) |
| Push       | push               | Effect (Push)   | power(0), distance(1) |
| Dig        | dig                | Effect (Dig)    | damage(0), power(0) |
| Move       | move               | Effect (Move)   | distance(1), through(`false`) |
| Status     | status             | Stats           | Added by modules |
| AttackRes  | resistance         | Stats           | armor(0), pierce(0), maxDamage(`math.huge`), minDamage(1), normal(1) (expandable) |
| PushRes    | resistance         | Stats           | normal(1) (expandable) |
| DigRes     | resistance         | Number          | dig(1) |
| StatusRes  | resistance         | Stats           | Added by modules |
| Invincible | resistance         | Number          | invincible(0) |

#### Examples:

```lua
local StatTypes = require('logic.decorators.dynamicstats').StatTypes
 
local attack = entity:getStat(StatTypes.Attack)
-- attack.damage, attack.pierce

local push = entity:getStat(StatTypes.Push)
-- push.power, push.distance
-- also, you can use push:toMove(direction)

local move = entity:getStat(StatTypes.Move)
-- move.distance, move.through
-- you can use move:toPos(grid, target)

local attackRes = entity:getStat(StatTypes.AttackRes)
-- now you can do
attackRes:get('armor') -- 0, by default
attackRes:set('armor', 1)
attackRes:get('armor') -- 1
attackRes:add('pierce', 1) -- increment pierce


local digRes = entity:getStat(StatTypes.DigRes)
-- digRes is a number, 1 by default
```

### `setStat(statIndex, ...)`

Used to reset a particular stat to the particular amount. **Shorthand function:** `Entity:setStat(...)`.

This function is probably going to be used rarely.

```lua
local StatTypes = require('logic.decorators.dynamicstats').StatTypes

-- set attackRes.armor to 2
-- statName == 'armor', newAmount = 2
entity:setStat(StatTypes.AttackRes, 'armor', 2)

-- set digRes to 0
-- amount == 0
-- can be used only if the value is returned as a number
entity:setStat(StatTypes.DigRes, 0)

-- set attack damage to 1
entity:setStat(StatTypes.Attack, 'damage', 1)

-- decrement all statuses
-- getStatRaw() returns stats without traversing handlers
local status = entity.decorators.DynamicStats:getStatRaw(StatTypes.Status)
status:decrement()
entity:setStat(StatTypes.Status, status)

-- increment attack damage by 1
local attack = entity:getStat(StatTypes.Attack)
entity:setStat(StatTypes.Attack, 'damage', attack.damage + 1)
```

### `addStat(statIndex, ...)`

Works the same way as the `setStat` method, except it adds the specified amount/stats to the existing stats instead of resetting the selected stat values. **Shorthand activation**: `Entity:addStat(statIndex, ...)`

```lua
local StatTypes = require('logic.decorators.dynamicstats').StatTypes

entity:setStat(StatTypes.Attack, 'damage', 1)
entity:getStat(StatTypes.Attack) -- damage == 1

entity:addStat(StatTypes.Attack, 'damage', 1)
entity:getStat(StatTypes.Attack) -- damage == 2

local stats = Stats.fromTable({ damage = 5 })

-- Addition
entity:addStat(StatTypes.Attack, stats)
entity:getStat(StatTypes.Attack) -- damage == 7

-- Subtraction
entity:addStat(StatTypes.Attack, -stats)
entity:getStat(StatTypes.Attack) -- damage == 2

-- Be careful, as it can get below 0 this way
entity:addStat(StatTypes.Attack, -stats)
entity:getStat(StatTypes.Attack) -- damage == -3
```

### `addHandler()` and `removeHandler()`

For each of the stat types, a chain is created. By default, these chains are empty. These chains are traversed before returning the stat. The chains can be employed to implement a more complex logic, rather than just increasing or decreasing stats by some amount. For that, use `Entity:addStat()` instead. 

This function is less common, so it lacks a shorthand function on `Entity`.

#### Here's a contrived example.

```lua
local StatTypes = require('logic.decorators.dynamicstats').StatTypes
local dynamicStats = entity.decorators.dynamicStats

local handler = function(event)
      -- check if a weapon is not equipped. 
      -- up the attack by 5 in this case.
      if event.actor.weapon == nil then
          -- note that the attack is set on `event.stats` and not `event.attack`
          event.stats.damage = event.stats.damage + 5
      end
  end 

dynamicStats:addHandler(StatTypes.Attack, handler)

entity:setStat(StatTypes.Attack, 'damage', 0)
local damage

entity.weapon = 1
damage = entity:getStat(StatTypes.Attack).damage
-- damage is 0

entity.weapon = nil
damage = entity:getStat(StatTypes.Attack).damage
-- damage is 5

dynamicStats:removeHandler(StatTypes.Attack, handler)

damage = entity:getStat(StatTypes.Attack).damage
-- damage is 0 again
```

### `getStatRaw(statIndex)`

Get the stat without a pass through the chain of this stat. Always returns a `Stats` object.

```lua
-- Consider the handler from the previous example.

entity.weapon = nil
local damage = entity:getStat(StatTypes.Attack).damage
-- damage is 5

-- returns the attack as stored internally, i.e. as a Stats object
local attackStats = dynamicStats:getStatRaw(StatTypes.Attack)

local damage = attackStats:get('damage')
-- damage is 0
```

If the same `Stats` object is internally used to store multiple stats, in other words, if multiple stats reference the same modifier, it is returned in its entirety as a `Stats` object.

```lua
local resistances = dynamicStats:getStatRaw(StatTypes.AttackRes)

local pushRes = resistances:get('push')
local armor   = resistances:get('armor')
local digRes  = resistances:get('dig')
-- ...
```


### `registerStat(name, config, howToReturn)`

In short, dynamic stats keeps all stats in three config objects:
1. `StatConfigs`, which keeps track of what properties are returned on stat request;
2. `StatTypes`, which maps name of a stat to the index in `StatConfig` array;
3. `HowToReturn`, which can be:
    1. `EFFECT`, which would treat *the second entry in the config* as an `Effect` constructor and will return stats wrapped with that constructor; 
    2. `STATS` which would treat it as an array of individual stats; 
    3. `NUMBER`, which would treat it as numbers.

This function adds the specified stat in all entities that are decorated with the `DynamicStats` constructor.

#### Examples

##### With effect:
```lua
local SampleEffect = class('SampleEffect', Effects)
SampleEffect.modifiers = {
    { 'field1', 0 }, -- default value is 0
    { 'field2', 8 }  -- default value is 8
}
-- Sample indicates that the stats will be stored in 
-- a special for `sample` stats object 
-- and are assumed to optionally be overriden via 
-- `Entity.baseModifiers.sample` field
local effectConfig = { 'sample', SampleEffect }

-- get the decorator itself
local DynamicStats = require 'logic.decorators.dynamicstats'
local HowToReturn = require 'logic.decorators.stats.howtoreturn'
local StatTypes = DynamicStats.StatTypes
-- register the new stat
DynamicStats.registerStat(
    'Sample', 
    SampleEffect, 
    -- since we want to wrap it in a custom effect
    HowToReturn.Effect
)

-- assume FooBar is a subclass of Entity decorated with DynamicStats
local entity = FooBar()
local sample = entity:getStat(StatTypes.Sample)
-- sample.field1 == 0, sample.field2 == 8
```

##### With stats:
```lua
local SampleModifier = {
    { 'field1', 0 }, -- default value is 0
    { 'field2', 8 }  -- default value is 8
}

local effectConfig = { 'sample', SampleModifier }

-- get the decorator itself
local DynamicStats = require 'logic.decorators.dynamicstats'
local HowToReturn = require 'logic.decorators.stats.howtoreturn'
local StatTypes = DynamicStats.StatTypes
-- register the new stat
DynamicStats.registerStat(
    'Sample', 
    SampleModifier, 
    -- since we want to return it as Stats
    HowToReturn.Stats
)

-- assume FooBar is a subclass of Entity decorated with DynamicStats
local entity = FooBar()
local sample = entity:getStat(StatTypes.Sample)
-- sample:get('field1') == 0, sample:get('field2') == 8
```

##### With Number:
```lua
local SampleModifier = { 'field1', 5 }
local effectConfig = { 'sample', SampleModifier }

-- get the decorator itself
local DynamicStats = require 'logic.decorators.dynamicstats'
local HowToReturn = require 'logic.decorators.stats.howtoreturn'
local StatTypes = DynamicStats.StatTypes
-- register the new stat
DynamicStats.registerStat(
    'Sample', 
    SampleModifier, 
    HowToReturn.Number
)

-- assume FooBar is a subclass of Entity decorated with DynamicStats
local entity = FooBar()
local sample = entity:getStat(StatTypes.Sample)
-- sample == 5
```

##### With baseModifiers:
```lua
local SampleEntity = class('SampleEntity', Entity)

-- ...
-- Assume DynamicStats from the previous example with number
-- decorate with DynamicStats and other decorators
-- ...

SampleEntity.baseModifiers = {
    sample = {
        field1 = 2
    }
}

-- instantiate the new class
local entity = SampleEntity()

local sample = entity:getStat(StatTypes.Sample)
-- sample == 2
-- if we try this with any other entity that did not define
-- an override value in `baseModifiers`, the result will be 
-- the default value, 5 in this case
```
