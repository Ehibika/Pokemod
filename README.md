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

### Moves
Within this mod, moves are essentially individual weapons each stored in their own file, these weapons inherit from **WepMasterBase** and consist almost entirely of the weapon's specific actor properties, and firing states. weapon actors have no hud sprites to account for, and all ready, select, deselect,altfire, and noammo functions are all handled by the inherited **WepMasterBase**.

The firing states for moves are typically split up into three main sections, the starting section that leads with the default "Fire:" state, the intermediary section where most of the weapon logic takes place. and lastly the afterfire state, typically defined with "Fire.After" that houses a Refire function for continuous fire moves, and finalizes things by establishing the recovery frames and leading directly into Fire.End, a state that exists within the **WepMasterBase** actor. Recovery frames are actually defined by inventories and are not hard coded, so the fire.after state is usually no longer than a single frame. as such:

```typescript
Fire.After:
TNT1 A 1 A_GiveInventory("Recovery_B",20)
TNT1 A 0 A_REfire("Fire")
TNT1 A 0 A_Jump(256,"Fire.End")
Goto Fire.End
```

Within the starting section of a weapon, the piece of code below is of utmost importance:

```typescript
Fire:
TNT1 A 0 A_Jumpif(CallACS("CheckWeaponFire",RC_B)==0,"NoAmmo") //<==
TNT1 A 0 A_Jump(256,"Fire.Bullet")
Goto Fire.Bullet
```

This takes the place of **A_Jumpifnoammo()** as typically found in most 8bdm weapons, not only does this check for ammo, but it also checks if the weapon is in or out of recovery, and for any special conditions that would otherwise deny the use of weapons. such as:
* most moves are innactive when grappled save for **Rapid Spin**
* most moves are innactive while asleep, save for **Snore** and **SleepTalk**
* Ramming moves are disabled when paralyzed
* fire moves will defrost you if frozen

the one argument this ACS call has dictates what recovery type this weapon uses, there are 3 recovery types, recovery_A, recovery_B, and recovery_C. for reference:
* all ranged/projectile moves use recovery_A.
* all melee/close ranged moves use recovery_B, including the altfire.
* all utility moves use recovery_C.

The generic altfire attack also triggers short recovery in both recovery_A and recovery_C, despite primarily checking for recovery_B

Two other important ACS scripts are at work as well, all of which are located in **WEPACS**, these scripts are **MoveSetup** and **MoveSwap**, and are called within the move's select and deselect states respectively.

**MoveSetup** initiates all the code necessary for the move upon swapping to it. moves that employ the targeter system (defined in the 4th element of the **weapons_ammo** array) will have their specific targeting set up initiated as designated by a switch block. Moves that can heal others (defined in the 5th element of the **weapons_ammo** array) will run code to make ally healthbars visible. any other move specific actions will also be done here, such as hydropump getting a flag actor if the wielder is a blastoise.

**MoveSwap** does cleanup in preparation for the next move, turning off targeter scripts and removes any specialty flags. this script will also remove any temporary weapons that the player has (detailed below).

#### -=- slot weapons and temporary weapons -=-

after the base weapon is completed, you must define copies of it for all 4 slots below it as such:

```typescript
actor BiteMove1 : BiteMove {Weapon.SlotNumber 1}
actor BiteMove2 : BiteMove {Weapon.SlotNumber 2}
actor BiteMove3 : BiteMove {Weapon.SlotNumber 3}
actor BiteMove4 : BiteMove {Weapon.SlotNumber 4}

actor BiteMove5 : BiteMove 
{
Weapon.SlotNumber 5
States
{
Fire.End:
TNT1 A 0
TNT1 A 0 A_TakeInventory("BiteMove5",9)
TNT1 A 0 A_Jump(256,"Ready")
Goto Ready
}
}
```

The 5th slot weapon is defined as a temporary weapon, Temporary weapons are acquired by moves that involve calling up random moves or calling up existing moves from someone else, this includes:
* mimic
* metronome
* mirror move
* me first
* assist

when acquired, you are automatically swapped to the temporary weapon, where you then get 1 use (or 1 button press and release for continuous fire moves) before the move is purged from your inventory, swapping off the move also removes it as well.

#### -=- Weapon design Principles -=-
*WIP*
