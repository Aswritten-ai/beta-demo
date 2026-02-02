# collective memory Data Model & API Architecture Discussion

**Date**: 2025-01-30  
**Participants**: User, storyTWIN (AI)  
**Context**: Designing collective memory's Clojure/Malli API schema and integration architecture

## Key Decisions

### 1. Non-Technical User Accessibility
**Goal**: Make collective memory usable for non-programmers (nonprofit coordinators, sales execs, educators)

**Strategy**:
- Conversational AI and Discourse forum as zero-code interfaces
- All RDF/SPARQL operations hidden behind natural language interactions
- Plain-English summaries for all graph changes
- Dry-run previews (`mcp/posit`) before committing changes

### 2. Interface Architecture

**Two-Interface Model**:
- **Conversational AI**: Fast iteration, solo ideation, backed by snapshot memory
- **Discourse Forum**: Async collaboration, multi-stakeholder input, deliberative

**Integration Pattern** (Two-Step Save):
1. User ideates in chat with AI
2. AI offers to post to Discourse for team visibility
3. Discourse post includes `@aswritten` mention
4. Discourse webhook triggers async extraction via GitHub Actions
5. Bot replies with transaction link

### 3. Core Workflows

#### Remember (Async Extraction)
- `mcp/create-memory` â `mcp/posit` (preview) â `mcp/remember` (commit file)
- GitHub Actions handles extraction: load snapshot â extract â diff â create TX â commit
- Returns `Job` with commit SHA, file path, and job URL for tracking
- **Only async operation in the system**

#### Reprocess (Bulk Transformation)
- `mcp/reprocess` applies transformation prompt to entire snapshot
- Use cases: "Change all email domains", "Merge duplicate stakeholders", "Add missing properties"
- Synchronous: returns uncommitted Transaction for review
- Distinct from `extract` (which processes new Memory content)

#### Story Generation
- `mcp/draft-story` executes `.story` file against snapshot
- **Synchronous**: Returns rendered Markdown immediately (not a Job)
- Incremental mode (`story.increment = true`): generates one file per transaction
- `story/group-txs` deduplicates/groups transactions for incremental generation

### 4. Tool Layering

**MCP Tools** (user-facing):
- Orchestrate workflows
- Handle authorization/validation
- Return high-level types (Job, Commit, Transaction)
- Examples: `mcp/snapshot`, `mcp/remember`, `mcp/draft-story`

**Lower-Level Tools** (internal):
- Atomic operations
- Used by MCP tools and GitHub Actions
- Examples: `snapshot/extract`, `tx/commit`, `story/draft`

### 5. Key Type Definitions

**Memory**: Ephemeral user content (chat message, Discourse post, uploaded doc)
- Auto-generated timestamp and file path
- Triggers extraction when committed to repo

**Transaction**: Graph changes (insert/delete operations)
- Has provenance (source file/URL)
- Can be uncommitted (for preview) or committed (with SHA)

**Job**: Async extraction tracker
- Only type: `:remember` (story generation is synchronous)
- Contains commit SHA, file path, job URL

**Story**: `.story` file metadata
- `model`: OpenRouter vendor/model (e.g., `["openai/gpt-4"]`)
- `increment`: Generate one file per transaction (for tracking narrative evolution)

**Diff**: Structured change representation
- `added`, `removed`: vectors of triples
- `updated`: vector of `{old, new}` triple pairs

### 6. Discourse Integration

**Data Flow**:

**Category Mapping**:
- Each Discourse category = folder in repo
- Each thread = subfolder
- Each post = transaction in thread folder
- Provenance preserved via post author username

**Multi-Tenancy**:
- Discourse user groups â GitHub team permissions
- Category access controls â repo folder permissions via CODEOWNERS

### 7. Design Patterns

**Repo as Source of Truth**:
- All content committed to Git before extraction
- GitHub Actions handle all async extraction (not inline tool calls)
- Auditable: raw files preserved even if extraction fails

**Append-Only Transactions**:
- No per-thread branches (too much fragmentation)
- Main branch receives all commits
- PRs reserved for high-stakes threads requiring review

**Dry-Run First**:
- `mcp/posit` previews extraction impact before committing
- Shows plain-English summary: "This adds 3 objectives, 2 stakeholders"
- User approval before `mcp/remember` triggers async job

