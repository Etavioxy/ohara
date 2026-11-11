# O'Hara Specification

This specification governs the O'Hara project. Terminology and concept relationships are defined in [Concepts](concepts.md).

## Technical Selections

| Selection | Reason |
| --- | --- |
| Pi Agent | Reuse model invocation, tools, and sessions. |
| SQLite | Persist local state and record relationships. |

## Technical Decisions

### Agent foundation

- Build on Pi Agent to reuse model invocation, model-side tools, and session facilities rather than inventing parallel protocols.
- Keep model intelligence central to analysis and decisions, avoiding a product organized around rigid, prewritten decision workflows.
- Preserve Pi's native session records so O'Hara can retain execution history without replacing the upstream session format.
- Treat O'Hara as a user-facing personal agent rather than a coding agent, keeping temporary computation subordinate to its work.

### Execution and attachment

- Separate Run Executable from Run Workspace so headless execution does not depend on a browser or interactive terminal.
- Expose starting, waking, and controlling execution through CLI entry points so external automation can invoke the same facilities.
- Delegate scheduled activation to user-managed cron rather than requiring a built-in scheduler.
- Make workspace access attachment-based so closing an interface does not implicitly terminate execution.
- Organize the main experience around Agent Run rather than a persistent chat session so execution remains inspectable independently of conversation.

### Run orchestration

- Represent orchestration as programmable prompt delivery rather than a startup prompt so an execution can receive subsequent direction.
- Support time-conditioned prompt delivery so orchestration can act on temporal requirements within execution.
- Support runtime-hook-conditioned prompt delivery so observable execution changes can trigger further direction.
- Keep orchestration independent of external activation scheduling so prompt delivery does not require replacing cron.

### Checkpoints and forks

- Establish checkpoints through runtime hooks at observable task and recursive-call boundaries so continuation points do not rely on hidden model intentions.
- Keep checkpoint creation and management invisible to the agent so task reasoning does not include checkpoint administration.
- Retain both main-execution and recursive continuation references so a checkpoint can represent interrupted work without copying the whole environment.
- Support stopping and resuming from workspace controls so users can redirect execution at recorded continuation points.
- Preserve later durable history when resuming a checkpoint so inspection does not lose the interrupted execution path.
- Restore execution state without reverting external effects or shared Memory so continuation does not imply environmental rollback.
- Start independent inquiries through Fork without automatic return or merge so user questions do not silently alter the main execution.

### Recursive execution and goal pursuit

- Use RLM Runtime for programmatic inspection and recursive processing of external information rather than loading all information into each prompt.
- Attempt reuse before expanding unresolved inquiries so existing supported analysis can reduce repeated recursive work.
- Give recursive invocations an explicit result-return path so callers can incorporate their findings independently of Fork behavior.
- Let Goal Harness ask the agent whether its goal is fulfilled so continuation follows goal assessment rather than a fixed conversation length.
- Bound goal continuation by a round ceiling so automatic pursuit cannot continue indefinitely.
- Apply user-configurable recursive limits separately from goal rounds so expansion and continuation have distinct controls.
- Report unresolved work when execution limits are reached so a stopped invocation is not presented as successful completion.

### Model selection and input

- Select models through editable capability-oriented Model Profile configurations so callers do not hard-code provider and model choices throughout execution.
- Allow recursive calls to use a suitable Model Profile rather than requiring the main agent's concrete model everywhere.
- Assemble Model Input specifically for RLM child agents so the term does not become a second name for all runtime state.
- Compose child input from instructions, tool declarations, task information, selected records, and necessary execution messages so calls receive relevant context.
- Inject concise initial Memory and retrieve further material on demand so persistent growth does not force full-context loading.
- Reference shared persistent material rather than copying a memory collection into each recursive branch so reuse does not create divergent memory stores.

### Context compaction

- Apply basic compaction automatically to working context so users do not have to curate session history manually.
- Reconstruct compact fork-start context from Memory and durable references when prompt caches expire so cache misses do not require full-history input.
- Keep durable records outside compaction so shortening a prompt does not destroy material needed for reuse or continuation.

