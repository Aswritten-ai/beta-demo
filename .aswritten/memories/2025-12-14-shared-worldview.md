Memory 2: Target Use Cases & Market Positioning
Title: ADR-002: Three Pillars - Coding, Executive, Content

Date: 2024-12-23

Context:
Current AI tools are context-starved across different domains:

Coding agents (Claude Code, Cursor) rely on brittle personas ("You are a senior developer") and scattered docs (README, architecture.md)
Executive assistants lack persistent memory of strategic decisions, priorities, worldview
Content generation produces generic output disconnected from actual organizational perspective
Each domain typically uses separate tools with no shared context, leading to strategy-execution gaps.

Decision:
collective memory provides a shared and collaborative worldview that backs AI agents across three primary use cases:

AI-Assisted Development
Coding agents read from compiled snapshot (know product strategy, past decisions)
Developers save ADRs as memories via conversation with AI
Auto-generated documentation (README, architecture.md) stays current
New developers onboard by querying collective memory for decision rationale
Executive Assistance
Strategy discussions become queryable memories
AI agents maintain organizational worldview (goals, values, market position)
Cross-team alignment (sales, product, engineering work from same context)
Meeting notes and decisions persist as structured knowledge
Content Generation
Speeches, presentations, blog posts generated from organizational worldview
Brand voice consistency (narrative architecture encodes style/perspective)
Automatic updates when worldview changes (strategies compile into stories)
Unique Value Proposition:
The same collective memory snapshot backs all three use cases. When sales learns a customer pain point, that memory informs both:

Product roadmap discussions (executive AI sees it)
Feature implementation (coding agent builds solution to actual problem)
Marketing content (blog posts address real customer needs)
This eliminates strategy-execution disconnect - the "$10M mistake" in startups where engineering builds features customers don't want.

Target Customer (Initial Focus):
Developer-first adoption (PLG motion):

Solo developers / small teams using Claude Code, Cursor, Windsurf
Pain: AI agents forget context between sessions, docs go stale
Hook: "Add collective memory to your repo, never write README again"
Expansion: Once developers see value, expand to executive/content use cases
Alternatives Considered:

Single use case focus (e.g., just coding agents) - Rejected: loses unique value of unified worldview
Enterprise-first (sell to Capitol-like companies) - Rejected: slow sales cycles, custom work distraction
Horizontal "AI memory platform" (any use case) - Rejected: too vague, hard to message
Open Questions (See ADR-006 for prompts):

Which use case should launch first? (All three simultaneously, or phase?)
Do we need different pricing for different use cases?
Can solo developers afford this, or do we need team/enterprise pricing from day one?
Consequences:

Market positioning: "Worldview layer for AI agents" (not "semantic memory API")
Go-to-market: Developer tools first, expand to executive/content later
Product roadmap: Must nail coding agent experience before adding complexity
Status: Decided - informing product development priorities

Related Decisions: ADR-001 (Complete Product), ADR-003 (Technical Architecture), ADR-005 (MCP-First Strategy)