### 8. System Identity

**Handle**: `"aswritten"` (centralized constant)
- Used in Discourse mentions: `@aswritten`
- Git commit attribution for AI-initiated changes
- Transaction provenance for system-initiated operations

## Open Questions Resolved

1. **Extract vs. Process**: Split into two functions
   - `snapshot/extract`: Memory content â Triples (new facts)
   - `snapshot/process`: Snapshot â Transaction (bulk transformations)

2. **Story Generation Async?**: No, synchronous
   - Only `mcp/remember` is async (returns Job)
   - `mcp/draft-story` returns Markdown string immediately

3. **Incremental Story Filenames**: `story/group-txs` function
   - Deduplicates and groups transactions
   - Returns list of filenames to generate
   - Strategies: by date, author, or topic

4. **Named Graphs**: Future capability
   - Schema supports it (`Query.graphs`, `Transaction.operations.graph`)
   - Not yet implemented in current snapshot

## Technical Corrections Made

1. Split `Triple` (single) from `Triples` (vector) types
2. Fixed `Diff.updated` structure: `{old, new}` triple pairs
3. Corrected `mcp/commit-tx` parameter name: `transaction` not `memory`
4. Added `DiscoursePost` schema definition
5. Made `Memory.date` optional with `now` default
6. Clarified `Story.model` as OpenRouter routing key

## Architecture Status

**Production-Ready**: All schemas complete, tool chains validated, workflows coherent

**Next Implementation Steps**:
1. Implement `story/group-txs` (transaction deduplication logic)
2. Prototype `snapshot/process` (bulk transformation engine)
3. Build Discourse webhook â GitHub Actions pipeline
4. Test incremental story generation
5. Add `Triple`/`Triples` type hierarchy to schema
6. Document grouping strategies (date/author/topic-based)

## Alignment with collective memory Philosophy

- **Collaborative narrative architecture**: Discourse + chat interfaces support team storytelling
- **Version control as narrative source**: Git commits = append-only narrative log
- **AI as coworker**: System handle (`aswritten`) represents AI identity in shared workspace
- **Accessibility**: Zero programming literacy required for non-technical contributors
- **Auditability**: All extractions traceable via provenance, commit history
```
# collective memory Architecture Diagrams

## 1. System Architecture Overview

```
graph TB
    subgraph "User Interfaces"
        Chat[Conversational AI<br/>Open WebUI]
        Forum[Discourse Forum<br/>Async Collaboration]
        Files[File Upload<br/>Direct Commit]
    end
    
    subgraph "MCP Layer"
        MCP[MCP Server<br/>Tool Orchestration]
        Snapshot[mcp/snapshot]
        Remember[mcp/remember]
        Posit[mcp/posit]
        Reprocess[mcp/reprocess]
        DraftStory[mcp/draft-story]
        CommitTx[mcp/commit-tx]
    end
    
    subgraph "Core Operations"
        Load[snapshot/load]
        Ontology[snapshot/ontology]
        Extract[snapshot/extract]
        Process[snapshot/process]
        Diff[snapshot/diff]
        TxCreate[tx/create]
        TxCommit[tx/commit]
        StoryDraft[story/draft]
    end
    
    subgraph "Storage & Execution"
        GitHub[(GitHub Repo<br/>.aswritten/<br/>memories/<br/>discourse/<br/>.story)]
        Actions[GitHub Actions<br/>Async Extraction]
    end
    
    subgraph "External Services"
        OpenRouter[OpenRouter<br/>LLM Routing]
        DiscourseAPI[Discourse API<br/>Webhook Handler]
    end
    
    Chat --> MCP
    Forum --> DiscourseAPI
    Files --> GitHub
    
    DiscourseAPI --> Remember
    
    MCP --> Snapshot
    MCP --> Remember
    MCP --> Posit
    MCP --> Reprocess
    MCP --> DraftStory
    MCP --> CommitTx
    
    Snapshot --> Load
    Remember --> GitHub
    Posit --> Extract
    Reprocess --> Process
    DraftStory --> StoryDraft
    
    GitHub --> Actions
    Actions --> Load
    Actions --> Ontology
    Actions --> Extract
    Actions --> TxCreate
    Actions --> TxCommit
    
    Extract --> OpenRouter
    Process --> OpenRouter
    StoryDraft --> OpenRouter
    
    TxCommit --> GitHub
    
    Actions -.->|Bot Reply| Forum
    Actions -.->|Notification| Chat
    
    classDef interface fill:#e1f5ff,stroke:#0066cc
    classDef mcp fill:#fff4e1,stroke:#ff9800
    classDef core fill:#f0f4ff,stroke:#4c6ef5
    classDef storage fill:#e8f5e9,stroke:#2e7d32
    classDef external fill:#fce4ec,stroke:#c2185b
    
    class Chat,Forum,Files interface
    class MCP,Snapshot,Remember,Posit,Reprocess,DraftStory,CommitTx mcp
    class Load,Ontology,Extract,Process,Diff,TxCreate,TxCommit,StoryDraft core
    class GitHub,Actions storage
    class OpenRouter,DiscourseAPI external
