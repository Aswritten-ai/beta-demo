# Deck Editing Decisions: From Tony Feedback to Four Pillars Structure

**Date**: 2026-01-08  
**Context**: Iterative deck revision session incorporating Tony's feedback and evolving the pitch structure from scattered benefits to four clear pillars. Created two versions: full advisor/sales deck (~38 slides) and landing page deck (~8 slides).  
**Related**: Memory 01082026-tony-deck-review-feedback.md (Tony's critiques), original deck structures, witness terminology framework

---

## Major Structural Decisions

### **1. Pain-First Structure (Tony's Critical Feedback)**

**Decision:** Lead with human pain, move cost slides to end (for procurement audience)

**Original structure:**
- Led with 5 interlocking costs ($1.6M breakdown)
- Quantified pain upfront
- Cost as primary hook

**Revised structure:**
1. Pain first (human problem): "New people don't know what to do" / "You're constantly interrupted"
2. Solution: The witness concept
3. Proof: How it works, example, pillars
4. Cost last (for procurement): $1.6M breakdown

**Tony's exact words:**
> "You're making the point you're going straight to the cost. But that's not the problem. Cost isn't the problem, actually. It's a way to measure the problem. Right. The problem is people have no fucking clue what to do when they join a company."

> "That's the value. It's not the money. The money is there for the procurement guys. For ROI, sure, you need the slide, but it's for a different audience."

> "See, just not having to stop what I'm doing to tell somebody something all the fucking time. Yeah, I'm in. Yeah, right. I don't care what it costs."

**Implication:** Decision makers care about operational pain. CFO/procurement care about ROI. Different audiences, different slides. Pain leads, cost supports.

---

### **2. Four Pillars Framework (Not Scattered Benefits)**

**Decision:** Replaced 6 "what this solves" slides + 6 role-based slides with 4 pillar slides showing what AI *does* with the witness

**Original approach:**
- 6 separate benefit slides (onboarding, interruptions, knowledge loss, tool freedom, alignment, coding agents)
- 6 role-based slides (CEO, Strategy, Sales, Product, Engineering, Architecture)
- Total: 12 slides of scattered value propositions
- Hard to remember, no clear organizing principle

**New approach:**
- 4 pillars showing what AI does with collective memory:
  1. AI-Assisted Development
  2. Executive Assistance
  3. Content Generation
  4. Organizational Change Tracking (added later in session)
- Each pillar has: what it means, impact metrics, what's delivered
- Total: 4 slides (8 fewer than before)

**Why this works:**
- Clarifies what AI *does* (not just benefits)
- Matches how people think ("I need help with code/strategy/content" not "I need alignment by default")
- Shows breadth (not just onboarding, not just alignment—four major capabilities)
- Easier to remember and explain
- Role-agnostic (dev pillar relevant to agencies, executive pillar relevant to everyone)
- Covers target market (digital agencies do all four)

**Heading changed from:**
"Three Pillars, One Worldview" 

**To:**
"One Worldview, Aligned Output Across Teams"

Emphasizes the output alignment benefit rather than just the structure.

---

### **3. Coding Agents Value Prop Made Explicit**

**Problem identified mid-session:**
Scarlet: "One of the most powerful parts (for technical orgs, and I know we're moving away from that but digital agencies do still do a lot of dev, etc) is that the AI agent working on the code directly now understands the entire business context, so is much less likely to implement features in a way that doesn't solve actual customer need."

**What was happening:**
- Coding agents value was buried in Engineering role slide
- Generic language ("AI explains with full strategic context")
- Not clear this was about *coding agents* having context *while implementing*
- PM burden reduction not emphasized

**What we did:**
1. Created dedicated pillar: "AI-Assisted Development"
2. Made explicit: Cursor, Claude Code, GitHub Copilot query witness while implementing
3. Emphasized: Features align to org direction automatically because agent has context
4. Highlighted PM impact: "PM freed from constant context questions"
5. Quantified: Rework rate 15% → <5%, "random vector" implementations eliminated
6. Updated Product section: "Devs' AI coding agents have full context (no more PM as bottleneck)"
7. Updated Engineering section: "Your coding agent (Cursor, Claude Code) understands business strategy"
8. Added to technical architecture: "Coding agents (Cursor, Claude Code, GitHub Copilot) query while implementing"

**Why this matters:**
- Half of PM's job is keeping devs aligned through tickets, all-hands, PRDs
- This makes alignment automatic
- Relevant for digital agencies (target market)
- Differentiates from ChatGPT Teams/Claude Projects (just chat, not coding agents)
- Demonstrates tool freedom (Cursor for coding, Claude for reasoning—both query same memory)

**Scarlet's insight:**
> "Communicating vision etc with developers is half the point of all-hands, product process, putting context in tickets, etc and we're able to free devs now to focus on working and ask their coworker (the AI) for context when they need it. PMs don't have to constantly worry and field questions."

---

### **4. Organizational Change Tracking (Fourth Pillar Added)**

**Decision:** Added fourth pillar for quantifying organizational change and auto-generating stakeholder reports

**What this adds:**
- Every witnessed change creates queryable organizational history
- "What changed this quarter?" → AI generates report from commit history
- Auto-generate updates for VCs, board, advisors, clients
- Track strategic progress: "Are we executing on our goals?"
- See organizational evolution over time (not just current state)

**Impact metrics:**
- Board deck preparation: days → 15 minutes
- Investor updates: manually compiled → auto-generated from commits
- Strategy reviews: "what did we say we'd do?" → queryable, verifiable
- Client progress reports: manual → derived from witnessed work

**What's tracked:**
- Strategic shifts (market positioning, product direction)
- Execution progress (features shipped, decisions made)
- Knowledge accumulation (what the org has learned)
- Team alignment (how cross-functional context flows)

**Why this matters:**
- Positions product as infrastructure for organizational transparency (not just memory)
- Internal use: Track whether you're executing on strategy
- External use: Auto-generate stakeholder reports (VCs, board, advisors, clients)
- Novel: Most companies can't quantify organizational change—you can with git commit history
- Differentiator: Not just "better memory" but "organizational change as code"

**Scarlet's request:**
> "I'd also like to add something about organizational change as code, ie being able to quantify the way the org is changing, and using that both to automatically generate reports/updates to funders, advisors, clients etc, but also to track internally for strategy, product, etc."

---

### **5. VC/Investor Angle Restored and Strengthened**

**Problem:** When moving to pillars framework, lost the VC value proposition

**Solution:** Added dedicated "For VCs, Board Members, Advisors" section and VC/Investor GTM channel

**VC value proposition (new dedicated section):**

**What VCs get:**
- "What's actually happening in this company?" → queryable, not just reported
- Org change history: "What changed since last board meeting?" → auto-generated
- Strategy execution: "Are they doing what they said?" → verifiable from commits
- Knowledge accumulation visible (org getting smarter over time)
- Board decks auto-generate from witnessed organizational state

**Why this matters for VCs:**
- Portfolio companies report inconsistently (this standardizes it)
- Board prep is auto-generated, not manually compiled
- You see org evolution, not just quarterly snapshots
- Verify execution against stated strategy
- Early warning when companies drift from stated direction

**For portfolio support:**
- Best practices from one company → template for others
- Cross-portfolio learning (anonymized worldview patterns)
- Advisors can query portfolio company context before calls

**New GTM channel: VC/Investor**

**Why VCs/investors care:**
- Portfolio companies report inconsistently → this standardizes it
- Board prep auto-generates (CEOs save days per quarter)
- Verify execution vs stated strategy (no more "trust me" reporting)
- Early warning when companies drift from direction

**The ask:**
- Introduce to portfolio companies struggling with onboarding/alignment
- Pilot with 1-2 portfolio companies
- If successful, offer to entire portfolio

**What VCs get:**
- Better visibility into portfolio (queryable, not just reported)
- Faster board prep across portfolio
- Cross-portfolio learning (anonymized patterns)
- Value-add for portfolio companies (operational improvement)

**Why this opens new GTM path:**
- VCs want better portfolio visibility (this provides it)
- VCs want to add value to portfolio (this is concrete help)
- Portfolio companies get better reporting + operational improvement
- Cross-portfolio network effects (anonymized pattern sharing)
- Addresses "who's the buyer?": Primary = CEO/founder, Champion = Strategy/CTO, Channel = VCs/advisors

---

## Cleaner "How It Works" Flow

**Decision:** Simplified from long numbered list to key phrase slides + 4-step process

**Original:**
Long numbered list with 10 steps including regenerate content, review changes, comment/edit/approve, etc.

**Revised:**
1. Key phrase slides (from cleaner deck version):
   - "AI observes / You decide what to witness / All AIs work from what you've witnessed"
   - "Not every conversation / Just what you decided / Was worth witnessing"
2. Four-step process:
   - AI detects novelty → prompts "Should we witness this?"
   - Draft memory together (collaborative)
   - Commit to the witness (git-native)
   - All AIs can query
3. Example showing the flow (healthcare pivot example)

**Why this works:**
- Clearer escalation: observe (passive) → witness (active)
- Addresses surveillance concerns ("not every conversation")
- Git-parallel vocabulary natural for technical buyers
- Example makes it concrete

---

## Two Deck Versions Created

### **Full Advisor/Sales Deck (~38 slides)**

**Structure:**
1. Pain (6 slides): Hook + 5 pain points (human problems, not costs)
2. Solution concept (3 slides): The witness, observe/witness/query
3. How it works (5 slides): 4-step flow + key phrases
4. Example (1 slide): Healthcare pivot
5. Four Pillars (4 slides): Development, Executive, Content, Change Tracking
6. Case studies (1 slide): Coming soon, pilot offer
7. Technical architecture (1 slide): Git-native, MCP, your control
8. Implementation (3 slides): 30-day pilot, expand, full org
9. Target customer (4 slides): Small teams, agencies/consulting, champion, VCs/board
10. Costs (2 slides): $1.6M breakdown (for procurement), intangibles
11. Pricing (1 slide): Pilot/team/org-wide tiers
12. GTM (2 slides): Advisor-led, VC/investor channel
13. Success/Why (2 slides): 6-month metrics, why this will work
14. CTA (2 slides): Ready to pilot?, Contact

**Use case:** Tony-style advisor presentations, investor meetings, detailed sales conversations

---

### **Landing Page Deck (~8 slides)**

**Structure:**
1. Hook: "You can't transfer AI context"
2. Five costs (one slide): All costs together with $ figures
3. Solution: Collective memory concept
4. How it works: Flow + example on one slide
5. Four pillars: All four on one slide (condensed)
6. Who this is for: Small teams + VC angle
7. Implementation: 30-60 days + pricing + channels
8. CTA: Contact

**Use case:** Quick read, email attachment, landing page content, fast pitch

**Format differences:**
- Multiple concepts per slide (meant for reading, not presenting)
- Denser text but scannable
- All costs together, all pillars together
- Faster read (5 minutes vs 20-minute presentation)

---

## Key Principles Established

### **1. Audience segmentation:**
- Decision makers (CEO, Strategy, CTO): Care about pain (onboarding, interruptions, knowledge loss)
- Procurement/CFO: Care about ROI ($1.6M quantified)
- VCs/Board: Care about transparency, reporting efficiency, portfolio standardization
- End users (developers, PMs): Care about specific workflow improvements

### **2. Product positioning:**
Not just "better AI memory" but:
- Infrastructure for organizational transparency
- Four distinct capabilities (code, strategy, content, change tracking)
- Git-native (version-controlled, reviewable, owned by customer)
- Tool-agnostic (works with any AI)

### **3. Target market refined:**
- Primary: Creative agencies, consulting firms, digital agencies (5-50 people)
- Why: Heavy AI adoption, methodology is product, frequent onboarding, technical enough but not too technical
- Not: Tech startups (will want to build it themselves per Tony), enterprise (too slow)
- Channel: VCs/advisors for portfolio rollout

### **4. Value prop hierarchy:**
1. **Pain relief** (decision maker): Stop interruptions, onboarding crisis solved, knowledge persists
2. **Capabilities** (end user): What AI can do with the witness (code, strategy, content, change tracking)
3. **Cost justification** (procurement): $1.6M quantified savings
4. **Strategic value** (board/VCs): Organizational transparency, execution verification, reporting automation

---

## Files Created

1. **working-deck-v3.md** - Full advisor/sales deck with all revisions applied
2. **landing-page-deck.md** - Short 8-slide version for quick reads
3. **advisor-sales-deck-v2.md** - Earlier version (before four pillars restructure, kept for reference)

All versions incorporate Tony's feedback but working-deck-v3 has the cleanest structure with four pillars framework.

---

## Next Actions (Not Yet Done)

From this session:
1. Get case studies (2 companies, 30-day pilots, permission to share results)
2. Create demo video (real-time workflow: working in Claude → save memory → see sync)
3. Test pitch with Tony (he offered to help sell)
4. Explore VC channel (warm intros to VCs with portfolio in target market)

From Tony's feedback:
1. Find 2 companies for case studies (creative agencies or consulting)
2. Set up Tony as beta user (he offered)
3. Run 30-60 day pilots with agreement to share results
4. Create before/after examples with real customer data (not just collective memory examples)

---

## Summary

The deck evolved through three major revisions in this session:

1. **Applied Tony's feedback:** Pain-first structure, cost moved late, non-tech focus, case studies prioritized
2. **Added coding agents emphasis:** Dedicated pillar, strengthened Product/Engineering sections, made PM burden reduction explicit
3. **Restructured to four pillars:** Replaced scattered benefits with clear capabilities (development, executive, content, change tracking), restored VC angle

The result is two complementary decks: a full advisor/sales presentation (~38 slides) and a landing page quick-read (~8 slides), both organized around "One Worldview, Aligned Output Across Teams" via four distinct AI capabilities.

Key strategic shift: From "better AI memory tool" to "infrastructure for organizational transparency with four capabilities: code alignment, executive assistance, content generation, and change tracking."
