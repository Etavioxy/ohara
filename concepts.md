# O'Hara Concepts

## 1. About Me

A user-provided file describing personal information and present circumstances. **About Me** supplies the starting context for **Agent Run** and is prepared with **User Guide**. It describes the user's situation rather than desired outcomes or an agent-inferred profile.

## 2. Whither

A user-provided file expressing the outcomes the user wants to achieve. **Whither** gives direction to **Agent Run** alongside **About Me**; it is not the execution plan or the accumulated execution history. One **Whither** can guide multiple **Agent Run** executions.

## 3. Agent Run

A bounded execution of the main agent, grounded in **About Me** and **Whither**. **Agent Run** uses **RLM Runtime**, produces **Trajectory**, and can contain multiple **Checkpoint** and **Fork** instances. The term refers to main-agent execution, not a chat session or a recursive child invocation.

### 3a. Checkpoint

A runtime-managed continuation point containing the state and references needed to resume execution or start a **Fork**. **Checkpoint** belongs to an **Agent Run** and can capture the continuation state of a recursive invocation within that execution. Task transitions and recursive-call boundaries provide creation points; the agent expresses task changes without managing checkpoint storage. Restoring a **Checkpoint** restores execution context, not external side effects, and does not erase subsequent durable records.

### 3b. Fork

An independent inquiry agent started from one **Checkpoint**. **Fork** has no automatic return or result-merge channel into the originating **Agent Run**, unlike **RLM Invocation**. Multiple **Fork** instances can start from the same **Checkpoint**. A **Fork** does not create a private copy of **Memory** or imply a snapshot of external services.

### 3c. Trajectory

The durable, observable execution history of an **Agent Run**, including messages, tool outcomes, recursive-call relationships, and task transitions. **Trajectory** provides positions and record references for **Checkpoint** and process information for **Run Workspace**. It is not model-internal reasoning or curated **Memory**; continuation and compaction do not require deleting its original history. Sensitive material is excluded from public records.

## 4. RLM Runtime

The runtime that organizes programmatic inspection of external information and recursive model invocation. **RLM Runtime** uses **Model Profile**, assembles **Model Input**, applies **Capability Scope**, and manages multiple **RLM Invocation** instances for an **Agent Run**. **RLM Runtime** can use **RAG** before expanding an unresolved inquiry; recursive execution structure does not require tree-shaped storage of reusable information.

### 4a. Model Profile

An editable configuration connecting requested model capabilities to a provider, a concrete model, and its invocation settings. **Model Profile** is selected by **RLM Runtime** and can serve multiple invocations. It describes model selection and configuration, not the information supplied for an individual call.

### 4b. Model Input

The prompt input assembled specifically for an RLM child agent. **Model Input** combines system instructions, tool declarations, the current task, selected **Memory** and **Probe Record** content, and necessary execution messages and tool results. **Model Input** is assembled within **Capability Scope**; it is neither the complete persistent information collection nor a name for the main agent's entire state.

### 4c. Capability Scope

The allowed capabilities and resource access for an invocation. **Capability Scope** constrains available **Toolset** operations and readable information, and can be narrowed for an **RLM Invocation**. A child cannot grant itself capabilities beyond those delegated by its caller. Scope is an execution boundary, not merely a list of tools omitted from a prompt; unrestricted shell access would undermine that boundary.

### 4d. RLM Invocation

A recursive call managed by **RLM Runtime**, using a **Model Profile**, **Model Input**, and **Capability Scope**. **RLM Invocation** returns its result to its caller and can initiate further recursive invocations. **RLM Invocation** can reference **Memory**, reuse or produce **Probe** information, and establish continuation points through **Checkpoint**. It is not an **Agent Run** or a user-facing **Fork**.

## 5. Toolset

An extensible collection of CLI capabilities available to agent execution, using Pi's model-side tool facilities. **Toolset** organizes commands into paths separated by /, with shell-style arguments, quoting, and composition, and provides agent-readable descriptions of available commands. **Toolset** contains multiple **Alias** entries for external capabilities; its internal commands need not be aliases. Discoverability does not itself grant execution permission.

### 5a. Alias

A stable, named entry in **Toolset** that connects an external CLI to an agent-facing command path. **Alias** exposes its purpose, inputs, outputs, dependencies, and side effects through an inspectable description. Multiple aliases can adapt different capabilities of the same external CLI. It is an integration contract, not merely an interactive shell abbreviation or a newly invented Pi tool protocol.

## 6. Memory

A unified Markdown-based long-term memory system with directed references between documents. **Memory** provides concise initial information and on-demand reading for agent execution, rather than injecting its full contents into every call. **Memory** uses **Identity Gate** for selectively restricted documents and **Memory Link** for document relationships. It is not partitioned into session or fork-local copies, and is distinct from the analysis records held by **Probe**.

### 6a. Identity Gate