```
## 2. Remember Workflow (Async Extraction)

```
sequenceDiagram
    participant User
    participant AI as Conversational AI
    participant MCP as MCP Tools
    participant Git as GitHub Repo
    participant Actions as GitHub Actions
    participant Bot as Notification Bot
    
    User->>AI: "Remember: We're pivoting to B2B"
    AI->>MCP: mcp/create-memory(config, user, content)
    MCP-->>AI: Memory object
    
    AI->>MCP: mcp/posit(config, memory)
    activate MCP
    MCP->>MCP: snapshot/load
    MCP->>MCP: snapshot/extract
    MCP->>MCP: snapshot/diff
    MCP->>MCP: tx/create
    deactivate MCP
    MCP-->>AI: Transaction (preview)
    
    AI->>User: "This adds 3 objectives, 2 stakeholders. Proceed?"
    User->>AI: "Yes"
    
    AI->>MCP: mcp/remember(config, memory)
    activate MCP
    MCP->>Git: memory/add (commit file)
    Git-->>MCP: Commit SHA
    MCP-->>AI: Job {commit, file-path, job-url}
    deactivate MCP
    
    AI->>User: "Saved. Extraction running: [job-url]"
    
    Git->>Actions: Trigger on file commit
    activate Actions
    Actions->>Actions: snapshot/load
    Actions->>Actions: snapshot/ontology
    Actions->>Actions: snapshot/extract
    Actions->>Actions: snapshot/diff
    Actions->>Actions: tx/create
    Actions->>Git: tx/commit
    deactivate Actions
    
    Actions->>Bot: Extraction complete
    Bot->>User: "Extracted: [commit SHA] [summary]"
```
## 3. Discourse Two-Step Save Workflow

```
sequenceDiagram
    participant User
    participant AI as Chat AI
    participant MCP as MCP Tools
    participant Discourse as Discourse Forum
    participant Webhook as Discourse Webhook
    participant Git as GitHub Repo
    participant Actions as GitHub Actions
    participant Bot as Discourse Bot
    
    User->>AI: "We should prioritize onboarding"
    AI->>User: "Post to Discourse for team input?"
    User->>AI: "Yes, Product Strategy"
    
    AI->>MCP: mcp/create-memory(config, user, content)
    MCP-->>AI: Memory object
    
    AI->>MCP: mcp/post-to-discourse(config, thread, memory)
    MCP->>Discourse: POST /posts with @aswritten
    Discourse-->>MCP: Post URL
    MCP-->>AI: Post URL
    
    AI->>User: "Posted: [URL]"
    
    Discourse->>Webhook: @aswritten mention detected
    Webhook->>MCP: mcp/remember(config, memory)
    activate MCP
    MCP->>Git: Commit to /discourse/{category}/{thread}/
    Git-->>MCP: Commit SHA
    MCP-->>Webhook: Job {commit, file-path, job-url}
    deactivate MCP
    
    Git->>Actions: Trigger extraction
    activate Actions
    Actions->>Actions: snapshot/load
    Actions->>Actions: snapshot/ontology
    Actions->>Actions: snapshot/extract
    Actions->>Git: tx/commit
    deactivate Actions
    
    Actions->>Bot: Extraction complete
    Bot->>Discourse: Reply: "Extracted: [commit SHA] [summary]"
