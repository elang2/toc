# 6. Governance, observability, security and policy

Status: outline. Owner: Elankumaran Srinivasan.

- OpenTelemetry GenAI semantic conventions and spans; metrics, events, logs and traces; baggage
- LLM cost, token and latency tracking. Gap: these numbers are not comparable across implementations today, so an operator cannot bound spend from telemetry
- Policy and compliance as code, with OSCAL Compass as one implementation (Anca Sailer)
- 6a. Verifiable per-call evidence (subsection, proposed by Sankalp Gilda): independently verifiable records of gateway behavior that third parties can audit; vendor neutral, anchored on in-toto and the OWASP agent control work
- 6b. Telemetry schema and cost (subsection, Elankumaran Srinivasan): a cost figure carries its unit and its source, and a declared budget is checked after the run from its record; vendor neutral, anchored on the OpenTelemetry GenAI conventions and the FinOps FOCUS specification

## 6b. Telemetry schema and cost

Status: outline. Author: Elankumaran Srinivasan.

An amount without its unit and its source is not comparable. A reader cannot add two providers' costs when one is a currency and the other is credits, and cannot tell a figure the provider returned from one an operator derived against a price table that may be stale. Budget is a separate question from cost. Telemetry records what a run did, and a limit that was configured is not a limit that was enforced.

Each requirement is a checklist item plus one line on how an implementer checks it.

- [ ] R1. A cost figure carries its unit and its source. Record a monetary amount only alongside an ISO 4217 currency code, and never record a token or credit count in a currency-denominated field. Declare whether the amount was returned by the party that answered or computed locally. No attribute carries the pricing data behind a computed amount, so that is declared absent rather than defaulted. Check: aggregating two providers' costs fails closed when a currency is absent, a token count occupies a currency field, or the source of a computed amount is undeclared; and where two layers record one operation, each record identifies the instrumentation that emitted it, so a consumer can de-duplicate before summing.
- [ ] R2. A declared budget is checked after the run from its record. Record the declared limit with its unit, an observed count taken from a vantage the enforcing component does not control, and the producer's declared enforcement mechanism. If the enforcing component reports its own counter, record it with its unit; a counter without a unit is a missing fact. Treat the mechanism as a claim the record lets a reader falsify, and represent a run whose mechanism cannot be determined as explicitly unclassifiable and not as a pass. Check: a limit that held is told from one merely requested by the retained request ledger and not by the producer's counter; the conformance suite exercises a counterparty that ignores the stop request, since one that honours it makes the mechanisms indistinguishable; and a run that errored before its mechanism could be determined does not read as conformant.

Requirement numbering is scoped to this subsection. A cross-framework count of consumed iterations stays out of scope, because each implementation binds its limit to a different native unit and the resulting counts are not comparable.

How an implementer checks it today. OpenTelemetry's own GenAI conformance report is the neutral surface. Read at revision `6807b1d5e676`, it covers 57 instrumented GenAI targets, of which 3 emit a cost figure at all, under 12 attribute names and 2 metric names across four namespaces, none of those 14 names appears in the 63-attribute, 17-signal GenAI registry, and one of the four is a namespace the registry itself owns. A crosswalk of the emitted names accompanies this subsection so that 6a's per-call record and this schema do not specify the same value twice. Both checks are executable. R1's is a corpus of thirty vectors, ten accept and twenty reject, covering every clause of the requirement. The aggregator refuses to return a total where any clause is unmet, so a partial sum of the valid records is never produced. Its verdicts are invariant under renaming every cost field, which is what makes it a check on the requirement and not on a spelling. R2's runs against a counterparty that ignores the stop request, since one that honours it makes the mechanisms indistinguishable.

Neither requirement names an attribute. The name the ecosystem already queries for spend is a leaf, so it cannot carry a currency or a source. The widest cost vocabulary among the three emitters runs to ten attribute names in a namespace the registry does not define, and not one of the three records a currency or a source anywhere.

Builds on: the OpenTelemetry GenAI semantic conventions and their conformance report, and the FinOps FOCUS specification, whose v1.0 `BillingCurrency` column is already normative on the point this requirement needs, in requiring that it be present, be non-null, and conform to a currency-code format. The vocabulary is reused and not reinvented. 6a's per-call binding is how a reported cost points back to its source.

Open: which fields are shared with 6a so one value is not specified twice, whether a consumed-iteration count can be made comparable at all without a declared unit, and where a locally computed amount's price basis is retained.
