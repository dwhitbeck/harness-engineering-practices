# Harness Engineering — Architecture and Design Notebook

**Version:** 1.1 (documentation quality revision)  
**Date:** November 14, 2026  
**Status:** Working design, not an approved implementation specification  
**Editorial scope:** Improve the document, not the functionality.  

> **Reading guide:** The consolidated design below is the primary reference. The decision register distinguishes agreed directions from proposals. Historical notes remain in the appendix and may contain exploratory wording that should not override the primary design.

# Consolidated Architecture — Working Design

**Discussion date:** 2026-11-14
**Document revision:** 1.1 — editorial quality review  
**Status:** Working design, not an approved implementation specification.  
**Scope:** Consolidated primary design for using engineering harnesses effectively, with decisions, rationale, governance boundaries, and open questions. Earlier chronological discussion notes are retained in the reference appendix.
**Interpretation:** Agreed directions and fixed initial policies are not equivalent to approved executable specifications; examples, possible mechanisms, and thresholds are illustrative unless separately approved.

## 1. Executive direction: what is being built

The initial deliverable is **not** a domain-specific coding assistant or a completed central orchestrator. It is a **reusable, recursive, evidence-driven engineering lifecycle** for building *the tools and skills that themselves build, inspect, improve, and evaluate software and architecture*. The same lifecycle can serve architecture, security, compliance, performance, engineering tooling, or other goals.

The unit of work is a **goal**. Goals can contain nested goals, tasks, experiments, decisions, evaluation loops, and acceptance criteria. The lifecycle is uniform; the contracts and data for each task are specialized and extensible. Configuration and run policies can differ by goal and by execution.

Guiding optimization: **use the best verified capability available now; systematically replace repeatable model work with fast, inexpensive deterministic programs where feasible**, without compromising quality, required safeguards, or human approval. AI can be central for some tasks (e.g., interpretive security review), assist with tool development, or be an exception path; it must not be artificially inserted into every runtime operation.

### Lifecycle primitive

1. Define goal, constraints, applicable governance profile, dependencies, acceptance criteria, and budget.
2. Resolve inherited and local knowledge/policies; discover available capabilities and contracts.
3. Decompose into smaller goals where needed and select deterministic, model-based, or hybrid methods according to run policy.
4. Execute within bounded, least-privilege environments and capture observable evidence.
5. Evaluate against mandatory gates and configurable weighted/partial criteria; report unknowns explicitly.
6. Decide: accept, retry, revise task decomposition, improve a tool/skill, request human input, or stop.
7. Persist the attempts, outcomes, decision rationales, and artifact references in Git/GitHub.
8. Promote approved improvements through the applicable PR and release process; revalidate and monitor actual outcomes.
9. Extract verified reusable lessons; re-evaluate when assumptions, dependencies, or environments change.

This is a **versioned graph**, not a single forward-only pipeline: branches, checkpoints, failed experiments, reversals, and alternative decisions must remain traceable.

## 2. Component model and recursive goal contracts

### 2.1 Generic envelope + typed specialized payload

Use a common **self-describing envelope** for goals, tools, evaluations, conflicts, findings, and decisions, each carrying specialized typed extensions. Required envelope data should include:

- Stable ID; object kind; schema/contract ID and version; producer and consumer capability IDs/versions.
- Parent goal, run and iteration IDs; timestamps; status; applicability scope; provenance Git SHAs.
- Goal requirements; mandatory controls; optional weighted criteria; autonomy and AI-use policy.
- Typed payload descriptors, schema URLs/IDs and versions, required-versus-optional extension markers.
- Evidence references, result classifications, known/unknown/unsupported coverage, confidence/limitations and explanatory notes.
- Dependency links; error and conflict classifications; human decisions and approvals where required.

**Compatibility:** A consumer must understand the core envelope. A required unknown extension or incompatible semantic contract must fail or escalate explicitly; an optional unknown extension may be skipped but must be recorded. Support versioned migrations/adapters with compatibility tests, deprecation and retirement policy, and impact analysis across consumers. Capability discovery happens before invocation when feasible.

Example *illustrative* envelope (not a finalized schema):

```yaml
kind: tool_result
contract: {id: harness.tool-result, version: '1.0'}
run_id: run-example
iteration_id: iter-02
producer: {id: go-analyzer, version: '0.3.0'}
status: completed_with_unknowns
coverage: {examined: 1000, classified: 950, unknown: 30, unsupported: 20}
extensions:
  - {type: spanner-transactions, schema: 'spanner-analysis/v2', required: false, artifact_ref: 'artifact:example'}
evidence: [{sha256: '<content-hash>', locator: 'approved-artifact-profile/object-id'}]
limitations: ['dynamic call target unresolved']
```

### 2.2 Inheritance and resolution

Central knowledge and parent goals establish defaults. Children and repositories may **inherit, augment, override, or remove** inapplicable items, with explicit applicability, scope, rationale, approval, and evidence. Removal is a *recorded exclusion*, not erasure. Applicable governance controls (e.g., required change approvals) cannot be silently disabled. An effective-context resolution artifact lists every included and excluded rule and its provenance. Multiple-parent conflicts are represented in a standard conflict contract and routed through a configurable evaluation/escalation loop.

### 2.3 Goal completion

Support strict pass-all, partial success, weighted thresholds, mandatory child gates, and combinations. Dependencies invalidate or block downstream claims if prerequisites are unmet. Completion must distinguish success, partial success, blocked, failed, unknown and not applicable. No aggregate score can compensate for a mandatory security/compliance gate failure.

## 3. Tools, skills, coverage and progressive deterministic replacement

- **Skills** instruct an agent how to approach a task; **tools** perform execution, detection, transformation, measurement and potentially **deterministic recommendations and remediation**. Existing frameworks/libraries are preferred when suitable, rather than building proprietary engines by default.
- Deterministic output may contain a precise suggested change or automatically generated patch, not merely a finding. Tool-generated modifications still go through testing and human-approved PRs.
- Every tool should describe capabilities, input/output contracts, applicability, runtime characteristics, explicit *known*, *unknown*, *unsupported*, and *not-evaluated* cases, and relevant evidence—even after an otherwise successful execution.
- An unknown pattern should not be silently interpreted as absence of risk. Report actual examined population and limitations, rather than misleading global percentages.
- The improvement loop clusters recurring unsupported cases (e.g., language-specific syntax-tree structure, API family, error signature), chooses targeted remediation by expected *risk-weighted marginal benefit per effort*, and measures false positives, false negatives, regressions and coverage gains after each update.
- Model-driven analysis may be used immediately, deferred, or avoided according to per-run policy. AI-created deterministic replacements must be validated against independent truth/evidence, **not** just mimic prior AI responses.
- Store verified examples, counterexamples, applicability conditions, tests, and stale/expiry triggers as reusable lessons. AI can help discover new rules, but deterministic detectors, recommendations and fixes should replace recurring model work when practical.
- Profile all capabilities for measured correctness, coverage, unknowns, reliability, latency, compute cost, model/token cost, impact and security classification. Do not assume a perfect finite deterministic solution for ambiguous or changing tasks.

