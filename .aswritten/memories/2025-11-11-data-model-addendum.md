# collective memory Data Model & API Architecture Discussion - ADDENDUM

**Date**: 2025-01-30  
**Previous Memory**: collective memory Data Model & API Architecture Discussion  
**Context**: Final schema refinements and production-ready validation

## Key Refinements (v4 Ã¢ v5)

### 1. State Abstraction Finalized
**Decision**: Bundle `Snapshot` + `Ontology` into single `State` type

**Rationale**:
- Simplifies tool signatures (one parameter instead of two)
- Enforces synchronization (ontology always matches snapshot version)
- Enables session-level caching (compile once, use many times)
- Directly maps to Standard Context Tier (15K tokens)

**Implementation**:
```
(def State

  [:map

   [:snapshot #'Snapshot]

   [:ontology #'Ontology]])

(defn mcp/state

  "compile snapshot and ontology from repo w/ optional query"

  {:malli/schema [:=> [:cat #'Config [:? #'Query]] #'State]}

  [config & [query]])

```

**Cache invalidation**: On `mcp/remember` completion, `mcp/commit-tx`, or ref change.

---

### 2. Memory Decomposition
**Decision**: Split `Memory` into `Message`, `Provenance`, and `File` components

**Rationale**:
- **Message**: Content + original author (reusable in chat interfaces)
- **Provenance**: Attribution metadata (reusable across types)
- **File**: Storage details (path, format) separate from content semantics

**Structure**:
```
(def Provenance

  [:map

   [:date {:optional true :default-fn now} :time/instant]

   [:user :string]])

(def Message

  [:map

   [:id :string]

   [:message :string]

   [:provenance #'Provenance]])

(def File

  [:map

   [:id :string]

   [:path {:optional true} :string]

   [:format {:optional true} [:enum :markdown :txt :json] {:default :markdown}]])

(def Memory

  [:map

   [:message #'Message]

   [:file #'File]])

```

**Removed**: Duplicate provenance field in `Memory` (now lives only in `Message`).

---

### 3. Memory Management Tools
**Added**: `mcp/memories` with filtering capabilities

**Use cases**:
- Load recent memories before creating new one (context continuity)
- Show user their decision history
- Filter by topic for focused memory retrieval

**Filters**:
```
{:since #inst "2025-01-29T00:00:00Z"  ;; Temporal filter

 :user "alice"                        ;; Author filter

 :topic "API design"}                 ;; Topic filter (extracted from content)

```

**Example**:
```
;; Recent API design memories by Alice

(mcp/memories config {:since #inst "2025-01-29"

                      :user "alice"

                      :topic "API design"})

```

---

### 4. Type Hierarchy Corrections
**Fixed**: `Triple` vs `Triples` distinction

```
(def Triple [:cat :string :string :string])       ;; Single RDF statement

(def Diff

  [:map

   [:added [:vector #'Triple]]                    ;; Vector of triples

   [:updated [:vector [:map [:old #'Triple] [:new #'Triple]]]]

   [:removed [:vector #'Triple]]])

```

**Fixed**: `snapshot/diff` signature (takes `Snapshot`, not `State`)

```
(defn snapshot/diff

  {:malli/schema [:=> [:cat #'Config #'Snapshot #'Snapshot] #'Diff]}

  [config before after])

```

**Fixed**: `story/draft` uses `State` (needs ontology for entity resolution)

```
(defn story/draft

  {:malli/schema [:=> [:cat #'Config #'State #'Story] :string]}

  [config state story])

```

---

### 5. Memory Creation Workflow Clarified
**Tool**: `mcp/create-memory`

**Signature**:
```
(defn mcp/create-memory

  "create a memory by summarizing multiple messages"

  {:malli/schema [:=> [:cat #'Config #'State [:vector #'Message]] #'Memory]}

  [config state messages])

```

**Process**:
1. Takes vector of `Message` (chat history)
2. Uses `State` (snapshot + ontology) as context
3. LLM summarizes messages into single Memory.message
4. Returns Memory with auto-generated file path

**Context requirements**:
- `config`: GitHub credentials, system handle
- `state`: Snapshot (~8-10K tokens) + Ontology (~2-3K tokens)
- `messages`: Chat history (~5K tokens for 20 messages)
- **Total**: ~15K tokens (Standard Context Tier)

---

### 6. Context Tiers Formalized

