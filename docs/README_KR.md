# EU AI Act 컴플라이언스 로깅 — PRISM 표준 v1.0

**EU AI Act 제12조 추론 로깅을 위한 오픈소스 pre-standard. 시스템 프롬프트로 통합. MIT 라이선스. 전통적 이벤트 로그와 함께 감사관이 기대하는 구조화된 증거 레이어를 제공.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Version](https://img.shields.io/badge/version-1.0-blue.svg)](../CHANGELOG.md)
[![EU AI Act](https://img.shields.io/badge/EU%20AI%20Act-Article%2012-red.svg)](#컴플라이언스-매핑-먼저-읽으세요)
[![English](https://img.shields.io/badge/docs-English-blue.svg)](../README.md)

---

## ⚠️ 2026년 8월 2일

EU AI Act 고위험 의무 대부분(제12조 로깅 요건 포함)이 Annex III 등재 시스템에 대해 강제력을 갖는 날입니다. (Annex I 일부 분야 카테고리는 2027년 8월 2일 적용. 시스템 카테고리를 먼저 확인하십시오.)

귀사의 고위험 시스템이 해당 적용일까지 결정별 감사 가능 기록을 산출하지 못하면 운영자 의무가 충족되지 않습니다. 제99조는 가장 심각한 위반에 대해 1,500만 유로 또는 전 세계 연 매출 3%까지의 과징금 상한을 설정합니다. 제12조 위반은 일반적으로 운영자 의무 위반 티어(상한 1,500만 유로 또는 3%)에 해당하며 정확한 액수는 위반 정황에 따라 달라집니다. 첫 위반의 경우 일반적으로 과징금 부과 전에 시정 조치 통지가 선행됩니다.

전통적 로그(응답시간, 토큰 수, 요청 ID)는 제12조의 일부를 충족하지만 결정의 **추론** — 어떤 가치가 우선되었는지, 어떤 증거가 채택되었는지, 어떤 출처를 신뢰했는지 — 는 포착하지 못합니다. 제13조(투명성)와 제14조(인간 감독 가능성)는 단순 이벤트 추적을 넘어선 기대치를 요구합니다.

**공식 조화 표준(CEN/CENELEC)은 아직 개발 중입니다.** 운영자는 표준 확정을 기다린 후 구조화된 로깅을 시작할 수 없습니다.

이 저장소는 더 넓은 컴플라이언스 프로그램의 한 구성 요소로 오늘 바로 배포 가능한 작동하는 pre-standard입니다.

---

## 🟦 중요: 보완 레이어이며 대체재가 아닙니다

PRISM은 기존 이벤트 로그 파이프라인에 추가되는 **추론 추적 레이어**입니다. 자체적으로 제12조를 충족하지 않습니다.

여전히 다음을 로깅해야 합니다:
- 사용자 식별자, 타임스탬프, 세션 ID (제12조(1) 수명주기 기록)
- 입력과 출력 (생체인식 시스템의 제12조(2)(c); 일반적인 위험 식별)
- 시스템 가동 기간 (제12조(2)(a))
- 해당하는 경우 데이터베이스 참조 (제12조(2)(b))

PRISM은 이들에 결정별 추론 지문을 **추가**합니다. 어느 것도 대체하지 않습니다.

일반적인 컴플라이언스 배포는 다음과 같습니다:

```
[전통적 이벤트 로그] → 입력/출력/타임스탬프/사용자ID
        +
[PRISM 추론 로그]    → 결정별 C/V/E/S 코드
        +
[해싱 레이어]        → 양쪽에 걸친 SHA-256 체인 (변조 방지)
        ↓
[감사 저장소]        → 제12조(2)(a) 보관 요건에 맞게 인덱싱·보관
```

전통적 이벤트 로그 없이 PRISM만 도입하면 여전히 미준수 상태입니다. 이 분야 일부 마케팅이 그렇지 않다고 암시하지만 — 사실이 아니므로 — 명시합니다.

---

## 무엇을 얻나요

주요 LLM이 재학습 없이 즉시 출력할 수 있는 어휘 체계입니다. 이 저장소가 제공하는 시스템 프롬프트를 추가하기만 하면 됩니다. 모델이 수행하는 모든 실질적 결정마다 한 줄의 구조화된 코드가 생성됩니다:

```
<prism_log>
C:MD/IXi | V:Ben<Sel | E:Exp<Gui | S:Usr<Pro
</prism_log>
```

약 60자. 사용자 콘텐츠 원문 없음. 토픽 수준 메타데이터만 포함 (`C:` 도메인은 "이건 의료 질의였다" 정도만 드러내며 — 기존 라우팅 로그가 이미 노출하는 수준과 비슷합니다).

각 부분의 의미:

- **C** — 컨텍스트: 도메인, 영향 범위, 가역성, 시간 지평
- **V** — 가치 위계: 어떤 가치 우선순위가 적용되었는지
- **E** — 증거 위계: 어떤 증거 유형이 결정적이었는지
- **S** — 출처 위계: 어떤 출처 클래스를 신뢰했는지

`<`는 "~에 밀림"을 의미합니다. 왼쪽 = 덜 중시됨. 오른쪽 = 우선됨.

[전체 명세 →](../SPECIFICATION.md)

---

## 컴플라이언스 매핑 (먼저 읽으세요)

왼쪽 컬럼은 PRISM이 기술적으로 산출하는 것입니다. 오른쪽 컬럼은 EU AI Act 조항 중 PRISM이 **뒷받침하는**(단독으로 충족하지 않는) 것입니다. 전체 컴플라이언스 프로그램 — 위험 관리, 데이터 거버넌스, 시판후 모니터링 — 은 제9조부터 제17조까지의 의무 전체를 중심으로 구축되어야 하며, PRISM 단독으로 가능하지 않습니다.

| PRISM이 산출하는 것 | 뒷받침하는 EU AI Act 조항 | 감사관 관점의 유용성 |
|---|---|---|
| 실질적 결정마다 PRISM 한 줄 | **제12조(1)** — 자동 기록 보관 | 전통적 이벤트 로그를 보완하는 머신 생성 추론 추적 |
| `C:` 레이어 (도메인/범위/가역성/시간) | **제12조(2)** — 위험 상황 추적성 | 결정별 위험 컨텍스트 태그 |
| 가동 기간 전반의 `C:` 분포 집계 | **제12조(2)(a)** — 사용 기간 기록 (타임스탬프와 결합) | 시간에 따른 결정의 분량과 카테고리 |
| [`tools/prism_hash.py`](../tools/prism_hash.py)를 통한 SHA-256 체인 해시 | 변조 방지 (일반 추적성 원칙 및 Recital 73에서 함의됨; 제12조(3) 직접 텍스트는 아님) | 로그 스트림에 대한 보관 사슬 증거 |
| `V:` 및 `E:` 위계 | **제13조** — 추론의 투명성 *(뒷받침; 제13조의 주된 프레임은 deployer 대상 투명성)* | 출력 뒤의 가치 우선순위와 증거 유형 보고 |
| 구조화된 한 줄 코드, grep·SQL 친화적 | **제14조** — 인간 감독 가능성 *(기여; 제14조 핵심은 인간 정지·재정의 권한)* | 인간 감사관이 대규모로 검사 가능한 형식 |
| 버전 간 코드 분포 집계 | **제15조** — 정확성·견고성 모니터링 | 모델 버전 간 드리프트 신호 |
| 이상 코드 패턴 (예: 예상치 못한 `V:` 반전, `S:Ano` 급증) | **제73조** — 중대 사고 보고 | 사전 조사용 신호 출처 |
| `S:` 출처 위계 | **제10조** — 데이터 거버넌스 *(보조 증거)* | 답변을 이끈 출처 클래스의 결정별 기록 |

**해석 가이드:** "뒷받침"·"기여"는 "충족"보다 의도적으로 약한 표현입니다. PRISM 로그 자체가 위 의무를 면제해주지 않습니다. 컴플라이언스 프로그램이 자체 거버넌스 문서, 전통적 이벤트 로그, 위험 관리 문서, 시판후 모니터링 결과와 함께 감사관·인증기관에 제출할 수 있는 구조화된 증거를 제공합니다. [DISCLAIMER.md](../DISCLAIMER.md) 참조.

제12조(3)에 대해 명시: 해당 항은 생체인식 시스템(Annex III 1(a))에 대한 최소 로깅 내용을 열거하며 일반적 해싱을 요구하지 않습니다. 본 저장소가 제공하는 해시 도구는 방어적 무결성 조치이며 제12조(3)을 직접 구현하는 것이 아닙니다.

---

## 통합

세 가지 출력 모드 — 모델이 지원하는 것을 선택하세요.

| 모드 | 사용 시점 | 사용자에게 노출? |
|---|---|---|
| **A. Inline 태그** | 구조화 출력 미지원 구형 모델 | 안 됨 (호스트가 `<prism_log>` 태그 제거) |
| **B. Structured Output (JSON)** | OpenAI, Gemini, Claude의 JSON 모드 | 안 됨 (별도 JSON 필드) |
| **C. Tool Call** | Claude, OpenAI, Gemini의 네이티브 도구 사용 | 안 됨 (별도 tool_use 블록) |

시스템 프롬프트 (기존 system 메시지에 추가):

**10-가치 프로필 (기본 — Schwartz 1992 기반):**

- [`system_prompts/prism_v1_kr_inline.md`](../system_prompts/prism_v1_kr_inline.md) — 모드 A
- [`system_prompts/prism_v1_kr_json.md`](../system_prompts/prism_v1_kr_json.md) — 모드 B
- [`system_prompts/prism_v1_kr_tool.md`](../system_prompts/prism_v1_kr_tool.md) — 모드 C

**19-가치 프로필 (확장 — Schwartz et al. 2012, *Refined Theory of Basic Values* 기반):**

- [`system_prompts/prism_v1_kr_inline_19v.md`](../system_prompts/prism_v1_kr_inline_19v.md) — 모드 A
- [`system_prompts/prism_v1_kr_json_19v.md`](../system_prompts/prism_v1_kr_json_19v.md) — 모드 B
- [`system_prompts/prism_v1_kr_tool_19v.md`](../system_prompts/prism_v1_kr_tool_19v.md) — 모드 C

19-가치 프로필은 10가치를 19개의 더 세분화된 카테고리로 분할합니다 (예: `Sel` → `Sdt`/`Sda`; `Sec` → `Sep`/`Ses`). 모델 간 일관성 최대화는 10v, 더 세밀한 감사 신호는 19v를 선택하십시오. 시스템당 하나의 프로필을 선택하고 유지하십시오 — 두 프로필은 직접 비교 불가능합니다.

**중요:** 모든 모드와 프로필에서 PRISM 로그는 감사 저장용입니다 — 최종 사용자에게 절대 노출하면 안 됩니다. 모드 A는 호스트가 태그를 제거해야 합니다. 모드 B와 C는 분리가 자체 보장됩니다.

시스템 프롬프트 추가 자체는 빠른 첫 단계입니다. 전체 배포에는 다음이 필요합니다:
- 출력 검증 (잘못된 코드 처리)
- `prism_code`를 기존 로그 파이프라인(데이터베이스, S3, SIEM)에 연결
- 저장 시점 해싱
- 제12조(2)(a) 및 분야 요건에 맞는 보관 정책
- 집계 분포에 대한 드리프트 모니터링

시스템 프롬프트 추가는 분 단위입니다. 프로덕션 배포는 더 오래 걸립니다.

### 최소 통합 예시 (모드 C, Claude)

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
    system=PRISM_MODE_C_PROMPT,  # system_prompts/prism_v1_kr_tool.md 에서
    tools=[PRISM_TOOL],
    messages=[{"role": "user", "content": user_input}]
)

# 사용자 노출 텍스트와 감사 로그는 별도 블록
user_facing = ""
prism_code = None
for block in response.content:
    if block.type == "text":
        user_facing += block.text
    elif block.type == "tool_use" and block.name == "record_prism_log":
        prism_code = block.input["code"]

# user_facing → 사용자 인터페이스
# prism_code → 감사 로그 저장소
```

`prism_code`를 기존 로그 파이프라인에 연결하세요. 저장 시점에 [`tools/prism_hash.py`](../tools/prism_hash.py)를 실행하면 체인 무결성을 충족합니다.

---

## 검증된 환경

| 제공자 / 모델 | 프로필 | 모드 A | 모드 B | 모드 C |
|---|---|---|---|---|
| Anthropic Claude (Sonnet 4.5) | 10v | 검증 완료 | 검증 완료 | 검증 완료 |
| Anthropic Claude (Sonnet 4.5) | 19v | 검증 완료 | 검증 완료 | 검증 완료 |
| OpenAI GPT 계열 | 양쪽 | 작동 예상; 커뮤니티 검증 진행 중 | 작동 예상 | 작동 예상 |
| Google Gemini 계열 | 양쪽 | 작동 예상; 커뮤니티 검증 진행 중 | 작동 예상 | 작동 예상 |
| 오픈소스 모델 (Llama, Mistral) | 양쪽 | 작동 예상; 커뮤니티 검증 대기 | 추론 스택에 따라 다름 | 추론 스택에 따라 다름 |

커뮤니티 검증 보고가 들어오는 대로 이 표를 갱신합니다. 출력 토큰 비용은 토크나이저에 따라 다릅니다 — 실측 시 코드당 약 25–40 토큰을 예상하십시오 (모델 의존). 시스템 사이징 전 자체 스택에서 측정하십시오.

---

## 왜 JSON이 아니라 코드 형식인가

컴플라이언스 로그는 수년간 수백만 건의 결정에 걸쳐 누적됩니다. 형식이 중요합니다.

| 속성 | PRISM 코드 | JSON |
|---|---|---|
| 길이 | 약 60자 | 약 300자 이상 |
| 출력 토큰 비용 | 약 25–40 토큰 (모델 의존) | 약 150 토큰 이상 |
| 프라이버시 | 원문 없음; 토픽 수준 메타데이터만 | 컨텍스트 누출 위험 |
| 집계 | 추출 후 직접 grep/SQL | 평탄화 필요 |
| 감사관 스캔 속도 | 높음 | 낮음 |

로그는 요약이 아니라 **구조 지문**입니다. 대화 콘텐츠는 기존 대화 로그에 있습니다. PRISM 한 줄은 대규모 행동 감사 가능성에 필요한 것만 포착합니다.

일일 100만 건 이상 규모의 배포는 ETL 파이프라인(Airflow / Spark / ClickHouse 등)을 계획해야 합니다. Python 파서([`tools/prism_parser.py`](../tools/prism_parser.py))는 배치 검증과 소규모 쿼리에 적합하며 스트리밍 집계용은 아닙니다.

---

## 이 표준이 주장하지 않는 것

DPO, 법무, 인증기관이 무엇을 읽고 있는지 명확히 알 수 있도록 직접적으로 명시합니다:

- **모델이 "진짜로" 그렇게 추론했다는 증명이 아닙니다.** LLM 자기 보고는 사후적 합리화일 수 있습니다. PRISM 로그는 *보고된* 추론이지, 인과적 추적이 아닙니다. (이 한계는 chain-of-thought, attention trace, 인간 작성 문서에도 동일하게 적용됩니다.)
- **독립 감사를 대체하지 않습니다.** 내부 로그는 규제적 무게를 갖기 위해 외부 검증이 필요합니다.
- **전통적 이벤트 로그를 대체하지 않습니다.** 위 보완 레이어 섹션 참조.
- **자동으로 컴플라이언스를 보장하지 않습니다.** 구조화된 증거 레이어입니다. 컴플라이언스는 전체 거버넌스 시스템에 의존합니다: 위험 관리(제9조), 데이터 거버넌스(제10조), 전통적 로깅(제12조), 투명성(제13조), 인간 감독(제14조), 정확성·견고성(제15조), 품질 관리(제17조), 시판후 모니터링(제72조), 사고 보고(제73조).
- **단독 솔루션이 아닙니다.** PRISM은 더 넓은 컴플라이언스 도구체인의 한 구성 요소입니다. 다른 관련 접근법으로 전통적 이벤트 로그, chain-of-thought 추론 캡처, NIST AI RMF 문서화 패턴, IBM AI FactSheets 등이 있습니다. 거버넌스 프로그램에 맞는 조합을 선택하십시오.

이것이 제공하는 것: 컴플라이언스 증거의 추론 레이어를 위한 오픈·구조화·규제 친화적 형식 — 오늘 사용 가능, 무료, fork 가능.

---

## 저장소 구성

| 파일 | 용도 |
|---|---|
| [`SPECIFICATION.md`](../SPECIFICATION.md) | v1.0 코드 명세 전문 (출력 모드 정의 포함) |
| [`DISCLAIMER.md`](../DISCLAIMER.md) | 범위, 한계, 무보증 조건 |
| [`system_prompts/`](../system_prompts/) | 즉시 사용 가능한 시스템 프롬프트 (EN + KR; 10v + 19v; 3 모드) |
| [`examples/`](../examples/) | 7개 도메인의 실제 로그 예시 |
| [`tests/validate.py`](../tests/validate.py) | 다중 모드 추출 검증기 (`--profile=10v|19v`) |
| [`tools/prism_parser.py`](../tools/prism_parser.py) | 코드 추출 및 구조화 파싱 |
| [`tools/prism_hash.py`](../tools/prism_hash.py) | SHA-256 무결성 (독립·체인 해시) |
| [`docs/integration_guide.md`](./integration_guide.md) | 제공자별 통합 가이드 |

---

## 라이선스

MIT. 자유롭게 사용, 수정, 상업적 임베드 가능. 출처 표기는 권장이지만 필수 아님. [LICENSE](../LICENSE) 참조.

이 표준은 기술 명세이며 법률 자문이 아닙니다. 사용 자체가 어떤 규제든 준수를 보장하지는 않습니다. 전체 범위와 한계는 [DISCLAIMER.md](../DISCLAIMER.md)를 참조하세요.

---

## 유지 관리

[AI Integrity Organization (AIO)](https://aioq.org), 스위스 등록 비영리 조직. AI 거버넌스와 행동 감사 가능성을 다룹니다.

PRISM 프레임워크의 학술 배경은 다음 워킹 페이퍼를 참조하세요:

- S. Lee (2026a). *AI Integrity: Definition, Authority Stack Model, and Enhanced Cascade Mapping Hypothesis.* arXiv:cs.AI.
- S. Lee (2026b). *The PRISM Framework for Measuring AI Value Hierarchies.* arXiv:cs.AI.
- S. Lee (2026c). *Measuring AI Value Priorities: Empirical Analysis.* arXiv:cs.AI.

19-가치 프로필은 다음을 직접 참조합니다:

- Schwartz, S. H., Cieciuch, J., Vecchione, M., Davidov, E., Fischer, R., Beierlein, C., et al. (2012). Refining the theory of basic individual values. *Journal of Personality and Social Psychology*, 103(4), 663–688.

---

## 연락처

- 이슈 / 제안: [GitHub issues](../../../issues)
- 상업적 문의: 2sk@aioq.org
- 웹사이트: [aioq.org](https://aioq.org)