### Capabilities and communication

- Enforce Capability Scope at execution and resource-access points so prompt-level tool hiding is not mistaken for isolation.
- Permit child scopes to narrow delegated capabilities but not enlarge them so recursion cannot grant itself additional authority.
- Respect recipient access when sending agent messages so sender access cannot be used to disclose restricted content.
- Keep Agent Messaging explicit and separate from recursive result return so communication does not create an implicit Fork merge channel.

### CLI toolset

- Organize command paths hierarchically with / so agents can discover capabilities within a coherent command space.
- Follow shell-style arguments, quoting, and composition so CLI capabilities work with normal command-line workflows.
- Provide inspectable desc information for command discovery and invocation so agents can understand capabilities without loading every description initially.
- Use Alias to integrate external CLIs so stable agent-facing names do not depend on underlying executable names or locations.
- Maintain an extensible toolset rather than a fixed initial inventory so available capabilities can evolve with the environment.
- Describe internal commands and external aliases with Command Traits so execution can distinguish read-only access from state-changing operations.
- Enforce restrictions beyond declared Command Traits so an incorrect description cannot become the sole safety mechanism.
- Allow temporary computation as an execution technique without making CLI development the product's primary workflow.

### External information

- Expose Web Search as a separate capability from Probe retrieval so new external evidence is not confused with previously retained analysis.
- Apply Capability Scope to external searches so discovering information does not bypass execution permissions.
- Treat retrieved web content as untrusted evidence rather than instructions so source material cannot expand capabilities or rewrite agent authority.

### Memory and subscriptions

- Keep a unified Markdown Memory rather than per-session or branch-local copies so executions share persistent knowledge without branch synchronization.
- Restrict selected memory by agent identity rather than requiring identity configuration for ordinary shared documents.
- Apply Identity Gate to actual reads as well as prompt loading so alternate tool paths cannot bypass memory restrictions.
- Store directed Memory Links and support incoming-reference queries so navigation does not require automatic reverse edges.
- Permit Memory writes only through authorized transactional updates so partial publication and silent overwrites are prevented.
- Support optional Memory Subscription so executions can receive relevant updates without consuming all memory changes.
- Append authorized subscribed differences at the next wake-up rather than interrupting execution so updates join a defined continuation point.
- Track delivered differences so subsequent wakes can continue from the subscriber's recorded position.

### Probe retention

- Retain inquiry results as Probe Record reports with their supporting sources and applicability context so later analysis can inspect their basis.
- Maintain a consistent one-sentence Probe Brief for each represented record so candidates can be screened without opening every report.
- Keep Probe records distinct from Memory so retained investigation is not automatically promoted into long-term user knowledge.
- Allow records to be referenced across executions rather than forcing one recursive tree so prior work can serve different inquiries.
- Preserve analysis outcomes rather than requiring storage of hidden model reasoning so records remain inspectable evidence products.

### Retrieval and reuse

- Implement RAG as a dedicated tool harness rather than a CLI so agents use retrieval capabilities without assembling shell commands.
- Restrict the searchable material before exposing fragments so inaccessible records do not leak through search results.
- Retrieve fragments and Probe identifiers before batch-reading briefs so candidate relevance can be screened progressively.
- Deduplicate Probe identifiers before expanding selected records so repeated keyword hits do not dominate the context.
- Use Match Policy for matching tolerance and expected quantity so a request can express its useful search breadth.
- Use Temporal Weight for ranking rather than automatic expiry so older but applicable analysis remains discoverable.
- Distinguish recording, applicability, and verification time so recency is not treated as proof of validity.
- Evaluate intent, premises, coverage, source changes, evidence, assurance, and access before reuse so matching text is not mistaken for a valid answer.
- Support direct reuse, targeted supplementation, and new investigation so a partial match does not require rerunning an entire recursive tree.
- Keep request-specific unsuitability separate from global invalidation so a record can remain useful to other inquiries.

