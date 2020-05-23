---
id: items
title: Items
---
The item system in this game is made up of these three components:
1. A piece of item logic, represented by instances of the `Item` class;
2. A dropped item in the world, which is an instance of the `DroppedItem` class, which is in turn a subclass of `Entity`;
3. An inventory, where items are stored.

## `DroppedItem`

The `DroppedItem` class is a subclass of `Entity` and it represents an item or a couple of items lying in the world at some particular location. It is nothing more than that, actually.

The way you spawn new items in the world is by calling `world:createDroppedItem(id, pos)`, where `id` is the id of the item and `pos` is the location in the world, `Vec`. More on item ids in the next section.

## `Item`

Instances of the `Item` class represent the logic of an item. The exact way an item influences the entity that `equips` it is defined by its [tinker](tinkers.md).

```lua
-- import the base class
local Item = require '@items.item'

-- define a tinker
local myTinker = Tinker({
    { 'attack', function(event) print('Attack called!') end }
})

-- create the item
local myItem = Item(myTinker)
```

When your item is `equipped`, the tinker is tinked and when it is `unequipped` or `detroyed`, the tinker is untinked.

### `Item:__construct(tinker)`

Sets the provided tinker as `item.tinker`. This is the tinker tinked by default.

### `item:beEquipped(entity)`

Called when the item is equipped by an entity. `Equipped` essentially means that an entity steps onto the dropped item lying in the world. It may also be called by e.g. an item that equips another item once you pick it up.

### `item:beUnequipped(entity)`

Called when either an inventory slot overflow takes place (e.g. when you pick up the second weapon, the first gets unequipped) or when the item is unequipped deliberately through some other logic. This function would both untink the tinker and drop the unequipped item in the world.

### `item:beDestroyed(entity)`

Called when an item's tinker needs be untinked but an item in the world need not be spawned.

### `item:getItemId()`

Is equivalent to `item.id`. This function has been introduced to remedy the distiction in the way item id is stored on `Item` vs `DroppedItem`.

### `item.slot`

Which slot in the `Inventory` the item belongs to.

## What are ids exactly?

There is a global object, `Items`, which contains all the items that have been registered by mods in the game. It provides a mapping of item id to item (the instance of `Item`).

You can also get references to items by their names. For example `Mods.Test.Items.sample` would get you the `sample` item of the `Test` mod. More on mods [here](mods.md).

It is common to get random items from the current item pool. See [this](pools.md) to find out how to do it.


## When are items equipped?

By default, no entities react to stepping on items, neither do dropped items react to being trampled.

The way you do it is by using retouchers. There are some predefined ones. 

### Example

Assume you have an entity class you want to be picking up items after it gets displaced.

```lua
-- just apply the retoucher
Retouchers.Equip.onDisplace(EntityClass)
```

For more equip retouchers, see [this](retouchers.md#equip)


## How does an Entity equip items?

The Entity class provides some methods for equipping (unequipping, destroying) items. In general, these functions call the appropriate methods on the `Inventory` decorator if it exists, or just equips (unequips, destroys) the item.

### `entity:equip(item)`

Calls `entity.inventory:equip(item)` or `item:beEquipped(entity)`.

### `entity:unequip(item)`

Calls `entity.inventory:unequip(item)` or `item:beUnequipped(entity)`.

### `entity:removeItem(item)`

Calls `entity.inventory:remove(item)` or `item:beDestroyed(entity)`.

### `entity:dropExcess(item)`

Drops overflown items in the world by calling `entity.inventory:dropExcess()`. Nothing is called if no inventory exists.


## `Inventory`

`Inventory`, as has been mentioned, is a decorator which allows an entity to store the equipped items.

It has a number of `slots`, initial sizes of which are predefined. New slots may be added by mods.
The `InventorySlots` object that contains a mapping from strings to slot ids (indices) is available globally.

```lua
local item = Item(myTinker)
item.slot = InventorySlots.body
```

The inventory stores the items in `Containers`. By default, a [variable-length cyclic buffer](https://github.com/AntonC9018/Dungeon-Hopper/blob/master/lib/cyclicbuf.lua) is used. When an overflow occurs, it would drop the old items.

### Methods

The methods have been explained previously, so I'm just going to list them. What they do, essentially, is call the corresponding method on the item getting discarded.

* `inventory:equip(item)`
* `inventory:unequip(item)`
* `inventory:remove(item)`
The method of item called is `item:beDestroyed(entity)`
* `inventory:dropExcess()`

And these are some miscellanious ones:

#### `inventory:get(slot)`
Return the container of the given slot.