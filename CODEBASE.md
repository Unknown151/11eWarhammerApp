# Warhammer 40K Army Builder — Codebase Reference

Generic reference for building a single-faction Warhammer 40K army builder app. All logic, rendering, and styling lives in **`index.html`** (~2500 lines). Unit data, abilities, and enhancements live in **`data/`** as JSON files.

---

## File Structure

```
index.html                          All JS, CSS, and HTML
data/
  abilities.json                    Ability definitions (weapon, core, wargear, leader, detachment)
  enhancements.json                 Enhancement definitions (grouped by detachment)
  army.json                         Reference pre-built army list (used as default on first load)
  units/
    <unit-id>.json                  One file per unit — filename matches the unit's "id" field
    ...
```

---

## Global Variables (index.html)

### Loaded Data
| Variable | Type | Source | Purpose |
|---|---|---|---|
| `abilities` | `Object` | `data/abilities.json` | Keyed ability definitions, looked up by ID |
| `enhancements` | `Object` | `data/enhancements.json` | Keyed enhancement definitions |
| `army` | `Object` | `data/army.json` | Default/reference army list |
| `units` | `Object` | `data/units/*.json` | All unit definitions keyed by unit ID |

### Army State
| Variable | Type | Default | Purpose |
|---|---|---|---|
| `armyList` | `Array<Instance>` | `[]` | Active army list — all added unit instances |
| `nextInstanceId` | `Number` | `1` | Auto-increment ID for new instances |
| `selectedDetachments` | `Array<String>` | `['hearthband']` | Active detachment IDs (limited by `DETACHMENT_POINTS`) |
| `savedLists` | `Array<SavedList>` | `[]` | All saved list slots: `{ name, armyList, nextInstanceId, detachments, pointsLimit }` |
| `currentListSlot` | `Number` | `0` | Index into `savedLists` for the active slot |
| `pointsLimit` | `Number` | `0` | Points cap for the active list (0 = no limit) |

### Buff/Stratagem State
| Variable | Type | Purpose |
|---|---|---|
| `stratagemState` | `Object` | Current toggle state of all stratagems — one boolean key per stratagem |

```javascript
// Example — replace keys with your faction's stratagems:
stratagemState = {
  strat-a: false,   // [Detachment A] stratagem
  strat-b: false,   // [Detachment A] stratagem
  strat-c: false    // [Detachment B] stratagem
}
```

Some stratagems have a two-step activation (e.g. spend a resource before the buff applies). Model with a companion boolean, e.g. `stratagemState.strat-a && stratagemPaid`.

### UI State
| Variable | Type | Purpose |
|---|---|---|
| `characterState` | `Object` | `{ leaderId: bool }` — tracks slain leaders |
| `bodyguardState` | `Object` | `{ leaderId: bool }` — tracks wiped bodyguards |
| `expandedCards` | `Object` | `{ cardId: bool }` — accordion state for units tab |
| `configExpandedCards` | `Object` | `{ instanceId: bool }` — accordion state for builder tab |

### Constants
| Constant | Type | Purpose |
|---|---|---|
| `ALL_UNIT_IDS` | `Array<String>` | All unit IDs to load from JSON files |
| `UNIT_GROUPS` | `Object` | Groups unit IDs for the unit picker by category |
| `DETACHMENTS` | `Object` | Detachment definitions (see Detachment System) |

```javascript
// Example UNIT_GROUPS structure — categories are arbitrary display labels:
const UNIT_GROUPS = {
  'Characters':          ['unit-id-a', 'unit-id-b'],
  'Infantry':            ['unit-id-c', 'unit-id-d'],
  'Vehicles & Mounted':  ['unit-id-e']
}
```

---

## Data Schemas

### Unit JSON (`data/units/<id>.json`)

