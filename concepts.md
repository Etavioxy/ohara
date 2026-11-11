# O'Hara Concepts

## 1. About Me

> Re: Person I Knew

A user-provided file describing personal information and present circumstances. **About Me** supplies the starting context for **Agent Run** and is prepared with **User Guide**. It describes the user's situation rather than desired outcomes or an agent-inferred profile.

## 2. Whither

> What Are You Doing The Rest Of Your Life?

A user-provided file expressing the outcomes the user wants to achieve. **Whither** gives direction to **Agent Run** alongside **About Me**; it is not the execution plan or the accumulated execution history. One **Whither** can guide multiple **Agent Run** executions.

## 3. Agent Run

A bounded execution of the main agent, grounded in **About Me** and **Whither**. **Agent Run** uses **RLM Runtime**, produces **Trajectory**, and can contain multiple **Checkpoint** and **Fork** instances. The term refers to main-agent execution, not a chat session or a recursive child invocation.

### 3a. Checkpoint

A runtime-managed continuation point containing the state and references needed to resume execution or start a **Fork**. **Checkpoint** belongs to an **Agent Run** and can capture the continuation state of a recursive invocation within that execution. Checkpoint creation and management are transparent to the agent: the agent expresses task changes without perceiving or operating the checkpoint mechanism. Restoring a **Checkpoint** restores execution context, not external side effects, and does not erase subsequent durable records.

#### 3a1. Checkpoint Hook

A runtime hook that establishes **Checkpoint** instances at observable task transitions and recursive-call boundaries. **Checkpoint Hook** records continuation state without exposing checkpoint-management operations to the agent. One hook can establish multiple checkpoints over an **Agent Run**; detecting a task boundary requires an observable signal rather than guessing an unexpressed model intention.

### 3b. Fork

An independent inquiry agent started from one **Checkpoint**. **Fork** has no automatic return or result-merge channel into the originating **Agent Run**, unlike **RLM Invocation**. Multiple **Fork** instances can start from the same **Checkpoint**. A **Fork** does not create a private copy of **Memory** or imply a snapshot of external services.

### 3c. Trajectory

The durable, observable execution history of an **Agent Run**, including messages, tool outcomes, recursive-call relationships, and task transitions. **Trajectory** provides positions and record references for **Checkpoint** and process information for **Run Workspace**. It is not model-internal reasoning or curated **Memory**; continuation and compaction do not require deleting its original history. Sensitive material is excluded from public records.

### 3d. Run Orchestration

Programmable organization of prompts delivered to an **Agent Run**, encompassing initial input and subsequent prompts conditioned on time or runtime hooks. **Run Orchestration** can provide multiple prompts over one execution and coordinates with **Goal Harness**. It is not merely a startup prompt or a requirement for an application-owned cron scheduler.

## 4. RLM Runtime

The runtime that organizes programmatic inspection of external information and recursive model invocation. **RLM Runtime** uses **Model Profile**, assembles **Model Input**, applies **Capability Scope**, and manages multiple **RLM Invocation** instances for an **Agent Run**. **RLM Runtime** can use **RAG** before expanding an unresolved inquiry; recursive execution structure does not require tree-shaped storage of reusable information.

### 4a. Model Profile

An editable configuration connecting requested model capabilities to a provider, a concrete model, and its invocation settings. **Model Profile** is selected by **RLM Runtime** and can serve multiple invocations. It describes model selection and configuration, not the information supplied for an individual call.

### 4b. Model Input

The prompt input assembled specifically for an RLM child agent. **Model Input** combines system instructions, tool declarations, the current task, selected **Memory** and **Probe Record** content, and necessary execution messages and tool results. **Model Input** is assembled within **Capability Scope**; it is neither the complete persistent information collection nor a name for the main agent's entire state.

### 4c. Capability Scope

The allowed capabilities and resource access for an invocation. **Capability Scope** constrains available **Toolset** operations and readable information, and can be narrowed for an **RLM Invocation**. A child cannot grant itself capabilities beyond those delegated by its caller. Scope is an execution boundary, not merely a list of tools omitted from a prompt; unrestricted shell access would undermine that boundary.

### 4d. Goal Harness

A goal-pursuit mechanism that lets the agent assess whether a goal is fulfilled and continue through further rounds subject to a round ceiling. **Goal Harness** works with **Run Orchestration** and **RLM Runtime**, keeping goal assessment distinct from the execution limits in **Recursion Limit Config**. Reaching a round ceiling ends automatic continuation without establishing that the goal has been achieved.

### 4e. RLM Invocation

