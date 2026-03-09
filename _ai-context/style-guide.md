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

> [!IMPORTANT]
> `*Relative path:*` refers exclusively to the **asset creation path in the Unity context menu** (i.e., `Assets > Create > Astra RPG Health > ...`). Use it only when documenting a standalone `ScriptableObject` type that the user creates via that menu. Do **not** use it for inspector subsections that are part of an existing SO (for example, the **True Damage Options** section of a `DamageType` inspector). In those cases, add an image directly with no path prefix.

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

## Lists

**Bullet lists** — use for unordered items: features, field descriptions, options, non-sequential information.

**Numbered lists** — use exclusively for **sequential steps or ordered procedures** (installation steps, tutorial walkthroughs, calculation breakdowns).

**Punctuation in list items**:
- Single-sentence items: **no trailing period**.
- Multi-sentence or complex items: **period at end of each sentence**.

**Nesting**: avoid nested lists. Use subsections or separate bullet groups instead.

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

For step-by-step calculation breakdowns, use a numbered list of steps with a bold inline final result:

```markdown
**Step-by-step calculation:**
1. Start with base value: 100
2. Apply flat modifiers: 100 + 20 = 120
3. Apply percentage modifiers: 120 × 1.3 = 156

**Final value: 156**
```

Inline math in prose uses the Unicode multiplication sign `×` and standard parentheses, never `*` for multiplication.

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

## Cross-References and Links

| Link type | Format | Example |
|---|---|---|
| Same-file anchor | `[Title](#anchor)` | `[Damage Reduction](#damage-reduction)` |
| Other file in same folder | `[Text](./filename.md)` | `[Package Configuration](./package-configuration.md)` |
| Other file + anchor | `[Text](./filename.md#anchor)` | `[Generic Modifiers](./package-configuration.md#generic-flat-damage-modification-stat)` |
| API type | `[ClassName](xref:Namespace.ClassName)` | `[DamageReductionFnSO](xref:ElectricDrill.AstraRpgHealth.DamageReductionFunctions.DamageReductionFnSO)` |
| External URL | Plain text `https://...` | Not wrapped in markdown |

Anchor IDs are generated from heading text: lowercase, spaces replaced by hyphens.

**Forward references within the same page**: when a section mentions a class, step, or system that is documented in a later section of the same page, always wrap the name in a same-file anchor link pointing to that section. This lets the reader navigate directly to the explanation without losing context. For example, when mentioning `ApplyBarrierStep` before the pipeline section has been reached, write `[ApplyBarrierStep](#damage-calculation-pipeline)` rather than plain inline code.

Known namespaces (xref):
- `ElectricDrill.AstraRpgHealth.DamageReductionFunctions` → `DamageReductionFnSO`
- `ElectricDrill.AstraRpgHealth.DefenseReductionFunctions` → `DefenseReductionFnSO`

> **Known typo in existing docs**: some existing xrefs use `ElctricDrill` (missing 'e'). New xrefs should use the correct `ElectricDrill`. Do not "fix" existing ones unless explicitly asked.

---

## Tone and Register

- Professional, instructional, third-person neutral.
- Avoid "you should" in favor of "ensure that" or the imperative ("Ensure all entities…").
- Explanations move from general concept → parameters → example → use cases/trade-offs.
- When referring to things explained elsewhere in the docs, use a cross-reference rather than re-explaining in full.
- Use **"Let's"** framing for collaborative tutorial walkthroughs: *"Let's see an example of how to define a `GrowthFormula`."*
- Use **parenthetical asides** `(like this)` for inline clarifications and side notes. Never use em-dashes for this purpose.
- Sections end by flowing into the next topic or providing next-step guidance. No formal "Summary" sections.

---

## Callout Box Details

Multi-paragraph callout boxes use a blank `>` line as separator between paragraphs or between a header and a list:

```markdown
> [!NOTE]
> **Header for the note**
>
> - First point
> - Second point
```

Use a bold inline header inside a callout only when the box covers multiple distinct sub-points. Single-topic callouts use plain prose.

---

## Code Fences and Inline Code

Use triple backtick fences for formulas and code snippets. No language tag for math formulas; use `csharp` for C# code.

```
Reduced Damage = Damage × (BaseValue / (BaseValue + Log(1 + DefensiveStat × ScaleFactor)))
```

Introduce code blocks with a colon at the end of the preceding sentence:

```markdown
For example, to get the Physical Attack value at level 5, you can do:
```

Backtick wrapping (`inline code`) is used universally for:
- Class/type names: `EntityHealth`, `DamageTypeSO`
- Method names: `TakeDamage`, `CalculateReducedDamage`
- Variable/field names in code context: `BaseValue`, `ScaleFactor`
- Unity menu paths: `` `Create > Astra RPG` ``, `` `Window > Astra RPG Health > Log Damage Reduction Graph` ``
- Folder/asset paths: `` `Assets/Events/GeneratedEvents` ``

No bold+code combination. Use one or the other.

---

## Special Inline Labels

Two recurring italic-label patterns precede content blocks:

```markdown
*Relative path:* `Dmg Reduction Functions -> Flat Dmg Reduction`
*Keyboard shortcut:* `Ctrl + Alt + S`
```

- Both use `*Italic label:*` followed by a backtick-wrapped value.
- No trailing punctuation after the closing backtick.
- Always on their own line, immediately before the relevant image or section.
- Keyboard shortcuts: capitalize key names, spaces around `+`.

---

## Version Tags

Use this format to indicate that a feature is available from a specific package version onwards:

```markdown
*🏷️ Version X.Y.Z+*
```

Or inline within a sentence: `(🏷️*vX.Y.Z+*)`.

Place on its own line after the heading it applies to, or inline after the feature name.

---

## HTML in Markdown

HTML is used in limited, specific cases:

**Colored field-status markers** (for inspector field legends):
```html
<span style="color:red;">*</span>           <!-- required field -->
<strong style="color:goldenrod;">R</strong>  <!-- replay field -->
<strong style="color:teal;">RO</strong>      <!-- read-only field -->
```

These three markers correspond to field properties that may appear in the custom inspectors of the package:
- **Red <span style="color:red;">*</span>** — the field is **required**: it must be assigned for the component to work correctly
- **Ochre/goldenrod <strong style="color:goldenrod;">R</strong>** — the field is a **replay** field: its value is re-evaluated at runtime under certain conditions
- **Teal <strong style="color:teal;">RO</strong>** — the field is **read-only**: it is displayed for information only and cannot be edited in the inspector

When documenting an inspector section that contains fields carrying one of these markers, reproduce the marker in the documentation using the corresponding HTML tag so the reader can visually match what they see in the Unity editor.

**Icon headings** (for architecture component definitions):
```html
### <img src="../images/icon.png" alt="Icon" width="30" class="icon-background"/> ComponentName
```

**HTML comments** for internal planning notes or TOC outlines visible in source but not rendered:
```html
<!-- This section covers: X, Y, Z -->
```

Avoid HTML for anything achievable with standard markdown.

---

## What NOT to Do

- Do not repeat detailed explanations that already exist in another section — link instead.
- Do not add `#####` headings.
- Do not reference existing image files with incorrect filenames — check `code-reference.md` for confirmed existing images.
- Do not change xref spelling in existing text unless the user asks.
- Do not add new sections without the user's explicit request.
- Do not use `*` for multiplication in prose — use the Unicode `×` character.
