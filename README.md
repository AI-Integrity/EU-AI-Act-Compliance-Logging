# PRISM Logging Standard

**An open standard for logging AI reasoning decisions — value, evidence, and source hierarchies — as structured one-line codes, for regulatory compliance and behavioral auditability.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Version](https://img.shields.io/badge/version-1.0-blue.svg)](./CHANGELOG.md)
---

## What a PRISM log looks like

```
<prism_log>
C:MD/IXi | V:Ben<Sel | E:Exp<Gui | S:Usr<Pro
</prism_log>
```

A single line. ~60 characters. Contains:

- **C** — Context: domain, scope of impact, reversibility, time horizon
- **V** — Value hierarchy: Schwartz value outranked by Schwartz value
- **E** — Evidence hierarchy: evidence type outranked by evidence type
- **S** — Source hierarchy: source type outranked by source type

The `<` reads as "outranked by." Left = deprioritized. Right = prevailed.

[Full specification →](./SPECIFICATION.md)

---

## Compliance Mapping Table

PRISM is a research framework. To make it usable for EU AI Act compliance programs, every research term maps directly onto a legal obligation. This table is the canonical mapping — auditors, DPOs, and compliance officers should read this first.

| Research term (PRISM) | Legal term (EU AI Act) | Provision | What the auditor receives |
|---|---|---|---|
| `C:` layer (domain / scope / reversibility / time) | **Traceability** | Art. 12(2) | Per-decision identification of the situation that may produce risk |
| Full PRISM code emitted per substantive decision | **Automatic Record-keeping** | Art. 12(1) | Machine-generated log entry over the system's lifetime |
| Timestamp + retention metadata (added at storage) | **Period of Use Records** | Art. 12(2)(a) | Evidence of when the system was operating |
| SHA-256 chained hash of the log stream ([`tools/prism_hash.py`](./tools/prism_hash.py)) | **Log Integrity** | Art. 12(3) | Tamper-evident protection against unauthorised modification |
| `V:` value hierarchy + `E:` evidence hierarchy | **Transparency of Reasoning** | Art. 13 | Reported value priorities and evidence types behind each output |
| Structured single-line code reviewable by humans | **Human Oversight Enablement** | Art. 14 | Format that lets a human auditor inspect AI behaviour at scale |
| Aggregate distribution of codes across time / versions | **Accuracy & Robustness Monitoring** | Art. 15 | Drift detection across model versions and deployment periods |
| Anomalous code patterns (e.g. unexpected `V:` flips, `S:Ano` surges) | **Serious Incident Reporting** | Art. 73 | Pre-investigation signal source feeding the incident-reporting workflow |
| `S:` source hierarchy (which source types prevailed) | **Data Governance Evidence** | Art. 10 (supporting) | Per-decision record of which source class drove the answer |

**Reading guide:** the left column is what the standard *technically* produces; the right column is what a regulator, auditor, or notified body *expects to see*. PRISM logs do not by themselves discharge any of these obligations — they provide the structured evidence layer that compliance programs and third-party auditors reference alongside their own governance documentation. See [DISCLAIMER.md](./DISCLAIMER.md).

---

## Why this exists

AI regulation is shifting from product-level compliance to behavior-level accountability. The EU AI Act's high-risk provisions, taking effect in August 2026, require that operators of high-risk AI systems maintain **auditable records of AI reasoning** — not just latency and token counts, but the values, evidence, and sources that drove each decision. Similar regimes are developing in other jurisdictions.

Current logging frameworks do not capture this. **PRISM Logging Standard does.**

Appending a structured vocabulary to a system prompt makes any modern LLM emit regulation-friendly reasoning codes. No new infrastructure. No API changes. No model retraining.

### A pre-standard in a standards gap

Official harmonised standards for EU AI Act compliance (developed by CEN/CENELEC and related bodies) are still under development at the time of v1.0 publication. Organizations facing the August 2026 deadline cannot wait for final standards to begin structured logging.

PRISM v1.0 is designed as a **pre-standard**: usable today, structurally aligned with the known requirements (Article 12's automatic logging, Article 13's transparency of reasoning, Article 14's human oversight), and modular enough to adapt when official standards converge.

**Modular design:** The vocabulary (domains, Schwartz values, evidence types, source types, scope, reversibility, time) is defined as fixed tables that can be extended or remapped without changing the structural format. If future harmonised standards require different taxonomies, adapting PRISM becomes a vocabulary-update task, not a reimplementation.

---

## Quick start

PRISM v1.0 supports three output modes. Pick based on your model's capabilities:

| Mode | When to use | User sees log? |
|---|---|---|
| **A. Inline tag** | Older models without structured output | No (host strips `<prism_log>` tag) |
| **B. Structured output** | Models with JSON mode (OpenAI, Gemini, Claude) | No (separate JSON field) |
| **C. Tool call** | Models with native tool use (Claude, OpenAI, Gemini) | No (separate tool_use block) |

**Critical:** In all modes, the PRISM log is for audit storage only — it must never be shown to end users. Mode A requires the host to strip the tag. Modes B and C provide the separation natively.

See the system prompt for your chosen mode:

- [`system_prompts/prism_v1_en_inline.md`](./system_prompts/prism_v1_en_inline.md) — Mode A
- [`system_prompts/prism_v1_en_json.md`](./system_prompts/prism_v1_en_json.md) — Mode B
- [`system_prompts/prism_v1_en_tool.md`](./system_prompts/prism_v1_en_tool.md) — Mode C

### Minimal integration (Mode C, Anthropic Claude)

```python
from anthropic import Anthropic

client = Anthropic()

PRISM_TOOL = {
    "name": "record_prism_log",
    "description": "Record the PRISM log for a substantive decision.",
    "input_schema": {
        "type": "object",
        "required": ["code"],
        "properties": {"code": {"type": "string"}}
    }
}

response = client.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=2000,
    system=PRISM_MODE_C_PROMPT,  # from system_prompts/prism_v1_en_tool.md
    tools=[PRISM_TOOL],
    messages=[{"role": "user", "content": user_input}]
)

# User-facing text and audit log are separate blocks
user_facing = ""
prism_code = None
for block in response.content:
    if block.type == "text":
        user_facing += block.text
    elif block.type == "tool_use" and block.name == "record_prism_log":
        prism_code = block.input["code"]

# user_facing → user interface
# prism_code → audit log storage
```

---

## Why code format, not JSON

PRISM logs are deliberately designed as compact codes rather than verbose JSON:

| Property | Code format | JSON format |
|---|---|---|
| Length | ~60 chars | ~300+ chars |
| Privacy | No user content | Risk of leaking context |
| Aggregation | Direct SQL/grep | Requires flattening |
| Cost per call | ~25 output tokens | ~150 output tokens |
| Human-readable at scale | Yes | No |

The log is a **structural fingerprint**, not a summary. Context and content live in your conversation log. The PRISM code captures only what's needed for behavioral auditability.

---

## What this standard does NOT claim

Being direct about limitations:

- **It does not prove the model "really" reasoned this way.** LLM self-reports can be post-hoc rationalization. PRISM logs are *reported* reasoning, not causal traces.
- **It does not replace independent audits.** Internal logs require external verification to carry regulatory weight.
- **It does not guarantee compliance with any regulation by itself.** It provides structured evidence; compliance depends on the full governance system around it.

These limitations are shared by all reasoning logs (chain-of-thought, attention traces, internal documentation). PRISM logs are the format best positioned for auditability at scale — not a complete solution.

---

## Theoretical foundation

PRISM is developed and maintained by [AI Integrity Organization (AIO)](https://aioq.org), a Swiss-registered nonprofit.

The standard draws on:

- **Schwartz Value Theory** (Schwartz, 1992, 2012) — 10 universal human values, validated across 80+ cultures
- **Walton Argumentation Schemes** (Walton, 2008) — taxonomy of evidence types in reasoning
- **Source Credibility Theory** (Hovland, Janis & Kelley, 1953; Pornpitakpan, 2004) — source trust hierarchies

Working papers:

- S. Lee (2026a). AI Integrity: Definition, Authority Stack Model, and Enhanced Cascade Mapping Hypothesis. arXiv:cs.AI.
- S. Lee (2026b). The PRISM Framework for Measuring AI Value Hierarchies. arXiv:cs.AI.
- S. Lee (2026c). Measuring AI Value Priorities: Empirical Analysis. arXiv:cs.AI.

---

## Repository contents

| File | Purpose |
|---|---|
| [`SPECIFICATION.md`](./SPECIFICATION.md) | Complete v1.0 code specification (includes output mode definitions) |
| [`DISCLAIMER.md`](./DISCLAIMER.md) | Scope, limitations, and non-warranty terms |
| [`system_prompts/prism_v1_en_inline.md`](./system_prompts/prism_v1_en_inline.md) | Mode A prompt — English |
| [`system_prompts/prism_v1_en_json.md`](./system_prompts/prism_v1_en_json.md) | Mode B prompt — English |
| [`system_prompts/prism_v1_en_tool.md`](./system_prompts/prism_v1_en_tool.md) | Mode C prompt — English |
| [`system_prompts/prism_v1_kr_*.md`](./system_prompts/) | Korean versions of all three modes (한국어) |
| [`examples/`](./examples/) | Real-world log examples across 7 domains |
| [`tests/validate.py`](./tests/validate.py) | Validator with multi-mode extraction |
| [`tools/prism_parser.py`](./tools/prism_parser.py) | Code extraction & structured parsing helper |
| [`tools/prism_hash.py`](./tools/prism_hash.py) | SHA-256 integrity helper (independent & chained hashes) |
| [`docs/integration_guide.md`](./docs/integration_guide.md) | Per-provider integration guide |

---

## License & disclaimer

MIT. Use it, modify it, embed it in commercial products. See [LICENSE](./LICENSE).

**Important:** This standard is a technical specification, not legal advice. Using it does not guarantee compliance with any regulation. See [DISCLAIMER.md](./DISCLAIMER.md) for full scope and limitations.

Attribution appreciated but not required.

---

## Contact

- Issues / suggestions: open a GitHub issue
- Commercial inquiries: 2sk@aioq.org
- Website: [aioq.org](https://aioq.org)

