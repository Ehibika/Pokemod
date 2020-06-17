# Pokemod
a 3rd person shooter/moba like project made within zandronum that employs pokemon mechanics, 
intended for play through the Megaman 8 bit deathmatch full conversion mod

This README assumes basic knowledge of modding within megaman 8 bit deathmatch.

### Table of contents
* Decorate
  1. important files
  1. stats
  1. moves
  1. pokemon classes
  1. items
* ACS
  1. types of ACS files
      * Dex Files
      * Standalone ACS files
      * General ACS files
      * Database files
      * Modified core ACS files
      
      
      
## Decorate

### important files

#### MasterMoveList.txt
This file isn't loaded in the **DECORATE** lump at all, it's main purpose is to serve as a master reference for all the moves
in the game in order of creation. this is also used for creating entries through copy/pasting and regEx modifications for things
like damagefactors or language definitions.

#### Jobconstants.txt
Constants are stored largely within the **Jobconstants.txt** file, a carryover name from the mod this mod was built off of. 
this txt file contains constant values defining all of the game's types, moves, and bitwise operators for scripts meant to be
called within decorate. **it is imperative that the type constants and move constants called within ACS are the same as the ones 
in decorate**.

#### BaseActors.txt
Contains primary inheritance actors for most gameplay actors, such as projectiles, watchers, clienside FX, and more.

#### BaseFlags.txt
Contains general purpose or script specific inventories and custominventories

#### ArcTargeter.txt
contains actors for the arching targeting system, lifted from the healshots golf mod

#### CustomRecover.txt
Contains replacements for map recovery items, currently they are replaced with money spawns

#### DamageDisplay.txt
Contains actors relevant to the damagedisplay script

#### GiverPackage.txt
Contains the master custominventory used for the radiusgive script, this works through a list of state jumps that check for the **PackageID** inventory from highest to lowest, and each state jump leads to **Pickup.After**.

#### HealerUtilities.txt
Primarily contains actors means for Tinybars, which manifest to display the health of allies while a healing move is currently selected.

#### Mapitems.txt
Contain actors for Item pickups that only appear on the map.

#### Replacingstuff.txt
Contains replacements for map weapons, currently they are replaced with item spawners.

#### ThirdPCamera.txt
Contains actors relevant to the 3rd person camera system, including the diagetic reticle

### Stats
The mods stat system all involves the following files:
* StatList.txt
* **AttackBase.txt**
* **DefenseBase.txt**
* PhysicalAttack.txt
* SpecialAttack.txt
* PhysicalDefense.txt
* SpecialDefense.txt
* **PKMNType.txt**

Files in **Bold** are subject to future modification as new content and moves are created. the **AttackBase.txt** and **DefenseBase.txt** contain the defining powerups and powerprotects for each type in accordance to it damage category, this uses an __exclusionary system__ where inherited actors utilize "Damagefactor "NORMAL"" to modify all damagetypes except for the ones brought up within the parent powerup, so in otherwords, the **PAtk** actor modifies all physical attacks by defining all special attack to be unmodified.

both attack powerdamages and defense powerprotects all inherit from **ATK** and **DEF** respectively, these contain damagefactors that are not modified by either physical or special offenses and defenses.