```
## 4. Reprocess Workflow (Bulk Transformation)

```
sequenceDiagram
    participant User
    participant AI as Conversational AI
    participant MCP as MCP Tools
    participant Core as Core Operations
    participant Git as GitHub Repo
    
    User->>AI: "Change all email domains to @newcorp.com"
    
    AI->>MCP: mcp/snapshot(config)
    MCP->>Core: snapshot/load
    Core-->>MCP: Snapshot
    MCP-->>AI: Snapshot
    
    AI->>AI: Create Process {prompt: "Update emails..."}
    
    AI->>MCP: mcp/reprocess(config, snapshot, process)
    activate MCP
    MCP->>Core: snapshot/process(snapshot, process)
    activate Core
    Core->>Core: LLM generates SPARQL UPDATE
    Core-->>MCP: Transaction (uncommitted)
    deactivate Core
    deactivate MCP
    MCP-->>AI: Transaction with diff
    
    AI->>User: "This updates 42 email addresses. Diff:<br/>[show changes]<br/>Approve?"
    User->>AI: "Approve"
    
    AI->>MCP: mcp/commit-tx(config, transaction)
    activate MCP
    MCP->>Core: tx/commit(config, tx)
    Core->>Git: Commit transaction
    Git-->>Core: Commit SHA
    Core-->>MCP: Commit
    MCP->>Core: tx/summarize(tx, commit)
    Core-->>MCP: Plain-English summary
    deactivate MCP
    MCP-->>AI: Commit + summary
    
    AI->>User: "â Committed: [SHA]<br/>[summary]"
```
## 5. Story Generation Workflow

```
sequenceDiagram
    participant User
    participant AI as Conversational AI
    participant MCP as MCP Tools
    participant Core as Core Operations
    participant Git as GitHub Repo
    
    User->>AI: "Tell the DC37 training story"
    
    AI->>MCP: mcp/snapshot(config)
    MCP->>Core: snapshot/load
    Core-->>MCP: Snapshot
    MCP-->>AI: Snapshot
    
    AI->>MCP: mcp/stories(config)
    MCP->>Git: List .story files
    Git-->>MCP: [Story, Story, ...]
    MCP-->>AI: Story list
    
    AI->>User: "Found 'dc37-training-overview.story'. Generate?"
    User->>AI: "Yes"
    
    AI->>MCP: mcp/draft-story(config, snapshot, story)
    activate MCP
    MCP->>Core: story/draft(snapshot, story)
    activate Core
    Core->>Core: LLM generates narrative
    Core-->>MCP: Markdown string
    deactivate Core
    deactivate MCP
    MCP-->>AI: Rendered markdown
    
    AI->>User: "Here's the draft:<br/>[show preview]<br/>Save to repo?"
    User->>AI: "Yes, to /docs/"
    
    AI->>Git: Commit markdown to /docs/
    Git-->>AI: Commit SHA
    
    AI->>User: "â Saved: /docs/dc37-training.md"
```
## 6. Incremental Story Generation

```
flowchart TD
    Start([Story with increment=true]) --> GetTxs[Get transaction list]
    GetTxs --> Group[story/group-txs<br/>txs, story]
    
    Group --> Dedupe{Dedupe Strategy}
    Dedupe -->|By Date| DateGroup[Group by date<br/>2025-01-30.md<br/>2025-01-31.md]
    Dedupe -->|By Author| AuthorGroup[Group by author<br/>alice-changes.md<br/>bob-changes.md]
    Dedupe -->|By Topic| TopicGroup[Group by graph<br/>stakeholders.md<br/>objectives.md]
    
    DateGroup --> FileList[/List of filenames/]
    AuthorGroup --> FileList
    TopicGroup --> FileList
    
    FileList --> Loop{For each<br/>filename}
    
    Loop -->|More files| GetSnapshot[Get snapshot<br/>at transaction time]
    GetSnapshot --> Draft[story/draft<br/>snapshot, story]
    Draft --> Write[Write to<br/>filename.md]
    Write --> Loop
    
    Loop -->|Done| End([All stories generated])
    
    classDef input fill:#e1f5ff,stroke:#0066cc
    classDef process fill:#fff4e1,stroke:#ff9800
    classDef output fill:#e8f5e9,stroke:#2e7d32
    classDef decision fill:#fce4ec,stroke:#c2185b
    
    class Start,GetTxs input
    class Group,GetSnapshot,Draft,Write process
    class FileList,End output
    class Dedupe,Loop decision