## 4. Evaluation framework — agreed initial implementation focus

**First implementation target:** a **small, reusable evaluation and evidence framework** around the basic goal lifecycle, rather than a domain-specific tool or full orchestrator. A minimal end-to-end vertical slice should:

1. Read a goal manifest, run policy and applicable contracts.
2. Execute one simple deterministic tool (AI optional).
3. Capture standardized results, known/unknown coverage, elapsed time, costs, logs and references.
4. Evaluate independently defined acceptance criteria (mandatory and optional) and compare iterations.
5. Record human-readable rationale, alternatives, decisions, outcomes and evidence in Git.
6. Propose a next iteration or human-reviewed change, but **do not autonomously merge PRs**.

Evaluation supports per-goal domain-specific extensions; correctness, coverage, latency, resource use, financial cost, security/compliance, observability, maintainability and risk. Always distinguish *characteristic coverage* from correctness. Severity-weighted errors and mission-critical false negatives matter even when aggregate accuracy is high. The harness should compare predicted benefit with measured benefit and re-rank remaining opportunities after every attempt.

**Run-configurable decision policy:** parameters may choose analysis-only/human decision, AI recommendation, or bounded automatic analysis/test selection. Additional configuration controls risk classes, acceptable uncertainty, completeness/partial-result rules, time/iteration/cost budgets, test scope, impact-versus-effort ranking and risk gates. Confidence is capability-specific and reversible following regressions. Human approval requirements are not relaxable by an AI confidence score.

### 4.1 Representative test selection

Characterize repositories by relevant versions, architecture patterns, dependencies, usage and history. Select a representative subset using a deterministic coverage/risk objective (e.g., weighted set cover), with AI permitted to propose missing characteristics. Document why each repository was selected or excluded, gaps in coverage, incremental expected value and cost. Compare expected versus observed additional coverage; expand testing if evidence/risk requires. A subset is useful for early evaluation, but broad validation may remain mandatory for promotion depending on risk and policy.

### 4.2 Cross-category remediation

Detect and validate whether apparently different failures (security, classification, performance) share root causes or fixes. Track causal evidence separately from hypotheses; estimate unique improvement without double counting; prioritize high-severity exposure and expected marginal benefit, constrained by cost, implementation effort and regression risk.

## 5. GitHub-first evidence, history and knowledge architecture

**Authoritative repository content:** Markdown decisions and notes, YAML/JSON manifests, contracts, schemas, skills, policies, tests, small sanitized findings, links to approved evidence storage. Preserve successful and failed attempts, rejected alternatives, reasons, measured outcomes, reversals, and proven reusable lessons; a decision log is not a substitute for full attempt history.

**Large or sensitive content:** traces, logs, sample datasets, binaries, and sensitive evidence live in governed external object/database storage where appropriate. Commit **metadata** for retrieval: dataset/record identity, approved connection profile (no secrets), URI/locator, schema and table names where relevant, immutable content hash/version, creation time, access controls, retention/deletion conditions, and evidence-to-run link. A mutable database query is not an immutable historical snapshot unless version/snapshot semantics are provided.

**Shared + local knowledge:** one central GitHub repository can hold versioned constraints, tool limitations, known failure patterns and contracts. Each solution repo carries local additions, exclusions and justified overrides. The resolver forms the smallest accurate context for each goal, while honoring applicable organizational policy. Each override records its validity predicate; changes to APIs, code patterns, dependencies and tool versions trigger re-evaluation. Knowledge itself has owners, status, applicable version range, confidence, verification date and revalidation trigger. A database may be used as a derived index or as necessary for large data, not automatically the source of truth.

**Suggested repository layout** (illustrative; refine during implementation):

```text
harness-engineering/
  README.md
  goals/                 # Goal and child-goal manifests
  contracts/             # Envelope + specialized schemas and migrations
  skills/                # Agent instructions, versions, tests
  tools/                 # Deterministic analyzers and tool tests
  policies/              # Governance, AI-use, risk, budget and escalation
  evaluations/           # Suites, baselines, scoring definitions
  knowledge/             # Central constraints and lessons
  decisions/             # ADRs, trade-offs and supersessions
  runs/                  # Compact sanitized run/iteration manifests
  artifacts/             # External artifact reference manifests only
  templates/             # PR, finding, conflict, override templates
  docs/                  # Design and operation guidance
```

Each solution repository may contain `.harness/` goal/config/override/decision files, pointing to pinned shared knowledge and tool versions. Each run identifies source/tree SHAs and exact inputs evaluated; generated PR summaries link source, evidence, policy, reviewers, and approval outcomes.

**Git safety:** no committed credentials, secrets, prohibited data or raw sensitive evidence. Use access controls, branch protection, secret scanning, review requirements, and artifact-retention policies. Prefer immutable references/checksums and append-only evidence where possible. Reviewers should inspect immutable tested revisions; isolate writing agents with worktrees/branches or containers. Never claim exact replay of opaque model behavior.

## 6. Solution and dependency lifecycle

- Each solution has a versioned manifest of its effective contracts, skills, tools, model configurations, evaluation suite, dependency versions and governance profile.
- Operations: **replay** historical state where feasible, **upgrade/migrate** dependency/contract/tool versions, then **revalidate** original and newly applicable acceptance criteria. A schema migration passing is not enough to establish behavioral equivalence.
- Shared foundational updates should be proposed proactively across consumers to reduce version drift, similar to Dependabot; specialized components may follow risk/effort/business priority. Track transitive dependencies across repos (e.g., C → B → A), deployed artifact SBOM/resolved versions, and actual adoption instead of assuming publication means deployed uptake.
- Evaluate compatible deployments independently when mixed-version interfaces remain safe; isolate failures for separate investigation and human intervention. Test cross-service behavior even where all services individually pass.
- Shared constraints include known limitations in internal and third-party tools/emulators, triggering conditions, affected versions, and whether a repo actually exercises the problematic code path. Revalidate stale findings after version or code changes.

## 7. Local-first execution and environment fidelity

