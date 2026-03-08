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

**Be proactive about adding images.** Study the existing documentation to understand where screenshots are used (inspector views, custom windows, example configurations) and apply the same pattern in new sections. When a screenshot would naturally belong in a section — for example, to show a new inspector section, a custom editor window, or a configuration example — add the image reference using the conventional format even if the file does not yet exist on disk. The user will provide the missing screenshots and place them in the correct folder.

When referencing an image that does not yet exist, add a comment immediately after to signal that the file is missing:

```markdown
*Relative path:* `True Damage Options`  
![True Damage Options](../../images/AstraRPG/workflows/damage/damage-type/true-damage-options.png)
<!-- IMAGE MISSING: true-damage-options.png — screenshot of the True Damage Options inspector section -->
```

This way the user can search for `IMAGE MISSING` in the docs to find all placeholders that still need a screenshot.

---

## Callout Boxes

DocFx supports five alert types. Use them as follows:

```markdown
> [!NOTE]
> Text of the note.

> [!TIP]
> Text of the tip.

> [!IMPORTANT]
> Text of the important notice.

> [!CAUTION]
> Text of the caution.

> [!WARNING]
> Text of the warning.
```

| Type | When to use |
|---|---|
| `[!NOTE]` | Optional-but-important clarifications, pairing rules, configuration tips, nuances the user should be aware of. |
| `[!TIP]` | Practical suggestions, workflow shortcuts, or recommendations that improve the development experience but are not required. |
| `[!IMPORTANT]` | Information that is critical for correct behavior and must not be overlooked — stronger than NOTE, but not an error condition. |
| `[!CAUTION]` | Potential pitfalls or design choices that may lead to unintended behavior if not carefully considered. |
| `[!WARNING]` | Runtime errors that **will** occur if the user misconfigures something (missing stats, unset fields, incompatible settings, etc.). |

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
