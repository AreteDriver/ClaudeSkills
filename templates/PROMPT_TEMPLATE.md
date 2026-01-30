# Prompt Template

Use this format for documenting prompts in this library.

---

## [Prompt Name]

**Pattern:** extraction | transformation | reasoning | orchestration | generation  
**Domain:** enterprise | development | operations | general  
**Complexity:** simple | moderate | complex  
**Tool Requirements:** none | web search | code execution | specific APIs

### Intent

*What problem does this prompt solve? When should you reach for it?*

[2-3 sentences describing the use case]

### Prompt

```
[The actual prompt text]
```

### Variables

| Variable | Type | Required | Description |
|----------|------|----------|-------------|
| `{variable_name}` | string/list/object | yes/no | What this represents |

### Model Assumptions

- **Best with:** Claude Sonnet/Opus, GPT-4, etc.
- **Context needed:** Minimum context window size, prior conversation
- **Tool dependencies:** List any required tools or integrations

### Example Input

```
[A concrete example of what you'd send]
```

### Example Output

```
[What good output looks like]
```

### Failure Modes

| Symptom | Cause | Fix |
|---------|-------|-----|
| [What goes wrong] | [Why it happens] | [How to address] |

### Iteration Notes

**v1 (date):** Initial version — [what you learned]  
**v2 (date):** [Change made] — [improvement observed]

### Related Prompts

- `[other-prompt-name]` — When to use instead
- `[another-prompt]` — Often used together

---

## Template Usage Tips

1. **Intent first** — If you can't explain why this prompt exists, it probably shouldn't
2. **Show don't tell** — Concrete examples > abstract descriptions
3. **Document failures** — The failure modes section is often more valuable than the prompt itself
4. **Track iterations** — Your v1 → v2 → v3 journey teaches more than the final version