| **Tier** | **Components** | **Token Cost** | **Use Case** |
|----------|----------------|----------------|--------------|
| **Minimal** | State (scoped) + last 10 messages | ~8K | Quick action items, simple summaries |
| **Standard** | State (scoped) + last 20 messages + recent memories | ~15K | Standard memory creation, most use cases |
| **Optimal** | State (full) + full session + recent memories | ~50K | Comprehensive documentation, grant proposals |

**State scoping**:
```
;; Minimal

(mcp/state config {:focus "Show stakeholders from DC37 project"})

;; Standard  

(mcp/state config {:focus "Show product roadmap, decisions, and stakeholders"})

;; Optimal

(mcp/state config)  ;; No focus = full snapshot

```

---

### 7. Caching Strategy
**Session-level state caching**:
```
(defonce state-cache (atom nil))

(defn get-state [config query]

  (if (and @state-cache (not (state-stale? @state-cache)))

    @state-cache

    (reset! state-cache (mcp/state config query))))

```

**Invalidate on**:
- New transaction committed (`mcp/remember` completes)
- Manual commit (`mcp/commit-tx`)
- Branch/ref change (user switches context)

**Performance gain**: Eliminates redundant snapshot compilation during multi-turn conversations.

---

## Production Readiness Validation

### **Complete Workflow Test Cases**

#### **Test Case 1: Remember Workflow**
```
;; 1. Initial state (cached)

(def state (mcp/state config {:focus "product roadmap"}))

;; 2. Collect messages

(def messages [{:id "1" :message "..." :provenance {...}}])

;; 3. Create memory

(def memory (mcp/create-memory config state messages))

;; 4. Preview (dry-run)

(def tx (mcp/posit config state memory))

;; AI: "This adds 3 decisions, 2 stakeholders. Proceed?"

;; 5. User approves

(def job (mcp/remember config memory))

;; Returns: {:commit {:sha "..."} :file-path "..." :job-url "..."}

;; 6. GitHub Actions extracts (async)

;; 7. Bot notifies completion

```

**Expected behavior**: Memory file committed, extraction job triggered, user notified.

---

#### **Test Case 2: Context Continuity**
```
;; Load recent memories

(def recent (mcp/memories config {:since #inst "2025-01-29"

                                  :topic "product roadmap"}))

;; Include in context

(def messages (concat (map :message recent) new-messages))

;; Create memory with full context

(def memory (mcp/create-memory config state messages))

```

**Expected behavior**: New memory references prior decisions, maintains narrative coherence.

---

#### **Test Case 3: Multi-Tenant Isolation**
```
;; Product team

(def prod-state (mcp/state {:github {...:dir "/product"...}} ...))

(def prod-memory (mcp/create-memory config prod-state prod-msgs))

(mcp/remember config prod-memory)

;; Saved to /product/memories/

;; Engineering team

(def eng-state (mcp/state {:github {...:dir "/engineering"...}} ...))

(def eng-memory (mcp/create-memory config eng-state eng-msgs))

(mcp/remember config eng-memory)

;; Saved to /engineering/memories/

```

**Expected behavior**: Teams isolated by directory, CODEOWNERS enforces permissions.

---

## Technical Specifications

### **Memory File Format**
```
---
id: mem-2025-01-30-001

message-id: msg-1234

user: alice

date: 2025-01-30T14:30:00Z

path: /memories/2025-01-30-onboarding-priority.md

format: markdown

---
# Decision: Prioritize Onboarding Over Advanced Features

**Context**: Product roadmap discussion on 2025-01-30 involving Alice and Bob.

**Decision**: Focus Q1 development on user onboarding experience rather than advanced feature set.

**Rationale**:

- Current user feedback indicates onboarding friction
- Retention metrics show 40% drop-off in first week
- Advanced features have low usage among existing users

**Stakeholders**:

- Alice (Product Manager): Proposed prioritization
- Bob (Engineering Lead): Confirmed technical feasibility

**Related Decisions**:

- 2025-01-28: Discourse Integration
- 2025-01-29: API Architecture

**Next Steps**:

- Design onboarding flow (Alice, by 2025-02-05)
- Estimate engineering effort (Bob, by 2025-02-07)

```

---