Develop and test first in isolated local environments where practical. Use representative service emulators and telemetry substitutes when available. Track precisely what substitutes do and do not reproduce: API semantics, transactions, retries, concurrency, failure modes, observability and scalability/performance differences. A local pass does not establish production equivalence. Maintain fidelity-gap findings and triggered tests; run environment-specific validation before release and operational monitoring afterward. Feed discovered gaps back into shared tooling. Keep local environments reproducible and bounded in resources.

## 8. Governance, PRs, deployment and audit

**Human review boundary:** Agents may prepare a proposed change while findings or uncertainties remain unresolved, but those concerns must be visible. Creating a pull request is not approval or permission to merge. Require review and authorization appropriate to the change's risk and the repository's governance. Capture review feedback as contextual evidence; it can inform improvements, but does not automatically establish a universal rule.

**Release lifecycle:** Integrate with each project's existing validation, approval, deployment, and rollback processes rather than assuming a particular toolchain. Separate authorization to change source from authorization to deploy where the governing process requires it. Use staged rollout, feature controls, or canaries when appropriate. Link observed outcomes to the goal, tested revision, artifacts, review, deployment, and evaluation record. Shared components may have independent release lifecycles; consuming projects should adopt versions through their normal reviewed update process. A harness must not grant itself merge or production authority.

**Compliance is an input**: per-solution governance profile covers applicable regulations/policies/contractual constraints, data classifications, enforced provider controls, customer-configurable controls, customer responsibility, audit evidence and authorized exception pathways. Internal-only tools may have different applicable profiles. Supporting a compliant configuration does not guarantee customer operation complies. Mandated gates cannot be canceled by unrelated weighted performance results. Human risk acceptance follows actual authority levels and policy, not assumed exceptions.

## 9. Policy conflicts and exceptions

Conflict detection is standardized and should distinguish **detect**, **recommend**, and **authorize**. A conflict finding includes source policy IDs/versions, affected scope, contradictory requirements, evidence, risk, alternative resolutions and rationale. Initially escalate to a human; later propose or automate narrowly scoped low-risk resolutions under explicit approved rules. Mandatory governance controls require authorized exception workflows when exceptions are permitted; some restrictions cannot be overridden. All outcomes and follow-up tests are logged. Historical decisions inform recommendations but are not universal precedents.

## 10. Phased implementation roadmap

**Phase 0 — Define governance and evidence boundaries**: establish human PR authority, secret/retention policies, base contract envelope, minimum goal schema, and what counts as verified evidence. Avoid heavyweight orchestration.

**Phase 1 — Foundational vertical slice (START HERE)**: goal + nested-task manifests, lightweight deterministic runner, pluggable evaluator, run/iteration ledger, decision log and Git commits, simple coverage/unknown reporting, one human review path. Measure execution and evaluation overhead.

**Phase 2 — Composable skills/tools**: typed extensions, schema negotiation, capability discovery, central/local knowledge resolution, local containers/emulators, configurable AI-on-unknown policy, simple rule-based remediation. Use real workloads to uncover needed changes.

**Phase 3 — Improvement and diagnostic loops**: writer/reviewer/tester/analyzer roles; tool/skill improvement PRs; clustered unknowns; risk/effort ranking; benchmark/regression suites; representative multi-repository evaluation; lessons registry and constraint detection.

**Phase 4 — Lifecycle and organization-scale operation**: transitive dependency impact/migration, contract deprecation, environment fidelity knowledge, governance profiles, adaptive risk-based autonomy, shared knowledge promotion.

**Phase 5 — Central orchestrator**: build only once foundation and actual flows have demonstrated what coordination semantics are needed. Keep agents and tools independently replaceable via contracts; add scheduling, concurrency, durable state, compensation/retry and recovery as proven necessary.

## 11. Decision register — choices, reasons, status

The following register separates **user-established direction** from **proposals that still require design/testing**. Reasons summarize the rationale stated in the discussion, rather than attributing fabricated quotations. Dates are the discussion date, not implementation dates.

| ID | Decision / direction | Why / intended effect | Status |
|---|---|---|---|
| D-001 | Build a reusable, recursive engineering SDLC first; domain tools and central orchestration follow | The same evidence/iteration principles must work across architecture, security, compliance, and code; premature orchestration would lock in untested assumptions | **Agreed direction** |
| D-002 | Goal is the basic unit; nested goals have specialized contracts, criteria and policies | Enables decomposition and re-composition of differently scoped problems with a common lifecycle | **Agreed direction** |
| D-003 | Prefer deterministic analysis, recommendations and fixes when available; AI is optional and can be the initial solution | Conventional code is often faster and cheaper; AI can help create deterministic replacements for repeatable tasks | **Agreed direction** |
| D-004 | Choose execution method per run: deterministic, AI, hybrid, defer, or human | Available capabilities and task requirements vary; no universal fallback strategy is correct | **Agreed direction** |
| D-005 | Begin with a common evaluation/evidence framework | Measured correctness, coverage, latency, cost and risk are prerequisites for rational optimization and increased autonomy | **Agreed first implementation focus** |
| D-006 | Use generic versioned envelopes with specialized typed payloads and capability discovery | Allows extensibility and cross-component understanding without forcing domain payloads into one schema | **Agreed direction; schema TBD** |
| D-007 | Use inherit/augment/override/remove for goals, policies, knowledge and configuration | Keeps a consistent resolution model and avoids irrelevant or false context while preserving provenance | **Agreed direction** |
| D-008 | Record unknown/unsupported cases and tool coverage even on success | A technically successful run can still contain blind spots; those gaps drive tool improvement | **Agreed direction** |
| D-009 | Persist successful and failed iterations, plus reusable lessons, and record decisions **with reasons** | Prevents repetition of failed approaches; enables audits, regression diagnosis, reversals and learning | **Agreed direction** |
| D-010 | GitHub-first, human-readable knowledge and decision records; external storage for large/sensitive evidence | Easy inspection and history without unnecessary database infrastructure or bloated repos | **Agreed direction** |
| D-011 | Central shared knowledge plus repository-specific augmentation, overrides and applicability conditions | Shares learning across many repos without injecting irrelevant context or overriding valid local constraints | **Agreed direction** |
| D-012 | Apply human review and approval requirements defined by the relevant governance profile | Human review provides accountable oversight; model scores do not replace required approval | **Fixed initial policy** |
| D-013 | Integrate with each project's approved release lifecycle and keep required source and deployment approvals distinct | Respects local governance and avoids assuming or duplicating deployment machinery | **Fixed initial integration target** |
| D-014 | Favor early foundational dependency upgrades and track transitive impacts | Avoids version drift and costly postponed migrations; failing consumers can be investigated in parallel | **Agreed direction** |
| D-015 | Isolate local runs with containers/emulators and report fidelity gaps | Fast, inexpensive early testing with explicit awareness of differences from cloud environments | **Agreed direction** |
| D-016 | Use representative subsets for early tests; make selection explainable and risk aware | Reduces test cost while targeting diverse and high-risk patterns; broader testing remains available | **Agreed direction** |
| D-017 | Make prioritization, thresholds, budgets, partial completion, and AI autonomy configurable per run | Different goals warrant different cost/risk/quality trade-offs; humans control early operation | **Agreed direction** |
| D-018 | Increase autonomy only from verified capability-specific evidence; revoke after regressions | Aggregate accuracy can hide mission-critical errors; trust must be bounded and reversible | **Agreed direction** |
| D-019 | Resolve policy conflicts with human review initially, recording structured rationale | Avoids silent policy choices and allows future automation to learn within approved authority limits | **Agreed direction** |
| D-020 | Keep central orchestration a long-term goal, not the first component | Foundational flows must mature and expose real coordination requirements before architectural commitment | **Agreed direction** |
| D-021 | Initially prefer risk/impact per effort, while allowing run-specific priority overrides | A fast improvement can provide value sooner, but security impact, overlap and constraints can change the optimal choice | **Agreed configurable default** |
| D-022 | Create PRs with unresolved findings attached rather than suppressing them | Human review may supply missing context; ambiguity must remain visible and feed improvement loops | **Agreed direction** |
| D-023 | Treat human feedback as evidence for cross-repo tool improvement, not an automatic global policy | A locally correct reviewer explanation may not generalize; broad adoption requires validation and approval | **Agreed direction** |
| D-024 | Maintain independent source/release provenance for deployable repositories and library chains | Published dependencies are not necessarily adopted or running in production; need exact lineage | **Agreed direction** |

