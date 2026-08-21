# Character creation, declared

Working notes and a draft declaration for the guided character builder. The
application-side design is
[`docs/character-builder-plan.md`](https://github.com/tpskowski/the-devils-toys/blob/main/docs/character-builder-plan.md)
in the core repository; this file is what Monolith has to say and do to use it.

Nothing here is executable. `characterCreation` is data in `system.json`, the way
`characterSheet` and `warningRules` are, and the application performs it.

## What the book asks for

CHARACTER CREATION, in the book's own order:

1. **Ability scores.** 3D6 for each of STR, DEX, WIL in order. The player may swap any two of the results.
2. **Background.** Roll 1D12, consult the background, then roll on all three tables listed under it — one of which is read at the HP roll rather than rolled. See item 4 below; the declaration therefore takes these two in the other order.
3. **Hit Protection.** 1D6.
4. **Finishing touches.** Optionally roll on the tables in that section.
5. **Shared debt.** One D12 for the whole company. "Optional but highly suggested."
6. **Vices.** With the scores in place, make a WIL save; on a failure roll D10 and start with that vice.
7. **Starting gear.** Three days of rations, a cheap data-comm, a glo-torch, and 3D6 credits.

And GEAR PACKS as a variant: skip the background, cross your highest ability
score against your HP, and take the package that falls out.

## The four things this repository has to fix first

These are found by writing the declaration against the sheet that exists, and
none of them is the application's to solve.

### 1. Three fields the sheet has not got

The PC sheet declares `level`, `xp`, `corruption`, the five current/maximum
pairs, `criticalDamage`, `abilities`, and `vices`. Steps 4 and 7 have nowhere to
land. Adding a field is safe — the installer only refuses a sheet that _drops_
one — so `characterSheet.sections` gains:

| Key          | Section    | Kind       | Why                                                                               |
| ------------ | ---------- | ---------- | --------------------------------------------------------------------------------- |
| `background` | `identity` | `text`     | Which of the twelve, so the sheet says it rather than only the notes              |
| `credits`    | `identity` | `number`   | 3D6 credits at creation, and the currency the whole COST OF LIVING section spends |
| `details`    | `identity` | `textarea` | Where finishing touches and the background's rolled results are written           |

`details` is the same key and the same purpose the Freelancer sheet already uses
for exactly this, which is the argument for the name.

### 2. The vices table is called AMBITION

The D10 vices table sits under the last of the ten `### <VICE>` prose headings,
so the roller names it after that heading: the catalogue holds
`VICES · AMBITION`, a d10 with columns `Vice`, `Triggers`, `Satisfying`.
`viceCatalog` finds it by column and does not care, but a creation step naming a
table by name cannot.

Give the table a heading of its own — `### Vices Table`, matching the shape
`Scars Table` already has — and record it in `rules/corrections.md` beside the
other table-structure repairs. Then rebuild:

```sh
npx tsx scripts/tables-md-to-json.ts --repo ../devils-toys-monolith
```

### 3. There is no backgrounds table

The twelve backgrounds are `### 01 — MERCENARY` … `### 12 — BOUNTY HUNTER`
headings. The book says roll 1D12 and consult, and there is nothing for the
roller to land on. This is why the application's `packet` step rolls its own die
against the sections it enumerates rather than reading a table — do **not** add a
d12 backgrounds table to the Markdown to work around it. That would put the list
of backgrounds in two places, and the headings are already the authority.

### 4. One roll gives both HP and the background's signature gear

CHARACTER CREATION says each background has "three tables, one corresponding to
HP", and the book means it. The first table under every background — Signature
Weapon, Side-Effects, Pet Project, and the nine others — was printed with a die
column reading `1 HP` through `6 HP`: you do not roll it, you read it at the Hit
Protection you already rolled. GEAR PACKS says the same thing in prose, and the
`4 HP | 5 HP | 6 HP` headings of its own chart are the surviving trace of it.

**Do not record this as a correction.** The book is right. The annotation was
stripped here by the existing "die values in background tables written as plain
numbers" repair, whose own _Worth knowing_ paragraph names exactly this
consequence: the cross-reference now lives only in the surrounding prose. There
is nothing further to record and nothing in `rules/Monolith.md` to change.

Two things follow for the declaration, and both are in the draft below:

- The **Hit Protection step comes before the background step**, not in the book's printed order. The background cannot be read until the d6 is on the table.
- The background's first table takes its total from that step — `fromStep` — rather than rolling a d6 of its own. Its other two tables roll normally.

Restoring the `1 HP` annotation is the alternative, and it is worse: the die
column would parse as text and the table would stop being rollable at all.

## The draft declaration

Ten steps. Written against the application vocabulary as planned; field names
here move if that plan's do, which is why nothing should be committed to
`system.json` before the schema is published.

```jsonc
{
  "characterCreation": {
    "label": "Build a freelancer",
    "rulesQuery": "CHARACTER CREATION",
    "steps": [
      {
        "id": "scores",
        "kind": "roll-scores",
        "label": "Ability scores",
        "rulesQuery": "ABILITY SCORES",
        "hint": "3D6 for each, in order. You may swap any two of the results.",
        "scores": [
          { "label": "STR", "dice": "3d6", "currentKey": "strCurrent", "maximumKey": "strMax" },
          { "label": "DEX", "dice": "3d6", "currentKey": "dexCurrent", "maximumKey": "dexMax" },
          { "label": "WIL", "dice": "3d6", "currentKey": "wilCurrent", "maximumKey": "wilMax" }
        ],
        "rearrange": { "kind": "swap", "count": 2 }
      },
      {
        "id": "hit-protection",
        "kind": "roll-scores",
        "label": "Hit Protection",
        "rulesQuery": "HIT PROTECTION (HP)",
        "hint": "1D6. Roll it before your background: your background's signature gear is read at this number.",
        "scores": [{ "label": "HP", "dice": "1d6", "currentKey": "hpCurrent", "maximumKey": "hpMax" }]
      },
      {
        "id": "background",
        "kind": "packet",
        "label": "Background",
        "rulesQuery": "BACKGROUNDS",
        "hint": "Roll 1D12, or pick. Its first table is read at your HP; the other two are rolled.",
        "under": "BACKGROUNDS",
        "dice": "d12",
        "prose": "PROFILE",
        "grantFrom": "STARTING GEAR",
        "listKey": "equipment",
        "reuse": [{ "position": 1, "fromStep": "hit-protection" }],
        "into": { "field": "background", "joinInto": { "field": "details", "separator": "
", "prefixWith": "table" } }
      },
      {
        "id": "finishing-touches",
        "kind": "roll-table",
        "label": "Finishing touches",
        "rulesQuery": "FINISHING TOUCHES",
        "optional": true,
        "section": "FINISHING TOUCHES",
        "tables": [
          { "table": "Physique", "column": "Result" },
          { "table": "Hair", "column": "Result" },
          { "table": "Face", "column": "Result" },
          { "table": "Mannerisms", "column": "Result" },
          { "table": "Clothing Style", "column": "Result" }
        ],
        "joinInto": { "field": "details", "separator": "\n", "prefixWith": "table" }
      },
      {
        "id": "name",
        "kind": "roll-table",
        "label": "Name",
        "section": "FINISHING TOUCHES",
        "optional": true,
        "tables": [
          { "firstOf": ["Male Names", "Female Names", "Ambiguous Names"], "column": "Result" },
          { "table": "Last Names", "column": "Result" }
        ],
        "joinInto": { "field": "$name", "separator": " " }
      },
      {
        "id": "vice",
        "kind": "save",
        "label": "Vices",
        "rulesQuery": "GAINING A VICE",
        "optional": true,
        "hint": "With your scores in place, make a WIL save. On a failure you begin with a vice.",
        "key": "wilCurrent",
        "type": "WIL",
        "on": "failure",
        "then": [
          {
            "id": "vice-roll",
            "kind": "roll-table",
            "label": "Your vice",
            "section": "VICES",
            "tables": [{ "table": "Vices Table", "column": "Vice", "field": "vices" }]
          }
        ]
      },
      {
        "id": "starting-gear",
        "kind": "grant",
        "label": "Starting gear",
        "rulesQuery": "STARTING GEAR",
        "listKey": "equipment",
        "items": ["monolith/rations", "monolith/data-comm", "monolith/glo-torch"],
        "roll": [{ "dice": "3d6", "field": "credits", "label": "Credits" }]
      },
      { "id": "level", "kind": "set", "label": "Level", "values": { "level": 1, "xp": 0, "armorMax": 0 } },
      {
        "id": "currents",
        "kind": "derive",
        "label": "Fill in your current scores",
        "derive": [
          { "key": "armorCurrent", "op": "copy", "from": ["armorMax"] }
        ]
      },
      {
        "id": "debt",
        "kind": "rules",
        "label": "Shared debt",
        "rulesQuery": "GROUP DEBT",
        "hint": "One D12 for the whole company, rolled once. Your GM rolls it on the group page."
      }
    ]
  }
}
```

### Notes on the draft

- **`$name` is the character's name column**, not a sheet field. The application declares it as `CREATION_NAME_KEY` and permits it as a write target nowhere else; a sheet that declared a field of that name would be refused at install.
- **`type` is `WIL`, not `wil`.** A save step names one of the system's own `dice.save.types` ids, and Monolith writes its three in capitals.
- **`rollTablesUnder` defaults to true and is left out.** A packet is its tables; saying so would be noise. `reuse` is a field of its own beside it, naming the tables that are read rather than rolled.
- **`position: 1` is the book's own ordering, not a guess.** All twelve backgrounds put their HP-indexed table first, under `#### STARTING GEAR`, ahead of the two that are rolled. Naming it by position is the only handle there is — the twelve tables share no name, no column, and no tag that the other twenty-four do not also carry.
- **`hp`, `str`, `dex`, and `wil` currents are set by their own `roll-scores` step**, which writes both keys of a pair. Only `armorCurrent` needs `derive`, because armour is the one pair no creation step rolls.
- **`listKey` is not optional here in practice.** This sheet has two lists, `equipment` and `augmentations`, so an unmatched gear line has no obvious home. Naming `equipment` is what the book means — a background's bullets are things you carry, and nothing in BACKGROUNDS grants an augmentation.
- **The background's gear is a checklist, not an application.** The bullets under a background's `#### STARTING GEAR` are prose — "Combat Knife (D6) or 2 Flash Grenades" is a choice, and "Cigar" is not in the catalogue at all. The application matches what it can against `items.json` and offers the rest as the text a slot holds anyway. Nothing here should try to make those bullets machine-readable; that would be restating the book in `system.json`, which is the thing the whole repository is arranged to avoid.
- **The Mercenary's Old Crew Specialty rows grant gear too** — "Take holo-binocs and a claymore" — and so do half the background tables in the book. Same treatment: rolled, written into `details`, offered as text. A player stows what they want.
- **Gear packs are not in the draft.** The variant needs a two-axis lookup — highest ability score against HP, split across two tables because it did not fit the page — and the roller has no notion of such a table. It stays a `rules` step pointing at GEAR PACKS until that exists, if it ever does. Note that its HP axis is the same 1d6 the background's first table reads, so a step that can reuse a roll is half of what it would need.
- **Talents, psionics, astromancy, artifacts, and ex-tech** are referenced by background results by number ("Psionic #22", "Talent #18") and each has a table of its own. Rolling them from a background result is a link between tables, which the roller already has — worth trying once the builder works, and out of the first pass.

## Before this is committed

`characterCreation` is not in the published schema yet. Until it is, adding this
block to `system.json` makes the system uninstallable — the schema is `.strict()`
and an unknown top-level key is refused, which is the behaviour we want.

Order of work in this repository:

1. Add `background`, `credits`, and `details` to `characterSheet.sections`. Independent of the builder, useful on its own, and safe.
2. Give the vices table its own heading, record it in `rules/corrections.md`, rebuild `tables/monolith.json` with `--repo`.
3. Record the HP-sentence correction.
4. Wait for `characterCreation` in the published schema, then add the block and validate:

```sh
npm run systems:validate -- ../devils-toys-monolith
```

Run from a core checkout beside this one. It runs exactly what an install runs,
which will by then include the creation cross-references: every field a step
writes, every table it names, every heading it enumerates.
