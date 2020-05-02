---
id: chains
title: Chains and Chain Templates
---

## Chains

### What are chains?

The concept of a chain is essential in this project. They allow for flexible dynamic customizable algorithms, and easier splitting of code into components.

These chains are just a modified form of responsibility chains, called just 'chains' for convenience.

### Example

Essentially, a chain is just a series of handler functions that operate on an event. You may want to have a chain that, for example, checks for multiple things like this:

```lua

local APPLE = 1
local POTATO = 2
local PEAR = 3


local chain = Chain()

local function checkApple(event)
    if event.actor == APPLE then
        event.propagate = false
    end
end

local function checkPotato(event)
    if event.actor == POTATO then
        event.propagate = false
    end
end

-- register the handlers.
-- their number is unlimited.
-- in this case, we used just two
chain:addHandler(checkApple)
chain:addHandler(checkPotato)

-- now, test out our chain
local event = { actor = PEAR, propagate = true }

-- do a pass through all handlers in order.
-- the second argument indicates that we should stop if event.propagate is false
chain:pass(event, Chain.checkPropagate)

print(event.propagate)
-- >> true


-- test for an apple
event = { actor = APPLE, propagate = true }

chain:pass(event, Chain.checkPropagate)

print(event.propagate)
-- >> false
```

The beauty of this method, though, is that the chain may be modified on the fly!

```lua
event = { actor = APPLE, propagate = true }
chain:pass(event)
-- event.propagate = false

-- remove the handler that check apple
chain:removeHandler(checkApple)

event = { actor = APPLE, propagate = true }
chain:pass(event)
-- event.propagate = true
```

A chain's handler may not merely check something, but also perform any action and update the event according to the results of that action. Believe me, these things are immensely useful! 

### Stop condition

One more thing, the stop condition may be programmed as a function. The `Chain.checkPropagate` you've seen previously is actually just a simple function. Here's how it's defined.

```lua
Chain.checkPropagate = function(event)
    return not event.propagate 
end
```

So, once `event.propagate` turns false, it will stop the propagation of the event, that is, the handlers after the one that made the stop condition be satisfied won't be executed.

### The order of execution

There is a special version of chains called *Sorted Chains* or, simply, *schains*. These store handlers with priority numbers and sort them based on those numbers on each traversal. This version of chains is by default available globally throughout the project as `Chain`. If you wish to use the normal version of chains, require it:
```lua
local Chain = require 'lib.chains.chain'
-- if you wish to use both or emphasize the fact that 
-- you're using the sorted version, you may include it too
local SChain = require 'lib.chains.schain' 
```
Same goes for *Chain Template* and *Sorted Chain Template*.

#### Ranks
There are four predefined *ranks*: `HIGHEST`, `HIGH`, `MEDIUM`, `LOW`, `LOWEST`. To use the ranks enum, require it:
```lua
local Ranks = require 'lib.chains.ranks'
```

The handlers will be executed in order from highest to lowest.
```lua
chain:addHandler({ h1, Ranks.LOW     })
chain:addHandler({ h2, Ranks.HIGHEST })
-- although h2 was added after h1, it will be executed before h1
```

If you're adding new handlers without specifying the ranks, they will end up in the medium rank.
```lua
-- this:
chain:addHandler({ myHandler, Ranks.MEDIUM })

-- is equivalent to this:
chain:addHandler(myHandler)
```

The handlers added to the same rank are always executed in the order they were added.
```lua
chain:addHandler({ h1, Ranks.MEDIUM })
chain:addHandler({ h2, Ranks.MEDIUM })
-- h1 is always executed before h2
```

#### Ranks to Numbers conversion
Under the hood, ranks are mapped to priority *numbers*, which are plain integers. They are stored in the `Numbers.rankMap` enum, and stuff like this is also possible:
```lua
local Numbers = require 'lib.chains.numbers'
local Ranks = require 'lib.chains.ranks'

chain:addHandler({ h1, Ranks.Medium })
chain:addHandler({ h2, Numbers.rankMap[Ranks.Medium] + 1 })
-- h2 will be executed just before h1
```

Under the hood, when you add handlers to the same rank, their priority numbers are decremented with each new handler.
```lua
chain:addHandler({ h1, Ranks.LOW }) -- maps to 200000 (Numbers.rankMap[Ranks.LOW] == 200000)
chain:addHandler({ h2, Ranks.LOW }) -- maps to 199995
chain:addHandler({ h3, Ranks.LOW }) -- maps to 199990

-- the other ranks are unaffected
chain:addHandler({ h4, Ranks.MEDIUM }) -- still 300000
chain:addHandler({ h5, Ranks.HIGH   }) -- still 400000

-- other chain instances are intact too
anotherChain:addHandler({ h6, Ranks.LOW }) -- maps to 200000
```

> NOTE: The priority numbers are not restored once a handler is removed.

Techinically, you may use floats, but you are recommended to use integers.

#### Why is it useful?

For example, the water tile prevents attacks, digs and moves by placing a high-priority handler that blocks the propagation of the event on the corresponding `do` chains: `attack`, `dig` and `move`. It must place the handler onto the `do` chain, since the tried action should succeed so that we don't have a situation like: the entity tried to attack -> it did not succeed because of being submerged -> the entity is getting unstuck, that is, the submerge handlers are removed -> the entity tries the next handler in the action chain, that is, tries to move, since the attack weren't successful. The moving succeeds, which is not what should've happened. By having the handler be of high priority, we guarantee that no other action will be tried, since the attack is considered successful.

Another example would be a ring that is destroyed if the player is about to take damage and prevents the damage. Such a ring would have low or lowest priority so that it is broken after all other damage reducers have had their turn at diminishing attack damage. 

## Chain Templates

### Overview

Chain templates offer a way of planning out a standart structure of a set of chains. 

### Example

```lua
local template = ChainTemplate() 

local function handler00(event) end
local function handler01(event) end
local function handler10(event) end

-- add a chain with 2 handlers
template:addChain('chain0')
template:addHandler('chain0', handler00)
template:addHandler('chain0', handler01)

-- add a chain with 1 handler
template:addChain('chain1')
template:addHandler('chain1', handler10)

-- add a chain with 0 handlers
template:addChain('chain2')

local chains = template:init()

print(inspect(chains))
-- {
--     chain0 = Chain,
--     chain1 = Chain,
--     chain2 = Chain
-- }

```

### Checking if a chain is set

For checking the existence of a chain, use `template.isSetChain(chainName)`