### Proposed mechanisms still subject to validation

- **P-001:** Exact file/folder layout, run manifest and decision-record schema, ID/versioning conventions.
- **P-002:** Weighted set-cover or another selection algorithm for representative repositories.
- **P-003:** Specific confidence calibration, risk scores, thresholds, and rollback triggers.
- **P-004:** How to determine that an AI-generated rule is valid over sufficiently diverse cases.
- **P-005:** Exact contract compatibility guarantees, adapter generation, deprecation windows, and retirement authority.
- **P-006:** Which evidence is retained in Git versus external storage, and enforceable retention/authorization per data classification.
- **P-007:** How multi-parent precedence is resolved when no authorized policy decides it automatically.
- **P-008:** Details of goal completion scoring, mandatory children, partial success and dependency propagation.
- **P-009:** Choice of initial target repository/workload and minimal code stack for the foundational slice.
- **P-010:** What data must be redacted before model use and what vendor/model configurations are approved for regulated tasks.

## 12. Decision record template (to be used for every meaningful choice)

```yaml
id: ADR-0001
status: proposed # proposed | approved | superseded | rejected | reversed
scope: {goal_id: GOAL-001, repositories: []}
decided_at: null
owner: null
approvers: []
question: "What decision is needed?"
context: "Business and technical conditions; what triggered this decision."
constraints: ["Mandatory requirements and limits"]
options:
  - {id: option-a, summary: "...", anticipated_benefits: [], risks: [], evidence_refs: []}
  - {id: option-b, summary: "...", anticipated_benefits: [], risks: [], evidence_refs: []}
selection: null
why: "Why this option was selected rather than alternatives."
assumptions: []
uncertainties: []
policy_versions: []
inputs: []
expected_outcomes: []
validation_plan: []
observed_outcomes: []
impact: {risk: null, cost: null, latency: null, coverage: null, correctness: null}
related_runs: []
related_pull_requests: []
revisit_triggers: []
supersedes: null
```

**Minimum audit rule:** each significant choice should answer *what, why, alternatives, evidence, authority, result, and what would cause us to revisit it*. Decisions can be made by humans, deterministic policies, or approved AI-assisted workflows, but the actor, authority, and limits must be explicit. These records contain **decision explanations and observable evidence**, not purported complete private model thought traces.

## 13. Initial implementation acceptance tests

A foundational implementation is credible when a reviewer can:

1. Run one simple goal end-to-end locally, with a pinned input manifest, a deterministic tool and measured evaluation.
2. See a status that differentiates full pass, partial success, failure, blocked, unknown and unsupported, without hiding incomplete analysis.
3. Compare at least two attempts, including one failed attempt, and determine precisely *what changed and why*.
4. Inspect machine-readable and human-readable evidence records pointing to exact tested Git revisions and external artifacts if any.
5. Verify inherited controls and repository-specific overrides, including provenance and a failed unauthorized mandatory override.
6. Review a proposed improvement and a generated PR summary showing tests, trade-offs, known risks and approval requirement.
7. Confirm no auto-merge or production bypass and no credentials/sensitive evidence committed.
8. Select AI-off, AI-on-request, and (later) approved bounded-auto modes without changing the shared lifecycle contract.
9. Observe elapsed time, resource use, confidence/coverage limits, and model cost separately, without conflating throughput with correctness.
10. Re-run a known regression scenario and see that its lesson/evidence is linked to the subsequent change.

## 14. Open questions for later sessions

1. Exact minimal goal envelope and schema/extension system to implement first.
2. Local tech choices for the first runnable evaluation/ledger slice; how an agent runtime is invoked versus plain deterministic scripts.
3. How to represent executable acceptance criteria, domain-specific rubrics and human sign-offs.
4. Approval policy for skill/tool-generated patches and contract migration PRs across multiple repositories.
5. Storage identity/retention for sensitive enterprise evidence and cross-repo audit queries.
6. Human override expiry and event-triggered re-evaluation rules.
7. Thresholds for promoting or demoting capability-level autonomy and handling high-severity false negatives.
8. When to introduce parallel execution, state-machine durability, retries, and central orchestration.

## Document quality review — v1.1 editorial decisions

**Scope guard:** This revision improves clarity, organization, provenance, and testability of the *documentation*. It does not authorize or add harness functionality. Any new capability ideas are isolated in the future-considerations section below.

| Review ID | Editorial decision | Why the change was made | Effect on agreed design |
|---|---|---|---|
| QR-001 | Promote the consolidated architecture to the primary body; preserve chronological v0.1–v0.4 material as a reference appendix | Readers previously encountered older, partly superseded notes first and could mistake them for the current design | None; organization only |
| QR-002 | Explicitly label agreed direction, fixed initial policy, candidate mechanism, and open question | A draft should not suggest that example schemas or recommendations are approved implementations | None; status clarity |
| QR-003 | Explain that numerical thresholds, schemas, repository layouts and examples are illustrative unless approved | Prevents sample values from becoming accidental engineering requirements | None; precision only |
| QR-004 | State the human PR and production-promotion boundaries together | Avoids confusing automated *deployment after approval* with authority to approve or merge | None; resolves ambiguity |
| QR-005 | Clarify Git versus external evidence storage and limits of reproducibility | Prevents implying that sensitive/large evidence belongs in Git or that AI responses replay exactly | None; accuracy only |
| QR-006 | Preserve the decision register and prior change history rather than silently rewriting earlier rationale | Maintains an audit trail for how the design evolved | None; traceability only |