```jsonc
{
  "id": "unit-id",                    // matches filename, used as key in units{}
  "name": "Unit Display Name",        // display name
  "type": ["Infantry"],               // unit type labels (display only, can be multiple)
  "keywords": ["INFANTRY", "KEYWORD"],
  "isLeader": false,                  // true if this unit can be attached to lead another
  "isEpicHero": false,                // Epic Heroes cannot take enhancements
  "isBattleline": false,              // display flag (optional)
  "points": 100,                      // base points for baseModels count
  "models": 5,                        // default starting model count
  "baseModels": 5,                    // minimum model count
  "maxModels": 10,                    // maximum model count (omit if fixed size)
  "modelBreakdown": "1 Sergeant + 4 Troopers",  // flavour text (optional)

  "stats": {
    "m": "6\"",  "t": 4,  "sv": "3+",  "w": 1,  "ld": "6+",  "oc": 1
  },

  // For units with multiple different statlines (e.g. sergeant + troopers):
  "modelProfiles": [
    { "name": "Sergeant", "count": 1, "stats": { "m": "6\"", "t": 4, "sv": "3+", "w": 2, "ld": "6+", "oc": 1 } },
    { "name": "Trooper",  "count": 4, "stats": { "m": "6\"", "t": 4, "sv": "3+", "w": 1, "ld": "6+", "oc": 1 } }
  ],

  "invulnerableSave": "4+",           // displayed in invuln row (null if none)
  "invulnerableSource": "ability-id", // ability ID that grants it (optional — for popup)
  "transportCapacity": 12,            // TRANSPORT units only (optional)
  "damagedProfile": "1-7",           // degraded profile wound range (optional)

  "weapons": {
    "ranged": [
      {
        "name": "Weapon Name",
        "count": 5,                   // how many models carry this weapon
        "range": "24\"",
        "a": "2",  "bs": "3+",  "s": "4",  "ap": "-1",  "d": "1",
        "abilities": ["rapid-fire"],  // ability IDs shown as weapon ability badges
        "inheritedAbilities": [],     // set by leader bonus system — do not populate in JSON
        "model": "Sergeant",          // which model type has this weapon (optional)
        "weaponGroup": "group-id",    // groups weapons into a choose-one block (optional)
        "weaponGroupLabel": "Choose one profile per shooting phase"
      }
    ],
    "melee": [
      // same structure, no "range" field
      {
        "name": "Close Combat Weapon",
        "count": 5,
        "a": "2",  "ws": "3+",  "s": "4",  "ap": "0",  "d": "1",
        "abilities": [],
        "inheritedAbilities": []
      }
    ]
  },

  "abilities": ["ability-id-a"],           // unit ability IDs (show unit ability badges)
  "coreAbilities": ["feel-no-pain-5"],     // core abilities (shown with blue badge)
  "wargear": ["wargear-ability-id"],       // static wargear, always equipped
  "wargearChoices": ["optional-item-id"], // optional wargear, user picks (up to max defined in JS)
  "wargearExclusiveChoices": ["item-a", "item-b"],  // radio-style — user picks exactly one
  "wargearOptions": ["All models can replace X with Y."],  // display text only (no mechanical effect)
  "canBeLeadBy": ["leader-unit-id"],       // leader unit IDs that can lead this unit

  // ── LEADER-ONLY FIELDS ──
  "isLeader": true,
  "canLead": ["bodyguard-unit-id"],
  "leaderBonus": {
    "text": "While this unit contains a [Leader], ...",  // displayed in leader bonus box
    "grantsAbilities": ["lethal-hits"],                  // ability IDs inherited by led unit's weapons
    "grantsAbilities": [
      { "id": "sustained-hits", "display": "Sustained Hits 1", "abilityAppliesTo": "melee" }
    ],                                                   // OR objects to restrict to weapon type
    "grantsFnp": "feel-no-pain-5",                       // FNP upgrade applied to bodyguard (optional)
    "appliesTo": "ranged"                                // filter inherited abilities to weapon type (optional)
  }
}
```

### Enhancement Entry (`data/enhancements.json`)

