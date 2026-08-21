# Monolith VTT Notes

This is practical guidance for material written for Monolith in The Devil's
Toys. The live authority is always `system.json`; use the campaign check command
at the end rather than relying on this page remaining in sync by hand.

## Campaign vocabulary

| Campaign field | Monolith value |
| --- | --- |
| Manifest `system` | `monolith` |
| NPC statblock keys | `hp`, `armor`, `str`, `dex`, `wil`, `attacks`, `secondWeapon` |
| Numeric NPC fields | `hp`, `armor`, `str`, `dex`, `wil` |
| Text NPC fields | `attacks`, `secondWeapon` |
| Character item lists | `equipment`, `augmentations` |
| Encounter sides | `party`, `enemies` |
| Initiative | Side initiative; do not set `individualInitiative` |
| Hirelings | Supported; Monolith calls them Freelancers |
| Shared asset kinds | `starship` |

An omitted NPC ability is 10 by Monolith's rules. Omit an ordinary `str`, `dex`,
or `wil` rather than filling every statblock with three tens. Armor runs from 0
to 3; omit `armor` when it is 0. Put the primary attack in `attacks` and only use
`secondWeapon` when the NPC has another action the GM needs in the combat rail.
Behavior, tactics, saves caused by attacks, and critical-damage consequences
belong in the NPC's Markdown `notes`.

Items under `equipment` are ordinary gear, weapons, drugs, biological curios,
and artifacts. Use `augmentations` only when the thing is installed into one of
the character sheet's augmentation slots. A campaign asset is property the
party already owns; a ship they might steal, repair, or earn remains a location,
reference, or NPC note until that happens. The same distinction applies to
Freelancers: importing one puts them on the party's roster immediately.

## Converting percentile science-fiction adventures

Monolith's own NPC rules say to convert fiction first. Percentile monster
abbreviations are therefore threat signals, not fields to copy or numbers to
calculate from.

| Source notation | Monolith treatment |
| --- | --- |
| Combat or attack percentage | Do not convert it. Monolith attacks deal damage without an attack roll. Preserve the weapon and what makes the attack dangerous. |
| Instinct | Decide whether the fiction describes reflexes (`dex`), resolve or psychic force (`wil`), or neither. There is no one-to-one field. |
| Wounds and health per wound | Do not multiply them into HP. Use 3 HP for average creatures, 6 for hardy ones, and 10 or more for serious threats, with 18 as the ceiling. Use `str` and critical-damage consequences for bodily resilience. |
| Armor Save or armor percentage | Give 1–3 Armor only when equipment or the creature's body actually soaks damage. |
| Body, Speed, or Sanity save | Use a STR, DEX, or WIL save respectively, adjusting when the fiction points elsewhere. |
| Stress or Panic | Use a WIL save against corruption, fatigue, or deprivation, as appropriate. |
| Wound or mega-damage attack | Make it a severe damage die or a clearly stated special/critical consequence; do not turn it into dozens of HP damage. |

Damage dice are chosen by threat and fiction. D4 is weak or improvised, D6 is a
standard armed attack, D8 is dangerous, D10 is severe, and D12 should be rare.
Keep unusual effects—paralysis, assimilation, swallowing, psychic domination—in
notes with the save and consequence that make them playable. This is more useful
than preserving a source die whose combat economy Monolith does not share.

## Encounters and tables

Prepared encounters use `party` and `enemies`. Leave their side initiatives
`null`: Monolith resolves the start of combat through PC DEX saves, then uses the
PC turn followed by the enemy turn. NPC combatants default to `enemies` and
Freelancers default to `party`, so explicit sides are optional when those
defaults are right.

Campaign random tables are system-independent and may keep supported dice such
as d5, d10, and d100. They import into the server-wide table catalogue, not just
the Monolith room, and should omit tags unless the destination server's tag
vocabulary is known.

## Validate a campaign against Monolith

From the campaign importer kit beside this repository:

```sh
node tools/check.mjs campaigns/<name> --system ../devils-toys-monolith
node tools/build.mjs campaigns/<name> --system ../devils-toys-monolith
```

The system-aware check reads this repository's current definition and catalogue.
It catches a wrong manifest id, unknown or wrongly typed statblock fields, wrong
item lists or retired ids, unsupported sides or initiative, and unsupported
hireling or asset data before the bundle reaches a room.
