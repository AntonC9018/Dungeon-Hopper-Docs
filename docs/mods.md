---
id: mods
title: Mods
---

Mods are a collection of base classes for entities and items, entities and items themselves, stats, status effects and attack sources. They may also expose decorators, actions, effects, retouchers, interactors and tinkers.

Mods provide the content for the game. On its own, the game has no entities and no items in the pool. It has no tiles, no floors, levels etc. All this content is added by mods.

Currently, there is ONE mod available: the test mod.

## How do I create mods?

1. Navigate to `/modules` in the repository.
2. Create a folder by the name of your mod. Lower case is the convention.
3. Add a `main.lua` file listing all the content.

```lua
return {
	requiredMods = {},
    bases = {
        entity = {},
        item = {}
    },
    entities = {},
	items = {},
	decorators = {}, -- you can choose not to expose e.g. decorators
	attackSources = {
		'Bounce'
	},
	stats = {
		StuckRes = { 'resistance', { 'stuck', 1 } }      
	},
	status = {
        -- dot at the front replaces `modules.MOD_NAME.`
		freeze = require '.status.freeze'
    }
    -- ...
}
```

You may omit any of the fields.


## How to reference the content of the mod?

Most exposed content of your mod is saved at `Mods.MOD_NAME`.

```lua
-- assume the mod is test
-- Mods.Test.Bases.Items.Whatever
-- Mods.Test.Bases.Entities.Whatever
Mods.Test.Items.whatever
Mods.Test.Entities.Whatever
-- Mods.Test.Decorators.Whatever
-- Mods.Test.Retouchers.Whatever
-- Mods.Test.Interactors.Whatever
-- Mods.Test.Effects.Whatever
-- Mods.Test.Actions.Whatever
-- Mods.Test.Tinkers.Whatever
-- not complete yet
```

You can use this content inside any other mod loaded after yours.

The other content (status effects, attack sources, stats) are registered automatically and available globally as per usual.

```lua
StatTypes.statFromMod     -- yes
StatusTypes.statusFromMod -- yes
-- there's currently no way of getting a reference to the status effect itself
```