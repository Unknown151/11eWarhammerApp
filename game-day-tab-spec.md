# Feature Spec: Game Day Tab — Phase-by-Phase Companion

## Overview

Add a new "Game Day" tab to the existing Leagues of Votann army builder app. This tab provides a phase-by-phase walkthrough of a Warhammer 40K game turn, showing relevant reminders, abilities, and stratagems for each phase. It pulls data from the army list the user has already built in the app.

The tab is **mobile-first** (phone at the game table) and built with **plain HTML/JS/CSS** — no frameworks, no build tools.

---

## Core Concept

The user taps through game phases in order. Each phase displays a focused screen with only the information relevant to that phase. Stratagems are interactive — tapping to use one deducts from the Command Point and/or Yield Point trackers automatically.

---

## Persistent State (visible across all phases)

At the top of the Game Day tab, show persistent resource trackers that remain visible regardless of which phase the user is viewing:

- **Command Points (CP):** Counter with +/- buttons. Starts at 0 (user sets starting CP). Automatically incremented by +1 at the start of each Command Phase if the user taps a "New Turn" or "Next Command Phase" action.
- **Yield Points (YP):** Counter with +/- buttons. Tracks accumulated Yield Points across the game. Show a reminder of how many YP are gained per Command Phase (this depends on the Hearthband detachment rules and army composition — pull from existing data if available, otherwise let the user configure it).
- **Turn Counter:** Simple turn number (1–5) so the app knows whether it's Turn 1 (no Deep Strike) or Turn 2+ (Deep Strike available).

---

## Phase Screens

### 1. Command Phase

**Section: Start of Turn**
- Auto-increment CP by +1 (or prompt user to confirm).
- Auto-increment YP by the per-turn amount.
- Show turn number.

**Section: Battleshock**
- Reminder: "Test Battleshock for any unit below half strength."
- Brief summary of Battleshock rules (roll 2D6 vs Leadership — fail if roll is greater).

**Section: Abilities**
- List any abilities from the user's army list that trigger during the Command Phase.
- Pull these from the existing abilities JSON data — filter by phase relevance.
- Each ability shown as a compact card with: ability name, which unit has it, and the effect.

**Section: Stratagems**
- List Command Phase stratagems available to the Hearthband detachment.
- Each stratagem shown as a tappable card (see Stratagem Interaction below).

---

### 2. Movement Phase

**Section: Deep Strike Reminder (Turn 2+ only)**
- If turn counter is 2 or higher, show a reminder: "Reinforcements (Deep Strike) can arrive this turn."
- List any units in the army that have the Deep Strike ability.
- If turn counter is 1, show: "No reinforcements allowed Turn 1."

**Section: Movement Reminders**
- Any movement-relevant abilities from the army list (e.g., Scout moves, transport embark/disembark rules if applicable).

**Section: Stratagems**
- List Movement Phase stratagems available to the Hearthband detachment.
- Each stratagem shown as a tappable card.

---

### 3. Shooting Phase

**Section: Stratagems**
- List Shooting Phase stratagems available to the Hearthband detachment.
- Each stratagem shown as a tappable card.
- **Interactive usage:** When the user taps "Use" on a stratagem:
  - Deduct the CP cost from the CP tracker.
  - If the stratagem also costs Yield Points, deduct from the YP tracker.
  - Visual confirmation (brief highlight/animation) that the cost was deducted.
  - If the user doesn't have enough CP or YP, show a warning instead of deducting.

---

### 4. Charge Phase

**Section: Stratagems**
- List Charge Phase stratagems available to the Hearthband detachment.
- Each stratagem shown as a tappable card with the same interactive usage as Shooting Phase.

---

### 5. Fight Phase

**Section: Stratagems**
- List Fight Phase stratagems available to the Hearthband detachment.
- Each stratagem shown as a tappable card with the same interactive usage as Shooting Phase.

---

## Stratagem Interaction Pattern

All stratagem cards across all phases should work the same way:

**Display:**
- Stratagem name
- CP cost (and YP cost if applicable)
- Brief description of the effect
- Which units it can target (if specific)

**"Use" Button:**
- Tap to activate. Deducts CP (and YP if applicable) from the persistent trackers.
- If insufficient resources, show a warning (e.g., "Not enough CP") and don't deduct.
- After use, the card could briefly show a "Used" state, but should reset — stratagems can be used multiple turns (the once-per-phase restriction is the player's responsibility to track).

---

## Navigation

- Phase screens are navigated by tapping phase names in a horizontal tab bar or by swiping left/right.
- The phase bar should show all 5 phases and highlight the current one.
- The persistent CP/YP/Turn trackers stay fixed at the top above the phase navigation.

---

## Data Sources

This tab should pull from the existing data in the app:

- **Unit abilities** → filter by phase relevance to show in the correct phase screen.
- **Stratagems** → the Hearthband detachment stratagems (and any universal stratagems if they're in the data). These need to be tagged or filterable by which phase they can be used in.
- **Unit keywords** → to determine which units have Deep Strike, Scout, etc.
- **Army list** → the currently loaded list determines which units and abilities are shown.

If stratagem data isn't in the app yet, it will need to be added. Each stratagem needs at minimum:
- `name`
- `phase` (which phase it's usable in — can be multiple)
- `cpCost` (number)
- `ypCost` (number, 0 if none)
- `description` (effect text)
- `target` (which units or keywords it applies to, if restricted)

---

## Design Notes

- **Mobile-first.** Assume a phone screen in portrait orientation.
- **Fast and glanceable.** The user is looking at this mid-game between dice rolls. Large tap targets, minimal text, clear hierarchy.
- **Match the existing app's visual style.** Use the same color scheme, fonts, and card patterns already in the army builder.
- **No unnecessary chrome.** No animations that slow things down. Quick transitions between phases.
- **Offline-capable.** All data is local — no network requests during a game.

---

## Out of Scope (for now)

- Opponent faction reference (may add later).
- Per-unit wound tracking.
- Game history or match logging.
- Sharing or exporting game state.

These can be added as future iterations once the core phase flow is solid.

---

## Summary

The Game Day tab is a focused, phase-by-phase companion that answers the question: "What do I need to remember right now?" It shows relevant abilities and stratagems for each phase, lets the user tap to use stratagems (auto-deducting CP/YP), and keeps resource trackers visible at all times. It's read-heavy with light interactivity, mobile-first, and built on the data already in the app.
