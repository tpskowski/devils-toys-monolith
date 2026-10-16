# Work items

Moved from the tabletop roadmap and checked against this repository on September 30, 2026. This repository contains declarative system data; application changes belong in `../the-devils-toys`.

## Dice rules reference

- [ ] Fix the dice roller's rules-reference query. `system.json` declares `rollRulesQuery: "Tests"`, but `rules/Monolith.md` has `## SAVES` and no matching Tests section. The tabletop filters the book using this query, so the current value produces an empty reference. Use the book's actual heading and verify the dice modal opens the intended save rules after installation. Validate the system and bump the marker version when publishing the correction.

## Starship declaration cleanup

- [ ] Review Monolith's starship declaration as a shared party asset and move it to the explicit `groupPage.groupAssets` representation where appropriate. It currently uses the legacy `groupPage.starshipSheet`. Keep the stored kind `starship`, all existing field/list keys, hull sizes, base values, parts, and hold capacities stable so existing ships survive the change. The tabletop already reads explicit asset definitions and falls back to the legacy declaration; migration alone does not add generic base or transport views.
- [ ] Verify Monolith's requirements against the tabletop's pending generic group-asset views. The remaining generic UI work is in [tabletop issue #45](https://github.com/tpskowski/the-devils-toys/issues/45): `GroupPage` and its tab selection still only offer the starship kind. Declare only asset kinds the book needs; do not put executable behavior in this repository. Any declaration requiring new application behavior must wait for that behavior to ship, or stay in a draft PR.

## Starship printing acceptance

- [ ] When the tabletop adds group-asset printing, verify every Monolith hull prints its full hold capacity, including empty holds and bulky-part continuations. Verify hull stats, scores, and installed parts against this declaration. The print action and layout belong in [tabletop issue #46](https://github.com/tpskowski/the-devils-toys/issues/46).

## Already available

Removed from the active tabletop roadmap after checking the implementation: starships start expanded unless the viewer has saved a different preference; expansion is stored per viewer and room; ship images can be uploaded, replaced, and removed. These need no new system declaration.