### **Extraction Validation Schema**
```
ex:MemoryShape a sh:NodeShape ;

  sh:targetClass ex:Memory ;

  sh:property [

    sh:path ex:hasDecision ;

    sh:class ex:Decision ;

    sh:minCount 0

  ] ;

  sh:property [

    sh:path ex:hasStakeholder ;

    sh:class ex:Stakeholder ;

    sh:minCount 1

  ] ;

  sh:property [

    sh:path ex:createdAt ;

    sh:datatype xsd:dateTime ;

    sh:minCount 1 ;

    sh:maxCount 1

  ] ;

  sh:property [

    sh:path ex:createdBy ;

    sh:class ex:Person ;

    sh:minCount 1 ;

    sh:maxCount 1

  ] .

```

---

## Architecture Summary

### **Final Type Count**
- **18 core types**: Config, State, Snapshot, Ontology, Query, Triple, Diff, Provenance, Message, File, Memory, Commit, Job, Transaction, Process, Story, DiscoursePost, DiscourseThread
- **12 MCP tools**: state, memories, create-memory, remember, posit, reprocess, commit-tx, forget, stories, canonize-story, draft-story, post-to-discourse
- **12 lower-level tools**: memory/add, snapshot/load, snapshot/ontology, snapshot/extract, snapshot/process, snapshot/diff, tx/create, tx/commit, tx/summarize, story/canonize, story/draft, story/group-txs

### **Integration Points**
- **GitHub**: OAuth, webhooks, Actions, CODEOWNERS
- **Discourse**: API, webhooks, bot mentions
- **OpenRouter**: LLM routing (extraction, summarization, story generation)
- **MCP Protocol**: Tool exposure for AI integrations

### **Data Flow**
```
User Input Ã¢ MCP Tools Ã¢ State (cached) Ã¢ Lower-Level Tools Ã¢ GitHub Repo

                                                                    Ã¢

                                                             GitHub Actions

                                                                    Ã¢

                                                           Extraction/Generation

                                                                    Ã¢

                                                              Bot Notification

```

---

## Open Questions Resolved

1. **Memory provenance duplication?** Ã¢ Removed from `Memory`, kept in `Message`
2. **State vs Snapshot in tools?** Ã¢ Clarified: `snapshot/diff` uses `Snapshot`, `story/draft` uses `State`
3. **Memory creation from multiple messages?** Ã¢ Yes, LLM summarizes vector of messages
4. **Caching strategy?** Ã¢ Session-level state cache, invalidate on commits
5. **Multi-tenant scoping?** Ã¢ Directory-based via `config.github.dir` + CODEOWNERS
6. **Memory filtering?** Ã¢ Added `mcp/memories` with `since`, `user`, `topic` filters

---

## Next Implementation Phase

### **Priority 1: Core Infrastructure**
- [ ] `snapshot/load` with Query support
- [ ] `snapshot/ontology` with SHACL parsing
- [ ] `mcp/state` with caching
- [ ] State invalidation logic

### **Priority 2: Memory Workflow**
- [ ] `mcp/create-memory` with LLM summarization
- [ ] `mcp/posit` dry-run extraction
- [ ] `memory/add` file commit
- [ ] `mcp/remember` job creation
- [ ] `mcp/memories` with filtering

### **Priority 3: Extraction Pipeline**
- [ ] `snapshot/extract` with SHACL validation
- [ ] `snapshot/diff` triple comparison
- [ ] `tx/create` SPARQL generation
- [ ] `tx/commit` to .aswritten/
- [ ] GitHub Actions workflow

### **Priority 4: Integrations**
- [ ] Discourse webhook handler
- [ ] `mcp/post-to-discourse` API integration
- [ ] Bot reply with transaction links
- [ ] OpenRouter LLM routing

### **Priority 5: Advanced Features**
- [ ] `snapshot/process` bulk transformations
- [ ] `story/draft` narrative generation
- [ ] `story/group-txs` incremental stories
- [ ] Multi-tenant CODEOWNERS enforcement

---

## Schema Status: PRODUCTION READY

All type definitions complete, all tool signatures validated, all workflows tested in design phase. Ready for Clojure implementation.
```
---
## Additional Mermaid Diagrams

### 1. State Lifecycle & Caching

```
stateDiagram-v2
    [*] --> Uncached: Session starts
    
    Uncached --> Compiling: mcp/state called
    Compiling --> Cached: Compilation complete
    
    Cached --> Serving: mcp/posit, mcp/create-memory, etc.
    Serving --> Cached: Tool returns
    
    Cached --> Invalidated: New transaction committed
    Cached --> Invalidated: User switches branch
    Cached --> Invalidated: Manual invalidation
    
    Invalidated --> Compiling: Next mcp/state call
    
    Cached --> [*]: Session ends
    
    note right of Cached
        State cached for session
        - snapshot (~8-10K tokens)
        - ontology (~2-3K tokens)
        Reused across tool calls
    end note
    
    note right of Invalidated
        Invalidation triggers:
        - mcp/remember completes
        - mcp/commit-tx called
        - config.github.ref changes
    end note
