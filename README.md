# Pawn — Ascension CoA Edition

**⚠️ This is a modified version of Pawn for [WoW Ascension's Chronicles of Azeroth (CoA)](https://ascension.gg/) server ONLY.**

If you play retail WoW, WoW Classic, or any other private server, **use the [official Pawn addon](https://www.vgermods.com/pawn) instead** — this fork will not behave correctly there, and isn't meant to.

If you play **WoW Ascension CoA**, regular Pawn shows you **wrong stat values and wrong upgrade comparisons**, because of how CoA's server works (see below). This fork exists to fix that.

---

## Why does this exist? (the actual problem)

CoA **rescales items dynamically**, server-side. The stat values you see on an item depend on *how* its tooltip was opened, not just on the item itself:

- Opening a tooltip via a plain item link (`SetHyperlink`) — what regular Pawn always does internally to evaluate items — shows the item's **base, un-rescaled** values.
- Opening a tooltip via the item's actual live location (bag slot, equipped slot, loot, quest reward) shows the **correct, rescaled** values — but only under specific conditions the original Pawn code has no idea about.

The practical effect on unmodified Pawn: item comparisons, upgrade percentages, and stat totals can all be **wrong**, sometimes significantly, because Pawn is reading base values instead of your actual (rescaled) item stats.

On top of that, CoA has **21 fully custom classes** in addition to the original 13 — none of which exist in Pawn's built-in "who can wear what" tables.

## What this fork fixes

- **Correct, rescaled stat values** everywhere Pawn shows them: bags, equipped items, the loot window, and group loot rolls (Need/Greed/Pass).
- **No more tooltip glitches or flicker** when equipping gear or comparing items — the original naive fix for CoA's rescaling caused visible tooltip corruption; this version avoids ever colliding with a tooltip that's actively on screen.
- **Upgrade % comparisons update immediately** after equipping something, instead of requiring you to manually hover your character panel first.
- **Works with all 21 custom classes**, not just the original 13 — equip-eligibility and armor-type checks no longer depend on Pawn's built-in vanilla class tables (which don't apply to any class on this server), and instead read the truth directly from the game's own tooltip.
- **A disabled, half-implemented "loot upgrade advisor" UI element** (a glow box that would've appeared next to loot roll popups) that pointed at functions that were never written — removed so it can't cause a Lua error on login. The loot/roll **tooltip** comparison (hover to compare) is unaffected and works normally.
- General performance pass: background scans only touch the parts of your gear that actually changed, use near-zero overhead when idle, and update your gear cache within a fraction of a second of equipping something — without any perceptible impact on FPS.

## Installation

1. Download the latest release (or clone this repo).
2. Copy the folder into your Ascension `Interface\AddOns` directory, e.g.:
   ```
   C:\Ascension\Launcher\resources\ascension-live\Interface\AddOns\Pawn
   ```
   (If you downloaded a ZIP from GitHub, the extracted folder will be named something like
   `Pawn-Ascension-CoA-main` — rename it to just `Pawn` before placing it in `AddOns`.)
3. Restart the game (or `/reload`) and log in.

## No built-in scales for CoA's custom classes (read this before you start)

Regular Pawn ships with a "suggested starting scale" for each of the 13 original classes and their
specs — that system is driven by internal tables keyed to those 13 classes, which **don't exist on
this server**. Practically, this means:

- When you open Pawn for the first time, it will **not** have anything pre-filled or suggested for
  your class. You need to create your own scale (or import one someone else made) reflecting what
  your spec actually values.
- This is expected, not a bug — see [Status / known limitations](#status--known-limitations) below.

### Importing a scale

The fastest way to get a working scale is to import one someone else has already tuned, via
**Pawn's options → Import Scale** (or `/pawn import`), pasting in a scale string like this one.

Here is the author's own scale for a PvE damage-focused Piety SunCleric, as a
real example of the format and a starting point if that's close to your own spec:

```
( Pawn: v1: "Piety PvE": Intellect=0.65, CritRating=0.85, Mp5=0.25, HasteRating=0.75, SpellPenetration=0.1, Stamina=0.1, SpellPower=1, Spirit=0.25, HitRating=1.3 )
```

This will not be suitable for your class/specialization if it isn't a similar "damage caster" type—consider this
just an example of the format, not a universal answer. Values represent how much 1 point of that stat is
worth relative to 1 point of the stat weighted at `1` (SpellPower, here).

## Status / known limitations

This is an actively-maintained personal fork, not an official Ascension or Pawn project. Current status:

- ✅ Bag items, equipped items, loot window, and group loot roll — tested and working.
- ⚠️ Quest reward tooltips — received the same fix as loot/roll, but not yet extensively tested in-game.
- ⚠️ Equip-eligibility checks now work generically for all 21 custom classes (no longer rely on vanilla class tables), but haven't been individually verified for every single class. **If you play a class other than SunCleric and something looks wrong (an item shown as usable that isn't, or vice versa), please open an issue** — see [Reporting bugs](#reporting-bugs) below.
- ❌ The loot-roll glow box advisor (visual "this is an upgrade" badge on the Need/Greed/Pass popup) is present in the UI files but was never functional even in the base file this was forked from, and is currently disabled rather than implemented.

## Screenshots

<img width="421" height="633" alt="image" src="https://github.com/user-attachments/assets/dcde1481-ba8f-413a-b58e-822425a942e1" />


## Credits & license

This is a modified fork of **[Pawn](https://www.vgermods.com/pawn) by Travis Spomer (Vger-Azjol-Nerub)** — all credit for the original addon, its scale system, and the vast majority of its code goes to him. This fork only adds the CoA-specific compatibility layer described above.

The original Pawn is released under a Creative Commons Attribution-NonCommercial-**NoDerivatives** 3.0 license, which technically does not permit distributing modified versions like this one. This fork is offered in good faith, for free, non-commercially, specifically to help the WoW Ascension CoA community — a use case the original addon was never built for and that the CoA server's mechanics require. If you're the original author and would like this taken down or handled differently, please open an issue.
