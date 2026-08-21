# Monolith for The Devil's Toys

> Freelancers at the edge of a sprawling, dangerous cosmos.

This repository is a game system for [The Devil's Toys](https://github.com/tpskowski/the-devils-toys),
a virtual tabletop. It carries Monolith's rules text, its extracted tables, its
gear, and the declarative definition that tells the application how to lay out a
character sheet, roll its dice, and cut its rules for players.

It is data. There is no code in this repository, and nothing in it is executed by
the server that installs it.

## Installing it

A server admin opens **Management → Systems** and installs Monolith from the
catalogue. To install straight from here instead, give that panel this
repository and a ref:

    tpskowski/devils-toys-monolith    main

The server downloads the tree, checks it against the system schema, and installs
it into the running instance. No restart, and no rebuild of the application.

## What is in here

| File | What it is |
| --- | --- |
| `devilsystem.json` | The marker that says this repository is a game system, and which one |
| `system.json` | The system definition: sheet layout, dice, content modules, warnings, NPC statblocks, group views |
| `items.json` | The gear catalogue |
| `traits.json` | What the book's weapon and gear words mean |
| `rules/Monolith.md` | The rules text the application serves |
| `rules/corrections.md` | Every repair made to the source, and why |
| `tables/monolith.json` | The tables extracted from the rules, ready to roll on |
| `notes.md` | Working notes on how this system was fitted to the application |
| `character-creation-plan.md` | The draft declaration for the guided character builder, and the repository work it needs first |

`source/` holds the original book and is not published here — see
[NOTICE.md](NOTICE.md).

## Optional rules

Monolith offers **tags** and starts with them off. A table that switches them on
in the room's settings can put words of its own on the crew, the cast, the
freelancers, and everything in the Library, and find any of it again by those
words. It is a facility the application provides rather than anything the book
asks for, which is exactly why it is offered rather than imposed.

## Working on it

See [AGENTS.md](AGENTS.md).

When adapting an adventure into a Monolith campaign, use the exact campaign
vocabulary and fiction-first conversion guidance in [notes.md](notes.md). From a
campaign importer checkout beside this repository, validate the result against
the live system definition rather than copied field names:

```sh
node tools/check.mjs campaigns/<name> --system ../devils-toys-monolith
```

## Licence and attribution

Monolith is by Adam Hensley and its text is licensed under
[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). See [NOTICE.md](NOTICE.md) for the full
attribution and the record of what was changed.

Adam Hensley does not endorse this project and has not reviewed it. The
application code that reads these files is MIT-licensed and lives in the core
repository, not here.
