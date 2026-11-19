# Stream plan: governance, observability, security and policy

Technical chapter: [Chapter 06](../06_Governance_Observability_Security_and_Policy.md)
Stream owner: Elankumaran Srinivasan (@elang2), as recorded in the [initiative README](../README.md#workstreams). The working document's outline and register still name Sankalp Gilda as section 6 owner, from before the initiative moved into this repository; 6a remains his and this plan does not change that.
Team: Elankumaran Srinivasan (@elang2), owner, confirmed; Sankalp Gilda (@astrogilda), 6a author, confirmed; Anca Sailer, confirmed for OSCAL Compass; Vikas Agarwal, invited; contributor seats open (see section 2)
Status: scoping
Last updated: 2026-11-18
Next walkthrough: November 19, 2026 session

## 1. Purpose and intended outcome

Platform teams running agents in Kubernetes can already see that a call happened. Three questions they cannot answer matter once an agent spends money and acts on a user's behalf. What did this run cost, in units an operator can add up across providers? Did the limit that was configured actually stop it, or did the script simply end? And can a party who does not trust the operator check either answer from what was retained?

This chapter helps platform engineers, operators and auditors decide where identity, authorization, policy and operational limits are enforced, and what telemetry and evidence have to exist for those decisions to be reviewable afterwards.

What differs from observability generally. A policy decision and an observed effect are separate facts, so an allowed call is not evidence the effect happened. A provider-reported cost and a locally computed one are indistinguishable unless the record says which. And an agent's own producer is usually the only witness, which is the condition under which a completeness claim needs a population the producer cannot edit.

Minimum useful outcome for v0.1, within the 1.5 pages the outline allots chapter 6, with the control-responsibility mapping going to an appendix if it does not fit:

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
| Anca Sailer | Safety guardrails, compliance as code and policy as code; OSCAL Compass as an implementation | OSCAL Compass into this stream; the wider policy and compliance subsection is scope to confirm with her | Confirmed, for OSCAL Compass, per the [October 8 action](../meeting-notes/2026-10-08.md), which names her alone |
| Vikas Agarwal | OSCAL Compass, same focus area | To agree | **Invited, not confirmed.** Recorded against this focus area in the contributor register and the working document's owner table, which is interest rather than agreement; he has not been asked. A third contributor on the same focus area is recorded in the register |
| Open seat | Workload identity, delegated authority, authorization | Control-responsibility mapping for identity and delegation; the boundary with chapter 03 | Open |
| Open seat | Security practitioner, ideally with agent deployments in production | Threat review of the control mapping; what the evidence patterns do not cover | Open |
| Open seat | End-user operator running agents at scale | Whether the telemetry guidance answers an operator's actual questions, and what it costs to emit | Open |

- Missing expertise or perspectives: workload identity and authorization; security practitioners; and operators running agents in production who can say what they currently cannot measure. Cross-stream reviewers from protocol, gateway, runtime and state.
- Recruitment action: the owner's action from the [October 22 session](../meeting-notes/2026-10-22.md), "identify the people working with them on each stream and coordinate in Slack", is past due and this plan is the first half of it. The second half is a call in `#initiative-distributed-agentic-systems` naming a first task per open seat, to go before this pull request merges. Checkpoint: contributors named, or the seat declared unfilled, by the December 3 session.
- How to join: reply in `#initiative-distributed-agentic-systems`, or comment on the pull request that carries this plan, naming the area you would like to take.
- First task for a new contributor: establish what the API server audit log records for an admission decision on a side-effecting tool call, and establish **both** bounds on its completeness, because they differ — webhook scope and `failurePolicy` bound which requests got a decision, the audit policy bounds which were recorded. Fill one row of the control-responsibility mapping from the Kubernetes documentation with section references, and record what the documentation does not say. About an hour, and it needs nobody's cluster.

Agreement means a session record or a public statement, not a mention, and it is agreement to contribute rather than to own an output. Anca Sailer is confirmed because the October 8 notes record her presenting this material and carry an action assigning OSCAL Compass here. A register entry is interest, so Vikas Agarwal is invited. A name appearing only in a chapter outline stays an open seat.

## 3. Scope and priority questions

- In scope: GenAI telemetry and MELT for agent runs; cost, token and latency measurement and its comparability; declared budgets and whether enforcement is checkable from a record; policy and compliance as code; verifiable per-call evidence; and the cross-cutting security implications of the above.
- Out of scope: vendor-specific controls, legal compliance guarantees, naming individual attributes, and treating telemetry alone as proof that behaviour was correct or compliant.
- Named rather than silently dropped, since output 1 promises that unowned gaps are named. The working document's focus-area-5 list is wider than this plan's v0.1 scope. Of the remainder: OAuth2 and JIT/ABAC/PBAC least agency are later investigations here, reached through the authorization-enforcement boundary accepted from chapter 02; prompt injection and tool hijacking are later investigations here and overlap the security review the open seat is for; model cards and Agent-as-a-Judge evaluation are not claimed by this chapter and need an owner; and TEEs and confidential computing belong to chapter 04, which already claims them. ANS and MCP authorization belong to chapter 02.
- Assumptions and terminology: distinguish user identity from workload identity; delegated authority from direct authority; a policy decision from an observed effect; operational telemetry from independently verifiable evidence; and a provider-returned amount from a locally computed one. Align these with chapter 07's shared terms before comparing mechanisms.
- Single-cluster baseline: explicit trust and tenancy boundaries between agents, tools, gateways and stores, with the enforcement point for each control named. RBAC and SPIFFE/SPIRE carry workload identity, backing the delegation half of the control mapping; admission control is one denominator 6a R4 permits and the API server audit log is what makes that population readable; NetworkPolicy bounds which tools an agent can reach. Each stops somewhere. Webhook scope and `failurePolicy` bound which requests received a decision; the cluster's audit policy bounds which were recorded; no control-plane object records a cost or a budget verdict.
- Multi-cluster, edge, or disconnected considerations: identity federation, authorization behaviour when the policy service is unreachable, policy distribution, and whether telemetry and evidence remain continuous across clusters. Recorded as later investigations.
- Boundaries with other streams. These are proposed by this stream. Rather than wait for a session, they will be posted in `#initiative-distributed-agentic-systems` for the charter's 72-hour lazy consensus, which is the mechanism the charter names and is why they are marked proposed rather than blocked:
  - Protocol (chapter 02): the chapter 02 plan hands governance "identity, authorization enforcement, and per-call evidence", which is wider than the evidence check this plan had claimed. **Accepted as stated**: this chapter takes identity assertion, authorization enforcement points and per-call evidence, and protocol keeps the security schemes, card signing and mandates the specifications define. Whether declared capabilities match behaviour is also handed here, and is recorded as a later investigation rather than a v0.1 output.
  - Gateway (chapter 03): gateway leads on routing and proxy behaviour; this chapter covers what the gateway must record for interception-based evidence to be checkable. 6a already carries one bullet in chapter 03 for the response-phase test, flagged to that owner on [#2369](https://github.com/cncf/toc/pull/2369).
  - Runtime (chapter 04): the chapter 04 plan hands isolation and identity requirements to governance, and this plan hands issuance to runtime, so identity currently points both ways. Proposed split, for that owner to accept or correct: runtime issues and rotates workload identity; governance specifies what the identity must assert for delegation to be reviewable.
  - State (chapter 05): state leads on durable storage and retention; this chapter covers which retained inputs an evidence claim depends on, and what happens to the claim when retention expires.
  - Reference architecture (chapter 07): control boundaries and the telemetry and evidence flows, so the same value is not specified twice.

| Priority | Question to answer | Intended output | Contributor |
|---|---|---|---|
| v0.1 | Where are identity, delegation, approval and least-privilege policies enforced, and which of those are currently unowned? | Cross-stream control responsibility mapping | Open seat, identity and authorization |
| v0.1 | Which telemetry supports task diagnosis and comparable cost measurement, and with what limits? | Semantic-convention and measurement guidance (6b) | @elang2, drafted, see section 5 |
| v0.1 | What evidence supports auditing a delegated action without exposing unnecessary sensitive data? | Evidence pattern, trust assumptions and trade-offs (6a) | @astrogilda, drafted, see section 5 |
| v0.1 | How are policy and evaluation decisions, and failure behaviour, made explicit and reviewable? | Governance decision guidance | Anca Sailer, per the October 8 action; Vikas Agarwal invited |
| Later | Does any of this hold across clusters, or when the authorization or telemetry path is unreachable? | Recorded limitations for chapter 08 | To agree |

## 4. Starting knowledge and sources

| Source / project | Relevant section or capability | Why it matters / question informed | Version / date and evidence limitations |
|---|---|---|---|
| Kubernetes control plane | RBAC, the API server audit log, admission control (the webhook model, Kyverno, Gatekeeper), NetworkPolicy, and SPIFFE/SPIRE for workload identity | These are the existing capabilities this chapter's requirements rest on, so the chapter says what they already give and where each stops | Documented and widely deployed, not demonstrated here. Two separate bounds: webhook scope and `failurePolicy` bound which requests receive an admission decision; the cluster's audit policy bounds which requests are recorded. The API server logs a request whether or not a webhook ran on it |
| [Cloud Native Agentic Standards](https://www.cncf.io/blog/2026/03/23/cloud-native-agentic-standards/) | Observability, governance and security | Baseline to operationalise, not restate | Published 2026-03-23; recommendations need deployment context |
| [OpenTelemetry GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) | Agent and client instrumentation, token usage | The neutral vocabulary this chapter builds on | Record the convention version and stability with every claim; all GenAI stability declarations currently read `development`, so nothing here is frozen |
| [OpenTelemetry GenAI conformance report](https://github.com/open-telemetry/semantic-conventions-conformance) | What instrumented projects actually emit, per target | Separates what the conventions define from what implementations do | Pin the revision; the figures are in 6b. The report carries **no date field at all** — its only provenance is `generated_by: otel-conformance-report build` — so the revision SHA is the only thing dating an observation, and it dates the report rather than an emitter's release |
| [semantic-conventions#1598](https://github.com/open-telemetry/semantic-conventions/issues/1598) | Cloud Financial Management conventions | Shows the gap is acknowledged upstream and unclaimed | Open since 2024-11-20, labelled `triage:accepted:needs-sig`. Triage accepted it; no SIG has taken it |
| [FOCUS specification](https://github.com/FinOps-Open-Cost-and-Usage-Spec/FOCUS_Spec) | `BillingCurrency` column requirements | The currency constraint is settled vocabulary to borrow, not a new proposal | Read at tag `v1.0`. FOCUS governs billing exports, not runtime telemetry, so the constraint is borrowed and not inherited |
| in-toto, DSSE, IETF SCITT | Statement envelopes, canonical signed bytes | What 6a's signature and binding requirements rest on | Per 6a; record the exact profile and test-vector revision |
| OWASP Agent Control Standard | Agent control expectations | Cross-check for the control mapping | Draft status; treat as input, not as a settled standard |
| OSCAL Compass | Policy and compliance as code | The policy subsection's anchor, per the October 8 session | Capability is documented rather than demonstrated in this chapter; the reference architecture Anca described applies across regulations and risk taxonomies |

- Which Cloud Native Agentic Standards recommendations apply, and what are we adding? The observability, governance and security recommendations apply. What this chapter adds is the condition under which each is checkable afterwards: a cost figure that carries its unit and source, a budget whose enforcement can be falsified from a record, and a per-call record a reader who distrusts the producer can verify.
- What do Kubernetes and existing projects already provide, and what remains insufficient? Kubernetes supplies one of the three populations 6a's R4 permits, under the two bounds stated in the single-cluster baseline above. What remains insufficient is on both sides: a population derived from the audit log is complete only to the audit policy's scope, and nothing in the control plane records a cost or a budget verdict at all, which is why 6b's requirements are about telemetry rather than a Kubernetes object.
- Where do sources disagree, and which contact could help? Two places, both measured in 6b rather than restated here: the GenAI registry defines no cost attribute while instrumented projects emit one anyway, and FOCUS constrains currency for billing exports where the metric-unit guidance is silent for runtime telemetry. The contacts are the OpenTelemetry GenAI SIG, and the triage on `semantic-conventions#1598`, which accepted the cost-conventions request and has had no SIG take it.

## 5. Milestones and substantive work

| Milestone / output | Responsible contributor(s) | Target checkpoint | Status | Issue / PR |
|---|---|---|---|---|
| Initial scope and team plan | @elang2 | November 19 session | This plan, as a pull request | This PR |
| 6a verifiable per-call evidence outline, R1 to R5 | @astrogilda | November 19 session | Open, needs an approver | [cncf/toc#2369](https://github.com/cncf/toc/pull/2369) |
| 6b telemetry schema and cost outline, two requirements | @elang2 | November 19 session, to merge after 6a | Drafted, to be filed | To be filed |
| First useful chapter draft, 6a and 6b integrated with a chapter-level opening | @elang2, @astrogilda | December 3 session | Proposed | Not yet created |
| Cross-stream control responsibility mapping | Open seat, identity and authorization | December 3 session | Proposed | Not yet created |
| Policy and compliance decision guidance | Anca Sailer, with Vikas Agarwal invited | To agree; Anca's action dates from October 8 | Proposed | Not yet created |
| Boundaries agreed with chapters 02, 03, 04, 05 and 07 | @elang2, with each stream owner | 72-hour lazy consensus from the channel post | Proposed | Not yet created |

Create issues for substantive work once scoped; keep detailed tasks in those issues.

**What lands by November 19**, which is the third part of the question the lead asked at the
November 5 session. This plan, as a pull request. 6b as a pull request, to merge after 6a. Nothing
else: the control mapping and the policy guidance have no draft, and one of them has no contributor.

## 6. Current biweekly walkthrough — November 19, 2026

- What changed or did we learn? The November 5 session happened and its record is in the working
  document rather than in `meeting-notes/`; at it the lead asked each owner what is written, what is
  blocked and what lands by November 19, and this stream was not represented. Since then 6a has been
  open and unreviewed, 6b has been drafted, and this plan is its first answer to that question.
- What concrete material can we walk through? The shared reference scenario for one side-effecting
  tool call, including the delegation and tenancy steps and starting from a Pod restart and a broken
  connection, stopping at each point where this chapter has no owner.
- Which recommendation, assumption, or boundary needs feedback? The identity boundary, which
  currently points both ways: chapter 04 hands isolation and identity requirements to governance
  while this plan hands issuance and lifecycle to runtime. Proposed resolution in section 3.
- What is blocked, and what help is needed? 6a needs an approver and has never had a human review
  comment. The identity and authorization seat is unfilled. Five boundaries are proposed and not
  agreed, which the channel post in section 3 is meant to close.
- What will we bring to the next meeting? The first integrated chapter draft, @elang2 with
  @astrogilda, by the December 3 session.
- Related issues, PRs, or meeting notes: [#2369](https://github.com/cncf/toc/pull/2369), this plan's
  pull request, and the [October 8](../meeting-notes/2026-10-08.md) and
  [October 22](../meeting-notes/2026-10-22.md) notes.

## 7. Findings and integration — develop progressively

Technical detail stays in the chapter. These are one-line summaries pointing at where the
measurement lives.

| Finding / pattern / open gap | Evidence and limitations | Related streams / coordination needed | Destination / next action |
|---|---|---|---|
| Cost figures are not comparable across implementations, and the conventions define nowhere to make them so | Measured at a pinned conformance revision; stated in full in 6b. Dates the report, not an emitter's release | None; this is chapter 6's own | 6b, and `semantic-conventions#1598` upstream |
| A configured limit and an enforced limit are different facts, and telemetry does not distinguish them | Reasoned from the record shape, not yet from a run; the longer-script run that would separate the two is outstanding | None | 6b |
| Kubernetes supplies one of the three populations 6a R4 permits, under two separate bounds | Documented, not demonstrated here; stated in full in section 4 | Runtime, for identity issuance | Chapter 07, and the control mapping |
| No control-plane object records a cost or a budget verdict | Absence, established from the control-plane documentation | None | Chapter 08 |

### Reference architecture handoff

For [chapter 07](../07_Reference_Architecture.md): the control boundaries in section 3, the
telemetry and evidence flows for one side-effecting tool call, and the two populations a
completeness claim can rest on, so the same value is not specified twice.

### Gap-analysis handoff

For [chapter 08](../08_Gap_Analysis_and_Recommendations.md): the unmet requirement is a comparable
cost figure, the evidence is the pinned conformance measurement in 6b, the existing capability is
the OpenTelemetry GenAI conventions, and its limitation is that they define no cost attribute. The
upstream destination is `semantic-conventions#1598`, which triage accepted and no SIG has taken.
Named rather than claimed; this stream has not yet raised it.

## 8. Review and readiness — use later

Use the [common readiness checklist](./TEMPLATE.md#8-review-and-readiness--use-later). Record review feedback and deferred questions here or in linked issues.

This plan is scoping, not ready for integration as technical guidance. Two of the four v0.1 outputs have drafts, one has a named contributor and no draft, and one has no contributor. The cross-stream boundaries in section 3 are proposed by this stream and have not been agreed with the other owners.
