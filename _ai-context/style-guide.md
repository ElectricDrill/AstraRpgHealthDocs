# Style Guide — Astra RPG Health Documentation

> All documentation is written in **English**. Conversations with the user happen in **Italian**.
> Adhere to this guide strictly. When in doubt, look at existing written sections in `damage.md` as the canonical reference.

---

## Heading Structure

- `#` — page title (one per file)
- `##` — major sections (e.g., `## Damage Sources`, `## Damage Types`, `## Damage Modifiers`)
- `###` — subsections within a major section
- `####` — sub-subsections (e.g., each built-in function under `### Damage Reduction`)
- **No `#####` headings** for inline labels. Use bold inline text instead (e.g., `**Pros**:`, `**Cons**:`, `**Use Cases**:`).

---

## Inline Formatting

| What | Format | Example |
|---|---|---|
| Parameter / field names (inspector) | `**Bold**` | **Defensive Stat**, **Damage Reduction Fn** |
| Class names, method names, asset types | `` `inline code` `` | `DamageTypeSO`, `TakeDamage`, `EntityHealth` |
| Both (named parameter that is also a class) | Bold preferred for inspector fields | **Damage Reduction Fn** (not `DamageReductionFn`) |
| Paths (Unity asset menu paths) | `*Relative path:* \`...\`` | `*Relative path:* \`Dmg Reduction Functions -> Flat Dmg Reduction\`` |

---

## Images

Always precede an image with its Unity relative path on a separate line:

```markdown
*Relative path:* `Dmg Reduction Functions -> Flat Dmg Reduction`  
![Alt text](../../images/AstraRPG/workflows/damage/damage-type/filename.png)
```

Image base path from `MD/workflows/`: `../../images/AstraRPG/workflows/`

**Never reference an image file that has not been confirmed to exist on disk.** Check with glob/view before adding new image references.

---

## Callout Boxes

```markdown
> [!NOTE]
> Text of the note.

> [!WARNING]
> Text of the warning.
```

- `[!NOTE]`: use for optional-but-important clarifications, pairing rules, configuration tips.
- `[!WARNING]`: use for runtime errors that will occur if the user misconfigures something (missing stats, unset fields, etc.).

---

## Numeric Examples

Use a bullet-list of premises followed by a prose result sentence:

```markdown
For example:
- **Damage Type**: Physical damage.
- **Defensive Stat for the Physical Damage Type**: Armor.
- **Damage Reduction Function for the Physical Damage Type**: Flat Dmg Reduction with a factor of 1.

In this case, if the target has Armor equal to 30 and the attacker has Armor Penetration equal to 10, the effective Armor value fed into the Damage Reduction Fn will be 30 (Armor) − 10 (Armor Penetration) × 1 (factor) = 20. [...]
```

Always close examples with: "This example assumes no other damage modifications are active." (or a variant).

---

## Pros / Cons / Use Cases

Use bold inline paragraph headers, **not** extra heading levels:

```markdown
**Use Cases**:
- ...

**Pros**:
- ...

**Cons**:
- ...
```

Order: Use Cases → Pros → Cons (when all three are present).

---

## Cross-References (xref)

Format:
```markdown
[ClassName](xref:Namespace.ClassName)
```

Known namespaces:
- `ElectricDrill.AstraRpgHealth.DamageReductionFunctions` → `DamageReductionFnSO`
- `ElectricDrill.AstraRpgHealth.DefenseReductionFunctions` → `DefenseReductionFnSO`

> **Known typo in existing docs**: some existing xrefs use `ElctricDrill` (missing 'e'). New xrefs should use the correct `ElectricDrill`. Do not "fix" existing ones unless explicitly asked.

---

## Anchor Links (internal)

Format: `[Section Title](#section-anchor)` where the anchor is the heading lowercased with spaces replaced by hyphens.

Example: `[Damage Reduction](#damage-reduction)`, `[Damage Modifiers](#damage-modifiers)`

---

## Tone and Register

- Professional, instructional, third-person neutral.
- Avoid "you should" in favor of "ensure that" or imperative ("Ensure all entities…").
- Explanations move from general concept → parameters → example → use cases/trade-offs.
- When referring to things explained elsewhere in the docs, use a cross-reference rather than re-explaining in full.

---

## Code Fences

Use triple backtick fences for formulas and code snippets. No language tag for math formulas; use `csharp` for C# code.

```
Reduced Damage = Damage × (BaseValue / (BaseValue + Log(1 + DefensiveStat × ScaleFactor)))
```

---

## What NOT to Do

- Do not repeat detailed explanations that already exist in another section — link instead.
- Do not add `#####` headings.
- Do not change xref spelling in existing text unless the user asks.
- Do not add new sections without the user's explicit request.