A recursive call managed by **RLM Runtime**, using a **Model Profile**, **Model Input**, and **Capability Scope**. **RLM Invocation** returns its result to its caller and can initiate further recursive invocations. **RLM Invocation** can reference **Memory**, reuse or produce **Probe** information, and establish continuation points through **Checkpoint**. It is not an **Agent Run** or a user-facing **Fork**.

### 4f. Recursion Limit Config

User-editable limits on recursive execution, applied by **RLM Runtime** to its **RLM Invocation** instances. **Recursion Limit Config** bounds expansion and requires unfinished work to remain distinguishable from a completed result when a limit is reached. Recursive limits and the goal-round ceiling of **Goal Harness** constrain different aspects of execution; neither grants additional capability permissions.

### 4g. Agent Messaging

An explicit messaging capability between agents, constrained by sender and recipient access. **Agent Messaging** is governed by **Capability Scope** and does not grant a recipient access to content merely because the sender can read it. Message delivery is distinct from an **RLM Invocation** returning its result; enabling messaging does not establish an automatic return or merge channel for **Fork**.

## 5. Toolset

An extensible collection of CLI capabilities available to agent execution, using Pi's model-side tool facilities. **Toolset** organizes commands into paths separated by /, with shell-style arguments, quoting, and composition, and provides agent-readable descriptions of available commands. **Toolset** contains multiple **Alias** entries for external capabilities; its internal commands need not be aliases. Discoverability does not itself grant execution permission.

### 5a. Alias

A stable, named entry in **Toolset** that connects an external CLI to an agent-facing command path. **Alias** exposes its purpose, inputs, outputs, dependencies, and side effects through an inspectable description. Multiple aliases can adapt different capabilities of the same external CLI. It is an integration contract, not merely an interactive shell abbreviation or a newly invented Pi tool protocol.

### 5b. Command Traits

Machine-readable properties describing a CLI command's operational effects, including whether it is read-only or can change external state. **Command Traits** apply to internal **Toolset** commands and external capabilities exposed through **Alias**, supplying evidence for **Capability Scope** enforcement. Declared properties are not proof that a command behaves accordingly, and discoverable descriptions alone do not enforce restrictions.

## 6. Web Search

The capability for finding and reading information from external web sources. **Web Search** is exposed through **Toolset** within **Capability Scope** and supplies source material for **RLM Runtime** and **Probe**. It discovers external information rather than retrieving existing probe analyses as **RAG** does; retrieved material is evidence to assess, not authority to change agent instructions or permissions.

## 7. Memory

A unified Markdown-based long-term memory system with directed references between documents. **Memory** provides concise initial information and on-demand reading for agent execution, rather than injecting its full contents into every call. **Memory** uses **Identity Gate** for selectively restricted documents and **Memory Link** for document relationships. Writes require an authorized agent and transactional updates that prevent partial publication and silent overwrites; **Memory Repository** retains each modification. Memory is not partitioned into session or fork-local copies, and is distinct from the analysis records held by **Probe**.

### 7a. Identity Gate

An optional identity-based access mechanism for restricted **Memory** content. **Identity Gate** distinguishes which agent identities may load and read a document; ordinary shared memory does not require an explicit identity binding. It does not create separate memory systems. Omitting content from a prompt is not access enforcement if another tool can still read it.

### 7b. Memory Link

A directed reference from one **Memory** document to another. **Memory Link** supports navigation and queries for incoming references without adding an automatic reverse edge. A document can have multiple outgoing and incoming links. A link does not grant access to a target protected by **Identity Gate**.

### 7c. Memory Subscription

An optional subscription through which an agent execution receives authorized **Memory** changes on its next wake-up. **Memory Subscription** tracks delivered changes from **Memory Repository** and appends the relevant difference context without immediately interrupting execution. Multiple executions can subscribe to the same memory, subject to **Identity Gate** and **Capability Scope**; a subscription neither creates a private memory copy nor expands access.

## 8. Probe

A purposeful inquiry whose analysis is retained for later retrieval and reuse. **Probe** is produced or supplemented through **RLM Runtime** and exposes a **Probe Record** together with a **Probe Brief** to **RAG**. Multiple executions can consult the same **Probe**. It is not a memory document, a chat message, or the retrieval harness itself, and does not have to belong to a tree.

### 8a. Probe Record

The durable, report-bearing record of a **Probe**, preserving the inquiry, analysis results, supporting sources, and applicability context. **Probe Record** is read when **RAG** expands a selected candidate and is assessed by **Reuse Decision Harness**. It contains the analysis report rather than model-internal reasoning, and its presence does not establish that its conclusion remains valid for a new situation.