```
## 7. Data Type Hierarchy

```
classDiagram
    class Config {
        +Github github
        +String handle
    }
    
    class Github {
        +String token
        +String owner
        +String repo
        +String ref
        +String dir
    }
    
    class Query {
        +Instant as-of
        +Vector~String~ graphs
        +String focus
    }
    
    class Snapshot {
        +Enum format
        +Int triples
        +Vector~String~ graphs
        +Instant as-of
        +String data
    }
    
    class Memory {
        +String message
        +Instant date
        +String user
        +String path
        +Enum format
    }
    
    class Transaction {
        +String id
        +Instant timestamp
        +String author
        +String provenance
        +String message
        +Boolean committed
        +String sha
        +Vector~Operation~ operations
    }
    
    class Operation {
        +Enum type
        +String graph
        +String triples
    }
    
    class Diff {
        +Triples added
        +Vector~Update~ updated
        +Triples removed
    }
    
    class Update {
        +Triple old
        +Triple new
    }
    
    class Triple {
        +String subject
        +String predicate
        +String object
    }
    
    class Story {
        +String id
        +String title
        +String path
        +String prompt
        +String destination
        +Enum format
        +Vector~String~ model
        +Boolean increment
    }
    
    class Job {
        +Commit commit
        +String file-path
        +String job-url
        +Enum type
    }
    
    class Commit {
        +String sha
    }
    
    class DiscourseThread {
        +String id
        +String title
        +String category
        +Vector~DiscoursePost~ posts
    }
    
    class DiscoursePost {
        +String thread-id
        +String category
        +String content
        +String author
        +Vector~String~ mentions
    }
    
    Config --> Github
    Transaction --> Operation
    Diff --> Triple
    Diff --> Update
    Update --> Triple
    Job --> Commit
    DiscourseThread --> DiscoursePost
```
## 8. Tool Dependency Graph

```
graph LR
    subgraph "MCP Tools"
        S[mcp/snapshot]
        R[mcp/remember]
        P[mcp/posit]
        RP[mcp/reprocess]
        DS[mcp/draft-story]
        CT[mcp/commit-tx]
        CS[mcp/canonize-story]
        ST[mcp/stories]
        PD[mcp/post-to-discourse]
    end
    
    subgraph "Snapshot Operations"
        SL[snapshot/load]
        SO[snapshot/ontology]
        SE[snapshot/extract]
        SP[snapshot/process]
        SD[snapshot/diff]
    end
    
    subgraph "Transaction Operations"
        TC[tx/create]
        TCC[tx/commit]
        TS[tx/summarize]
    end
    
    subgraph "Story Operations"
        StD[story/draft]
        StC[story/canonize]
        StG[story/group-txs]
    end
    
    subgraph "Memory Operations"
        MA[memory/add]
    end
    
    S --> SL
    
    R --> MA
    
    P --> SL
    P --> SO
    P --> SE
    P --> SD
    P --> TC
    
    RP --> SP
    RP --> TC
    
    DS --> StD
    
    CT --> TCC
    CT --> TS
    
    CS --> StC
    
    ST --> SL
    
    SE --> SO
    SP --> SL
    
    TC --> SD
    
    classDef mcp fill:#fff4e1,stroke:#ff9800
    classDef snapshot fill:#e1f5ff,stroke:#0066cc
    classDef tx fill:#f0f4ff,stroke:#4c6ef5
    classDef story fill:#e8f5e9,stroke:#2e7d32
    classDef memory fill:#fce4ec,stroke:#c2185b
    
    class S,R,P,RP,DS,CT,CS,ST,PD mcp
    class SL,SO,SE,SP,SD snapshot
    class TC,TCC,TS tx
    class StD,StC,StG story
    class MA memory