### Clarifications used throughout the document

- **Agreed direction:** An architectural intent explicitly accepted during discussion, subject to subsequent detailed design and governance review.
- **Fixed initial policy:** A process selected for an example context; it is not assumed to apply to every project.
- **Proposed mechanism:** An implementation approach to investigate; it is *not* a decided technology choice.
- **Evidence:** Observable inputs, tool calls/results, tests, model-visible explanations when supplied, and recorded decisions. This does not imply access to a model's complete private reasoning.
- **Replay:** Reconstruction of run configuration, versions, and evidence. Exact reproduction of nondeterministic model responses is not guaranteed.
- **PR lifecycle:** An agent may prepare a PR with unresolved findings; human approval is required before merge. Deployment steps and approval boundaries depend on each project's documented governance and release process.

## Future considerations — not part of the agreed implementation scope

These are **candidates for future discussion only**, not features added or approved by this document revision:

- Build tooling to check consistency across decision records, manifests, and policy citations.
- Generate a human-readable change-impact report when decisions are revised or superseded.
- Provide a concise reviewer summary alongside the full archival evidence to reduce audit-review effort.

Each candidate requires an explicit decision record before it moves into the design or roadmap.

## 15. Change log

- **v1.1 — 2026-11-14:** Editorial review: promoted consolidated design to primary body, retained earlier notes as history, clarified approval/evidence boundaries and example status, and documented the reasons for each revision. No new functionality approved.
- **v1.0 — 2026-11-14:** Consolidated subsequent conversation into a recursive goal-driven lifecycle, deterministic-first capability evolution, comprehensive contracts and knowledge inheritance, evaluation-first implementation plan, GitHub-first evidence and decision rationale, governance/release integration, environment fidelity, solution lifecycle, adaptive autonomy, and formal decision register with explicit **why** fields. Existing v0.4 sections retained for historical continuity.

---

## Appendix A — Chronological notes from versions 0.1–0.4

These notes are preserved for historical traceability. When wording differs, use the consolidated design and decision register above.

### Original v0.4 notebook (historical reference)

**Status:** Working draft, version 1.0  
**Created:** November 14, 2026  
**Purpose:** Capture our evolving design discussion for a customizable, secure, evidence-driven AI agent harness built from existing tools or custom components. This is a living reference, not a finalized architecture.

#### A1. Vision

Build a hierarchical agentic software-engineering system that uses AI models for reasoning and decisions, deterministic programs for repeatable analysis and verification, and a harness for orchestrating tools, sessions, constraints, quality gates, and feedback. Support codebases and distributed systems of varying size, with measurable outcomes, bounded autonomy, and human oversight.

#### Foundational distinction

- **Model:** Interprets context, reasons, generates recommendations and code, and requests tool calls.
- **Harness:** Manages the agent loop, context, tool dispatch, sessions, event handling, permissions, and coordination. The harness controls execution capabilities, not the model's internal thinking.
- **Tools:** Deterministic scripts and other callable programs that inspect systems, collect evidence, perform checks, and execute authorized changes.
- **Orchestration layer (custom):** Coordinates multiple agents and nested control loops. An existing agent runtime can provide building blocks, but the proposed hierarchical system may require additional custom orchestration.

#### A2. Design principles established in discussion

1. **Deterministic first.** Use repeatable scanners and scripts to extract structured facts from codebases rather than repeatedly asking models to read every source file. AI consumes the structured outputs and applies reasoning only where necessary. This can reduce latency and token cost and improve consistency.
2. **Decompose and compose.** Break complex work into independently verifiable subtasks; compose them into feature-, service-, product-, and system-level results.
3. **Evidence-based evaluation.** Check requirements using both AI reviews guided by explicit rubrics and deterministic verification (tests, policy checks, benchmarks, security scanners).
4. **Separate hard gates from weighted objectives.** Critical security, correctness, and compliance requirements must pass; favorable performance scores cannot compensate for failed hard gates.
5. **Least privilege.** Apply capability and OS/container boundaries regardless of model instructions. Review-only agents receive read-only access; coding agents receive narrowly scoped write access.
6. **Bounded loops.** Put time, iteration, cost, and resource budgets on all agent workflows. Escalate or change strategy after repeated failure.
7. **Observability and human review.** Capture task inputs, model-visible outputs, tool invocations/results, tests, diffs, diagnostics, metrics, and decisions. Do not assume access to a model's complete internal reasoning.
8. **Protect the definition of success.** An improvement agent must not quietly weaken acceptance tests or requirements to make a failing solution pass.
9. **Progressive verification.** Run cheap static and unit checks before costly integration, end-to-end, fault-injection, and performance tests.
10. **Risk-tiered autonomy.** The amount of unattended action permitted should reflect impact and reversibility, not only model confidence.

#### A3. Proposed agent hierarchy

```text
Business / engineering goal and non-negotiable constraints
                         |
              System orchestration loop
                         |
             Decompose into bounded tasks
                         |
       +-----------------+------------------+
       |                                    |
  Task execution loop                  System evaluation
  - Coding agent                       - Cross-service contracts
  - Review agent (read-only)           - End-to-end invariants
  - Deterministic tools                - Quality / performance
  - Targeted tests                     - Security / observability
       |                                    |
       +-----------------+------------------+
                         |
                 Results and evidence
                         |
               Diagnostic / meta-loop
               - Analyze repeated failures
               - Test hypotheses
               - Propose improvements to
                 prompts, tools, workflows
                         |
                  Approval / validation
                         |
             Controlled retry or escalation
```

#### Roles

| Role | Responsibilities | Indicative permissions |
|---|---|---|
| Orchestrator | Break down work, assign subgoals, coordinate dependencies, enforce budgets and gates | Workflow control; no default broad filesystem privileges |
| Coding agent | Implement scoped changes, address review findings | Read/write only in assigned workspace |
| Reviewing agent | Evaluate quality against requirements and rubrics | Read-only code/artifacts; ability to produce findings |
| Deterministic tool runner | Scan code, run tests, collect metrics and evidence | Least privilege per command/tool |
| Diagnostic agent | Investigate competing root-cause hypotheses | Mostly read-only access to evidence and relevant sources |
| Improvement/meta-agent | Propose better tools, prompts, evaluation or decomposition | Proposal-only by default; cannot change protected tests without approval |
| Human approver | Resolve ambiguous requirements, approve consequential changes, adjudicate uncertain failures | Explicit approval authority |