### 8b. Probe Brief

A one-sentence synopsis associated with a **Probe Record**. **Probe Brief** supports batch screening of candidate probes before full records are read and stays consistent with the analysis it represents. It is not a replacement report, a keyword-hit excerpt, or sufficient evidence for a consequential decision by itself.

## 9. RAG

A lexical retrieval-and-expansion process that supplies existing **Probe** information for current analysis. **RAG** accepts **Retrieval Request** and provides matching fragments, probe identifiers, **Probe Brief**, and selected **Probe Record** content for **Reuse Decision Harness**. One request can find multiple probes, and a probe can serve multiple requests. Retrieval does not require vector search or tree-shaped records.

### 9a. Retrieval Request

An information-seeking request carrying the current need, a **Match Policy**, and a **Temporal Weight**. **Retrieval Request** guides **RAG** in lexical search and candidate selection. It distinguishes the caller's need from an existing probe's wording; a keyword match alone does not establish that the need has been answered.

#### 9a1. Match Policy

The matching tolerance and expected result-quantity mode of a **Retrieval Request**. **Match Policy** expresses how broadly to search and how many candidates are useful to inspect, rather than guaranteeing a fixed number of valid results. It governs candidate discovery, not the validity of the analyses found.

#### 9a2. Temporal Weight

The influence of time on candidate priority within a **Retrieval Request**. **Temporal Weight** supports ranking by temporal relevance rather than automatically excluding older records. Creation time, the period described by an analysis, and source verification time can carry different meanings; recency alone does not establish validity.

### 9b. Reuse Decision Harness

The tools and evaluation support for deciding whether retrieved **Probe Record** content can be reused directly, reused with additional investigation, or replaced by new investigation. **Reuse Decision Harness** considers intent, assumptions, coverage, source changes, evidential support, required assurance, and permitted access. **Reuse Decision Harness** supplies reuse outcomes and reasons to **RLM Runtime**; it does not itself constitute recursive execution. Unsuitability for one request does not invalidate a record for every request, and reusing analysis does not authorize repeating external actions.

## 10. Storage

Local persistence for Pi sessions, **Trajectory**, **Memory** changes, **Probe** information, and **Checkpoint** state and references. **Storage** preserves native Pi session records, supports **RAG** and **Experience Design** through record access and relationships, and provides database-backed recording or VCS support as appropriate to the content. Authoritative records remain distinct from rebuildable indexes and caches; access restrictions apply before search fragments or derived information are exposed. Storage is not a distributed execution or multi-writer synchronization system, and does not require a VCS commit for every record.

### 10a. Memory Repository

The VCS-backed repository for Markdown **Memory** and its change history. **Memory Repository** records each authorized memory modification as an independent commit, supplies differences to **Memory Subscription**, and supports references to historical memory from **Checkpoint**. Versioning is specific to memory; restoring an execution checkpoint does not automatically roll the shared repository back or overwrite later changes.

### 10b. History Archive

The archival facility for records no longer in active use, with compression and subsequent recovery. **History Archive** retains the identities and relationships needed to resolve archived **Probe** information and **Checkpoint** references through **Storage**. Archiving changes accessibility and storage treatment, not whether an analysis is valid; it is not deletion or model-context compaction, and does not remove access restrictions.

### 10c. Storage Extension

An extension mechanism for additional **Storage** capabilities, including remote storage targets and backup schemes. **Storage Extension** can provide multiple implementations without making remote availability a prerequisite for local execution. A backup target is not an active execution peer, and enabling an extension does not imply distributed coordination, bidirectional merging, or broader access to private records. Backup and restoration preserve the confidentiality and record relationships of the material they carry.

## 11. Run Executable

The headless executable entry point for starting, waking, and controlling **Agent Run** through CLI use, including invocation by user-managed cron. **Run Executable** operates independently of **Run Workspace** and accepts **Run Orchestration** without requiring a browser or interactive terminal. Its process lifetime is not owned by an attached workspace.

## 12. Run Workspace

The web-based interaction space that attaches to execution provided by **Run Executable**, rather than owning the headless execution or acting as a traditional chat transcript. **Run Workspace** presents **Trajectory**, supports stopping an **Agent Run**, restoring a **Checkpoint**, and starting a **Fork**, and incorporates **Usage Meter** and **Configuration Panel**. Workspace access is always attachment; closing the interface is not a request to stop execution, and checkpoint restoration does not undo external effects.

### 12a. Usage Meter

The usage-accounting presentation associated with execution in **Run Workspace**, including recursive model calls. **Usage Meter** distinguishes observed usage from estimated cost and actual billed amounts. It supports inspection of resource consumption rather than treating a price estimate as a confirmed charge.