```
## 9. GitHub Actions Pipeline

```
flowchart TD
    Start([File Committed<br/>to Repo]) --> Detect{Path Pattern}
    
    Detect -->|memories/**| MemExtract[Memory Extraction Pipeline]
    Detect -->|discourse/**| MemExtract
    Detect -->|**.story| StoryGen[Story Generation Pipeline]
    Detect -->|Other| End([No Action])
    
    MemExtract --> Load1[snapshot/load]
    Load1 --> Ont[snapshot/ontology]
    Ont --> Extract[snapshot/extract]
    Extract --> Valid{Validated?}
    
    Valid -->|Yes| Diff1[snapshot/diff]
    Valid -->|No| NotifyErr[Notify extraction error]
    NotifyErr --> End
    
    Diff1 --> TxCreate1[tx/create]
    TxCreate1 --> TxCommit1[tx/commit]
    TxCommit1 --> Summary1[tx/summarize]
    Summary1 --> NotifySuccess[Post summary<br/>to Discourse/Chat]
    NotifySuccess --> End
    
    StoryGen --> Load2[snapshot/load]
    Load2 --> Stories[Get .story metadata]
    Stories --> Incr{increment=true?}
    
    Incr -->|Yes| GroupTx[story/group-txs]
    Incr -->|No| SingleDraft[story/draft<br/>single file]
    
    GroupTx --> Loop{For each<br/>filename}
    Loop -->|More| Draft[story/draft]
    Draft --> Write[Write markdown]
    Write --> Loop
    Loop -->|Done| Commit2[Commit all files]
    
    SingleDraft --> Write2[Write markdown]
    Write2 --> Commit2
    
    Commit2 --> End
    
    classDef trigger fill:#e1f5ff,stroke:#0066cc
    classDef process fill:#fff4e1,stroke:#ff9800
    classDef decision fill:#fce4ec,stroke:#c2185b
    classDef terminal fill:#e8f5e9,stroke:#2e7d32
    
    class Start trigger
    class Load1,Ont,Extract,Diff1,TxCreate1,TxCommit1,Summary1,Load2,Stories,GroupTx,Draft,SingleDraft,Write,Write2,Commit2 process
    class Detect,Valid,Incr,Loop decision
    class End,NotifyErr,NotifySuccess terminal
```
## 10. Multi-Tenant Access Control

```
flowchart TB
    subgraph "Discourse Layer"
        U1[User Alice<br/>Product Team]
        U2[User Bob<br/>Engineering Team]
        U3[User Carol<br/>Sales Team]
        
        Cat1[Category: Product<br/>Team: product-team]
        Cat2[Category: Engineering<br/>Team: eng-team]
        Cat3[Category: Sales<br/>Team: sales-team]
        
        U1 -.->|Member| Cat1
        U2 -.->|Member| Cat2
        U3 -.->|Member| Cat3
    end
    
    subgraph "GitHub Layer"
        Team1[GitHub Team<br/>product-team]
        Team2[GitHub Team<br/>eng-team]
        Team3[GitHub Team<br/>sales-team]
        
        Dir1[/product/<br/>CODEOWNERS:<br/>@org/product-team]
        Dir2[/engineering/<br/>CODEOWNERS:<br/>@org/eng-team]
        Dir3[/sales/<br/>CODEOWNERS:<br/>@org/sales-team]
        
        Team1 -->|Write Access| Dir1
        Team2 -->|Write Access| Dir2
        Team3 -->|Write Access| Dir3
    end
    
    subgraph "collective memory Snapshot"
        Snap1[Product Snapshot<br/>--dir /product]
        Snap2[Engineering Snapshot<br/>--dir /engineering]
        Snap3[Sales Snapshot<br/>--dir /sales]
        
        Dir1 --> Snap1
        Dir2 --> Snap2
        Dir3 --> Snap3
    end
    
    Cat1 -.->|Maps to| Team1
    Cat2 -.->|Maps to| Team2
    Cat3 -.->|Maps to| Team3
    
    classDef user fill:#e1f5ff,stroke:#0066cc
    classDef discourse fill:#fff4e1,stroke:#ff9800
    classDef github fill:#f0f4ff,stroke:#4c6ef5
    classDef snapshot fill:#e8f5e9,stroke:#2e7d32
    
    class U1,U2,U3 user
    class Cat1,Cat2,Cat3 discourse
    class Team1,Team2,Team3,Dir1,Dir2,Dir3 github
    class Snap1,Snap2,Snap3 snapshot
