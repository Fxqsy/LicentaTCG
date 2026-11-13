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
| **Cognitive drift** |
| **State tampering** |


### 1. Privilege Inflation

**Definition:** Agents accumulate unnecessary permissions over time.
**Formal property:** A privilege must never be both proposed and validated by the same phase.

**Causes**
- Each grant is judged on its own merits.
- Nothing expires.
- Nothing measures the total.

The survey literature calls this pattern *capability creep*. Permissions pile up across steps, so a minimal starting privilege can grow into full access through a chain of steps that are each valid on their own. Checks that look at each call separately can't see it happening.

**Exploit in this platform**
- **Salami slicing:** An intern files many small, low-risk requests that all get auto-approved, until together they add up to an admin role.
- **Proposer/validator violation:** If the LLM can suggest which scope to grant *and then* approve it, Zouari's property is broken.

**Controls**
- Grants that expire
- A per-user cumulative-risk budget, computed over all effective grants
- An LLM that only gives a verdict on the requested scope and never proposes a different one

---

### 2. Collusive Escalation

**Definition:** Two or more agents coordinate to approve a privilege set that is unsafe or excessive.
**Formal property:** A grant is allowed only while the risk of the resulting state stays below a threshold [1].

**Causes**
- Requests are evaluated in isolation, but the danger lives in *combinations*. These are toxic combinations such as *create vendor* + *approve payment*.
- Trust between parties helps attackers. In one study, models that refused a malicious request from a human went along with it when the same request came from a peer agent.

**Exploit in this platform**
- Two accounts request complementary permissions. Each one is harmless alone, but together they break separation of duties.
- A justification such as *"my teammate already got this"* borrows another request's approval.
- A requester and an approver collude.
- Fake accounts build clean behavior histories in advance.

**Controls**
- Risk computed on the **combined** privilege set
- Cross-user correlation features
- One account's approval is never treated as evidence for another account

---

### 3. Cognitive Drift

**Definition:** An agent's behavior slowly degrades, which leads to unintended and insecure actions.

**Causes**
There are three separate sources:
1. **Model change:** version updates, prompt edits, nondeterminism.
2. **Data change:** behavior baselines shift.
3. **Feedback loop:** Behavior enabled by earlier auto-approvals becomes part of the "normal" history that justifies the next approval.

**Exploit in this platform**
- **Boiling frog:** An attacker raises their requests gradually, so each one looks like a small step from a baseline that has already been normalized.
- **Persistent state:** Research on agent memory shows that repeated exposure can make an agent prioritize a user differently. The risk then comes from state drift rather than from a single authentication event.

**Controls**
- Pin the model version.
- Keep a fixed regression set of requests that must produce the same verdicts after any change.
- Monitor approval rates and score distributions over time.
- Compute baselines against **peer groups**, not only against the user's own history.
- Make the decider **stateless** by rebuilding its context from the store on every call.

---

### 4. State Tampering

**Definition:** Agents maliciously alter system configurations or data. In practice this means manipulating internal state variables or decision records to forge an approval.

**Causes**
- Decision inputs and outputs live in writable places. They are often writable by the very people they describe.

**Exploit in this platform**
- Editing or inserting rows in the grants table
- Deleting denied attempts from the behavior history
- Injecting fake events through log or SQL injection in the justification text
- Changing the threshold configuration
- Replaying an old approval
- Altering user attributes that the context builder trusts (department, manager)

**Controls**
- An append-only, hash-chained audit log
- Grants signed by the gateway and verified on every use
- Database roles, so that only the gateway can write grants and events
- Configuration kept under version control
- Periodic reconciliation that re-checks grants against decisions (Zouari's *reanalyze* step)

---

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
