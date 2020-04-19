---
id: world
title: Game Loop
---

Here is an overview of what happens during the game loop and in what order.

1. All game objects have priority, which determines in what order they act. For example, *Enemies* have higher priority than *projectiles*, this is why they move first, and then do the projectiles. **All game objects are sorted by priority among their categories at the start of each loop**. Amongst *traps* and *floors*, things are too sorted by priority at the start of each loop. This ensures thing are always in order.

2. After that, all game objects are asked to decide on their next action, and save it inside them (see action.lua for a list of action types). These calculations must not affect the grid (world) state.

3. Now all actions are executed in this order:
    1. *Player* actions
    2. All *Reals* but player
    3. *Explosions*
    4. *Floor* hazards
    5. finally, *Traps*

4. Destroyed (*dead*) things are filtered out, things are rendered and then *reset*. *Resetting* means deleting the actions that the objects chose inside those objects and some other variables for doing those action calculations. 

