# Context-Aware Authorization with AI Agents


## Research Question

This thesis sets out to answer one central question:

> **Can an agent be built and trained to reason about authorization decisions based on context?**

---

## Background

### Toward Agentic IAM — Zouari (2025)

Zouari identifies four main sources of risk in agentic Identity and Access Management systems:

| Threat | Description |
|---|---|
| **Privilege inflation** | Over time, an agent accumulates permissions it no longer needs. |
| **Collusive escalation** | Multiple agents coordinate to obtain access none of them is authorized to have. |
| **Cognitive drift** | — |
| **State tampering** | — |

### Formal Model: Dec-POMDP

The study models the problem as a **Decentralized Partially Observable Markov Decision Process (Dec-POMDP)**. This is a mathematical framework for cooperative, multi-agent sequential decision-making under uncertainty.

The model is defined by the following components:

- **System state:** the global state of the environment
- **Role action space:** the set of actions available to each role
- **Observation model:** what each agent can perceive about the state
- **State transition function:** how the system evolves in response to actions
- **Global reward:** the shared objective the agents optimize
- **Action schedule:** an encoding of the mandatory workflow the agents must follow

---

## References

- Zouari, 2025. *Toward Agentic IAM.*
