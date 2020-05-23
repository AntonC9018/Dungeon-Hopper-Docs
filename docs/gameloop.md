---
id: gameloop
title: Game Loop
---

*Game Loop* is equivalent to a beat in the game, with all actions and iteractions between game objects that happen during that beat. The term *Game Loop* refers in particular to the logic associated with that beat, that is, no graphics is involved.

Here is an overview of what happens during the game loop and in what order.

1. All game objects have priority, which determines in what order they act. For example, some enemies have higher priority than other enemies, this is why they would move first. **All game objects are sorted by priority among their categories at the start of each loop**. The categories are: *players*, *misc* (abbreviation of miscellanious, include e.g. bombs), *non-player reals*, *traps*, *projectiles* and *floors*, things are too sorted by priority at the start of each loop. This ensures things are executed in the right order.

2. After that, all game objects are asked to decide on their next action, and save it inside them (see [Action](action.md)).

3. Now all actions are executed in this order:
    1. *Player*
    2. *Misc*
    3. *Non-Player Reals*
    1. *Projectiles*
    5. *Floor* hazards
    4. *Traps*

4. These groups are `ticked` once all entities from the group have executed their action. `Ticking` means resetting or advancing some internal state. For example, decreasing the amount of the applied status effects, advancing the sequence step etc.

5. Destroyed (`dead`) things are filtered out, things are rendered and then *reset*. *Resetting* means deleting the actions that the objects chose inside those objects and some other variables for doing those action calculations. 