#### A4. Inner loop: implementation and review

1. Receive one clearly defined task with required inputs, constraints, and acceptance criteria.
2. Generate a plan and identify minimal required evidence.
3. Implement changes in a scoped working tree/container.
4. Run deterministic checks: formatting, static analysis, unit tests, contract tests, security checks as appropriate.
5. Send code diff plus structured evidence to a read-only reviewing agent.
6. Classify findings by impact and confidence; return actionable feedback to the coder.
7. Repeat only within time, resource, iteration, and spend budgets.
8. Publish a structured result: accepted, needs another step, blocked, or escalated.

**Important:** A passing test percentage alone does not imply corresponding real-world correctness. Measure the right properties and track failed critical gates separately.

#### A5. Outer diagnostic and improvement loops

Triggered by stalled progress, repeated failed invariants, unexpected costs, regression, or poor evaluation quality.

- Collect full observable execution history: starting inputs, policies, artifacts, model-visible response summaries, tool calls/results, diffs, test outputs, timings, budgets, and state transitions.
- Cluster repeated failures and distinguish unclear specifications, broken tools, incorrect tests, service defects, interface defects, and architectural assumptions.
- Form competing hypotheses, assign discriminating tests, and allow **insufficient evidence** as an outcome.
- Where justified, propose changes to prompt context, decomposition, deterministic analysis scripts, tool logic, or evaluation procedures.
- Validate changes against independent/protected acceptance checks before enabling them in the task loop.
- Human approval for modifications to protected criteria, permissions, security-sensitive code, or production infrastructure.

**Failure mode to prevent:** An improvement agent changes tests or scoring rules to make a poor solution appear successful (specification gaming / reward hacking).

#### A6. Security and execution boundaries

- Model instructions guide behavior but are not security controls.
- Harness extensions/tool hooks can inspect or reject requests; do not rely on simplistic textual command allowlists. `python -c` or shell redirection can perform operations without a blocked command name.
- Enforce filesystem and network restrictions at the execution boundary (e.g., separate Linux identities, containers, read-only mounts, scoped credentials, or stronger sandboxes as the threat model requires).
- On Linux, file deletion generally depends on permissions on the containing directory, not just the file's mode bits.
- Minimize credentials, secrets exposure, ambient cloud IAM rights, writable mounts, and outbound network access.
- Keep reviewers read-only and proposal-generating improvement agents separate from approvers.
- Log security-relevant decisions without placing secrets into the audit trail.
- Treat containers as a useful isolation layer, not an automatic security guarantee; configuration, capabilities, mounts, and runtime privileges matter.

#### A7. Deterministic codebase intelligence

**Existing working approach described in conversation:** Use AI to help author non-AI scripts that scan selected sections of codebases, extract useful structure, and produce information more cheaply and quickly than repeated full-codebase model reads.

Potential reusable extractors:

- Repository/module/package maps and dependency graphs.
- Service entry points, routes, RPC contracts, API schemas, and message/event boundaries.
- Data schemas and ownership mappings.
- Security posture, vulnerable dependencies, policy deviations.
- Observability: structured logs, metrics, traces, correlation IDs, failure paths.
- Complexity, resource, and performance signals.
- Change impact: affected services, contracts, tests, and rollout risks.

**Proposed extension:** Machine-readable architecture knowledge graph or catalog kept fresh by these scanners, with provenance, commit/version identity, and freshness indicators. Agents query scoped facts rather than rediscovering architecture on every task.

#### A8. Quality and acceptance framework

#### Candidate hard gates

- Mandatory security and compliance controls.
- Required business invariants and correctness tests.
- Interface compatibility and data integrity.
- Authorized filesystem/cloud access boundaries.
- Reproducibility and evidence availability.

#### Candidate weighted objectives (conditional on passing gates)

- Latency, throughput, scalability.
- Memory, CPU, network, and disk use.
- Cloud resource and model-token costs.
- Observability and diagnostic quality.
- Maintainability, complexity, test depth, and operational burden.

Define each metric's measurement method, baseline, threshold, weight, acceptable uncertainty, environment, and owner. Avoid rolling unrelated measurements into one score without transparent tradeoffs.

#### A9. Distributed-system diagnosis example

**Environment used for illustration:** A distributed service system with multiple independently deployed components and a transactional datastore.

**Example workflow:** Order service validates/creates an order; inventory service reserves stock; payment service processes payment.

**Protected invariant:** Never confirm an order unless inventory is reserved and payment succeeds.

If all services pass unit tests but the end-to-end invariant fails:

1. Reproduce the failing end-to-end workflow and capture a correlation ID.
2. Collect traces, structured logs, API/event payloads, versioned service contracts, and relevant datastore state and transaction evidence.
3. Compare the actual timeline and states with the intended workflow contract.
4. Form hypotheses:
   - Local service bug (wrong state transition).
   - Interface/contract mismatch (different meaning of a status or event).
   - Architecture-level flaw (unclear ownership, missing invariant, incorrect transactional assumption).
5. Design targeted tests that distinguish hypotheses; do not repair code before isolating the likely cause.
6. Recheck both the local tests and the protected end-to-end invariant after any fix.

**Caution:** Consistency guarantees within one datastore do not automatically provide atomicity across independently committed operations or services.

#### Test ladder

1. Static checks and unit tests.
2. API/event contract tests.
3. Two-service/component integration tests.
4. End-to-end business-invariant tests.
5. Fault injection: retries, partial failures, race conditions, timeouts, duplicate events.
6. Load/soak and cost/performance evaluation when warranted.

#### A10. Practical implementation with an existing agent runtime

The architecture in this notebook is a **proposed system design**, intended to be adapted to an existing agent harness or runtime where practical. Verify implementation details and version-specific APIs against the selected tools before building.

Potential construction order:

1. Establish a constrained agent session/runtime and a reproducible task sandbox.
2. Create deterministic scanners returning versioned, structured outputs.
3. Define task contracts, evidence schemas, hard gates, and scoring rules.
4. Add a bounded coding/review/test loop.
5. Add traceable execution events, diffs, test reports, and an escalation bundle.
6. Build a read-only diagnostic agent that compares hypotheses.
7. Add a proposal-only improvement loop and protected evaluation suite.
8. Expand to multiple services/products with dependency-aware decomposition.
9. Introduce approval tiers based on risk, data sensitivity, reversibility, and deployment scope.

#### A11. Open design questions

