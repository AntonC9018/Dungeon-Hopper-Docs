---
id: pools
title: Pools
---
In this article, the mechanics of pools and the way you register pools are examined.

## What is a pool?

A *pool* is a collection of some items (*records*), either limited or unlimited (*infinite*). These records can be *retrieved* or *drawn* from the pool. Once a pool has no records it in, it is said to have been *exhausted*.

A pool is essentially a *simple connected graph*, meaning it has a *root node* (also called the *root pool*) which splits in a number of subnodes (*subpools*), which split again until some final depth is reached. There are no loops within the graph.

In the game there are two most important places where pools are employed:
1. Getting `items` from specific pools, e.g. by rarity;
2. Getting `entities` from specific pools, e.g. by zone, floor and type (enemy, tile etc.)


## Examples and the Notation

The two most notable examples are the *item pool* and the *entity pool*.

### The item pool

The item pool has a root pool, which includes all items there are in the game. In code you would address this root pool using `i` as the pool id (discussed later). Its direct subpools are the rarities: common `i.common` or `i.1`, rare `i.rare` or `i.2` etc. Each of these have direct subpools that distribute the items among categories:
`i.common.weapon` would mean 'weapons of common rarity', `i.*.weapon` would mean 'weapons of any rarity' and `i.rare.*` would mean 'any rare items'. 

The item pool is a finite pool, meaning it can be exhausted.

### The entity pool

The entity pool also has a root pool. Its split over zones: `e.z1`, `e.z2` etc. These are in turn split among floors: `e.*.f1`, `e.z1.f2` etc. These are split among types: `e.z1.f1.entity`, `e.z2.f2.wall` etc.

The entity pool is an infinite pool, meaning it will never be exhausted.

### The tilda notation

The `~` sign (tilda) means take the currently active setting. For example, if the current rarity level is rare, the notation `i.~.weapon` would mean a rare weapon; if the current zone is 1 and floor is 2, `e.~.~.enemy` will be equivalent to `e.z1.f2.enemy`.

The notation will come in handy in a moment.

## How to draw from the item/entity pool?

The `World` class defines the method `World:drawFromPool(poolId)` for drawing items at random from pools. The pool id is the string representing the particular subpool.

```lua
-- getting a weapon of the current quantity at random
local itemId = world:drawFromPool('i.~.weapon')
-- convert to an actual item
local item = Items[itemId]
-- create the item in the world at position x = 5, y = 5
local droppedItem = world:createDroppedItem(itemId, Vec(5, 5))

-- getting a random enemy of zone 1, floor 1
local entityId = world:drawFromPool('e.z1.f1.enemy')
-- convert id to entity class
local entityClass = Entities[entityId]
-- spawn the entity in the world at position x = 4, y = 4
local entity = world:create(entityClass, Vec(4, 4))
```

You cannot use the star notation to draw from pools.
```lua
world:drawFromPool('e.*.*.*')   -- error
world:drawFromPool('e.*')       -- error
world:drawFromPool('e.*.item')  -- error
world:drawFromPool('e.z2.*')    -- error
world:drawFromPool('e.z2.f1.*') -- error
world:drawFromPool('*')         -- error
world:drawFromPool('e')         -- ok
```

## Adding items to subpools

Use the `Pools` API to add items to particular subpools, or to define new subpools. Custom root pool definition is examined further in the article.

First thing you would do is import `Pools`.

```lua
local Pools = require 'game.pools'
```

To add an item to a subpool, use `addToSubpool()`:
```lua 
-- this adds the item ONLY to the common weapon subpool
Pools.addToSubpool('i.common.weapon', myItem:getItemId())
-- it does not modify the common subpool
-- so drawing from 'i.common' will never get you your item
-- unless you also add it to 'i.common'. For that, also do:
Pools.addToSubpool('i.common', myItem:getItemId())

-- add an entity to all zones
-- the `.global_id` thing will probably be changed 
Pools.addToSubpool('e.*', myEntity.global_id)
-- add to the subpool of enemies of all zones, second floor
Pools.addToSubpool('e.*.f2.enemy', myEntity.global_id)

-- adding to the root pool amount to an error
Pools.addToSubpool('e', myEntity.global_id) -- error
-- do not use the tilda notation here since it is meaningless
Pools.addToSubpool('e.~', myEntity.global_id) -- doesn't make sense

-- you may also provide the mass as parameters
-- in this case, it will appear 5 times as much
Pools.addToSubpool('e.z1', myEntity.global_id, 5) 
```

