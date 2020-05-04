---
id: tinkers
title: Tinkers
---

`Tinkers` are objects that help to add handlers onto the chains of an instance of an entity. They also may modify stats via the interaction with the DynamicStats decorator. In addition, `Tinkers` provide a simple interface of removing these handlers off the target and restoring initial stats.

There are 2 classes: the `Tinker` class is responsible for handlers and the `StatsTinker` class is responsible for stats and stat handlers.

## Terminology

The process of adding a handler onto a chain is called **tinking** (an abbreviation of *tinkering*). Similarly, the process of removing one is called **untinking**.

The process of activating a tinker is called **tinking** too, but refers to all tinking done by a tinker. The process of deactivating it is refered to as the **untinking** of a tinker.

The same terminology by extension also applies to modifying and restoring stats via the `StatsTinker`. So, the process of adding a stat may be called **stat-tinking**.

## Example
```lua
local Move = require 'logic.tinkers.move'
local Tinker = require 'logic.tinkers.tinker'
local StatsTinker = require 'logic.tinkers.stattinker'
local DynamicStats = require 'logic.decorators.dynamicstats'
local StatTypes = DynamicStats.StatTypes
local Stats = require 'logic.stats.stats'

local tinkElements = {
    Move.afterAttackIfNotFirstPiece
    -- ...
}

local tinker = Tinker(tinkElements)


local statChanges = {
    { StatTypes.Attack, Stats({ damage = 1, pierce = 1 }) },
    { StatTypes.PushRes, -1 },
    { StatTypes.Push, 'distance', 1 },
    { StatTypes.Push, function(event) event.stats.distance = 5 end}
}

local statsTinker = StatsTinker(statChanges)


local Item = class('Item')

function Item:onPickup(entity)
    tinker:tink(entity)
    statsTinker:tink(entity)
end

function Item:onDrop(entity)
    tinker:untink(entity)
    statsTinker:untink(entity)
end

return Item
```

## SelfUntinkingTinker

This version has a reference to the tinker itself in the handler function. Note the usage of dots, as this tinker is based on upvalues.

```lua
local function generator(suTinker)
    return function(event)
        if event.something == 'something' then
            -- remove this handler from inside the handler
            suTinker.untink()
        end
    end
end

-- attach the function to the 'sampleChain' chain
local suTinker = utils.SelfUntinkingTinker(entity, 'sampleChain', generator)

suTinker.tink()   -- the function output by generator gets onto the chain
suTinker.untink() -- the function output by generator is taken off the chain
```

## RefTinker

A more useful version of the `SelfUntinkingTinker` is the `RefTinker`, which means tinker with self reference.

```lua
local RefTinker = require 'logic.tinkers.reftinker'

-- it is given a list of { chainName, handler or { handler, priority } }
local function generator(tinker)
    return 
    {
        { 
            'move', function (event)
                print("Hello")
                tinker:untink(event.actor)
            end
        },
        {
            'attack', {
                function (event)
                    print("Hello")
                    tinker:untink(event.actor)
                end,
                Ranks.HIGH
            }
        }
    }
end

local refTinker = RefTinker(generator)
refTinker:tink(entity)
```

If you want to define functions separately, use generators for those functions. I think this is the only way to make definitions less ugly and closures are the only way of getting a reference to the tinker inside the handlers (contact me if you know any other).

```lua
local RefTinker = require 'logic.tinkers.reftinker'

local function testGenerator(tinker)
    return function(event)
        print("Hello")
        tinker:untink(event.actor)
    end
end

local function testGenerator2(tinker)
    return {
        function(event)
            print("Hello")
            tinker:untink(event.actor)
        end,
        Ranks.HIGH
    }
end

local function generator(tinker)
    return 
    {
        { 'move',   testGenerator(tinker)  },
        { 'attack', testGenerator2(tinker) }
    }
end

local refTinker = RefTinker(generator)
refTinker:tink(entity)
```

Similarly, a version for stats also exists and is called `RefStatTinker`.