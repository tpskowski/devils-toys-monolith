# Working on this repository

This is a game system for [The Devil's Toys](https://github.com/tpskowski/the-devils-toys).
It is data only. If you find yourself wanting to add a `.ts` file, the change
belongs in the core repository instead.

## The shape of it

    devilsystem.json      the marker: app, formatVersion, systemId, systemName, licenses, version
    system.json           the GameSystem, as data
    items.json            the gear catalogue
    traits.json           what the book's weapon words mean
    rules/                the Markdown the application serves
    tables/               the tables extracted from that Markdown
    source/               the original book — gitignored, never published

`system.json` is the whole of the system's behaviour. It is declarative: every
field is data the application reads, and there is no hook, callback, or
expression anywhere in it. A player warning is a `warningRules` entry, not a
function. That is what makes a system installable at all, and it is not
negotiable — the installer will not run code and there is nowhere to put it.

`version` is this repository's own release, and it is written by hand: nothing
derives it from a tag, a commit, or the version of the book. Keep it dotted
numbers — `MAJOR.MINOR.PATCH` — because two versions that both read as numbers
are compared as numbers, and anything else can only be reported as *different*,
never as *newer*.

Bumping it is what makes an update visible. A server with Monolith installed
reads this one file from the repository, sees a later version than the one it
holds, and offers its admin the update; never bumping it means never offering
one, however much has changed underneath. So bump it in the commit that carries
the change worth pulling — a rules repair, a new sheet field, a rebuilt
`tables/` file — and leave it alone for anything a server has no reason to
fetch.

## Before you push

    npm run systems:validate -- ../devils-toys-monolith

Run from a checkout of the core repository beside this one. It runs exactly the
checks the server runs at install time — the schema, the sheet and statblock
cross-references, and the table parse — so anything it accepts will install.

The same command runs in CI on every push. A red build means the system will not
install, not that a style rule was broken.

It runs against whatever the core repository has on `main`, which is what makes
a declaration using something the application has not shipped yet a red build
rather than a wrong one. So **open such a pull request as a draft**, and mark it
ready once the core change it depends on is merged. The order is the core change
first, then this one: a system declaring a field the installed application does
not know is refused at the door, and CI is that door.

That is only for a change that needs a new build. A rules repair, a rebuilt
`tables/` file, or anything else the shipped application already understands has
nothing to wait for and goes up ready for review.

## Changing the rules text

`rules/Monolith.md` is the text the application serves. It is the book,
converted, and it is not the place for corrections made silently: every
divergence from the source goes in `rules/corrections.md`
with what was changed and why. That file travels with the book so the record
cannot be separated from the thing it corrects.

If you change the rules Markdown, rebuild `tables/monolith.json` from it —
that file is generated, not hand-maintained. The extraction tooling lives in the
core repository; run it from a checkout beside this one:

    npx tsx scripts/tables-md-to-json.ts --repo ../devils-toys-monolith

Add `--check` to assert the committed JSON is current without rewriting it,
which is what you want in CI and before a release.

Rebuild it this way and not with the older `--in`/`--out` pair. That pair cannot
see `gmOnlyHeadings`, so it writes tables carrying no `classification` — and a
table with no classification is refused to *players*, not to the GM. A system
rebuilt that way looks fine to whoever rebuilt it and has silently lost every
table its players could reach.

The application only ever reads the committed JSON, so this is an authoring step
and never a build step: a stale `tables/` file installs perfectly well and is
simply out of step with the book. `systems:validate` will tell you the JSON
parses; only `--check` tells you it still matches the Markdown beside it.

## Optional rules

`optionalRules` is what this system offers a table rather than imposes on it.
Each entry names the `feature` the application switches on — the ids the
application knows are its own, not this repository's to invent — and `default`
is where a new room starts. Monolith declares one, `tags`, defaulting off.

Two things follow from a room recording only where it has *moved* a rule.
Changing a `default` here changes it for every room that never said otherwise,
and for none of the rooms that did. And an `id` is what those rooms' settings are
recorded against, so it is as permanent as the system's own: rename one and every
room that had switched it goes back to the default.

Mark a rule `required` for something Monolith is played by rather than offered —
no room is then given a switch, and the rule is on everywhere.

## Changing the sheet

Adding a field to `characterSheet` is safe. **Removing one is not** — a
character already written against it would lose what it holds, and the installer
refuses a replacement that drops a field in use. Retire a field by leaving it in
place rather than deleting it.

Every key named in `warningRules` must exist in `characterSheet`, and
`npcStatblock.hitPointsKey` and `armorKey` must be among `npcStatblock.fields`.
The validator checks all three, because a rule pointing at a field nothing writes
fails silently rather than loudly.

## Ids

Five things are namespaced by the system id and move together: the id itself,
each content module's `id`, the `provides` and `requires` that reference those
ids, each module's `storageNamespace`, and every item id in `items.json`. Do not
change the id of a published system — rooms and characters reference it forever.

## Getting it into the catalogue

The catalogue is [devils-toys-systems](https://github.com/tpskowski/devils-toys-systems).
Open a pull request adding an entry with this repository, the ref to install
from, and the licence. Until it is listed, an admin can install this repository
by name and ref.