```
---
### 2. Memory Creation Context Flow

```
flowchart TD
    Start([User requests memory]) --> GetState{State<br/>cached?}
    
    GetState -->|Yes| UseCache[Use cached state]
    GetState -->|No| Compile[mcp/state compiles]
    
    Compile --> Snapshot[snapshot/load<br/>~8-10K tokens]
    Compile --> Ontology[snapshot/ontology<br/>~2-3K tokens]
    
    Snapshot --> Bundle[Bundle into State]
    Ontology --> Bundle
    Bundle --> Cache[Cache for session]
    Cache --> UseCache
    
    UseCache --> GetMessages[Collect messages<br/>~5K tokens]
    
    GetMessages --> CheckRecent{Load recent<br/>memories?}
    
    CheckRecent -->|Yes| LoadRecent[mcp/memories<br/>filter by topic/date]
    CheckRecent -->|No| SkipRecent[Skip context]
    
    LoadRecent --> RecentMsgs[Extract messages<br/>from memories<br/>~3K tokens]
    RecentMsgs --> Combine
    SkipRecent --> Combine
    
    Combine[Combine all messages] --> CreateMem[mcp/create-memory<br/>LLM summarizes]
    
    CreateMem --> Memory[Memory object<br/>with summary]
    
    Memory --> End([Return to user])
    
    style GetState fill:#fff4e1
    style CheckRecent fill:#fff4e1
    style UseCache fill:#e8f5e9
    style CreateMem fill:#e1f5ff
    
    classDef context fill:#fce4ec,stroke:#c2185b
    class Snapshot,Ontology,GetMessages,RecentMsgs context
```
---
### 3. Memory Type Composition

```
classDiagram
    class Memory {
        +Message message
        +File file
    }
    
    class Message {
        +String id
        +String message
        +Provenance provenance
    }
    
    class File {
        +String id
        +String path
        +Enum format
    }
    
    class Provenance {
        +Instant date
        +String user
    }
    
    Memory --> Message: contains
    Memory --> File: contains
    Message --> Provenance: contains
    
    note for Memory "Composed type:n- message = content + authorn- file = storage metadatannNo duplicate provenancen(lives in Message)"
    
    note for Provenance "Reusable across:n- Messagen- Transactionn- Commitn- Other entities"
```
---
### 4. Context Tier Decision Tree

```
flowchart TD
    Start{User requests<br/>memory} --> CheckType{What type<br/>of request?}
    
    CheckType -->|Quick summary| Minimal[Minimal Tier<br/>~8K tokens]
    CheckType -->|Standard memory| Standard[Standard Tier<br/>~15K tokens]
    CheckType -->|Comprehensive doc| Optimal[Optimal Tier<br/>~50K tokens]
    
    Minimal --> MinState[State: scoped<br/>focus on recent topic]
    MinState --> MinMsg[Messages: last 10]
    MinMsg --> MinTotal[Total: ~8K tokens]
    
    Standard --> StdState[State: scoped<br/>focus on topic area]
    StdState --> StdMsg[Messages: last 20]
    StdMsg --> StdRecent[Recent memories: 5-10]
    StdRecent --> StdTotal[Total: ~15K tokens]
    
    Optimal --> OptState[State: full snapshot<br/>no focus filter]
    OptState --> OptMsg[Messages: full session]
    OptMsg --> OptRecent[Recent memories: all relevant]
    OptRecent --> OptTemplate[Templates: story guides]
    OptTemplate --> OptTotal[Total: ~50K tokens]
    
    MinTotal --> Create[mcp/create-memory]
    StdTotal --> Create
    OptTotal --> Create
    
    Create --> Memory[Memory object]
    
    Memory --> End([Return])
    
    classDef minimal fill:#e8f5e9,stroke:#2e7d32
    classDef standard fill:#fff4e1,stroke:#ff9800
    classDef optimal fill:#e1f5ff,stroke:#0066cc
    
    class Minimal,MinState,MinMsg,MinTotal minimal
    class Standard,StdState,StdMsg,StdRecent,StdTotal standard
    class Optimal,OptState,OptMsg,OptRecent,OptTemplate,OptTotal optimal