1. What is the exact structured representation for goals, constraints, subtask contracts, and acceptance criteria?
2. Which checks are always mandatory and which are weighted objectives? Who can modify either category?
3. How should orchestration budgets be allocated across tasks, agents, models, and loops?
4. How are context snapshots and tool outputs versioned so evaluations are reproducible?
5. How do we distinguish tool defects from product defects without allowing automated weakening of tests?
6. What should be the default recovery strategy after repeated attempts: isolate evidence, change model, change tool, replan, or escalate?
7. What level of automated repair is safe for sandbox code versus shared repositories or deployed services?
8. How should the knowledge graph represent architecture assumptions and confidence/freshness?
9. How do we prevent cross-service drift and coordinate concurrent agent writes?
10. How should we benchmark improvements to the harness itself against a fixed suite of representative tasks?

#### A12. Next discussion topic

**Risk-based autonomy and authorization:** Should a diagnostic agent's proposed fix proceed automatically when evidence is strong, or should authorization depend primarily on blast radius, reversibility, trust boundaries, and required approvals? How are those thresholds expressed as deterministic policy?

---

#### A13. Deterministic-first execution and evidence-driven improvement (discussion update)

#### Guiding objective

The **end solution should use as little AI at runtime as practical**. Prefer deterministic, fast, testable code whenever the behavior can be expressed reliably in software. AI is principally a development-time and exception-analysis resource: discover missing cases, propose rule/tool/skill changes, and help convert recurrent model reasoning into repeatable code. A fully deterministic run is a success even if a later run reveals a new unsupported case; that exception should inform the next improvement cycle.

Two performance dimensions should be measured independently:

- **Runtime solution performance:** latency, throughput, memory, disk, network usage, reliability, and AI calls/tokens/cost on the actual production path.
- **Development/improvement-loop performance:** time to diagnose and repair exceptions, test execution time, model tokens/cost, human review burden, regressions, and improvement lead time.

Optimization goal: steadily reduce runtime reliance on AI *without* weakening correctness, security, coverage, or maintainability. A deterministic classification should not be accepted merely because it is cheap: unknowns must stay visible.

#### Human-reviewed decision learning

At first, decisions about whether to extend an existing tool, create a specialized one, revise a skill, or change task decomposition are **case-specific and made with human review**. Save for each decision: triggering evidence; candidate strategies; tradeoffs (risk, accuracy, coverage, performance, complexity, cost); test outcomes; selected option; explanation/rationale available from the model or human; reviewer/approval; and the actual observed outcome. Do not assume access to private, complete model internal reasoning. Over time, turn repeated validated decisions into explicit policy while retaining an override/approval route.

#### Instrument the deterministic runtime itself

Each scan/tool should emit a versioned structured result including run ID, tool/skill version, repository revision, elapsed time and resource usage, counts of classified/unsupported/unknown cases, specific unknown pattern examples with file locations, diagnostics/error categories, and relevant artifacts/traces. Avoid storing secrets or unnecessarily large source snippets. Group/reduce repeated errors into fingerprints to support incremental triage.

The improvement cycle is: **run deterministic tool → capture unknowns/failures and telemetry → classify root cause → generate alternative changes → test against preserved examples and independent acceptance criteria → human approve when needed → release/version tool or skill → measure whether coverage and runtime performance improved**. The meta-loop must not silently change protected quality gates or relax tests to manufacture success.

#### Provisional design choices

- Optimize for deterministic execution first; AI assistance should retreat from the production path when repeatable rules cover the case.
- Make observability an explicit acceptance criterion of generated tools, not a post-hoc feature.
- Treat rule/skill/tool selection as a tradeoff requiring measured comparison and initially human review; there is no universal default answer.
- Preserve machine-readable decisions and observable evidence for both human inspection and later AI analysis.
- A future policy engine can progressively automate well-understood, low-risk cases; keep exceptional and high-impact decisions reviewable.

#### Further measurable indicators

- Autonomous deterministic handling rate; AI fallback rate; unknown/unsupported rate.
- P50/P95/P99 latency and resource/cost per completed task.
- False-positive/false-negative rates against labeled examples.
- Regression count and coverage on real-world edge cases.
- Percentage of improvement proposals accepted and validated; human minutes per improvement.

#### Immediate next architecture question

Define the **fallback contract**: when deterministic tools encounter an unknown, do they abstain and emit an evidence bundle; invoke a read-only diagnostic model; or request human review? Which thresholds govern each path, and how are the outcomes used to produce a tested deterministic improvement?

#### A14. Run-configurable improvement prioritization (discussion update)

#### Decision

**Impact per unit effort is a recommended default, not a fixed global rule.** Each invocation of an execution, diagnostic, or improvement loop should accept a **prioritization policy as an input**, enabling the human or calling workflow to change the tradeoffs for that run. Preserve the policy, candidate measurements, selected action, rationale, and outcome in the audit record.

Example from the discussion: when comparing a candidate that resolves approximately **300 unknown cases in two hours** with one resolving **500 cases in two weeks**, prioritize the two-hour improvement by default, provided both satisfy mandatory quality, security, and regression gates. Estimates are not guaranteed successes: validate the actual coverage gain after implementation. Re-rank the remaining candidates after each improvement because pattern groups may overlap.

#### Suggested per-run policy inputs

- **Primary objective:** fastest validated coverage gain, maximum absolute coverage, security-risk reduction, cost efficiency, highest correctness, or another specified goal.
- **Weights:** expected correctly resolved cases, severity/business impact, engineering time, operational cost, runtime overhead, maintainability, confidence in estimates, and reusability across products.
- **Hard constraints:** security/correctness gates, protected tests, time/deadline, resource and spending budgets, required human approvals, and forbidden changes.
- **AI allocation:** AI-led analysis permitted/required/forbidden for designated steps, with per-run budgets and fallback modes. Deterministic-first remains a preference when the task permits, not an unconditional rule.
- **Risk posture:** allowable blast radius and reversibility, and whether uncertain changes are deferred or escalated.
- **Stop/reconsider conditions:** diminishing marginal coverage gain, regression, breached budget, insufficient confidence, or a higher-priority newly discovered issue.

#### Proposed prioritization sequence

1. Capture task goals and the per-run policy before a loop starts.
2. Reject proposals violating non-negotiable gates; do not allow a high weighted score to cancel a gate failure.
3. Estimate incremental **correct** coverage, effort, runtime impact, risk, and uncertainty for each remaining candidate.
4. Rank by the configured goal/weights; use expected validated impact per effort as the default when no other preference is supplied.
5. Execute under bounded budgets, validate against an independent test corpus, and record the observed gain.
6. Recompute rankings after each cycle; allow human override with recorded explanation.