An optional identity-based access mechanism for restricted **Memory** content. **Identity Gate** distinguishes which agent identities may load and read a document; ordinary shared memory does not require an explicit identity binding. It does not create separate memory systems. Omitting content from a prompt is not access enforcement if another tool can still read it.

### 6b. Memory Link

A directed reference from one **Memory** document to another. **Memory Link** supports navigation and queries for incoming references without adding an automatic reverse edge. A document can have multiple outgoing and incoming links. A link does not grant access to a target protected by **Identity Gate**.

## 7. Probe

A purposeful inquiry whose analysis is retained for later retrieval and reuse. **Probe** is produced or supplemented through **RLM Runtime** and exposes a **Probe Record** together with a **Probe Brief** to **RAG**. Multiple executions can consult the same **Probe**. It is not a memory document, a chat message, or the retrieval harness itself, and does not have to belong to a tree.

### 7a. Probe Record

The durable, report-bearing record of a **Probe**, preserving the inquiry, analysis results, supporting sources, and applicability context. **Probe Record** is read when **RAG** expands a selected candidate and is assessed by **Reuse Decision Harness**. It contains the analysis report rather than model-internal reasoning, and its presence does not establish that its conclusion remains valid for a new situation.

### 7b. Probe Brief

A one-sentence synopsis associated with a **Probe Record**. **Probe Brief** supports batch screening of candidate probes before full records are read and stays consistent with the analysis it represents. It is not a replacement report, a keyword-hit excerpt, or sufficient evidence for a consequential decision by itself.

## 8. RAG

A lexical retrieval-and-expansion process that supplies existing **Probe** information for current analysis. **RAG** accepts **Retrieval Request** and provides matching fragments, probe identifiers, **Probe Brief**, and selected **Probe Record** content for **Reuse Decision Harness**. One request can find multiple probes, and a probe can serve multiple requests. Retrieval does not require vector search or tree-shaped records.

### 8a. Retrieval Request

An information-seeking request carrying the current need, a **Match Policy**, and a **Temporal Weight**. **Retrieval Request** guides **RAG** in lexical search and candidate selection. It distinguishes the caller's need from an existing probe's wording; a keyword match alone does not establish that the need has been answered.

#### 8a1. Match Policy

The matching tolerance and expected result-quantity mode of a **Retrieval Request**. **Match Policy** expresses how broadly to search and how many candidates are useful to inspect, rather than guaranteeing a fixed number of valid results. It governs candidate discovery, not the validity of the analyses found.

#### 8a2. Temporal Weight

The influence of time on candidate priority within a **Retrieval Request**. **Temporal Weight** supports ranking by temporal relevance rather than automatically excluding older records. Creation time, the period described by an analysis, and source verification time can carry different meanings; recency alone does not establish validity.

### 8b. Reuse Decision Harness

The tools and evaluation support for deciding whether retrieved **Probe Record** content can be reused directly, reused with additional investigation, or replaced by new investigation. **Reuse Decision Harness** considers intent, assumptions, coverage, source changes, evidential support, required assurance, and permitted access. **Reuse Decision Harness** supplies reuse outcomes and reasons to **RLM Runtime**; it does not itself constitute recursive execution. Unsuitability for one request does not invalidate a record for every request, and reusing analysis does not authorize repeating external actions.

## 9. Run Workspace

The web-based interaction space for following and directing an **Agent Run**, rather than a traditional chat transcript. **Run Workspace** presents **Trajectory**, supports stopping execution, restoring a **Checkpoint**, and starting a **Fork**, and incorporates **Usage Meter**. It exposes execution controls and state without implying that restoring a checkpoint undoes external effects.

### 9a. Usage Meter

The usage-accounting presentation associated with execution in **Run Workspace**, including recursive model calls. **Usage Meter** distinguishes observed usage from estimated cost and actual billed amounts. It supports inspection of resource consumption rather than treating a price estimate as a confirmed charge.

## 10. Experience Design

The design model governing how users understand, guide, and receive results from agent execution. **Experience Design** comprises **User Guide**, **Presentation Harness**, and **Reuse Heatmap**, and informs **Run Workspace**. It describes concrete guidance, presentation capabilities, and inspection experiences rather than a separate execution runtime.

### 10a. User Guide

Guidance that helps users prepare **About Me** and **Whither** and use **Run Workspace**. **User Guide** explains continuation, interruption, and the independent nature of **Fork**, including its lack of automatic return to the main execution. It is part of the product's use, not an assumed requirement that users understand Pi's internal sessions.

### 10b. Presentation Harness

A collection of reusable presentation capabilities with agent-readable descriptions, information requirements, and supported controls. **Presentation Harness** lets an agent respond to a user's natural-language request with an interactive visual report without making every output a new page or component. Main-agent use is authorized by default, while fork use is enabled on demand; generation is tied to an active viewer. A **Checkpoint** can be an opportunity for presentation, not an unconditional generation trigger. Presentation does not itself grant capabilities or merge a fork's results.