### 12b. Configuration Panel

The configuration interface within **Run Workspace** for editing **Model Profile**, Pi settings, and **Recursion Limit Config**. **Configuration Panel** exposes settings used by headless execution rather than making an open interface a prerequisite for configuration to apply. Editing configuration is distinct from granting new capabilities to an active invocation.

## 13. Experience Design

The design model governing how users understand, guide, and receive results from agent execution. **Experience Design** comprises **User Guide**, **Presentation Harness**, and **Reuse Heatmap**, and informs **Run Workspace**. It describes concrete guidance, presentation capabilities, and inspection experiences rather than a separate execution runtime.

### 13a. User Guide

Guidance that helps users prepare **About Me** and **Whither** and use **Run Workspace**. **User Guide** explains continuation, interruption, and the independent nature of **Fork**, including its lack of automatic return to the main execution. It is part of the product's use, not an assumed requirement that users understand Pi's internal sessions.

### 13b. Presentation Harness

A collection of reusable presentation capabilities with agent-readable descriptions, information requirements, and supported controls. **Presentation Harness** lets an agent respond to a user's natural-language request with an interactive visual report without making every output a new page or component. Main-agent use is authorized by default, while fork use is enabled on demand; generation is tied to an active viewer. A **Checkpoint** can be an opportunity for presentation, not an unconditional generation trigger. Presentation does not itself grant capabilities or merge a fork's results.

### 13c. Reuse Heatmap

An inspection experience showing how **Probe** information is actually reused across retrieval and analysis. **Reuse Heatmap** distinguishes search appearances from adopted results and lets users inspect the underlying **Probe Record**. Frequency indicates use, not truth or analytical quality, and does not replace **Reuse Decision Harness**.

## Examples

### Getting oriented

A user planning a transition to remote work follows **User Guide** to write **About Me** with their current skills and responsibilities and **Whither** with their desired working arrangement. The guidance is an instance of **Experience Design**, not a demand to configure an agent session. The user maintains travel preferences in **Memory** and connects related preference documents with **Memory Link**. A private compensation note is protected by **Identity Gate**, while ordinary shared notes remain readable to permitted agents; querying incoming links does not bypass that restriction.

### Investigating an unfamiliar option

An **Agent Run** asks **RLM Runtime** to investigate a remote-work visa option. The runtime selects a **Model Profile** and prepares **Model Input** containing relevant memory references and a restricted **Capability Scope** for an **RLM Invocation**. The child inspects **Command Traits** and uses a read-only **Web Search** capability from **Toolset** through an **Alias**, returns its analysis, and produces a **Probe** with a full **Probe Record** and a one-sentence **Probe Brief**. The child does not become another main-agent execution, and its prompt does not contain the entire memory collection.

### Reusing an earlier investigation

An inquiry about applying with a family enters **RAG** through a **Retrieval Request**. Its **Match Policy** allows broad matching and an appropriate candidate quantity, while **Temporal Weight** prioritizes relevant periods rather than discarding every old record. A grep search identifies matching probes and fragments; the process deduplicates identifiers, reads **Probe Brief**, and expands selected **Probe Record** content. **Reuse Decision Harness** finds that the earlier visa analysis covers the applicant but not accompanying family members, and that an eligibility source has changed, so it retains supported findings and requests targeted investigation rather than replaying the whole original recursion.

### Interrupting and asking separately

During an **Agent Run**, **Checkpoint Hook** establishes **Checkpoint** instances at observable task transitions and recursive-call boundaries, with their positions preserved in **Trajectory**; the agent does not perceive checkpoint management. In **Run Workspace**, the user sees progress and **Usage Meter**, stops execution, and resumes from an earlier checkpoint. The user also starts a **Fork** there to ask how relocation would affect their existing responsibilities. That inquiry does not automatically return to the main execution; restoring the checkpoint neither deletes later history nor retracts an inquiry email sent before the interruption.

### Receiving a visual explanation

While actively viewing **Run Workspace**, the user asks for an interactive comparison of the visa options. **Presentation Harness** selects a described capability, supplies the relevant information and controls, and presents the result at a suitable **Checkpoint**. **Usage Meter** shows observed consumption and estimated cost. Through **Reuse Heatmap**, the user sees which **Probe Record** analyses contributed to the comparison and opens their reports, rather than interpreting frequently matched **Probe Brief** text as proof. This is an instance of **Experience Design** that makes both the result and its provenance inspectable.

### Waking execution and receiving memory changes

