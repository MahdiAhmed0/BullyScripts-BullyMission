# Bully Mission Script

A custom story mission for Bully: Scholarship Edition, written in Lua against the game's scripting API.

## Overview

This repository contains a single script, `Script.lua`, that implements a complete mission for Bully: Scholarship Edition on PC. It is one of the author's earliest large-scale Lua scripts. The mission follows Jimmy through a chain of objectives: meeting an ally, beating a series of opponents under the school, staging a brawl between cliques to distract a teacher, a scripted (unwinnable) confrontation in an office interior, a sequence with eggs, and a final boss fight in "the hole". Each stage is driven by position checks, enemy death checks and short cutscene-style conversations.

The characters carry custom nicknames rather than the game's character names. The dialogue is informal and contains mature language, and a few lines are written in Arabic chat alphabet (Arabic transliterated with Latin letters and digits).

The author notes that, as an early project, some inconsistencies may exist because of engine limitations, in particular the lack of exception handling mechanisms. `Script.lua` was uploaded in August 2023 and has not changed since; this README was last revised in 2026. The project is archived and shared for learning, experimentation and reference. The repository does not include a license file.

## Requirements

- Bully: Scholarship Edition for PC. The script calls the game's Lua API and the base game's animation groups, action trees (`Act/Conv/6_02.act`, `Act/Anim/G_Striker_A.act` and others) and music streams.
- A way to run a mission-style script (see Installation and usage).

## Installation and usage

The repository does not document how the author ran the script. Two approaches are documented by the community:

1. Replacing a built-in script.
   - The game's Lua 5.0.2 build cannot load Lua source (derpy54320's reference: "it can't read from source (parser doesn't load)"), so the base game's scripts are compiled `.lur` files packed in `Scripts/Scripts.img` (with `Scripts.dir`).
   - Compile `Script.lua` with the Lua 5.0.2 `luac` build provided on the Bully Scripting Docs download page (Bully's Lua uses 4-byte float numbers, so a stock build does not match), for example `luac -o ArcRace1.lur Script.lua`. `ArcRace1.lur` is the script modders commonly replace; a mission installed that way starts when the player interacts with the arcade machine in the boys' dormitory.
   - Open `Scripts.img` with an IMG archive editor such as IMGTool, replace the original `.lur` with the compiled file under the same name and rebuild the archive (the installation notes of community mods such as Repeatable Errands and Selector Mod Remake on libertycity.net describe this step). Back up `Scripts.img` and `Scripts.dir` first.
2. Derpy's Script Loader (DSL) by derpy54320.
   - DSL loads scripts from its `_derpy_script_loader/scripts` folder (a file, a folder or a zip), does not rely on replacing files in `Scripts.img`, runs scripts from source without compiling, and calls a script's `MissionSetup` when the script starts and `main` after it completes.
   - The DSL docs note that a DSL script starts as soon as the game starts, and the Bully Scripting Docs page for `SystemIsReady` says such scripts should wait until it returns true before doing most things. This script has no such wait, so under DSL add a loop at the top of `MissionSetup` that waits for `SystemIsReady()` before the first `AreaTransitionXYZ`.

Once the mission is running, play it with the game's normal controls; the script defines no custom key bindings. `MissionSetup` warps the player into area 14 at (-502, 319, 31), while the first objective waits outside at (270, -101, 6), so leave the interior and follow the blip. The stages, in order (objective text paraphrased):

| Step | Stage goal | What to do |
|---|---|---|
| 1 | Meet with Saleh outside | Walk to the blip over Saleh. |
| 2 | Go to Ali-ALJBAR under the school | Walk to the blip, then win the fight. |
| 3 | Keep going and beat the next two | Approach, then win the fight. |
| 4 | They are running, go after them, then face the prefects | Chase the runners, then beat the two prefects. |
| 5 | Go to see the first teacher, get help from Saleh, hide, go upstairs | Follow the blips; punch the marked Bullies leader to start the brawl. |
| 6 | Office scene, find Khader, see Saleh, take ABO-J3FR to the hole | Follow the blips; the office fight is scripted to be lost. |
| 7 | Throw the eggs at ABO-J3FR and go to the hole | Hit the target with an egg (the player gets 12 eggs, Saleh 10), then run to the hole entrance. |
| 8 | Beat ABO-J3FR | Win the boss fight with the help of the allies. |

## How it works

### Entry points and structure

`MissionSetup` and `main` are the functions a mission script defines; the DSL documentation describes the flow as `MissionSetup` when the script starts, `main` after it completes, and `MissionCleanup` on shutdown (this script defines no `MissionCleanup`). `MissionSucceed` and `MissionFail` are game globals listed in derpy54320's Bully LUA Reference; `MissionSetup` is not listed there.

`MissionSetup` disables player control, warps the player with `AreaTransitionXYZ(0, 270, -110, 6.4)` and then `AreaTransitionXYZ(14, -502.2488, 318.8277, 31.4111)`, re-enables control, calls `DisablePunishmentSystem(true)`, starts the "MS_BikeFun" low, mid and high intensity music streams, disables action-tree speech, and then preloads animation groups and action trees. Lines 30 to 586, a little over a third of the file, are this preload list (550 `LoadAnimationGroup` and 7 `LoadActionTree` calls).

The mission logic is a chain of six stage functions, each called from the end of the previous one, so control never returns to `main`:

| Function | Line | Role |
|---|---|---|
| `MissionSetup` | 1 | Setup and preloading |
| `PedsPrepareConversation` | 590 | Helper: positions two peds for a conversation |
| `main` | 621 | Stages 1 and 2 |
| `AfterPedIsDead` | 744 | Stage 3, and the setup of stage 4 |
| `PedAreDead` | 783 | Predicate for stage 3 |
| `BLastMission` | 796 | Stage 4 |
| `PedIsDeadRunners`, `PedIsDeadAstad` | 873, 884 | Predicates for stage 4 |
| `LastMission` | 896 | Stage 5 |
| `MakePedsIgnore` | 1076 | Helper: calms the brawl |
| `AfterInOffice` | 1101 | Stage 6 |
| `EggsThrow` | 1440 | Stages 7 and 8 |

### Mission stages

| Order | Function | What the code does |
|---|---|---|
| 1 | `main` | Sets the clock to 9:30, spawns Saleh with a blip, waits until the player is within 2 units, runs a conversation, then Saleh walks away. |
| 2 | `main` | Spawns the target, waits for the player, plays the "MS_StealthLow" stream during the talk, then sets the target to attack the player (`PedAttackPlayer(ped, 3)`) and waits until he is dead. |
| 3 | `AfterPedIsDead` | Spawns two enemies that attack once the player reaches a trigger point; `PedAreDead` returns true when both are dead. Then spawns two runners, gives them `GameSetPedStat(ped, 20, 150)` and infinite sprint, sends them to a point with `PedMoveToXYZ`, and spawns the two prefects. |
| 4 | `BLastMission` | Waits until a runner reaches the point or both are dead (`PedIsDeadRunners`), then the two prefects walk up, talk and fight (`PedIsDeadAstad`). |
| 5 | `LastMission` | Conversation with a teacher character, recruit Saleh as an ally, wait until the player hits the marked Bullies leader (`PedIsHit`), start a brawl between a Bullies group and a Preps group, calm it with `MakePedsIgnore`, walk the teacher over to the brawl, then reach a blip to enter the office (`AreaTransitionXYZ(5, ...)`). |
| 6 | `AfterInOffice` | Office cutscene (dialogue text obfuscated), a scripted fight the player cannot win, knock-out animation, warp back, conversation with Khader, regroup with Saleh and friends, meet another teacher character. |
| 7 | `EggsThrow` | Gives the player and Saleh weapon 312 (the egg projectile model), waits until `PedGetLastHitWeapon(target) == 312`, then the target attacks and the player must reach the hole entrance. |
| 8 | `EggsThrow` | Area transition to the hole, boss with 1200 health plus three prefect guards, allies arrive after 6 seconds, scripted boss combos, `MissionSucceed()` when the boss is dead or no longer valid. |

### Conversations

`PedsPrepareConversation(ped1, ped2, distance, walkSpeed)` (written by derpy54320) prepares two peds to talk. It resets the player's punishment points, removes weapon 437 (the skateboard model) from the player if held, warps the player out of any vehicle, stops `ped2`, walks `ped1` toward `ped2` with `PedMoveToXYZ`, makes `ped2` face `ped1`, waits with `PedInRectangle` until `ped1` is inside a square of half-width `distance` around `ped2`, then stops `ped1` and makes both peds face each other. The script also uses it to arrange groups of peds around one character before the player arrives, and always passes `walkSpeed` 0.

Cutscenes follow a common pattern: `PlayerSetControl(0)` and `CameraSetWidescreen(true)`, usually `CameraSetFOV(65)` followed by `CameraSetFOV(40)` about two seconds later (six scenes) or an explicit `CameraSetXYZ` and `CameraLookAtXYZ` placement (three scenes), then a sequence of lines. Each line calls `PedSetActionNode(speaker, "/Global/6_02/FactionLeaderNIS/Preps/Chad01", "Act/Conv/6_02.act")` to play a talking animation, `TextPrintString(text, seconds, 2)` to show the subtitle, `SoundRemoveAllQueuedSpeech` to keep the ped from speaking its own lines, and a `Wait` of about the same length as the subtitle. Objective messages use `TextPrintString(text, seconds, 1)`; the one exception is the "they are running" message, which uses the dialogue style 2. Scenes end with `CameraReset()`, `CameraReturnToPlayer()`, `CameraSetWidescreen(false)`, in most cases `SoundSetAudioFocusPlayer()`, and `PlayerSetControl(1)`.

### Fights, factions and allies

Enemies are made hostile with `PedSetPedToTypeAttitude(ped, 13, 0)` followed by `PedAttackPlayer(ped, 3)`. In derpy54320's tables, faction 13 is the player and attitude 0 is "Hate" (the function's parameters themselves are not documented); the Bully Scripting Docs describe `PedAttackPlayer` priority 3 as relentless and 1 as a low priority that can be distracted. Bullies, Preps and Prefects are the game's faction names. The brawl in `LastMission` uses the same mechanism between peds: one side (ped models from the Bullies clique) is set to hate faction 5 (Preps) and the other side (Preps models) is set to hate faction 11 (Bullies), and each ped is given a specific opponent with `PedAttack`. `MakePedsIgnore` later switches every brawler's attitude to 3 ("Like") and lowers the attack priority to 1 so the fight winds down.