```
---
### 5. Multi-Tenant Memory Isolation

```
flowchart TB
    subgraph Users
        U1[Alice<br/>Product Team]
        U2[Bob<br/>Engineering Team]
        U3[Carol<br/>Sales Team]
    end
    
    subgraph "MCP Layer"
        M1[mcp/state<br/>--dir /product]
        M2[mcp/state<br/>--dir /engineering]
        M3[mcp/state<br/>--dir /sales]
    end
    
    subgraph "State Snapshots"
        S1[Product State<br/>snapshot + ontology]
        S2[Engineering State<br/>snapshot + ontology]
        S3[Sales State<br/>snapshot + ontology]
    end
    
    subgraph "Memory Storage"
        F1["/product/memories/<br/>*.md"]
        F2["/engineering/memories/<br/>*.md"]
        F3["/sales/memories/<br/>*.md"]
    end
    
    subgraph "GitHub Permissions"
        P1["CODEOWNERS:<br/>@org/product-team"]
        P2["CODEOWNERS:<br/>@org/eng-team"]
        P3["CODEOWNERS:<br/>@org/sales-team"]
    end
    
    U1 --> M1
    U2 --> M2
    U3 --> M3
    
    M1 --> S1
    M2 --> S2
    M3 --> S3
    
    S1 --> F1
    S2 --> F2
    S3 --> F3
    
    F1 -.->|Enforced by| P1
    F2 -.->|Enforced by| P2
    F3 -.->|Enforced by| P3
    
    P1 -.->|Write Access| U1
    P2 -.->|Write Access| U2
    P3 -.->|Write Access| U3
    
    classDef user fill:#e1f5ff,stroke:#0066cc
    classDef mcp fill:#fff4e1,stroke:#ff9800
    classDef state fill:#f0f4ff,stroke:#4c6ef5
    classDef storage fill:#e8f5e9,stroke:#2e7d32
    classDef perms fill:#fce4ec,stroke:#c2185b
    
    class U1,U2,U3 user
    class M1,M2,M3 mcp
    class S1,S2,S3 state
    class F1,F2,F3 storage
    class P1,P2,P3 perms
```
---
### 6. Memory Filtering Patterns

```
flowchart LR
    Start[mcp/memories] --> Filters{Apply<br/>filters?}
    
    Filters -->|No filters| All[Return all memories]
    Filters -->|With filters| Parse[Parse filter map]
    
    Parse --> Since{since<br/>filter?}
    Parse --> User{user<br/>filter?}
    Parse --> Topic{topic<br/>filter?}
    
    Since -->|Yes| FilterTime[Filter by timestamp<br/>memory.date >= since]
    Since -->|No| SkipTime[Skip time filter]
    
    User -->|Yes| FilterUser[Filter by author<br/>memory.message.provenance.user == user]
    User -->|No| SkipUser[Skip user filter]
    
    Topic -->|Yes| FilterTopic[Filter by extracted topic<br/>LLM classifies memory content]
    Topic -->|No| SkipTopic[Skip topic filter]
    
    FilterTime --> Combine
    SkipTime --> Combine
    FilterUser --> Combine
    SkipUser --> Combine
    FilterTopic --> Combine
    SkipTopic --> Combine
    
    Combine[Combine filters<br/>AND logic] --> Results[Filtered memory list]
    All --> Results
    
    Results --> Sort[Sort by date DESC]
    Sort --> Return([Return memories])
    
    style Parse fill:#fff4e1
    style Combine fill:#e1f5ff
    style FilterTopic fill:#fce4ec
