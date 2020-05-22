---
id: status
title: Status effects
---

It is hard to define a status effect, so I will illustrate it with an example instead.

## Example

Take for example the `stun` effect. It has an amount, which indicates for how many ticks it will last. So applying a stun to an entity of amount 2 means it would skip 2 bits.

This is achieved via a [tinker](tinkers.md). In this case, we have a tinker that blocks all actions from the entity.
```lua
local function forbidAction(event)
    event.propagate = false
end

tinker = Tinker({
    { 'checkAction', forbidAction }
})
```

The tinker is tinked once the `stun` is applied and is untinked once the remaining amount ticks down to 0.

The essence is as simple as it is, while the actual behavior achieved using it is pretty broad.

## Statused Decorator

Statuses are obviously pretty tighly entwined with the `Statused` decorator. 

### Overview

In short, this decorator provides methods for:
1. Applying status effects after checking resistances
2. Ticking (decreasing) status effects amounts each beat
3. Wearing off of the status effect
4. registering new status effects

### The stats associated with statuses

There are 2 stats for statuses. First one is `StatTypes.Status`, which indicates kind of the power level of the stat, and the second is `StatTypes.StatusRes`, which indicates the resistances against the particular statuses. So, for example, if you tried to apply a `stun` status effect while having your `stun` at `StatTypes.Status` being lower than the `stun` at `StatTypes.StatusRes` of the entity you are applying the status effect to, no status effect would be applied. This is managed entirely by the `Statused` decorator.

### `activate(actor, action, ?options)`

**Shorthand activation**: `entity:beStatused(action, ?options)`

It is assumed that the `StatTypes.Status` stat has been set on the action as `action.status`.

This function would traverse the added `status` chain. It would get the resistances, compare them to the levels statuses being applied and apply them only if needed by calling the `status:apply(entity, amount, option)` of the status effect (more on this and other methods later)

### Ticking the amounts

The decorator inserts a function into the `tick` chain to decrease the status amounts each beat.

The status amounts are saved on the entity as `Entity.statuses` `Stats` object.

The amounts applied are right now defined by the status effect itself, but may become parameters in the future.

### Setting a particular status amount directly

You may use the aforementioned status amounts object to reset status effects. 

For example, if you do `entity.statuses:set('stun', 0)`, the next time the tick event is triggered, the stun's tinker will be removed by means of a call to the `status:wearOff(entity, amount)` function (more later).

However, if the stat were 0 to begin with and you set it to a positive amount, no status will be applied and so no tinker will be tinked, but its removal will take place once the amount ticks back to zero, which may produce some unexpected errors.

### When are status effects applied?

By default, status effects are applied to all targets upon attacking. So if you were to set your player's `status.stun` stat to e.g. 1 (so that it is greater than or equal to the default resistance level), and assuming the `stun` status effect has been registered, your player would stun enemies by attacking them, given they are decorated with the `Statused` decorator.  

It may also be applied directly by calling `entity:beStatused(action, options)` on the given entity.

## Status

Now let's turn our attention to how exactly the statuses work.

### Basic example

Assume we wanted to create out `stun` status effect. This is all the code needed to do it:

```lua
-- create the tinker
local function forbidAction(event)
    event.propagate = false
end

local tinker = Tinker({
    { 'checkAction', forbidAction }
})

-- import the Status class
local Status = require '@status.status'

-- create our stun status effect
local stun = Status(tinker)

-- default amount of turns to wear off
stun.amount = 2

-- the overlay method.
-- here, we go for RESET, since we don't want repetitive
-- application of the stun status effect to stack the amount
-- of stun on `entity.statuses`. For that use Overlay.ADD
local Overlay = require '@status.overlay'
stun.overlay = Overlay.RESET
```

There isn't a lot going on under the hood. When our `stun` status effect is applied, the tinker is tinked, and once it wears off, the tinker is untinked.

A stat tinker, store tinker or ref tinker may also be used instead of the normal tinker.

You might have noticed that we got the new status by instantiating it, using the `Status` contructor (factory). That is right, the statuses are not classes, they are just instances of the `Status` class. You'll have to keep that in mind while using more advanced techniques.

### Overridable methods

To achieve a more sophisticated behavior, make a custom subclass of `Status` and override its methods.

```lua
local Sample = class('SampleStatus', Status)

function Sample:__construct(tinker, someParam)
    self.someParam = someParam

    Status.__construct(self, tinker) -- this sets up the tinker
    self.tinker = tinker -- you could use this instead, but ^ is more consistent
end

-- this method has been mentioned above
-- it is called once the status is first applied, that is, 
-- the with the amount being 0 before
function Sample:apply(entity, amount, options)
    -- the default method is to tink the tinker
    -- let's say we want to print something before doing it
    print('Before tinking')
    -- now tink, using the superclass
    Status.apply(self, entity)
    -- or do it manually
    self.tinker:tink(entity)
end

-- this one is called when the same effect is applied while
-- its amount on entity.statuses isn't 0
function Sample:reapply(entity, amount, options)
    printf('The new amount: %i', amount)
end

-- called at the time of tick when the amount reaches 0
-- the default behavior is to untink the tinker
function Sample:wearOff(entity)
    -- let's say we e.g. don't want to untink
    -- this is why we must override this and leave it blank
end

-- this one is experimental. It could be removed soon.
-- This one is called before the entity does the action, in
-- the `checkAction` chain (HIGHEST priority rank).
function Sample:free(entity)
    -- let's say we want to untink here
    self.tinker:untink(entity)
end
```

Now you may create instances of this new subclass to get status effects with a different behavior.

```lua
local sampleStatus = Sample(someTinker, someParam)
```

The given methods are all the methods of a status effect ever called by extraneous code.


### Useful predefined `Status` Subclasses

There are a couple of useful `Status` subclasses that I have defined.

#### `OptionStatus`

This tinker relies on the assumption that you are using a `StoreTinker`. Here is its entire source code. Pretty self-explanatory:

```lua
local Status = require '@status.status'

-- Options are an object with data.
-- this status saves those options when an effect is applied
-- and deletes them once it is removed.

local OptionStatus = class('OptionStatus', Status)

function OptionStatus:apply(entity, amount, options)
    self.tinker:setStore(entity, options)
    self.tinker:tink(entity)
end

function OptionStatus:wearOff(entity, amount)
    self.tinker:removeStore(entity)
    self.tinker:untink(entity)
end

return OptionStatus
```

So basically the options that were passed to the status effect can now be accessed from the tinker's handlers. More info on [StoreTinkers](tinkers.md).

#### `FlavorStatus`

This is actually a subclass of the previous subclass, the `OptionStatus`. 

By definition, a `status flavor` is a tinker component. It must include the rank or a priority number with the handler. Basically, the component(s) will be tinked once the effect is applied (`apply()`) and untinked once removed (`wearOff()`).

The flavor is expected to be passed with the options object, either as `options.flavor` if you're using a single component or `options.flavors` if you're using several ones.

It is assumed that you're using a `StoreTinker`.


## Standart ones

There are no predefined status effects in the core game logic. I have yet defined 3 in the `test` mod. More on this, mods and their content in designated future articles. You can find the information on how to register your status effects in there too. 