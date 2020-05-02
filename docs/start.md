---
id: start
title: What is this?
---

It's a game based on mechanics of NecroDancer. Right now I'm developing the logic for it. I've done an attempt at this previously, but the project had gone out of hand rather quickly. You may find the old code in old versions of the repo.

I pondered how I could refine what I had and decided to rewrite everything from scratch taking a more strict and manageable approach. Nonetheless, the initial attempt was fine for the first attempt, and good for realizing the right direction.

The core things have been written, and the project has been debugged, so it works!

## Dependencies

Among dependecies, there is the [Luaoop class library](https://github.com/ImagicTheCat/Luaoop) by ImagicTheCat(classes are used extensively throughout the project) and [inspect](https://github.com/kikito/inspect.lua) by kikito (dev dependency, useful for debugging). See their github repos for docs.

[Vec](https://github.com/AntonC9018/Dungeon-Hopper/blob/master/lib/vec.lua) and [Emitter](https://github.com/AntonC9018/lua-event-emitter) are also used but are not documented in these docs (have not been thus far). [Chains](https://antonc9018.github.io/Dungeon-Hopper-Docs/docs/chains) are both used extensively and documented here.

## Progress

List of things already implemented:
1. Chains and Chain Templates (Implemented sorting based on priority)
2. High-level Grid
3. General action execution algorithm for player and non-player entities
4. Attacking and Attackable decorators
5. Sequences
6. Weapon target selection logic

List of important things not implemented:
1. Basic graphics +
2. Pushing +
3. Moving +
4. Sequential decorator +
5. Ticking +
6. HP +
6. Basic Controls -

List of less significant things not implemented:
1. Digging, walls +
2. Traps +
3. Special tiles +
4. Explosions
5. Environment Objects
6. Status effects
7. Projectiles
8. Items
10. Better controls
9. Better renderer

List of dreams:
1. World generation
2. Enemy pools
3. Shopping
4. Secrets
5. Music
6. Lobby
7. Menu

More:
1. More Enemies!
2. More Items!
3. More Weapons!





