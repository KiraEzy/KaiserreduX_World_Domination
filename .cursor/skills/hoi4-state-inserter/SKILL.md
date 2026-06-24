---
name: hoi4-state-inserter
description: Generates VSCode regex find/replace for inserting additional HOI4 state IDs into owner/original_tag, add_core_of, or "state = X" rules. Use when the user mentions adding states to lines like "x = { owner = { OR = { original_tag = TAG is_subject_of = TAG } } }", "x = { add_core_of = TAG }", or "state = X".
---

# HOI4 State Inserter

## Instructions
When the user asks to add state IDs wherever a specific state line exists, produce a VSCode regex **Find** and **Replace**.

### Required inputs
- Base state ID (the existing line to match, e.g., 429)
- Additional state IDs to insert (e.g., 1360, 1361)
- Rule type: `owner/original_tag`, `add_core_of`, or `state = X`
- Whether tags must match (default: same tag in both fields, owner/original_tag only)

### Output format
Use two labeled blocks:
- **Find:** single regex line
- **Replace:** multiple lines, one per state ID, in the user’s preferred order

### Regex rules
- Escape all literal braces: `\{` and `\}`
- Anchor whole line with `^` and `$`
- Capture tag with `([A-Z]{3})`
- For same-tag requirement, use backreference `\1`
- Replace uses `$1` (and `$2` if different-tag mode)

### Find template (same-tag, owner/original_tag)
```
^{BASE} = \{ owner = \{ OR = \{ original_tag = ([A-Z]{3}) is_subject_of = \1 \} \} \}$
```

### Find template (different tags allowed, owner/original_tag)
```
^{BASE} = \{ owner = \{ OR = \{ original_tag = ([A-Z]{3}) is_subject_of = ([A-Z]{3}) \} \} \}$
```

### Find template (add_core_of)
```
^{BASE} = \{ add_core_of = ([A-Z]{3}) \}$
```

### Find template (state = X)
```
^state = {BASE}$
```

### Replace template (same-tag, owner/original_tag)
For each state ID in the list (including the base, if requested), output:
```
{ID} = { owner = { OR = { original_tag = $1 is_subject_of = $1 } } }
```

### Replace template (different tags allowed, owner/original_tag)
```
{ID} = { owner = { OR = { original_tag = $1 is_subject_of = $2 } } }
```

### Replace template (add_core_of)
```
{ID} = { add_core_of = $1 }
```

### Replace template (state = X)
```
state = {ID}
```

## Example
User: add states 1360, 1361 wherever 429 matches

Find:
```
^429 = \{ owner = \{ OR = \{ original_tag = ([A-Z]{3}) is_subject_of = \1 \} \} \}$
```

Replace:
```
1361 = { owner = { OR = { original_tag = $1 is_subject_of = $1 } } }
1360 = { owner = { OR = { original_tag = $1 is_subject_of = $1 } } }
429 = { owner = { OR = { original_tag = $1 is_subject_of = $1 } } }
```

## Example (add_core_of)
User: add states 1360, 1361 wherever 429 has add_core_of

Find:
```
^429 = \{ add_core_of = ([A-Z]{3}) \}$
```

Replace:
```
1361 = { add_core_of = $1 }
1360 = { add_core_of = $1 }
429 = { add_core_of = $1 }
```

## Example (state = X)
User: add states 1360, 1361 wherever state = 429

Find:
```
^state = 429$
```

Replace:
```
state = 1361
state = 1360
state = 429
```