To add a subpool, use `registerSubpool()`:
```lua
-- register zone 3
Pools.registerSubpool('e.z3')
-- register the wall type
-- please, use starts in this case so that the structure of
-- subpools is the same. The API expects this
Pools.registerSubpool('e.*.*.wall')
-- this would produce unexpected behavior later down the line
Pools.registerSubpool('e.z3.f1')
-- you may also provide the initial config, which is the initial records
Pools.registerSubpool('e.*.*.wall', { 
    { id = Mods.Test.Entities.Dirt.global_id, mass = 5 } 
})
```

## Pools in depth

As has been mentioned, a pool is essentially a *simple connected graph*, meaning it has a *root node* (also called the *root pool*) which splits in a number of subnodes (*subpools*), which split again until some final depth is reached. There are no loops within the graph.

The root pool contains the list of all records shared between all its subpools, where the position of the records in the list match their id. A direct subpool has the same or a lower amount of the records than the root pool. Subpools of same depth may share some records between each other.

Each record has an *id* and a *mass*. In case the pool is not infinite, it also includes a *quantity* (`q`).

The mass indicates the probability of getting this record from the pool. For example, consider a pool with the following items:
* id = 1, mass = 1 (, q = 1)
* id = 2, mass = 2 (, q = 1)

In this case, the probability of getting the record by id 2 is twice as large as getting the first one. 

* id = 1, mass = 1, q = 1
* id = 2, mass = 2, q = 2

The probability of getting the second record in this case is 4 times as large, as the quantity gets multiplied by the mass to produce the total mass. 

Once a record is drawn from the pool, its quantity in the pool decreases, so `q` would go from 2 to 1.

The subpools may each define their own mass for a particular record, which will be used when drawing from this particular pool. So the mass is said to be *private* to each of the subpools, while the id and the mass are said to be *shared* between subpools. 

Once a subpool is exhausted, the quantities of items it contained are reset back to their original quantities.

### The `Pool` class

The `Pool` class two useful high level methods. 
* `Pool:getRandom()` selects a random record, decreases its quantity by 1 and returns the id.
* `Pool:exhaust()` checks if the pool has been exhausted, then refreshes it if it has.

Each pool has a parent, indicating the node higher in the pool hierarchy, and a list of subpools, which are too instances of `Pool`.

#### Instantiation

```lua
-- import the Pool class
local Pool = require '@items.pool.pool'
local Record = require '@items.pool.record'

-- define the initial config
local config = {
    ids = { 
        -- record takes the id, mass and quantity 
        Record(1, 1, 1),
        Record(2, 2, 1)
    },
    subpools = {
        {
            -- share the first record
            ids = { 1 }
        },
        {
            -- specify a provate mass
            ids = { { id = 2, mass = 2 } }
        }
    }
}
local pool = Pool(config.ids, config)
```

You may also use the `createPool()` function, which takes a list of unmapped to `Record` set of initial records (like `{ 1, 1, 1 }` instead of `Record(1, 1, 1)`) and a list of subpools of the root pool.


### The `InfPool` class

This one also has `InfPool:getRandom()` method which does the same, except it doesn't exhaust the config.

There is no `InfPool:exhaust()` method.

The initialization is the same, except you omit the quantity. 

The imports will be:
```lua
local InfPool = require '@items.pool.infinite.pool'
local InfRecord = require '@items.pool.infinite.record'
```


### Adding new root pools with the `Pools` API

This API is not yet complete.