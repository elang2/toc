# Stream plan: governance, observability, security and policy

Technical chapter: [Chapter 06](../06_Governance_Observability_Security_and_Policy.md)
Stream owner: Elankumaran Srinivasan (@elang2), as recorded in the [initiative README](../README.md#workstreams)
Team: Elankumaran Srinivasan (@elang2), owner, confirmed; Sankalp Gilda (@astrogilda), 6a author, confirmed; Anca Sailer, confirmed for OSCAL Compass; contributor seats open (see section 2)
Status: scoping
Last updated: 2026-11-18
Next walkthrough: November 19, 2026 session

## 1. Purpose and intended outcome

Platform teams running agents in Kubernetes can already see that a call happened. Three questions they cannot answer matter once an agent spends money and acts on a user's behalf. What did this run cost, in units an operator can add up across providers? Did the limit that was configured actually stop it, or did the script simply end? And can a party who does not trust the operator check either answer from what was retained?

This chapter helps platform engineers, operators and auditors decide where identity, authorization, policy and operational limits are enforced, and what telemetry and evidence have to exist for those decisions to be reviewable afterwards.

What differs from observability generally. A policy decision and an observed effect are separate facts, so an allowed call is not evidence the effect happened. A provider-reported cost and a locally computed one are indistinguishable unless the record says which. And an agent's own producer is usually the only witness, which is the condition under which a completeness claim needs a population the producer cannot edit.

Minimum useful outcome for v0.1, within about two pages:

1. Cross-stream control responsibilities: where agent identity, delegation, approval and least-privilege policy are enforced across gateway, runtime, state and this chapter, with the unowned gaps named rather than assumed covered.
2. Telemetry and cost guidance with its limits stated: what an implementer records so a cost figure is comparable and a declared budget is checkable, and what the current conventions do not provide.
3. Verifiable per-call evidence: a per-call record a reader who does not trust the producer can check from retained bytes, with each pattern's blind spots declared.
4. A short list of evidence-backed gaps for chapter 08, each tied to a named upstream destination.

The chapter does not define attribute names, select products, or treat telemetry as proof of compliant behaviour. Naming conventions belong to the OpenTelemetry GenAI SIG; this chapter states the requirement and leaves the spelling to them.

## 2. Team and expertise

| Person / handle | Expertise / perspective | Intended contribution | Participation |
|---|---|---|---|
| Elankumaran Srinivasan (@elang2) | OpenTelemetry GenAI conventions, cost and budget measurement across agent frameworks, differential testing | Owner; scope, 6b telemetry schema and cost, cross-stream coordination | Confirmed |
| Sankalp Gilda (@astrogilda) | Verifiable evidence, in-toto and DSSE, test-vector corpora | 6a verifiable per-call evidence ([#2369](https://github.com/cncf/toc/pull/2369)) | Confirmed, for 6a |
| Anca Sailer | Safety guardrails, compliance as code and policy as code; OSCAL Compass as an implementation | OSCAL Compass into this stream, per the [October 8 action](../meeting-notes/2026-10-08.md); the wider policy and compliance subsection is scope to confirm with her | Confirmed, for OSCAL Compass |
| Open seat | Workload identity, delegated authority, authorization | Control-responsibility mapping for identity and delegation; the boundary with chapter 03 | Open |
| Open seat | Security practitioner, ideally with agent deployments in production | Threat review of the control mapping; what the evidence patterns do not cover | Open |
| Open seat | End-user operator running agents at scale | Whether the telemetry guidance answers an operator's actual questions, and what it costs to emit | Open |

- Missing expertise or perspectives: workload identity and authorization; security practitioners; and operators running agents in production who can say what they currently cannot measure. Cross-stream reviewers from protocol, gateway, runtime and state.
- Recruitment action: the owner will post a call in `#initiative-distributed-agentic-systems` naming a first task per open seat. This also closes the owner's outstanding action from the [October 22 session](../meeting-notes/2026-10-22.md), "identify the people working with them on each stream and coordinate in Slack". Checkpoint: contributors named, or the seat declared unfilled, by the second session after kickoff.
- How to join: reply in `#initiative-distributed-agentic-systems`, or comment on this plan's pull request naming the area you would like to take.
- First task for a new contributor: establish what the API server audit log actually records for an admission decision on a side-effecting tool call, and what its completeness depends on. 6a R4 permits an admission controller as the population a producer cannot edit. Admission control decides; the API server audit log records. Establish **both** bounds from the documentation, because they are different: webhook scope and `failurePolicy` bound which requests got a decision, and the audit policy bounds which were recorded. Fill one row of the control-responsibility mapping from the Kubernetes documentation with section references, and record what the documentation does not say. About an hour, and it needs no access to anyone's cluster.

Only people who have agreed are listed, and agreement means a session record or a public statement rather than a mention. Anca Sailer is listed because the October 8 notes record her presenting this material and carry an action assigning OSCAL Compass to this stream. That is agreement to contribute, and it is not agreement to own a v0.1 output, so her row names the action and marks the wider scope as still to confirm with her. A name that appears only in a chapter outline stays an open seat.

## 3. Scope and priority questions

- In scope: GenAI telemetry and MELT for agent runs; cost, token and latency measurement and its comparability; declared budgets and whether enforcement is checkable from a record; policy and compliance as code; verifiable per-call evidence; and the cross-cutting security implications of the above.
- Out of scope: vendor-specific controls, legal compliance guarantees, naming individual attributes, and treating telemetry alone as proof that behaviour was correct or compliant.
- Assumptions and terminology: distinguish user identity from workload identity; delegated authority from direct authority; a policy decision from an observed effect; operational telemetry from independently verifiable evidence; and a provider-returned amount from a locally computed one. Align these with chapter 07's shared terms before comparing mechanisms.
- Single-cluster baseline: explicit trust and tenancy boundaries between agents, tools, gateways and stores, with the enforcement point for each control named. Concretely: RBAC and SPIFFE/SPIRE carry workload identity and so back the delegation half of the control mapping; admission control is one of the separately controlled components 6a R4 permits as a denominator, and the API server audit log is what makes that population readable, since a webhook decides without recording; NetworkPolicy bounds which tools an agent can reach at all. Each stops somewhere, and the plan records where rather than assuming coverage. Which requests received a decision is bounded by webhook scope and `failurePolicy`; which requests were recorded is bounded by the audit policy; and no control-plane object records a cost or a budget verdict at all.
- Distributed considerations: identity federation, authorization behaviour when the policy service is unreachable, policy distribution, and whether telemetry and evidence remain continuous across clusters. Recorded as later investigations.
- Boundaries with other streams, proposed and to agree with each owner:
  - Protocol (chapter 02): protocol covers the security schemes, card signing and mandates the specifications define; this chapter covers whether those produce a record an untrusting reader can check.
  - Gateway (chapter 03): gateway leads on routing and proxy behaviour; this chapter covers what the gateway must record for interception-based evidence to be checkable. 6a already carries one bullet in chapter 03 for the response-phase test, flagged to that owner on [#2369](https://github.com/cncf/toc/pull/2369).
  - Runtime (chapter 04): runtime leads on workload identity issuance and lifecycle; this chapter covers what an identity has to assert for delegation to be reviewable.
  - State (chapter 05): state leads on durable storage and retention; this chapter covers which retained inputs an evidence claim depends on, and what happens to the claim when retention expires.
  - Reference architecture (chapter 07): control boundaries and the telemetry and evidence flows, so the same value is not specified twice.

| Priority | Question to answer | Intended output | Contributor |
|---|---|---|---|
| v0.1 | Where are identity, delegation, approval and least-privilege policies enforced, and which of those are currently unowned? | Cross-stream control responsibility mapping | Open seat, identity and authorization |
| v0.1 | Which telemetry supports task diagnosis and comparable cost measurement, and with what limits? | Semantic-convention and measurement guidance (6b) | @elang2, drafted, see section 5 |
| v0.1 | What evidence supports auditing a delegated action without exposing unnecessary sensitive data? | Evidence pattern, trust assumptions and trade-offs (6a) | @astrogilda, drafted, see section 5 |
| v0.1 | How are policy and evaluation decisions, and failure behaviour, made explicit and reviewable? | Governance decision guidance | Anca Sailer, per the October 8 action; scope to confirm with her |
| Later | Does any of this hold across clusters, or when the authorization or telemetry path is unreachable? | Recorded limitations for chapter 08 | To agree |

## 4. Starting knowledge and sources

| Source / project | Relevant section or capability | Why it matters / question informed | Version / date and evidence limitations |
|---|---|---|---|
| Kubernetes control plane | RBAC, the API server audit log, admission control (the webhook model, Kyverno, Gatekeeper), NetworkPolicy, and SPIFFE/SPIRE for workload identity | These are the existing capabilities this chapter's requirements rest on, so the chapter says what they already give and where each stops | Documented and widely deployed, not demonstrated here. Two separate bounds: webhook scope and `failurePolicy` bound which requests receive an admission decision; the cluster's audit policy bounds which requests are recorded. The API server logs a request whether or not a webhook ran on it |
| [Cloud Native Agentic Standards](https://www.cncf.io/blog/2026/03/23/cloud-native-agentic-standards/) | Observability, governance and security | Baseline to operationalise, not restate | Published 2026-03-23; recommendations need deployment context |
| [OpenTelemetry GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) | Agent and client instrumentation, token usage | The neutral vocabulary this chapter builds on | Record the convention version and stability with every claim; all GenAI stability declarations currently read `development`, so nothing here is frozen |
| [OpenTelemetry GenAI conformance report](https://github.com/open-telemetry/semantic-conventions-conformance) | What instrumented projects actually emit, per target | Separates what the conventions define from what implementations do | Pin the revision. Read at `6807b1d5e676`: of 57 instrumented GenAI targets, 3 emit a cost figure at all, under 12 attribute and 2 metric names spanning four namespaces, and the registry defines no cost attribute. The report carries **no date field at all** — its only provenance is `generated_by: otel-conformance-report build` — so the revision SHA is the only thing that dates an observation, and it dates the report rather than an emitter's release |
| [semantic-conventions#1598](https://github.com/open-telemetry/semantic-conventions/issues/1598) | Cloud Financial Management conventions | Shows the gap is acknowledged upstream and unclaimed | Open since 2024-11-20, labelled `triage:accepted:needs-sig`. Triage accepted it; no SIG has taken it |
| [FOCUS specification](https://github.com/FinOps-Open-Cost-and-Usage-Spec/FOCUS_Spec) | `BillingCurrency` column requirements | The currency constraint is settled vocabulary to borrow, not a new proposal | Read at tag `v1.0`. FOCUS governs billing exports, not runtime telemetry, so the constraint is borrowed and not inherited |
| in-toto, DSSE, IETF SCITT | Statement envelopes, canonical signed bytes | What 6a's signature and binding requirements rest on | Per 6a; record the exact profile and test-vector revision |
| OWASP Agent Control Standard | Agent control expectations | Cross-check for the control mapping | Draft status; treat as input, not as a settled standard |
| OSCAL Compass | Policy and compliance as code | The policy subsection's anchor, per the October 8 session | Capability is documented rather than demonstrated in this chapter; the reference architecture Anca described applies across regulations and risk taxonomies |

- Which Cloud Native Agentic Standards recommendations apply, and what are we adding? The observability, governance and security recommendations apply. What this chapter adds is the condition under which each is checkable afterwards: a cost figure that carries its unit and source, a budget whose enforcement can be falsified from a record, and a per-call record a reader who distrusts the producer can verify.
- What do Kubernetes and existing projects already provide, and what remains insufficient? Kubernetes supplies one of the three populations 6a's R4 permits. R4 requires a population a producer cannot edit and names an admission controller, a workload registry and a dispatch record as sources; admission control decides, and the API server audit log is what records the decision, so the admitted population is readable without trusting the agent. What remains insufficient is on both sides of that. A population derived from the log is complete to the audit policy's scope, and whether each member was evaluated is a separate fact the log carries only where the policy records admission annotations, so the two denominators must be stated separately. And nothing in the control plane records cost or a budget verdict at all, which is why 6b's requirements are about telemetry rather than about a Kubernetes object.
- Where do sources disagree, and which contact could help? Two places, both in this chapter's own material. The OpenTelemetry GenAI registry defines no cost attribute while three instrumented projects emit one anyway, and the two that emit a cost metric spell the currency differently. And FOCUS constrains currency for billing exports while the metric-unit guidance is silent for runtime telemetry, so the same question has an answer in one venue and none in the other. The contacts are the OpenTelemetry GenAI SIG, and the triage on `semantic-conventions#1598`, which accepted the cost-conventions request and has had no SIG take it.

## 5. Milestones and substantive work

| Milestone / output | Responsible contributor(s) | Target checkpoint | Status | Issue / PR |
|---|---|---|---|---|
| 6a verifiable per-call evidence outline, R1 to R5 | @astrogilda | Awaiting approver review | Open, needs approval | [cncf/toc#2369](https://github.com/cncf/toc/pull/2369) |
| 6b telemetry schema and cost outline, two requirements | @elang2 | To merge after 6a, so 6b follows it in the file | Drafted, to be filed | To be filed |
| Cross-stream control responsibility mapping | Open seat, identity and authorization | To agree at kickoff | Proposed | Not yet created |
| Policy and compliance decision guidance | Anca Sailer, scope to confirm | To agree; the action dates from October 8 | Proposed | Not yet created |
| Confirm subsection boundaries with chapters 02, 03, 04, 05 and 07 | @elang2, with each stream owner | Within two sessions | Proposed | Not yet created |
| Limitations and gap inputs for chapter 08 | Stream, as findings land | Progressive | Proposed | Not yet created |

Create issues for substantive work once scoped; keep detailed tasks in those issues.

## 6. Current biweekly walkthrough — November 19, 2026

- Changes or findings: no session notes exist after [October 22](../meeting-notes/2026-10-22.md), which set November 5 with the target "first drafts"; `meeting-notes/2026-11-05.md` returns 404. So no cross-stream boundary in section 3 has been agreed at a recorded session, and every one is proposed. 6a is on [#2369](https://github.com/cncf/toc/pull/2369) and awaiting an approver. 6b is drafted against the same file and will be opened to follow it. Both carry requirements as checklist items with an implementer check, which is the form proposed for this chapter.
- Suggested material: trace identity, policy check, human approval, telemetry and evidence through the [shared reference scenario](./README.md#proposed-shared-reference-scenario) for one side-effecting tool call, and stop at each point where the chapter currently has no owner.
- Feedback needed: what happens when the authorization or telemetry path is unreachable, and which claims the remaining evidence can still support. Also whether the two drafted subsections cover the chapter's cost and policy bullets or leave a third subsection needed.
- Blocker / help needed: an approver is needed on #2369, which has never received a human review comment; the identity and authorization seat is unfilled; and the boundaries with chapters 02 to 05 are proposed rather than agreed.
- Next output, contributor, and checkpoint: agree at the session; link meeting notes, issues and pull requests here.

## 7. Findings and integration — develop progressively

Two findings are established enough to record, both measured against pinned revisions rather than asserted. Neither is yet chapter text.

**Cost figures are not comparable across implementations today.** At conformance revision `6807b1d5e676`, 3 of 57 instrumented GenAI targets emit a cost figure at all, under 12 attribute and 2 metric names across four namespaces — 13 distinct, since `operation.cost` is emitted as both — and the registry defines none of them, though it does own one of the four namespaces. Both projects emitting a cost metric declare a currency as the instrument's unit and spell it differently, one `USD` and one `{USD}`, read from the library versions that revision pins because the report carries no unit field. So an operator cannot add two providers' costs without knowing which emitter produced each number.

**A configured limit and an enforced limit are different facts, and telemetry does not currently distinguish them.** A run that stops at a declared budget and a run that simply ends look the same unless the record carries an observed count from a vantage the enforcing component does not control. This is why 6b states the budget requirement as a record a reader can falsify rather than as a number to report.

Candidate upstream destinations for chapter 08: `semantic-conventions#1598` for the cost conventions gap, and the OpenTelemetry GenAI SIG for the unit and currency question. Both are named rather than claimed; neither has been raised by this stream yet.

Control boundaries and the telemetry and evidence flows go to [chapter 07](../07_Reference_Architecture.md). Supported measurement and policy limitations go to [chapter 08](../08_Gap_Analysis_and_Recommendations.md).

## 8. Review and readiness — use later

Use the [common readiness checklist](./TEMPLATE.md#8-review-and-readiness--use-later). Record review feedback and deferred questions here or in linked issues.

This plan is scoping, not ready for integration as technical guidance. Two of the four v0.1 outputs have drafts, one has a named contributor and no draft, and one has no contributor. The cross-stream boundaries in section 3 are proposed by this stream and have not been agreed with the other owners.