```jsonc
{
  "enhancement-id": {
    "name": "Enhancement Name",
    "type": "Enhancement (15 pts)",       // shown as modal type label
    "points": 15,
    "detachment": "detachment-id",        // only available when this detachment is active
    "restriction": "Character Name only", // human-readable (display only)
    "restrictedTo": ["unit-id"],          // unit IDs that can equip this (optional)
    "restrictedToWeapon": "weapon-name",  // case-insensitive weapon name match (optional)
    "description": "<strong>HTML description...</strong>"
  }
}
```

**Rules:**
- Max 3 enhancements active per army at once
- `detachment` prevents display when a different detachment is selected
- `restrictedTo` is checked against `instance.unitId` (standalone) or `instance.leaderId` (attached)
- Enhancements are cleared from instances when switching detachment

### Ability Entry (`data/abilities.json`)

```jsonc
{
  "ability-id": {
    "name": "Ability Name",
    "type": "Weapon Ability",    // or "Core Ability", "Wargear", "Detachment Rule — X", etc.
    "description": "<strong>HTML description...</strong>"
  },
  "lethal-hits": {
    "name": "Lethal Hits",
    "type": "Core Ability",
    "description": "Each unmodified <strong>hit roll of 6</strong> automatically wounds the target."
  }
}
```

### Army Instance (entries in `armyList[]`)

```javascript
{
  instanceId: "inst-1",               // unique key, format "inst-N"
  unitId: "unit-id",                  // key into units{}
  modelCount: 5,                      // current model count (between baseModels and maxModels)
  weapons: {
    "Weapon Name": { equipped: true, count: 5 },
    // key = weapon name string — tracks UI state per weapon
  },
  leaderId: "leader-unit-id" | null,  // attached leader unit ID (null = no leader)
  leaderWargear: ["wargear-id"],      // exclusive wargear choice for the attached leader
  enhancementId: "enh-id" | null,     // selected enhancement ID
  selectedWargear: ["wargear-id"]     // selected optional wargear IDs
}
```

### Detachment Definition (`DETACHMENTS` constant in index.html)

```javascript
const DETACHMENTS = {
  'detachment-id': {
    name: 'Detachment Name',
    dp: 2,                                // Detachment Point cost (1–3)
    category: 'Priority Assets',          // one of DETACHMENT_CATEGORIES (drives banner colour)
    unique: 'hearthband' | null,          // unique group — only one detachment per group may be active
    ruleId: 'detachment-rule-id',         // ID used for ability modal lookup
    ruleType: 'Detachment Rule — Name',   // modal type label
    ruleDescription: '<strong>HTML...</strong>'  // embedded HTML (does NOT use abilities.json)
  }
}
```

### Detachment Points (DP) customizer