### 10c. Reuse Heatmap

An inspection experience showing how **Probe** information is actually reused across retrieval and analysis. **Reuse Heatmap** distinguishes search appearances from adopted results and lets users inspect the underlying **Probe Record**. Frequency indicates use, not truth or analytical quality, and does not replace **Reuse Decision Harness**.

## Examples

### Getting oriented

A user planning a transition to remote work follows **User Guide** to write **About Me** with their current skills and responsibilities and **Whither** with their desired working arrangement. The guidance is an instance of **Experience Design**, not a demand to configure an agent session. The user maintains travel preferences in **Memory** and connects related preference documents with **Memory Link**. A private compensation note is protected by **Identity Gate**, while ordinary shared notes remain readable to permitted agents; querying incoming links does not bypass that restriction.

### Investigating an unfamiliar option

An **Agent Run** asks **RLM Runtime** to investigate a remote-work visa option. The runtime selects a **Model Profile** and prepares **Model Input** containing relevant memory references and a restricted **Capability Scope** for an **RLM Invocation**. The child uses a search capability from **Toolset** through an **Alias**, returns its analysis, and produces a **Probe** with a full **Probe Record** and a one-sentence **Probe Brief**. The child does not become another main-agent execution, and its prompt does not contain the entire memory collection.

### Reusing an earlier investigation

An inquiry about applying with a family enters **RAG** through a **Retrieval Request**. Its **Match Policy** allows broad matching and an appropriate candidate quantity, while **Temporal Weight** prioritizes relevant periods rather than discarding every old record. A grep search identifies matching probes and fragments; the process deduplicates identifiers, reads **Probe Brief**, and expands selected **Probe Record** content. **Reuse Decision Harness** finds that the earlier visa analysis covers the applicant but not accompanying family members, and that an eligibility source has changed, so it retains supported findings and requests targeted investigation rather than replaying the whole original recursion.

### Interrupting and asking separately

During an **Agent Run**, task transitions and recursive-call boundaries establish **Checkpoint** instances, with their positions preserved in **Trajectory**. In **Run Workspace**, the user sees progress and **Usage Meter**, stops execution, and resumes from an earlier checkpoint. The user also starts a **Fork** there to ask how relocation would affect their existing responsibilities. That inquiry does not automatically return to the main execution; restoring the checkpoint neither deletes later history nor retracts an inquiry email sent before the interruption.

### Receiving a visual explanation

While actively viewing **Run Workspace**, the user asks for an interactive comparison of the visa options. **Presentation Harness** selects a described capability, supplies the relevant information and controls, and presents the result at a suitable **Checkpoint**. **Usage Meter** shows observed consumption and estimated cost. Through **Reuse Heatmap**, the user sees which **Probe Record** analyses contributed to the comparison and opens their reports, rather than interpreting frequently matched **Probe Brief** text as proof. This is an instance of **Experience Design** that makes both the result and its provenance inspectable.

## Coverage Matrix

| Concept | Orientation | Investigation | Reuse | Interruption | Presentation |
| --- | :---: | :---: | :---: | :---: | :---: |
| 1. About Me | ✓ | | | | |
| 2. Whither | ✓ | | | | |
| 3. Agent Run | | ✓ | | ✓ | |
| 3a. Checkpoint | | | | ✓ | ✓ |
| 3b. Fork | | | | ✓ | |
| 3c. Trajectory | | | | ✓ | |
| 4. RLM Runtime | | ✓ | | | |
| 4a. Model Profile | | ✓ | | | |
| 4b. Model Input | | ✓ | | | |
| 4c. Capability Scope | | ✓ | | | |
| 4d. RLM Invocation | | ✓ | | | |
| 5. Toolset | | ✓ | | | |
| 5a. Alias | | ✓ | | | |
| 6. Memory | ✓ | | | | |
| 6a. Identity Gate | ✓ | | | | |
| 6b. Memory Link | ✓ | | | | |
| 7. Probe | | ✓ | | | |
| 7a. Probe Record | | ✓ | ✓ | | ✓ |
| 7b. Probe Brief | | ✓ | ✓ | | ✓ |
| 8. RAG | | | ✓ | | |
| 8a. Retrieval Request | | | ✓ | | |
| 8a1. Match Policy | | | ✓ | | |
| 8a2. Temporal Weight | | | ✓ | | |
| 8b. Reuse Decision Harness | | | ✓ | | |
| 9. Run Workspace | | | | ✓ | ✓ |
| 9a. Usage Meter | | | | ✓ | ✓ |
| 10. Experience Design | ✓ | | | | ✓ |
| 10a. User Guide | ✓ | | | | |
| 10b. Presentation Harness | | | | | ✓ |
| 10c. Reuse Heatmap | | | | | ✓ |