Allies are chained with `PedRecruitAlly(leader, follower)` and `PedSetAllyAutoEngage(follower, true)`. In the boss fight the allies are set to hate faction 0 (Prefects) and the main loop re-targets them with a list of `if PedIsDead(...)` rules as the guards die; once all three guards are dead, all allies attack the boss.

The office fight in `AfterInOffice` is scripted to be lost: the boss gets `PedSetDamageGivenMultiplier(boss, 2, 200)`, his health is reset to the player's maximum health value (`PlayerMaxHP`) whenever it drops to 100 or below, and the loop exits when the player's health falls to 30 or below (a drop to 20 or below is restored to maximum first). The script then sets the player's health to 1, plays the "KO_COLLAPSE" and "KnockoutDead" action nodes, and warps the player out with `AreaTransitionXYZ(2, -633, -291, 5)`.

### Fail conditions

- `PedIsDead(Saleh)` and `PedIsDead(Saleh1)` while the player is on the way to them, and `PedIsDead(Khader)` right after his conversation, while the player walks to the marked spot.
- `PedIsInAreaXYZ(gPlayer, 271, -122, 6, 2)`: entering a fixed point near the starting area, (271, -122, 6) with radius 2, calls `MissionFail()` during stages 2 to 4. The check appears seven times: twice in stage 2, twice in stage 3 (the loop that waits for the player to reach the two opponents has no check) and three times in stage 4. The script does not say what the point represents.