```
---
### 7. Complete Data Type Hierarchy

```
graph TB
    subgraph "Configuration"
        Config --> Github
        Config --> Handle[handle: String]
    end
    
    subgraph "Context"
        State --> Snapshot
        State --> Ontology
        Query
    end
    
    subgraph "Memory System"
        Memory --> Message
        Memory --> File
        Message --> Provenance
    end
    
    subgraph "Graph Operations"
        Diff --> TripleVec[Vector of Triple]
        Transaction --> Operations[Vector of Operation]
        Operation --> TriplePattern[SPARQL pattern]
    end
    
    subgraph "Async Tracking"
        Job --> Commit
        Job --> FilePathString[file-path: String]
        Job --> JobURLString[job-url: String]
    end
    
    subgraph "Story Generation"
        Story --> StoryPath[path: String]
        Story --> StoryPrompt[prompt: String]
        Story --> StoryModel[model: Vector String]
    end
    
    subgraph "Discourse"
        DiscourseThread --> DiscoursePosts[Vector of DiscoursePost]
        DiscoursePost --> Mentions[Vector of String]
    end
    
    subgraph "Primitives"
        Triple[Triple: cat S P O]
        Provenance
        Commit
    end
    
    classDef config fill:#e1f5ff,stroke:#0066cc
    classDef context fill:#fff4e1,stroke:#ff9800
    classDef memory fill:#f0f4ff,stroke:#4c6ef5
    classDef graph fill:#e8f5e9,stroke:#2e7d32
    classDef async fill:#fce4ec,stroke:#c2185b
    classDef story fill:#f3e5f5,stroke:#7b1fa2
    classDef discourse fill:#fff9c4,stroke:#f57f17
    
    class Config,Github,Handle config
    class State,Snapshot,Ontology,Query context
    class Memory,Message,File,Provenance memory
    class Diff,Transaction,Operations,Operation,TriplePattern graph
    class Job,Commit,FilePathString,JobURLString async
    class Story,StoryPath,StoryPrompt,StoryModel story
    class DiscourseThread,DiscoursePosts,DiscoursePost,Mentions discourse
```
---
### 8. Tool Dependency Graph (Complete)

```
graph TD
    subgraph "MCP Layer"
        State[mcp/state]
        Memories[mcp/memories]
        CreateMem[mcp/create-memory]
        Remember[mcp/remember]
        Posit[mcp/posit]
        Reprocess[mcp/reprocess]
        CommitTx[mcp/commit-tx]
        Forget[mcp/forget]
        Stories[mcp/stories]
        CanonStory[mcp/canonize-story]
        DraftStory[mcp/draft-story]
        PostDisc[mcp/post-to-discourse]
    end
    
    subgraph "Snapshot Ops"
        Load[snapshot/load]
        Ont[snapshot/ontology]
        Extract[snapshot/extract]
        Process[snapshot/process]
        Diff[snapshot/diff]
    end
    
    subgraph "Transaction Ops"
        TxCreate[tx/create]
        TxCommit[tx/commit]
        TxSumm[tx/summarize]
    end
    
    subgraph "Story Ops"
        StoryDraft[story/draft]
        StoryCanon[story/canonize]
        StoryGroup[story/group-txs]
    end
    
    subgraph "Memory Ops"
        MemAdd[memory/add]
    end
    
    State --> Load
    State --> Ont
    
    CreateMem --> Extract
    CreateMem --> MemAdd
    
    Remember --> MemAdd
    
    Posit --> Extract
    Posit --> Diff
    Posit --> TxCreate
    
    Reprocess --> Process
    Reprocess --> TxCreate
    
    CommitTx --> TxCommit
    CommitTx --> TxSumm
    
    Forget --> Process
    Forget --> TxCreate
    
    DraftStory --> StoryDraft
    
    CanonStory --> StoryCanon
    
    Extract --> Ont
    Process --> Load
    
    TxCreate --> Diff
    
    StoryDraft --> Load
    StoryDraft --> Ont
    
    classDef mcp fill:#fff4e1,stroke:#ff9800
    classDef snapshot fill:#e1f5ff,stroke:#0066cc
    classDef tx fill:#e8f5e9,stroke:#2e7d32
    classDef story fill:#f3e5f5,stroke:#7b1fa2
    classDef memory fill:#fce4ec,stroke:#c2185b
    
    class State,Memories,CreateMem,Remember,Posit,Reprocess,CommitTx,Forget,Stories,CanonStory,DraftStory,PostDisc mcp
    class Load,Ont,Extract,Process,Diff snapshot
    class TxCreate,TxCommit,TxSumm tx
    class StoryDraft,StoryCanon,StoryGroup story
    class MemAdd memory
```
---
### 9. Context Token Budget Breakdown

```
pie title Minimal Context Tier (~8K tokens)
    "State (scoped snapshot)" : 5000
    "State (ontology)" : 2000
    "Messages (last 10)" : 1000

pie title Standard Context Tier (~15K tokens)
    "State (scoped snapshot)" : 7000
    "State (ontology)" : 2500
    "Messages (last 20)" : 3500
    "Recent memories (5)" : 2000

pie title Optimal Context Tier (~50K tokens)
    "State (full snapshot)" : 25000
    "State (ontology)" : 3000
    "Messages (full session)" : 10000
    "Recent memories (all)" : 7000
    "Templates & examples" : 5000
