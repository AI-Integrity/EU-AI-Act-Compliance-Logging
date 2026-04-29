# EU AI Act Compliance Logging — PRISM Standard v1.0

**An open-source pre-standard for Article 12 reasoning logs. Drop-in system prompt. MIT licensed. Drives the structured evidence layer auditors expect alongside conventional event logs.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Version](https://img.shields.io/badge/version-1.0-blue.svg)](./CHANGELOG.md)
[![EU AI Act](https://img.shields.io/badge/EU%20AI%20Act-Article%2012-red.svg)](#compliance-mapping-read-this-first)
[![한국어](https://img.shields.io/badge/docs-한국어-lightgrey.svg)](./docs/README_KR.md)

---

## ⚠️ August 2, 2026

That is when most EU AI Act high-risk obligations — including the Article 12 logging requirement — become enforceable for systems listed under Annex III. (Some sectoral categories under Annex I follow on August 2, 2027. Confirm your system's category before assuming the date.)

If your high-risk system cannot produce auditable per-decision records by the applicable date, your operator obligations are not met. Article 99 sets penalty caps up to €15 million or 3% of worldwide annual turnover for the most severe breaches; Article 12 non-compliance typically falls in the operator-obligation tier (penalty caps up to €15 million or 3%) or lower depending on circumstances. First-time non-compliance is usually addressed through corrective action notices before fines.

Conventional logs (latency, token count, request ID) satisfy parts of Article 12 but do not capture the **reasoning** behind decisions: which values prevailed, which evidence was weighted, which sources were trusted. Article 13 (transparency) and Article 14 (human oversight enablement) raise expectations beyond simple event traces.

**Official harmonised standards (CEN/CENELEC) are still under development.** Operators cannot wait for them to begin structured logging.

This repository is a working pre-standard you can deploy today as one component of a broader compliance program.

---

## 🟦 Important: This is a complementary layer, not a replacement

PRISM is a **reasoning-trace addition** to your existing event log pipeline. It does not, by itself, satisfy Article 12.

You still need to log:
- User identifiers, timestamps, session IDs (Article 12(1) lifecycle records)
- Inputs and outputs (Article 12(2)(c) for biometric systems; risk identification generally)
- System operation periods (Article 12(2)(a))
- Database references where applicable (Article 12(2)(b))

PRISM **augments** these with a per-decision reasoning fingerprint. It does not replace any of them.

A common compliant deployment looks like:

```
[Conventional event log] → input/output/timestamp/user-ID
        +
[PRISM reasoning log]   → C/V/E/S code per substantive decision
        +
[Hashing layer]         → SHA-256 chain over both, for tamper-evidence
        ↓
[Audit storage]         → indexed, retained per Article 12(2)(a) requirements
```

If you adopt PRISM only, without conventional event logs, you remain non-compliant. We mention this prominently because some marketing in this space implies otherwise — it is not true.

---

## What you get

A vocabulary that any modern LLM can emit with zero retraining. Append the system prompt provided in this repo. Every substantive decision the model makes produces a single line of structured code:

```
<prism_log>
C:MD/IXi | V:Ben<Sel | E:Exp<Gui | S:Usr<Pro
</prism_log>
```

Approximately 60 characters. No verbatim user content. Topic-level metadata only (the `C:` domain reveals "this was a healthcare question" — comparable to what your existing routing logs already disclose).

The line decodes as:

- **C** — Context: domain, scope of impact, reversibility, time horizon
- **V** — Value hierarchy: which value priority prevailed
- **E** — Evidence hierarchy: which evidence type was decisive
- **S** — Source hierarchy: which source class was trusted

The `<` reads as "outranked by." Left = deprioritized. Right = prevailed.

[Full specification →](./SPECIFICATION.md)

---

## Compliance mapping (read this first)

The left column lists what PRISM technically produces. The right column maps each item to the EU AI Act provision it **supports** (not single-handedly satisfies). Your full compliance program — risk management, data governance, post-market monitoring — must be built around the entire Article 9–17 obligations, not PRISM alone.

| What PRISM produces | EU AI Act provision it supports | Auditor-facing usefulness |
|---|---|---|
| One PRISM line per substantive decision | **Art. 12(1)** — Automatic record-keeping | Machine-generated reasoning trace, complementary to conventional event logs |
| `C:` layer (domain / scope / reversibility / time) | **Art. 12(2)** — Traceability of risk situations | Per-decision risk-context tag |
| Aggregate `C:` distribution across operating period | **Art. 12(2)(a)** — Period of use records (in combination with timestamps) | Volume and category of decisions over time |
| SHA-256 chained hash via [`tools/prism_hash.py`](./tools/prism_hash.py) | Tamper-evidence (implied by general traceability principle and Recital 73; not literal Art. 12(3) text) | Chain-of-custody evidence for the log stream |
| `V:` and `E:` hierarchies | **Art. 13** — Transparency of reasoning *(supports; Art. 13's primary frame is deployer-facing transparency)* | Reported value priorities and evidence types behind outputs |
| Structured single-line code, grep- and SQL-friendly | **Art. 14** — Human oversight enablement *(contributes; Art. 14's core requirement is human stop/override capability)* | Format that lets a human auditor inspect AI behaviour at scale |
| Aggregate code distribution across versions | **Art. 15** — Accuracy and robustness monitoring | Drift detection signals across model versions |
| Anomalous code patterns (e.g. unexpected `V:` flips, `S:Ano` surges) | **Art. 73** — Serious incident reporting | Pre-investigation signal source |
| `S:` source hierarchy | **Art. 10** — Data governance *(supporting evidence)* | Per-decision record of which source class drove the answer |

**Reading guide:** "Supports" and "contributes to" are deliberately weaker than "satisfies." PRISM logs alone do not discharge any of these obligations. They give your compliance program structured evidence that auditors and notified bodies reference alongside your governance documentation, conventional event logs, risk management documentation, and post-market monitoring outputs. See [DISCLAIMER.md](./DISCLAIMER.md).

For Article 12(3) specifically: that subsection lists minimum logging contents for biometric identification systems (Annex III point 1(a)) and does not generically require hashing. The hash tool we provide is a defensive integrity measure, not a literal Art. 12(3) implementation.

---

## Integration

Three output modes — pick the one your model supports.

| Mode | When to use | User sees log? |
|---|---|---|
| **A. Inline tag** | Older models without structured output | No (host strips `<prism_log>` tag) |
| **B. Structured output (JSON)** | OpenAI, Gemini, Claude with JSON mode | No (separate JSON field) |
| **C. Tool call** | Claude, OpenAI, Gemini with native tool use | No (separate tool_use block) |

System prompts (drop into your existing system message):

**10-value profile (default — based on Schwartz 1992):**

- [`system_prompts/prism_v1_en_inline.md`](./system_prompts/prism_v1_en_inline.md) — Mode A
- [`system_prompts/prism_v1_en_json.md`](./system_prompts/prism_v1_en_json.md) — Mode B
- [`system_prompts/prism_v1_en_tool.md`](./system_prompts/prism_v1_en_tool.md) — Mode C

**19-value profile (extended — based on Schwartz et al. 2012, *Refined Theory of Basic Values*):**

- [`system_prompts/prism_v1_en_inline_19v.md`](./system_prompts/prism_v1_en_inline_19v.md) — Mode A
- [`system_prompts/prism_v1_en_json_19v.md`](./system_prompts/prism_v1_en_json_19v.md) — Mode B
- [`system_prompts/prism_v1_en_tool_19v.md`](./system_prompts/prism_v1_en_tool_19v.md) — Mode C

The 19-value profile splits the 10 values into 19 finer categories (e.g., `Sel` → `Sdt`/`Sda`; `Sec` → `Sep`/`Ses`). Use 10v for maximum cross-model consistency, 19v for finer audit signal. Pick one profile per system and stay with it — the two are not directly comparable.

**Critical:** In all modes and profiles, the PRISM log is for audit storage only — never shown to end users. Mode A requires the host to strip the tag. Modes B and C provide separation natively.

System prompt drop-in is a quick first step. Full deployment requires:
- Output validation (handling malformed codes)
- Wiring `prism_code` into your existing log pipeline (database, S3, SIEM)
- Hashing at storage time
- Retention policy aligned with Article 12(2)(a) and your sector requirements
- Drift monitoring on aggregate distributions

The system prompt addition itself takes minutes. Production deployment typically takes longer.

### Minimal example (Mode C, Claude)

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

Wire `prism_code` into your existing log pipeline. Run [`tools/prism_hash.py`](./tools/prism_hash.py) at storage time for chain integrity.

---

## Validated environments

| Provider / Model | Profile | Mode A | Mode B | Mode C |
|---|---|---|---|---|
| Anthropic Claude (Sonnet 4.5) | 10v | tested | tested | tested |
| Anthropic Claude (Sonnet 4.5) | 19v | tested | tested | tested |
| OpenAI GPT family | both | expected to work; community validation in progress | expected to work | expected to work |
| Google Gemini family | both | expected to work; community validation in progress | expected to work | expected to work |
| Open-source models (Llama, Mistral) | both | expected to work; community validation pending | varies by inference stack | varies |

We will update this table as community validation reports come in. Output token cost depends on tokenizer; expect roughly 25–40 tokens per code in practice (model-dependent). Measure on your stack before sizing.

---

## Why a code, not JSON

Compliance logs run for years across millions of decisions. Format matters.

| Property | PRISM code | JSON equivalent |
|---|---|---|
| Length | ~60 chars | ~300+ chars |
| Output token cost | ~25–40 tokens (model-dependent) | ~150+ tokens |
| Privacy | No verbatim content; topic-level metadata only | Risk of context leakage |
| Aggregation | Direct grep / SQL after extraction | Requires flattening |
| Auditor scan rate | High | Low |

The log is a structural fingerprint, not a summary. Conversation content stays in your conversation log. The PRISM line captures only what is needed for behavioral auditability at scale.

For deployments at >1M logs/day, plan an ETL pipeline (Airflow / Spark / ClickHouse / similar). The Python parser ([`tools/prism_parser.py`](./tools/prism_parser.py)) is suitable for batch validation and small-scale querying, not for streaming aggregation.

---

## What this does NOT claim

Stated plainly so your DPO, legal counsel, and notified body know what they are reading:

- **Not proof of true reasoning.** LLM self-reports can be post-hoc rationalisation. PRISM logs are *reported* reasoning, not causal traces. (This limitation applies equally to chain-of-thought, attention traces, and human-written documentation.)
- **Not a substitute for independent audit.** Internal logs need external verification to carry regulatory weight.
- **Not a replacement for conventional event logs.** See the complementary-layer section above.
- **Not automatic compliance.** This is a structured evidence layer. Compliance depends on your full governance system: risk management (Art. 9), data governance (Art. 10), conventional logging (Art. 12), transparency (Art. 13), human oversight (Art. 14), accuracy and robustness (Art. 15), quality management (Art. 17), post-market monitoring (Art. 72), and incident reporting (Art. 73).
- **Not a sole-source solution.** PRISM is one component of a broader compliance toolchain. Other relevant approaches include conventional event-log logging, chain-of-thought reasoning capture, NIST AI RMF documentation patterns, and IBM AI FactSheets. Pick the combination that fits your governance program.

What it is: an open, structured, regulator-friendly format for the reasoning layer of your compliance evidence — available today, free, and forkable.

---

## Repository contents

| File | Purpose |
|---|---|
| [`SPECIFICATION.md`](./SPECIFICATION.md) | Complete v1.0 code specification (includes output mode definitions) |
| [`DISCLAIMER.md`](./DISCLAIMER.md) | Scope, limitations, non-warranty terms |
| [`system_prompts/`](./system_prompts/) | Drop-in system prompts (EN + KR; 10v + 19v; all 3 modes) |
| [`examples/`](./examples/) | Real-world log examples across 7 domains |
| [`tests/validate.py`](./tests/validate.py) | Validator with multi-mode extraction (`--profile=10v|19v`) |
| [`tools/prism_parser.py`](./tools/prism_parser.py) | Code extraction and structured parsing |
| [`tools/prism_hash.py`](./tools/prism_hash.py) | SHA-256 integrity helper (independent and chained hashes) |
| [`docs/integration_guide.md`](./docs/integration_guide.md) | Per-provider integration guide |
| [`docs/README_KR.md`](./docs/README_KR.md) | 한국어 문서 |

---

## License

MIT. Use it, modify it, embed it in commercial products. Attribution appreciated, not required. See [LICENSE](./LICENSE).

This standard is a technical specification, not legal advice. Using it does not guarantee compliance with any regulation. See [DISCLAIMER.md](./DISCLAIMER.md) for full scope and limitations.

---

## Maintained by

[AI Integrity Organization (AIO)](https://aioq.org), a Swiss-registered nonprofit working on AI governance and behavioral auditability.

For the academic background of the PRISM framework, see the working papers:

- S. Lee (2026a). *AI Integrity: A New Paradigm for Verifiable AI Governance.* arXiv:[2604.11065](https://arxiv.org/abs/2604.11065) [cs.AI].
- S. Lee (2026b). *PRISM Risk Signal Framework: Hierarchy-Based Red Lines for AI Behavioral Risk.* arXiv:[2604.11070](https://arxiv.org/abs/2604.11070) [cs.AI].
- S. Lee (2026c). *Measuring the Authority Stack of AI Systems: Empirical Analysis of 366,120 Forced-Choice Responses Across 8 AI Models.* arXiv:[2604.11216](https://arxiv.org/abs/2604.11216) [cs.AI].

The 19-value profile draws specifically on:

- Schwartz, S. H., Cieciuch, J., Vecchione, M., Davidov, E., Fischer, R., Beierlein, C., et al. (2012). Refining the theory of basic individual values. *Journal of Personality and Social Psychology*, 103(4), 663–688.

---

## Contact

- Issues / suggestions: [GitHub issues](../../issues)
- Commercial inquiries: 2sk@aioq.org
- Website: [aioq.org](https://aioq.org)
