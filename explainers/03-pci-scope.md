# PCI scope and why the estate does not want it

*What "PCI scope held by the payment provider" means, and why a scope boundary is an architectural decision.*

---

## The short answer

**PCI DSS** is the Payment Card Industry Data Security Standard: the rulebook the card networks impose on anyone who handles cardholder data. It is not a law, it is a contractual condition of being allowed to accept cards at all.

**Scope** is the set of systems that touch that data. The estate's design keeps card data off every system it owns, so the scope sits with the payment provider instead.

## What being in scope costs

If a card number passes through a server, that server is in scope — and so, in practice, is everything connected to it. Being in scope means carrying, every year, indefinitely:

- Formal annual assessment, with documented evidence
- Quarterly network vulnerability scans and periodic penetration tests
- Network segmentation that provably isolates the cardholder environment
- Encryption at rest and in transit, with managed key rotation
- Access logging, access review, formal change control

For an organisation whose competence is 18th-century rides and exotic animals, that is a large recurring cost against no revenue, plus a liability the estate is poorly placed to carry. A breach means fines, a forced forensic audit, and potentially losing the ability to take card payments — which for a ticketed attraction is the same as closing.

## How the scope is pushed out

The mechanism is that **a card number never touches a system the estate owns**:

1. A guest clicks pay. The page loads a payment form that is not the estate's — a hosted field or iframe served from the provider's own domain, or a full redirect to the provider.
2. The card details travel from the guest's browser **straight to the provider**. They do not reach the estate's web tier, its network, or its logs.
3. The provider charges the card and returns a **token**: an opaque reference that is useless to anyone who steals it.
4. The estate stores the token against the order. That is all it ever holds.

ADR-003 states the resulting rule: payments are handled by a payment provider and only a payment reference is stored. The data classification in 04-data-flow.md carries the same line for orders and payments — *card data never enters the platform* — and the provider is a Level 1 compliant service provider carrying the assessment burden on the estate's behalf.

## Why this is in the architecture and not a footnote

Three reasons this earns a place in the design documents rather than a procurement note.

**It is a declared non-goal.** Handling card payments in-house sits in the same list as not automating ride safety inspection and not letting AI act on an animal without a keeper. All three are the same move: identify a liability, and place it with the party equipped to carry it. Writing down what you are deliberately *not* building is itself an architectural decision.

**It is a fitness function.** Security and privacy is the seventh ranked characteristic, and one of its tests is literally that payment card data never touches estate systems. That is checkable — it either does or it does not — which is what makes it a fitness function rather than an aspiration.

**It changes the failure analysis.** The context view names the payment provider as an external dependency and asks what happens when it is unavailable: online sales pause, gate sales fall back to a standalone card terminal with deferred settlement, and **existing tickets are unaffected, because validation is offline and signed**.

That last consequence is the interesting one, and it is the same instinct as the estate-to-cloud path: name the external dependency, assume it fails, and make sure the critical path — people getting through the gate — does not depend on it.

## What the estate gives up

Delegation is a trade, and the costs are worth stating:

- **Provider fees** on every transaction, in exchange for not running a compliance programme.
- **Less control over the checkout experience**, since the sensitive fields belong to someone else's domain.
- **A vendor dependency** on the money path, which is why it appears in the context view with an explicit degradation plan rather than being assumed always-up.
- **Reduced but not zero obligation.** The estate still completes a self-assessment questionnaire and still has to not do foolish things, such as logging a card number a guest typed into the wrong field.

## Where this is specified

- [ADR-003](../adr/ADR-003-offline-verifiable-signed-tickets.md) — payments delegated, only a reference stored, and why entry survives a provider outage
- [01-context.md](../architecture/01-context.md) — the payment provider as an external dependency, with its degradation path
- [02-containers.md](../architecture/02-containers.md) — the Ticketing and Payments container and its provider SDK boundary
- [04-data-flow.md](../architecture/04-data-flow.md) — the data classification row for orders and payments
- [05-characteristics.md](../architecture/05-characteristics.md) — the security and privacy fitness functions

Related explainers: [zone gateways](01-zone-gateways.md), [estate-to-cloud path](02-estate-to-cloud-path.md).
