# Client Context System

Modular per-client context layer for the claude-skills library. Implements the Skill Systems pattern — separating reusable skill logic from client-specific context so a single update propagates everywhere.

## The Problem This Solves

Without this:
- Brand voice, ICP, and formatting baked into each skill separately
- Client changes voice → update every affected skill file manually
- New client → duplicate context across every skill they use
- 15 skills for one client = 15 places to maintain

With this:
- Each client has one context folder with four files
- Skills load context at runtime rather than embedding it
- Voice changes once → every skill that uses it updates automatically
- New client → copy `_template/`, fill in four files, done

## Folder Structure

```
_client-context/
├── README.md                  ← this file
├── _template/                 ← copy this for each new client
│   ├── voice.md               ← brand voice and writing style
│   ├── icp.md                 ← ideal customer profile
│   ├── brand.md               ← visual identity
│   └── memory.md              ← session memory and key facts
└── [client-name]/             ← one folder per client
    ├── voice.md
    ├── icp.md
    ├── brand.md
    └── memory.md
```

## How Skills Reference Client Context

Add this to the "Before Starting" section of any SKILL.md:

```markdown
**Load client context first:**
Check for `_client-context/[client-name]/`. If it exists, read `voice.md`,
`icp.md`, and `brand.md` before starting. Apply that context throughout —
tone from voice.md, audience from icp.md, visuals from brand.md.
Only ask for information not already covered.
```

## How Skill Systems Chain Context

For a task like "write a LinkedIn post":

```
linkedin-post skill
  → loads _client-context/[client]/voice.md   (how they sound)
  → loads _client-context/[client]/icp.md     (who it's for)
  → applies its own post formatting logic
  → output: right voice, right audience, every time
```

One update to `voice.md` → every skill that references it updates automatically. No duplicates to chase.

## Adding a New Client

```bash
cp -r _client-context/_template _client-context/[client-name]
# Then fill in voice.md, icp.md, brand.md, memory.md
```

## Memory Pattern

`memory.md` is updated at the end of each significant session — capped at ~2,500 characters. When it fills up, remove the oldest/least relevant entries. This mirrors the Hermes memory injection pattern but keeps you in control of what is preserved.
