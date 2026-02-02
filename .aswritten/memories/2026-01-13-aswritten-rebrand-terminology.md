# aswritten.ai Branding & Terminology

**Date**: 2025-01-30  
**Context**: Strategic rebrand from "storyBASE/witness" terminology to "aswritten.ai" product identity

---

## Decision

We are rebranding the product as **aswritten.ai** and establishing new canonical terminology that shifts framing from human-centric documentation to AI-centric collective memory.

**Product name**: aswritten.ai (not "storyBASE" or "witness" as product names)  
**Technical layer**: Fully rebrand storyBASE → aswritten in all tools, APIs, and infrastructure

---

## Core Terminology

### Actions (what AIs do)
- **"remember as written"** - recall from collective memory
- **"draft a memory"** - observe conversation/decision and record it
- **"write a memory"** / **"write/commit to collective memory"** / **"save/commit as written"** - save to shared memory repository
- **"reference collective memory"** / **"read/reference as written"** - query for context
- **"pull from collective memory"** - sync with latest
- **"compile"** - generate narrative from collective memory

### Actions (what humans do)
- **"ask my AI"** - query for context
- **"review the PR"** - approve changes to collective memory
- **"connect to collective memory"** - setup/onboarding

### System/State
- **"collective memory"** - collaborative, append-only ledger stored in `.aswritten/`
- **"memory as written"** - persistence layer all AIs contribute to/query from
- **"the narrative"** - collective memory compiled into worldview at a point in time
- **"the present"** - current compiled snapshot
- **"as written"** / **"in collective memory"** - saved, available to all AIs

### Technical
- **`.aswritten/`** - dotfile directory in any repo (like `.git/`) containing collective memory
- Multiple repos per org, composed/combined in future

---

## Canonical Workflow

Human has experience → AI notices something important → AI prompts: "Should I draft a memory?" → Human and AI **draft memory together** → Human approves → AI **writes to collective memory** (`.aswritten/` in repo) → System compiles **the present narrative** → All connected AIs **remember as written**

**Key principle**: Writing memories is **intentional**, not automatic. AI prompts when it notices novelty against the present narrative; human approves.

---

## The Pitch

We're building **collective memory for AI** that creates day one-through-exit institutional knowledge for new hires, clients, and investors.

When your AI notices something important in your work, you **draft a memory together**, write it to collective memory repo, review with your team, and all connected AIs **remember, as written**.

Collective memory means:
- Day 1 productivity for new hires
- Zero knowledge loss when people leave
- Shared understanding without all-hands
- Coding agents that understand business strategy while implementing
- Auto-generated content from decks and docs to board updates about worldview changes

---

## Positioning Shift

**From**: Human-centric ("You need to document things for AI")  
**To**: AI-centric ("Your AI drafts memories alongside you")

**From**: AI as passive tool that needs feeding  
**To**: AI as coworker that observes and remembers

**From**: "storyBASE" (technical infrastructure framing)  
**To**: "aswritten.ai" (product identity emphasizing persistence & collaboration)

---

## Value Props (Reframed)

### Onboarding Problem
- **Before**: New hire's AI starts blind, 3-6 month ramp
- **After**: Connected to collective memory, day 1 productivity

### Knowledge Loss Problem
- **Before**: When people leave, their AI's context leaves with them
- **After**: Written to collective memory, persists in `.aswritten/`

### Fragmentation Problem
- **Before**: Sales AI, Product AI, Engineering AI work from different context
- **After**: All AIs work from same narrative, written to collective memory

---

## Rationale

1. **"Witness" too metaphysical**: Confused users expecting passive observation vs. intentional memory creation
2. **"storyBASE" too technical**: Implied infrastructure/dev tool rather than complete product
3. **"aswritten.ai" clearer value prop**: Emphasizes persistence, intentionality, collaborative memory creation
4. **Dotfile pattern familiar**: `.aswritten/` maps to developer mental model (like `.git/`)
5. **"Collective memory" resonates**: Solves onboarding/knowledge loss in natural language

---

## Migration Plan

- Technical rebrand: storyBASE → aswritten in all tools, MCP server, APIs
- Documentation update: All public-facing materials use new terminology
- Repo structure: `.aswritten/` as standard dotfile location
- Tool naming: `aswritten/compile`, `aswritten/remember`, etc.
- Preserve "story" terminology for templates (`.story` files) - already established, works well

---

## Natural Conversation Examples

**Real-time**: "My AI noticed this architectural decision. Should I draft a memory?" → "Yes" → "Written to collective memory"

**Cross-role**: "My AI drafted a memory about client patterns" → "Is it in collective memory?" → "Just committed, your AI should see it"

**Onboarding**: "Your AI is connected to collective memory now" → "So it knows everything?" → "Everything the team remembers as written"