### Blips and markers

Characters to meet get `AddBlipForChar(ped, 0, 0, 4)` and characters to fight get `AddBlipForChar(ped, 0, 26, 4)`. In derpy54320's blip icon table, 0 is a yellow cross and 26 a red cross, assuming the third argument is the icon ID (the function's parameters are not documented in the sources consulted). Fixed points (the office entrance and the hole) use `BlipAddXYZ`. One helper ped (`JuseForBlip`) is spawned only to carry a blip and is deleted when the player arrives.

### Cast and ped models

| Character (as named in objectives) | Ped model ID | Model name in derpy54320's ped table |
|---|---|---|
| Saleh (ally) | 32 | N_Chad |
| Ali-ALJBAR | 8 | N_Bucky |
| Two opponents under the school | 139, 71 | N_Gordon, N_Ray |
| Runners | 102, 146 | N_Troy, N_Wade |
| Two prefects | 49, 51 | N_Max, N_Edward |
| First teacher character | 61 | N_MrHuntingdon |
| Khader (teacher) | 151 | N_MrWiggins |
| Third teacher character | 57 | N_MrGalloway |
| ABO-J3FR (antagonist and boss) | 65 | N_DrCrabblesnitch |
| Office group | 8, 4, 7, 6, 28 | N_Bucky, N_Algernon, N_Thad, N_Melvin, N_Ricky |
| Brawl, Bullies side | 99, 102, 85, 147, 145 | N_Davis, N_Troy, N_Trent, N_Tom, N_Ethan |
| Brawl, Preps side | 37, 31, 33, 34, 35 | N_Darby, N_Tad, N_Bif, N_Justin, N_Bryce |
| Saleh's friends | 30, 31, 33, 34, 35 | N_Gord, N_Tad, N_Bif, N_Justin, N_Bryce |
| Boss fight guards | 52, 51, 50 | N_Karl, N_Edward, N_Seth |

## Notes and known limitations

- Lines 1151 to 1190 (the office conversation) have their subtitle text replaced with asterisks on purpose, for private reasons. The surrounding code is intact.
- Line 1355 passes `gplayer` (lowercase) to `PedRecruitAlly`; in Lua an unassigned global is nil, so the intended player handle `gPlayer` is not used there.
- Line 1548 tests `PedIsDead(AboJ3frBossFightABOJ3FRFOffice)`, an identifier that is never assigned, so `PedIsDead` is called with nil on every iteration of the boss combo loop; the intended handle is `BossFightABOJ3FR`. How the game's Lua runtime reacts to the nil argument was not tested.

## Credits

- Fibonacci: script development.
- derpy54320: implementation of the `PedsPrepareConversation` function, and the Bully LUA Reference used for the ID tables in this README.

## References

- derpy54320, Bully LUA Reference (repository archived October 2023, site still online): https://derpy54320.github.io/Bully-LUA-Reference/
- Bully Scripting Docs (community site, repository maintained by RanggaBS; includes the DSL reference): https://bully-scripting.vercel.app/
- Bully Scripting Docs, Lua 5.0.2 downloads: https://bully-scripting.vercel.app/docs/miscellaneous/downloads/lua
- Bully Scripting Docs, DSL downloads: https://bully-scripting.vercel.app/docs/miscellaneous/downloads/dsl
- Community mod pages describing the Scripts.img replacement step: https://libertycity.net/files/bully-scholarship-edition/225724-repeatable-errands.html and https://libertycity.net/files/bully-scholarship-edition/164620-selector-mod-remake-.html