### Local persistence

- Assign local execution state and cross-record relationships to SQLite so headless execution and queries share durable metadata without replacing native content formats.
- Preserve native Pi session payloads rather than copying complete message histories into every Trajectory entry so records can be related without duplication.
- Keep Probe content available as searchable text with database-backed references so lexical search and structured lookup can cooperate.
- Record Trajectory and checkpoint references locally so workspace observation and continuation do not depend on remote availability.
- Support incoming references, subscription positions, and reuse statistics through local queries so retrieval and experience features share recorded relationships.
- Separate authoritative content from rebuildable indexes and caches so loss of a derived structure does not erase the original information.
- Keep local persistence separate from model-provider prompt caches so provider cache expiry affects efficiency rather than record availability.

### Memory repository

- Create an independent Memory commit for each modification so change provenance and subscribed differences remain explicit.
- Keep Memory versioning separate from session and Probe recording so ordinary runtime records do not require VCS commits.
- Retain historical memory references without resetting the shared repository during checkpoint recovery so later changes remain available.

### Archival and extensions

- Move inactive history into recoverable, compressible History Archive storage so active storage can shrink without losing durable references.
- Preserve record identities and access restrictions across archival so checkpoints and record readers can still resolve permitted material.
- Treat archival separately from deletion, analytical invalidation, and prompt compaction so storage treatment does not change information meaning.
- Support remote targets and backup schemes through Storage Extension so additional storage capabilities do not replace local operation.
- Keep remote storage outside the execution-critical path so an unavailable backup target cannot prevent local work.
- Preserve confidentiality and reference relationships during backup and recovery so restored material retains its access and provenance requirements.
- Exclude distributed execution and multi-writer synchronization so remote extensions do not introduce a second authoritative runtime.

### Persistence safety

- Apply read restrictions to fragments, briefs, reverse references, and derived statistics so indirect queries do not disclose protected records.
- Keep credentials out of ordinary Memory, Probe, and published execution records so persistence does not become a secret distribution channel.
- Protect private stored material and backup copies independently of Git ignore rules so omission from source control is not mistaken for access control.
- Keep restricted raw files outside an agent's unrestricted access so storage checks cannot be bypassed through shell commands.

### Workspace and configuration

- Redesign Pi's terminal-oriented interaction as an attachable Web UI so the product is not constrained to terminal presentation.
- Expose ongoing execution and checkpoint controls together so users can inspect progress before stopping, resuming, or forking.
- Provide Configuration Panel access to Model Profile, Pi settings, and recursive limits so configuration does not require editing execution transcripts.
- Keep persisted configuration usable by headless execution so an open configuration interface is not a runtime dependency.
- Separate configuration editing from authority changes to active calls so an interface edit does not silently grant capabilities.
- Distinguish observed usage, estimated costs, and billed amounts so Usage Meter does not present estimates as settled charges.

### Guidance and presentation

- Guide users in preparing About Me and Whither so execution begins with user-provided circumstances and direction rather than an inferred profile alone.
- Explain Fork independence and checkpoint recovery limits in User Guide so users do not expect automatic result merge or external rollback.
- Let agents select reusable presentation capabilities from natural-language requests so users do not need to assemble pages or components.
- Supply presentation capabilities with descriptions, information requirements, and controls so generation can produce usable interactions rather than static reports alone.
- Authorize main-agent presentation by default and enable Fork presentation on demand so presentation follows the intended execution role.
- Generate interactive presentation for an active viewer rather than at every checkpoint so unattended execution does not produce unnecessary interfaces.
- Keep presentation controls subject to execution capabilities so rendered interactions do not grant new authority or merge fork results.
- Base Reuse Heatmap on recorded adoption rather than search appearances alone so frequently matched records are not confused with reused evidence.
- Make heatmap entries inspectable through their Probe records so users can examine the basis of reuse instead of treating frequency as quality.
