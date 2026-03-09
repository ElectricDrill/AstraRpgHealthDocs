# Documentation Progress — Astra RPG Health

> Update this file whenever a section is completed or its status changes.
> Last updated: 2026-03-09

---

## File Map

All markdown files live in: `C:\Users\emaci\Documents\AstraRpgHealthDocs\DocFx\MD\`

| File | Status | Notes |
|---|---|---|
| `introduction.md` | ✅ Written by user | Contains the "Damage Modifiers vs. Defensive Stat-Based Damage Reduction" section linked from damage.md |
| `package-configuration.md` | ✅ Written by user | Contains Generic Flat/Percentage Damage Modification Stat descriptions |
| `installation-instructions.md` | ✅ Written by user | |
| `requirements.md` | ✅ Written by user | |
| `package-contents.md` | ✅ Written by user | |
| `samples.md` | ✅ Written by user | |
| `advanced-topics.md` | ✅ Written by user | |
| `limitations.md` | ✅ Written by user | |
| `migration-guide.md` | ✅ Written by user | |
| `changelog.md` | ✅ Written by user | |
| `workflows/damage.md` | 🔄 IN PROGRESS | See detailed breakdown below |
| `workflows/entity-health.md` | ❓ Unknown | Not yet reviewed |
| `workflows/healing.md` | ❓ Unknown | Not yet reviewed |
| `workflows/lifesteal.md` | ❓ Unknown | Not yet reviewed |
| `workflows/resurrection.md` | ❓ Unknown | Not yet reviewed |
| `workflows/experience-collection.md` | ❓ Unknown | Not yet reviewed |
| `workflows/health-scaling-component.md` | ❓ Unknown | Not yet reviewed |
| `workflows/package-configuration.md` | ✅ Written by user | |

---

## `workflows/damage.md` — Section-by-Section Status

Lines are approximate; re-check with view tool if needed.

| Section | Heading | Status | Notes |
|---|---|---|---|
| `## Damage Sources` | line ~3 | ✅ Done | Written by user |
| `## Damage Types` | line ~19 | ✅ Done | Intro + three-section list written by user |
| `### Damage Reduction` | line ~34 | ✅ Done | Intro with NOTE + WARNING added (session 1) |
| `#### Flat Dmg Reduction` | line ~50 | ✅ Done | Written by user |
| `#### Percent Dmg Reduction` | line ~75 | ✅ Done | Written by user |
| `#### Log Dmg Reduction` | line ~93 | ✅ Done | Formula + Graph Window explanation added (session 1) |
| `#### Custom Dmg Reduction Functions` | line ~148 | ✅ Done | Written by user |
| `#### Defense Penetration` | line ~152 | ✅ Done | Full section written in session 1 |
| `### Damage Modifiers` (under Damage Types) | line ~188 | ✅ Done | Brief callout + link to full section (session 1) |
| `### True Damage Options` | line ~192 | ✅ Done | Written session 2: Ignore Barrier, Ignore Generic % Modifiers, Ignore Generic Flat Modifiers; IMPORTANT callout on generic-only scope |
| `## Damage Modifiers` | line ~194 | ✅ Done | Full section written in session 1; includes Generic, DamageSource, DamageType, Stacking |
| `## Damage Calculation Pipeline` | line ~224 | ❌ EMPTY | Container section; needs intro paragraph |
| `### PreDamageContext and DamageResolutionContext` | line ~226 | ❌ EMPTY | Key files: PreDamageContext.cs, DamageResolutionContext.cs |
| `### Damage Step` | line ~228 | ❌ EMPTY | Key file: DamageStep.cs abstract class |
| `### Damage Calculation Strategy` | line ~230 | ❌ EMPTY | Key files: DamageCalculationStrategySO.cs; per-entity override |
| `## Dealing Damage to an Entity` | line ~232 | ✅ Done | Written by user; do NOT modify |

---

## Session History

### Session 2 (2026-03-09)
- Wrote `### True Damage Options` section (Ignore Barrier, Ignore Generic % Modifiers, Ignore Generic Flat Modifiers; IMPORTANT callout clarifying generic-only scope)


- Initial familiarization of Framework + Health packages
- Added `#### Defense Penetration` section
- Added `[!NOTE]` and `[!WARNING]` to `### Damage Reduction`
- Added formula + Log Damage Reduction Graph Window explanation to `#### Log Dmg Reduction`
- Wrote full `## Damage Modifiers` section (promoted from `### Damage Modifiers` under Damage Types)
- Added brief callout to `### Damage Modifiers` under Damage Types
- Created this `_ai-context/` archive

---

## Next Steps (Priority Order)

1. **`### True Damage Options`** — shortest, self-contained; source: `DamageTypeSO.cs` fields `IgnoresBarrier`, `IgnoreGenericPercentageDamageModifiers`, `IgnoreGenericFlatDamageModifiers`
2. **`## Damage Calculation Pipeline`** intro paragraph
3. **`### PreDamageContext and DamageResolutionContext`** — source: `PreDamageContext.cs`, `DamageResolutionContext.cs`
4. **`### Damage Step`** — source: `DamageStep.cs`
5. **`### Damage Calculation Strategy`** — source: `DamageCalculationStrategySO.cs`, `EntityHealth.cs`