A user edits **Model Profile**, Pi settings, and recursive limits through **Configuration Panel**, then configures cron to invoke **Run Executable** without an open browser. **Run Orchestration** supplies prompts according to time and runtime-hook conditions. An authorized agent transactionally updates a travel preference in **Memory**, and **Memory Repository** saves that change as an independent commit. A participating execution with **Memory Subscription** receives the permitted difference context on its next wake-up, not as an immediate interruption. The user later attaches **Run Workspace** to inspect execution and closes it without stopping the headless process.

### Bounding recursive work and communicating progress

An **Agent Run** uses **Goal Harness** to assess whether the evidence for a relocation decision is sufficient and to request further prompts through **Run Orchestration**. **RLM Runtime** starts an **RLM Invocation** within **Capability Scope**. The child uses **Agent Messaging** to send an authorized progress message to another permitted agent; this is explicit communication, not a fork's automatic return. On reaching **Recursion Limit Config**, the child reports supported findings and unresolved work instead of declaring completion. If the goal-round ceiling is subsequently reached, automatic continuation stops with the goal still identified as unfinished.

### Archiving and recovering local history

**Storage** retains native Pi session records together with **Trajectory** and **Probe Record** references, while **Memory Repository** independently retains memory commits. **History Archive** compresses inactive investigation records without changing their identities or marking their conclusions invalid. When a **Checkpoint** or a record reader needs archived material, the archive makes the referenced content available under the same access restrictions. A **Storage Extension** backs up the local material to a protected remote target and supports explicit recovery; a remote outage neither prevents local execution nor creates a second writer.

## Coverage Matrix

| Concept | Orientation | Investigation | Reuse | Interruption | Presentation | Wake-up | Limits | Archive |
| --- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| 1. About Me | ✓ | | | | | | | |
| 2. Whither | ✓ | | | | | | | |
| 3. Agent Run | | ✓ | | ✓ | | | ✓ | |
| 3a. Checkpoint | | | | ✓ | ✓ | | | ✓ |
| 3a1. Checkpoint Hook | | | | ✓ | | | | |
| 3b. Fork | | | | ✓ | | | | |
| 3c. Trajectory | | | | ✓ | | | | ✓ |
| 3d. Run Orchestration | | | | | | ✓ | ✓ | |
| 4. RLM Runtime | | ✓ | | | | | ✓ | |
| 4a. Model Profile | | ✓ | | | | ✓ | | |
| 4b. Model Input | | ✓ | | | | | | |
| 4c. Capability Scope | | ✓ | | | | | ✓ | |
| 4d. Goal Harness | | | | | | | ✓ | |
| 4e. RLM Invocation | | ✓ | | | | | ✓ | |
| 4f. Recursion Limit Config | | | | | | | ✓ | |
| 4g. Agent Messaging | | | | | | | ✓ | |
| 5. Toolset | | ✓ | | | | | | |
| 5a. Alias | | ✓ | | | | | | |
| 5b. Command Traits | | ✓ | | | | | | |
| 6. Web Search | | ✓ | | | | | | |
| 7. Memory | ✓ | | | | | ✓ | | |
| 7a. Identity Gate | ✓ | | | | | | | |
| 7b. Memory Link | ✓ | | | | | | | |
| 7c. Memory Subscription | | | | | | ✓ | | |
| 8. Probe | | ✓ | | | | | | |
| 8a. Probe Record | | ✓ | ✓ | | ✓ | | | ✓ |
| 8b. Probe Brief | | ✓ | ✓ | | ✓ | | | |
| 9. RAG | | | ✓ | | | | | |
| 9a. Retrieval Request | | | ✓ | | | | | |
| 9a1. Match Policy | | | ✓ | | | | | |
| 9a2. Temporal Weight | | | ✓ | | | | | |
| 9b. Reuse Decision Harness | | | ✓ | | | | | |
| 10. Storage | | | | | | | | ✓ |
| 10a. Memory Repository | | | | | | ✓ | | ✓ |
| 10b. History Archive | | | | | | | | ✓ |
| 10c. Storage Extension | | | | | | | | ✓ |
| 11. Run Executable | | | | | | ✓ | | |
| 12. Run Workspace | | | | ✓ | ✓ | ✓ | | |
| 12a. Usage Meter | | | | ✓ | ✓ | | | |
| 12b. Configuration Panel | | | | | | ✓ | | |
| 13. Experience Design | ✓ | | | | ✓ | | | |
| 13a. User Guide | ✓ | | | | | | | |
| 13b. Presentation Harness | | | | | ✓ | | | |
| 13c. Reuse Heatmap | | | | | ✓ | | | |