- Every army has a fixed budget of `DETACHMENT_POINTS` (3 DP).
- `selectedDetachments` is an **array** of active detachment IDs (was a single `selectedDetachment` string before v0.1.0).
- `toggleDetachment(id)` adds/removes a detachment, enforced by `canAddDetachment(id)`: the DP cost must fit in `dpRemaining()` and its `unique` group must be free (`uniqueGroupTaken`).
- Helpers: `isDetachmentActive(id)`, `dpSpent()`, `dpRemaining()`.
- Enhancements and stratagems are available when their `detachment` is in `selectedDetachments` (`isDetachmentActive`). Removing a detachment drops orphaned enhancements (`clearOrphanedEnhancements`) and resets its quick-buffs (`resetDetachmentBuffs`).
- Persistence: saved lists store a `detachments` array; the URL pack stores `d` as an array. Both migrate the old single-string form on load.
- The header banner shows `dpSpent()/DETACHMENT_POINTS DP` and opens `showDetachmentRules()` (lists every active detachment's rule).

---

## Rendering Pipeline

### Startup
```
loadData()
  → fetch abilities.json, enhancements.json, army.json (parallel)
  → fetch all unit JSONs (parallel, keyed into units{})
  → restore armyList from URL hash or localStorage
  → renderApp()
      → loadDetachment()
      → updateStratagemVisibility()
      → renderUnits()
      → renderConfigurationTab()
```

### Units Tab
```
renderUnits()
  → buildActiveArmy()     — maps armyList instances to { unit, instance, ... }
  → Sort cards (e.g. alphabetically)
  → For each entry:
      if entry has leaderId → renderLedUnitCard(ledUnit)
      else                  → renderSoloUnitCard(entry)
```

**`renderSoloUnitCard(entry)`** — standalone unit card:
- Computes active stratagem flags from `stratagemState`
- Calls `renderWeapons(weapons, inheritedAbilities, appliesTo, ...activeFlags)`
- Calls `renderAbilities(unit, extraWargear)`

**`renderLedUnitCard(ledUnit)`** — leader + bodyguard merged card:
- `leader` = `units[ledUnit.leader]`, `bodyguard` = `units[ledUnit.bodyguard]`
- Builds `leaderInheritedAbilities` and `bodyguardInheritedAbilities` from `leaderBonus.grantsAbilities`
- Renders two `<div class="card-section">` blocks (Leader / Bodyguard)
- Handles death/wipe states (`.leader-dead`, `.bodyguard-wiped`)

### Configuration Tab (Army Builder)
```
renderConfigurationTab()
  → renderDetachmentSelector()   — buttons to pick detachment, rule preview
  → armyList.map(renderArmyInstance)
```

**`renderArmyInstance(instance)`**:
- Model count adjuster (if `maxModels > baseModels`)
- If bodyguard unit with `canBeLeadBy.length > 0`: leader selector dropdown; leader selected → enhancement selector
- If standalone leader and not Epic Hero: enhancement selector
- If `unit.wargearChoices`: wargear checkboxes (up to max)
- If `unit.wargearExclusiveChoices`: radio-style wargear selector
- Weapon equipped toggles and count adjusters

### Weapon Rendering
```
renderWeapons(weapons, inheritedAbilities, appliesTo, ...activeStratFlags)
  → renderWeaponCategory('ranged', ..., relevantFlags)
  → renderWeaponCategory('melee',  ..., relevantMeleeFlags)
      → renderWeapon(weapon, type, ...)
          → build allAbilities[]
          → append inherited abilities (filtered by appliesTo)
          → append stratagem buffs if active flags match
          → render stat grid + ability badges
```

**Weapon ability badge classes:**
- `.wab` — standard weapon ability
- `.wab.inherited` — inherited from leader bonus (green dashed border)
- `.wab.strat-buff` — added by stratagem (gold highlight)

---

## Stratagem / Buff System

Stratagems are toggled via a buff dropdown menu (hamburger icon) or Floating Action Buttons (FABs). Each toggle:
1. Flips `stratagemState[id]`
2. Syncs visual state (`.active` class on dropdown item and FAB)
3. Calls `updateBuffButton()` (updates badge count)
4. Calls `renderUnits()` (re-renders all unit cards with new effects)

### Adding a stratagem effect to weapon rendering

1. Add key to `stratagemState`
2. Compute `xyzActive` boolean in `renderSoloUnitCard` and `renderLedUnitCard`
3. Pass `xyzActive` through `renderWeapons` → `renderWeaponCategory` → `renderWeapon`
4. In `renderWeapon`: push to `allAbilities` with `{ id, display, stratBuff: true }`
5. Add UI toggle (dropdown item + optional FAB)

### FABs (Floating Action Buttons)
- One FAB per frequently-used stratagem toggle
- Shown/hidden per detachment via `updateStratagemVisibility()`
- Each FAB has an ID (`#strat-fab-id`) and a visual indicator when active

---

## Detachment System

```javascript
const DETACHMENTS = { 'detachment-a': {...}, 'detachment-b': {...}, ... }
```

**Adding a new detachment:**
1. Add entry to `DETACHMENTS` with `name`, `ruleId`, `ruleType`, `ruleDescription`
2. Add ability entry in `data/abilities.json` with matching `ruleId`
3. Add enhancements in `data/enhancements.json` with `"detachment": "<id>"`
4. If the detachment has unique buff buttons:
   - Add key to `stratagemState`
   - Add dropdown item with a `data-<detachment>="true"` attribute
   - Add FAB button HTML + CSS if needed
   - Add visibility logic to `updateStratagemVisibility()`
   - Add reset logic to `selectDetachment()`
   - Add FAB toggle function
   - Thread a new `xyzActive` flag through `renderWeapons` if it affects weapon display

**`selectDetachment(id)`** does:
- Sets `selectedDetachment`
- Saves to localStorage
- Resets detachment-specific stratagems
- Clears enhancements that don't belong to the new detachment
- Re-renders everything

---

## Enhancement System

### Enhancements vs Upgrades
- **Normal enhancements** — only on a non-Epic **CHARACTER** unit; max **one** of each in the army; each counts toward the army limit (`MAX_ENHANCEMENTS = 4`).
- **Upgrades** (`"upgrade": true` on the enhancement) — may go on **any non-Epic-Hero unit** (including non-characters); up to **three** of the same Upgrade in the army; only the **first** copy of each distinct Upgrade counts toward the limit (extra copies are free of the limit but still cost points each).
- **Epic Hero** units can take neither.

### Filter Logic (`renderInstanceEnhancementSelector(instance)`)
The selector figures out internally:
- `normalCharId` — the non-Epic character eligible for normal enhancements (the unit itself if a non-Epic leader, else a non-Epic attached leader; otherwise none).
- `upgradesAllowed` — true for any non-Epic-Hero unit.

An option is shown if `enh.detachment` is active and: for Upgrades, fewer than 3 copies are used elsewhere; for normal enhancements, a `normalChar` exists, it isn't already used, and `restrictedTo`/`restrictedToWeapon` pass. Returns `''` when nothing is available and none is selected (keeps non-eligible units uncluttered).

Helpers: `getEnhancementUsage(excludeId)` → `{enhId: countElsewhere}`; `countArmyEnhancements(excludeId, extraEnhId)` → army limit count (distinct Upgrades count once). Per-option disable = selecting it would push `countArmyEnhancements` over `MAX_ENHANCEMENTS`.

### Enhancement Card UI
Cards rendered as `.enh-option` divs in `.enh-option-list`:
- `.enh-option.active` — currently selected
- `.enh-option.disabled` — selecting it would exceed the limit
- `.enh-option-tag` — the "Upgrade" badge on upgrade options

---

## Leader–Bodyguard Attachment

### Display (Units Tab)
Attachment renders as a single merged card (`renderLedUnitCard`) when an instance has `leaderId` set and the leader is listed in `bodyguard.canBeLeadBy`.

`buildActiveArmy()` groups instances into `ledUnits` (pairs) and `soloUnits`.

### Leader Bonus
`leaderBonus.grantsAbilities` is injected into the bodyguard's weapons as `inheritedAbilities`. Rendered as `.wab.inherited` badges.

- `grantsFnp` — overrides bodyguard FNP ability
- `appliesTo: "ranged"|"melee"` — filters which weapon types receive the inherited abilities

### Exclusive Wargear on Attached Leaders
When a leader has `wargearExclusiveChoices`, the bodyguard instance stores the chosen ID in `leaderWargear[]`. The choice is applied in `renderLedUnitCard` via a `displayLeader` shallow-copy before rendering. The selector is shown in `renderArmyInstance` under the leader assignment.

### Death / Wipe States

| Action | State Key | CSS Class | Effect |
|---|---|---|---|
| Toggle leader dead | `characterState[leaderId]` | `.leader-dead` | "SLAIN" overlay; inherited bonuses hidden |
| Toggle bodyguard wipe | `bodyguardState[leaderId]` | `.bodyguard-wiped` | "WIPED" overlay; leader loses "while leading" bonuses |

**Wiring a new led unit:**
- Add `"canBeLeadBy": ["leader-id"]` to the bodyguard unit JSON
- Add `"isLeader": true` and `"canLead": ["bodyguard-id"]` to the leader unit JSON

---

## Modal Popup System

Clicking any element with `[data-ab="<id>"]` calls `showModal(id)`.

Lookup order:
1. `abilities[id]`
2. `enhancements[id]`
3. `DETACHMENTS` — matches on `ruleId` field

Displays: `name` as title, `type` as subtitle, `description` as HTML body.

---

## Points Calculation

**`calculateInstancePoints(instance)`**:
- Base: `getUnitSizePoints(unit, modelCount, copyIndex)` — looks up the exact cost for the chosen model count from `unit.costing.tiers`
- Add: enhancement points (`enhancements[enhancementId].points`)
- Add: leader points (if `leaderId` is set, add leader unit's `points`)

**`unit.costing`** (escalating / per-size pricing):
```json
"costing": {
  "escalationAfter": 2,                 // optional; copies beyond this index pay the last tier
  "tiers": [
    [{ "models": 3, "points": 80 }, { "models": 6, "points": 160 }],   // 1st–2nd copy
    [{ "models": 3, "points": 90 }, { "models": 6, "points": 170 }]    // 3rd+ copy
  ]
}
```
- Each tier lists explicit `{ models, points }` sizes (no doubling — the 10-model price can differ from 2× the 5-model price).
- `getUnitCopyIndex(instance)` returns the 1-based position of the instance among same-datasheet instances (army-list order). When it exceeds `escalationAfter`, the last tier is used (e.g. Hekaton uses `escalationAfter: 1` → the 2nd+ copy costs more).
- Units without `costing` fall back to legacy `unit.points` (doubled when `modelCount > baseModels`).
- The model-count stepper (`adjustModelCount`) snaps between the sizes defined in the first tier; it's hidden for single-size units.

**`updateConfigPoints()`**: sums all instances, updates the points display, and (if `pointsLimit > 0`) colours the total green/amber/red, shows `/ XXXX` label, and fills a progress bar.

**List management functions:** `switchToList(slot)`, `addNewList()`, `deleteCurrentList()`, `renameCurrentList(name)`, `setPointsLimit(value)`.

---

## localStorage Keys

Use a consistent faction-specific prefix for all keys. Example pattern:

| Key | Type | Default |
|---|---|---|
| `[faction]-theme` | `'light'` \| `'dark'` | `'dark'` |
| `[faction]-detachment` | detachment ID string | first detachment ID |
| `[faction]-saved-lists` | JSON `{ lists: SavedList[], slot: number }` | one empty slot |

> Replace `[faction]` with a short identifier for your faction (e.g. `space-wolves`, `tau`, `necrons`).

---

## URL Sharing

### Encode (`shareArmyUrl` — async)
1. `packArmy()` builds a compact object — short keys, only non-default values:
   - `d` — detachment ID
   - `p` — pointsLimit (omitted if 0)
   - `l` — array of packed instances: `u` (unitId), `m` (modelCount, omitted if default), `l` (leaderId), `e` (enhancementId), `g` (selectedWargear), `W` (weapon diffs vs. unit defaults)
2. `deflateToBase64()` compresses with browser-native `CompressionStream('deflate-raw')` → URL-safe base64
3. URL hash = `'2'` + compressed string (version prefix distinguishes from legacy formats)
4. Copies to clipboard

### Decode (`loadArmyFromUrl` — async)
- Hash starts with `'2'` → new format: decompress, `unpackArmy()` rebuilds full instances filling defaults from unit JSON, regenerates `instanceId`s
- Saves to localStorage, clears URL hash after load

`normalizeInstance(instance)` is called on every loaded instance to fill missing fields with defaults — important for backwards compatibility when new fields are added.

---

## CSS Conventions

### Unit Cards
| Class | Meaning |
|---|---|
| `.unit-card` | Standalone unit card |
| `.unit-card.attached` | Leader+bodyguard merged card (gold border) |
| `.unit-card.leader-dead` | Leader marked slain |
| `.unit-card.collapsed` | Card body hidden (accordion) |

### Ability Badges
| Class | Meaning |
|---|---|
| `.ab` | Unit ability badge |
| `.ab.core` | Core ability (blue) |
| `.ab.wg` | Wargear ability (gold) |
| `.ab.inherited` | Inherited from leader (green dashed) |
| `.wab` | Weapon ability badge |
| `.wab.strat-buff` | Added by stratagem (gold highlight) |

### Defence Row
| Class | Meaning |
|---|---|
| `.invuln-row` | Invulnerable save display (gold) |
| `.invuln-row.inherited` | Invuln granted by leader (green dashed) |
| `.invuln-row.fnp-only` | FNP without invuln |

### Configuration Tab
| Class | Meaning |
|---|---|
| `.enh-option` | Enhancement card |
| `.enh-option.active` | Selected enhancement |
| `.enh-option.disabled` | Greyed out (max reached) |
| `.leader-select` | Leader assignment dropdown |
| `.config-label` | Section label in instance card |
| `.wargear-exclusive-option` | Radio-style wargear choice |
| `.wargear-exclusive-option.selected` | Currently chosen exclusive wargear |

### Buff Dropdown
| Class | Meaning |
|---|---|
| `.buff-dropdown-item` | Single stratagem row |
| `.buff-dropdown-item.active` | Stratagem enabled |
| `[data-<detachment>]` | Hidden when not that detachment — add one attribute per detachment |

---

## Common Patterns for Adding Features

### Add a new unit
1. Create `data/units/<id>.json` following the unit schema above
2. Add the ID to `ALL_UNIT_IDS` in index.html
3. Add to `UNIT_GROUPS` in the appropriate category
4. If it can be led: add `"canBeLeadBy": ["leader-id"]`
5. If it's a leader: add `"isLeader": true`, `"canLead": ["bodyguard-id"]`, `"leaderBonus": {...}`

### Add a new ability
1. Add entry to `data/abilities.json`
2. Reference by ID in a unit's `"abilities"`, `"coreAbilities"`, or a weapon's `"abilities"`

### Add a new enhancement
1. Add entry to `data/enhancements.json` with `"detachment": "<id>"`
2. Add `"restrictedTo": ["unit-id"]` if only certain characters can take it

### Add a new detachment
See the **Detachment System** section above.

### Add a new weapon buff (stratagem effect)
See the **Adding a stratagem effect to weapon rendering** section above.

### Add exclusive wargear to a unit or leader
1. Add `"wargearExclusiveChoices": ["option-a", "option-b"]` to the unit JSON
2. Add ability entries for each option in `data/abilities.json`
3. Handle the choice in `renderSoloUnitCard` / `renderLedUnitCard` via a `displayUnit` shallow-copy
4. If on a leader: store in `instance.leaderWargear[]`, handle in `renderLedUnitCard` via `displayLeader` shallow-copy
5. Add case to `packArmy()` / `unpackArmy()` to serialize the choice

### Add scalable model counts (e.g. 5 or 10 models)
1. Set `"baseModels": 5, "maxModels": 10` in the unit JSON
2. Add a `costing` block with the explicit per-size prices (see **Points Calculation**). The stepper snaps between the sizes listed in `tiers[0]`.
3. For escalating costs (a copy beyond the Nth costs more), add `"escalationAfter": N` and a second tier.
4. Weapon counts scale automatically if set to match `baseModels` in the JSON

### Hearthkyn Warriors guided wargear
Hearthkyn Warriors don't use the free-form weapon selector. Their builder card renders
`renderHearthkynWargearPanel(instance)` — steppers/dropdowns modelling the datasheet's
wargear options (ion blasters 0–9, two no-duplicate special-weapon slots, 0–2 plasma
knives, Theyn ranged/melee choices). The slots live in `instance.hkWargear` (packed as
`hk` in share URLs) and `deriveHearthkynWeapons()` recomputes `instance.weapons` AND
`instance.splitWeapons` from them on every change/render (`ensureHkWargear` also
migrates old free-form loadouts). When the Sagitaur split is active, extra "Split
Assignment" controls choose which half gets the Theyn, each special, and how many ion
blasters/knives go to half A — all clamped to each half's warrior count.