#### Example input (illustrative, not a finalized schema)

```yaml
improvement_policy:
  objective: validated_coverage_gain_per_engineering_hour
  weights:
    correct_coverage_gain: 0.45
    engineering_effort: 0.30
    severity_reduction: 0.15
    runtime_cost: 0.10
  hard_gates:
    security_regressions_allowed: false
    protected_acceptance_tests_must_pass: true
  max_improvement_hours: 4
  ai_analysis: allowed_with_budget
  require_human_approval_for_tool_changes: true
  rerank_after_each_iteration: true
```

The illustrative weights are policy choices, not universal constants. They should be validated against actual outcomes; if a goal or weight changes, that should be reflected in the individual run record.

#### Open question

How should different policy profiles be selected and versioned by task type (e.g., rapid coverage recovery vs. security-critical investigation), while allowing an explicit override on each run?

#### A15. Git-first provenance, iteration snapshots, and reversible decisions (discussion update)

#### Decision

**Git is the default system of record for inputs, source changes, evaluation evidence, analysis, and decisions across each iteration.** Every reproducible loop execution must have a traceable identity and be linked to exact source, tool, skill, policy, and acceptance-test revisions. Capture the baseline, what changed, what ran, observed results, assessments, and the reason for the next action. Preserve failed runs as first-class records, not just successful outputs.

#### Proposed repository organization

```text
harness-repo/
  README.md
  docs/
    architecture.md
    decisions/                  # ADR-0001-...md; decisions and reversals
  policies/
    default.yaml
    profiles/
  skills/
    ...                         # Versioned skill definitions
  tools/
    ...                         # Deterministic scanners, schemas, test helpers
  tests/
    protected/                   # Permission-controlled acceptance corpus
    regression/
  runs/
    <run-id>/
      manifest.yaml             # Parent run, task, commit SHAs, tool/skill/model IDs
      inputs/                   # Sanitized goal, criteria, policies, context references
      iterations/
        0001/
          plan.json
          actions.jsonl        # Tool requests/observable results, redacted
          metrics.json
          results.json         # Test/scan output and coverage summary
          evaluation.json      # Findings, hard gates, scores, uncertainty
          decision.md          # Chosen next action, alternatives, justification
        0002/
          ...
      summary.md
  schemas/                      # Machine-readable artifact contracts
```

**Storage policy:** Commit concise, sanitized, reproducible text artifacts and manifests. Large logs, traces, snapshots, binaries, generated data, and sensitive evidence should live in governed artifact storage; commit immutable content hashes/URIs and retention rules instead. Never commit secrets, credentials, personal data, or vulnerable raw payloads. Configure secret scanning and `.gitignore`/artifact policies from day one.

#### Proposed commit and iteration protocol

1. **Baseline:** Record the starting Git commit SHA, task/goal, criteria, run policy, tool/skill versions, approved environment and data references. Commit the run manifest and baseline evidence.
2. **Execute:** Work in a dedicated branch or worktree. Run the tools and model-mediated steps; collect structured traces, outputs, performance metrics, test results, and classifications.
3. **Evaluate:** Compare against hard acceptance gates and weighted goals. Persist evidence plus a decision record describing the observed reasoning/justification, alternative hypotheses, confidence, and next step. Do not claim complete private model reasoning is available.
4. **Checkpoint:** Commit the iteration's source diff and **sanitized** evidence together when feasible. Refer to the exact tested tree SHA; if artifacts are finalized after code execution, a separate evidence commit must explicitly link to the tested SHA. Never imply a later metadata-only commit was the exact code under test.
5. **Improve and repeat:** Record the next tool/skill/policy/code change, rerun appropriate checks, and create another iteration checkpoint. Link child iteration to parent and document why it was attempted.
6. **Promote:** Merge only after protected gates pass and required review completes. Include a summary of validated gains, regressions, residual unknowns, cost, and approvals.

#### Traceability and historical diagnosis

- Treat **Git SHAs, run IDs, iteration IDs, artifact hashes, policy revisions, and test-suite revisions** as first-class join keys.
- Maintain an explicit **decision log (ADRs)**: context, hypotheses, alternatives, measurements, choice, approver, downstream consequences, and a `supersedes`/`reverses` link when a decision changes.
- Enable comparison of any two iterations: code and skill diffs, new/fixed regressions, acceptance-gate changes, AI usage, runtime performance, and unknown-pattern coverage.
- When a higher-level security or performance failure emerges, use those links to identify the earliest relevant change, replay from a known-good revision, and investigate whether an architectural assumption was invalid.
- Prefer **revert or a new corrective branch** to destructive history rewriting. Keep failed and reversed paths for future analysis. Where replay is impossible (external APIs, mutable datasets, model variance), preserve environment metadata and explicitly label reproducibility limitations.

#### Concurrency and integrity guardrails

- Isolate agent writers with branches/worktrees and merge gates. Read-only review agents inspect immutable revisions, not a concurrently changing directory.
- Protect baseline acceptance tests, approvals, policies, and audit records with repository permissions/CI; agents should not be able to rewrite their own pass criteria unnoticed.
- Make runs append-only by convention (and preferably enforce immutability in evidence storage), with signed commits/provenance where warranted.
- Use CI to validate manifests, referenced hashes, reproducibility metadata, and secret scanning before merging.
- Use explicit retention and aggregation for long-lived multi-product operation; a Git repository should not become an unbounded log database.

#### Next design choices

1. One repository versus separate control-plane, product-code, and evidence repositories (with cross-repo SHA linking).
2. Exact commit granularity: baseline, each iteration, and/or tested-change plus evidence commits.
3. Minimum reproducibility contract for data snapshots, external service state, model/version settings, and infrastructure.
4. Policy for log size, sensitive material, evidence retention, and cryptographic integrity.


## Earlier revision history (retained)

- **v0.4 — 2026-11-14:** Added Git-first provenance, run/iteration repository layout, checkpoint protocol, immutable evidence references, decision reversal history, branch/worktree isolation, and CI/integrity safeguards.

- **v0.3 — 2026-11-14:** Added run-configurable prioritization policies, impact-per-effort default, mandatory gates, per-run AI allocation, and measurable re-ranking after each improvement.

- **v0.2 — 2026-11-14:** Added deterministic-first runtime objective; separate runtime vs development performance; instrumented exception/coverage reporting; case-specific human-reviewed improvement decisions; evidence and policy-learning loop.
- **v0.1 — 2026-11-14:** Consolidated discussion of models vs harnesses; least-privilege agent roles; nested execution/evaluation/meta-loops; deterministic codebase scanners; measurable acceptance criteria; evidence/auditability; root-cause isolation across distributed services; next architectural decisions.
