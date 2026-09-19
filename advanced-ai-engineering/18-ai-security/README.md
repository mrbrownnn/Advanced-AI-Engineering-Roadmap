# Module 14 — AI Security

## Why This Module Exists

AI systems expand the attack surface in unique ways: the model processes untrusted input as instructions, retrieval systems can be poisoned, and agents can be tricked into taking harmful actions. This module covers threat modeling and defense for AI-specific attack vectors.

## Key Engineering Questions

- What are the trust boundaries in my AI system?
- How do I prevent prompt injection (direct and indirect)?
- How do I prevent retrieval poisoning?
- How do I scope agent capabilities to minimize damage from compromise?
- How do I audit what the AI system did and why?

## Prerequisites

- Module 08-09 (retrieval systems — for retrieval poisoning)
- Module 11 (agent runtime — for excessive agency)
- Module 12 (evaluation — for testing defenses)

## Topics

### Threat Modeling
- Threat modeling for AI systems: identifying assets, threats, and trust boundaries
- STRIDE applied to AI: Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege
- Trust boundaries: user input, retrieved content, tool outputs, model outputs

### Injection Attacks
- Direct prompt injection: user input that overrides system instructions
- Indirect prompt injection: malicious content in retrieved documents or tool outputs
- Defense strategies: input sanitization, instruction hierarchy, output validation

### System Attacks
- Retrieval poisoning: injecting malicious documents into the retrieval corpus
- Tool abuse: tricking the model into making harmful tool calls
- Excessive agency: agent taking actions beyond intended scope
- Confused deputy: model acting on behalf of attacker while believing it serves the user

### Defense Engineering
- Data exfiltration prevention: stopping the model from leaking sensitive information
- Secret management: keeping API keys, credentials, and PII out of model context
- Capability scoping: principle of least privilege for agent tools
- Sandboxing: isolating agent execution environments
- Audit trails: logging all model inputs, outputs, and actions for review

## Expected Artifacts

1. **Threat model** — complete threat model for a specific AI system
2. **Injection testing** — test a system against prompt injection attacks
3. **Capability scoping design** — principle of least privilege for an agent system
4. **Engineering report** — security architecture for an AI application

## Exit Criteria

The learner can:
- Produce a threat model for an AI system identifying trust boundaries and attack vectors
- Test and defend against prompt injection (direct and indirect)
- Design capability scoping and sandboxing for agent systems
- Implement audit trails for AI system actions
- Reason about the cost of security measures vs the risk they mitigate

## Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate → Create
  solo: Relational
  dreyfus: Competent
```
