# Bully Mission Script

A custom story mission for Bully: Scholarship Edition, written in Lua against the game's scripting API.

## Overview

`Script.lua` implements a complete mission for Bully: Scholarship Edition on PC. Jimmy meets an ally, fights a chain of opponents under the school, stages a brawl between cliques to distract a teacher, goes through a scripted office confrontation, an egg-throwing sequence and a final boss fight in "the hole". Each stage is driven by position checks, enemy death checks and short cutscene-style conversations.

The cast uses custom nicknames rather than the game's character names. The dialogue is informal and contains mature language, and a few lines are written in Arabic chat alphabet.

## Mission stages

| Step | Goal | What happens |
|---|---|---|
| 1 | Meet Saleh outside | Conversation, then Saleh walks away. |
| 2 | Find Ali-ALJBAR under the school | Conversation, then a fight. |
| 3 | Beat the next two opponents | Two enemies attack at a trigger point. |
| 4 | Chase the runners, then face the prefects | Runners flee with boosted speed; two prefects confront the player. |
| 5 | See the teacher, get Saleh's help, start the brawl | Punching the marked Bullies leader starts a Bullies versus Preps brawl that draws the teacher away. |
| 6 | Office scene, then find Khader | A scripted office fight, a knock-out, then regrouping with Saleh and friends. |
| 7 | Throw eggs at ABO-J3FR | The player and Saleh get eggs; a hit sends the target to the hole. |
| 8 | Beat ABO-J3FR | Boss fight with three guards; allies join after a few seconds. |

## How it works

- `MissionSetup()` warps the player, starts the music streams and preloads the animation groups and action trees the mission uses.
- `main()` runs stages 1 and 2, then hands off through `AfterPedIsDead`, `BLastMission`, `LastMission`, `AfterInOffice` and `EggsThrow`, each calling the next.
- `PedsPrepareConversation(ped1, ped2, distance, walkSpeed)` walks one ped to another and makes them face each other before every conversation.
- Cutscenes set widescreen, adjust the camera, play a talking action node per line and show subtitles with `TextPrintString`.
- Enemies are made hostile with `PedSetPedToTypeAttitude` and `PedAttackPlayer`; allies are chained with `PedRecruitAlly` and `PedSetAllyAutoEngage`.
- `MissionFail()` fires if a key character dies or the player leaves the mission area; `MissionSucceed()` ends the boss fight.

## Running the script

- Bully: Scholarship Edition for PC.
- Load it with Derpy's Script Loader (DSL) from its `_derpy_script_loader/scripts` folder, or compile it with the Bully build of the Lua 5.0.2 `luac` (available from the Bully Scripting Docs) and place the `.lur` in `Scripts/Scripts.img` in place of a built-in script such as `ArcRace1.lur`.
- Play with the game's normal controls; the script defines no custom key bindings.

## Notes

- Lines 1151 to 1190 (the office conversation) have their subtitle text masked on purpose, for private reasons.
- One of the author's earliest large-scale Lua scripts, shared for learning, experimentation and reference.

## Credits

- Fibonacci: script development.
- derpy54320: the `PedsPrepareConversation` function.
