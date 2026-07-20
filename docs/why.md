# 왜 만들었나 · 상세 사용법

**한국어** · [English](#english)

README에서 덜어낸 배경과 4개 Phase 상세, 사용 시나리오를 여기 둡니다.

---

## 왜 만들었나

한국에서 기관 문서를 쓰는 일의 진짜 비용은 **내용이 아니라 양식**에 있습니다.

내용은 대개 이미 손에 있습니다. 무슨 일이 있었고, 다음에 뭘 하고, 뭐가 문제인지는 담당자가 제일 잘 압니다. 정작 시간을 잡아먹는 건 그다음입니다.

- 어떤 문서는 절 제목이 `□`이고 항목이 `ㅇ`인데, 어떤 문서는 `Ⅰ.`부터 시작합니다. 하위 항목 들여쓰기가 1칸이냐 3칸이냐도 다릅니다.
- 어떤 문서는 개조식(`~함`, `~할 예정임`)이고, 어떤 문서는 경어체(`~합니다`)입니다. 같은 기관 안에서도 보고서와 공문이 다릅니다.
- 날짜를 `'26. 5. 7.(목)`으로 쓰는 곳이 있고 아닌 곳이 있습니다. 서명을 "직책 + 이름"으로 제목 아래 오른쪽에 두는 곳이 있고, 다르게 두는 곳이 있습니다.
- 표는 열 구성부터 헤더 음영까지 지난번 문서를 그대로 따라야 합니다.

그리고 이런 것들은 **틀리면 반려됩니다.** 내용이 아무리 정확해도 그렇습니다. 그래서 실무에서는 늘 지난번 파일을 옆에 띄워 놓고 형식을 눈으로 대조하면서 새 내용을 밀어 넣는 작업이 반복됩니다.

### 기존 도구는 반대 방향을 풀고 있었습니다

한글(HWP) 문서를 다루는 도구는 이미 많습니다. 그런데 대부분은 **"마크다운 → hwpx"**, 즉 *내용을 형식으로 바꾸는* 방향을 풉니다. 이때 형식은 도구가 정해 둔 표준 형식입니다.

실무에서 자주 필요한 건 반대 방향입니다.

> **"이 기관 양식 / 지난번 보고서 틀에 정확히 맞춰라."**

목표 형식이 이미 정해져 있고, 그건 도구가 아니라 **발주처나 상급 부서가 정한 것**입니다. 내가 고를 수 있는 게 아닙니다.

### 그렇다고 템플릿을 내장할 수는 없었습니다

가장 쉬운 해법은 기관별 템플릿을 잔뜩 넣어 두는 것입니다. 두 가지 이유로 실패합니다.

1. **커버가 안 됩니다.** 양식은 기관마다 다르고, 같은 기관 안에서도 문서 종류마다 다릅니다. 내장 템플릿 목록은 언제나 뒤처집니다.
2. **담아서는 안 되는 자산입니다.** 남의 기관 서식 파일을 저장소에 넣어 배포하는 것은 애초에 해서는 안 되는 일입니다.

그래서 반대로 설계했습니다. **양식을 코드가 아니라 런타임 입력으로 받습니다**(BYO-template). 사용자가 자기 샘플을 주면 거기서 틀을 데이터로 뽑아 씁니다. 이 저장소에는 **어떤 기관의 서식 자산도 들어 있지 않습니다.** 동봉된 [examples/example-walkthrough.md](../examples/example-walkthrough.md)조차 실제 양식이 아닌 가상의 예시입니다.

### 그리고 생성만으로는 부족했습니다

양식에 맞춰 뽑아 놓고 보면 대체로 그럴듯해 보입니다. 문제는 **그럴듯해 보이는 것과 실제로 맞는 것이 다르다**는 데 있습니다. 항목 세 개만 서술체로 남아 있어도, 하위 항목 들여쓰기가 3칸이 아니라 2칸이어도, 필수 「붙임」 절이 통째로 빠져 있어도 눈으로는 잘 안 걸립니다.

그래서 이 스킬은 생성으로 끝내지 않고 **마지막에 샘플과 항목별로 대조합니다**(Phase 4). 섹션 골격, 기호 체계, 종결 어미, 날짜 표기, 표 열 구성, 서명 형식을 각각 판정해 보고합니다. **생성 자체보다 이 대조가 이 스킬의 값입니다.**

---

## 무엇이 다른가

- 📥 **양식은 런타임 입력** — 어느 기관 양식에도 적용됩니다. 저장소에 기관 자산이 없습니다.
- 🔍 **learn-from-sample** — 샘플에서 골격·기호·문체·표·서명 형식을 추출합니다(hwpx는 kordoc의 양식 지능 활용).
- ✅ **충실도 점검 내장** — 생성 후 "정말 그 양식을 지켰는가"를 항목별로 대조 보고합니다.
- 🚫 **관측한 것만 기록** — 파서가 못 읽은 서식은 지어내지 않고 `UNOBSERVED`로 남깁니다. "본문 15pt"라고 단정하려면 실제로 읽혀야 합니다.
- 🔒 **사실 불변·개인정보 미복제** — 양식만 이식하고 수치·고유명사는 그대로 둡니다. 샘플에 있던 실명 서명은 새 문서로 복제하지 않습니다.

---

## 4개 Phase 상세

### Phase 1 — 프로파일 추출 ★사용자 확인 지점

샘플을 파싱해 틀을 데이터로 만듭니다. 문서 유형, 섹션 골격(계층·순서·필수 섹션), 레벨별 글머리 기호와 들여쓰기, 종결 어미, 제목·날짜·서명·문서번호 형식, 표 관행, 글꼴·여백을 뽑습니다. 형식마다 파싱 경로가 다릅니다.

| 형식 | 파싱 경로 |
|---|---|
| `.hwp` / `.hwpx` | **kordoc MCP** — `parse_document`·`parse_form`·`extract_profile`·`parse_table`·`detect_format` |
| `.docx` | **python-docx** — 글머리는 선행 글자 + 선행 공백 수로, 글꼴은 3단 폴백으로, 표 음영은 `w:shd` XML 직접 파싱으로, 여백은 EMU로 |
| `.pdf` | 텍스트·레이아웃 참고용 (정밀 서식 재현은 제한적) |

**여기서 한 번 멈춰 사용자에게 확인을 받습니다.** "이 샘플에서 이런 틀을 읽었습니다 — 관측하지 못한 항목은 이것입니다. 이대로 맞출까요?" 잘못 읽은 틀로 문서 전체를 만들어 놓고 나서 되돌리는 낭비를 막는 지점입니다.

> 비대화형·단발 실행(배치·서브에이전트 등)이라 확인을 기다릴 수 없으면 멈추지 않습니다. 프로파일을 산출물로 남기고 진행한 뒤, 확인이 필요한 항목을 Phase 4 점검표에 ⚠️로 모아 둡니다.

### Phase 2 — 내용 매핑

새 원고를 골격에 배치하고 문체 규칙을 적용합니다. 이 단계의 규율 세 가지가 중요합니다.

- **없는 정보는 지어내지 않습니다.** 날짜·서명자 같은 머리 정보가 새 내용에 없으면 형식만 적용한 placeholder를 넣고 ⚠️로 표시해 물어봅니다.
- **표에 데이터를 욱여넣지 않습니다.** 새 데이터의 자연스러운 열 수가 샘플 표와 다르면, 빈 칸을 "-"로 메우거나 한 값을 반복해 채우는 대신 사용자에게 묻습니다(샘플 열에 맞출지, 데이터에 맞는 새 표를 만들지). 무단 편입은 데이터를 왜곡합니다.
- **수치와 고유명사는 건드리지 않습니다.** 어미를 바꾸는 과정에서 의미가 흔들릴 것 같으면 원문을 유지하고 플래그합니다.

명사구 메모를 개조식으로 바꿀 때는 항목이 **어느 섹션에 속하는지**로 시제를 정합니다. 같은 "공모 마감"도 실적 섹션이면 "공모를 마감함", 계획 섹션이면 "마감할 예정임"입니다(→ [references/korean-form-conventions.md](../references/korean-form-conventions.md)).

### Phase 3 — 생성

프로파일에 맞는 형식으로 출력합니다. 글머리 기호와 들여쓰기는 이 단계에서 부착합니다(Phase 2 텍스트에 미리 넣지 않아 **기호 이중부착**을 막습니다).

- **hwpx 샘플 → hwpx 출력** — kordoc `fill_form`(폼필드 양식) 또는 스타일 공여 빌드로 원본 서식을 보존합니다.
- **docx 샘플 → docx 출력** — python-docx로 생성하되, 원본에서 **관측한** 서식(제목 글꼴·크기·정렬, 셀 음영, 여백 EMU 등)을 명시적으로 재적용합니다.

원본 샘플은 덮어쓰지 않고 새 파일로 만듭니다. 검토용 마크다운 초안을 함께 뽑을 수도 있습니다.

### Phase 4 — 충실도 점검 ★결과를 읽는 지점

생성물을 샘플 프로파일과 대조해 표로 보고합니다. hwpx는 kordoc `compare_documents`로 구조를 비교할 수 있습니다.

| 항목 | 샘플 | 생성물 | 판정 |
|------|------|--------|------|
| 섹션 골격·순서 | Ⅰ□ㅇ- 계층 | 동일 | ✅ |
| 글머리 기호체계 | ㅇ1칸/-3칸/*5칸 | 동일 | ✅ |
| 기호 이중부착 | — | 0건 | ✅ |
| 개조식 문체 | ~함/~임 | 3개 항목 서술체 잔존 | ⚠️ 위치 명시 |
| 날짜 표기 | '26. 5. 7.(목) | 형식 일치, 값 placeholder | ⚠️ 확인요청 |
| 표 열 구성 | 4열 | 데이터 형상에 맞춰 2열 생성 | ⚠️ 사유 명시 |
| 필수 섹션 | 붙임 포함 | 누락 | 🔴 |

**판정 기호는 이렇게 읽으면 됩니다.**

- ✅ — 샘플과 일치합니다. 그대로 두면 됩니다.
- ⚠️ — 부분 일치이거나, **스킬이 임의로 결정하지 않고 남겨 둔 자리**입니다. 대개 셋 중 하나입니다. ①문체 변환이 애매해 원문을 살려 둔 항목, ②날짜·서명자처럼 정보가 없어 placeholder를 넣은 자리, ③표 열 충돌처럼 결정이 필요한 지점. **여기가 사람이 봐야 하는 곳**이고, 답을 주면 그대로 반영합니다.
- 🔴 — 누락·위반입니다. 한 줄이라도 🔴이면 스킬은 "양식 준수"라고 보고하지 않습니다.

`UNOBSERVED`가 보인다면 그건 실패가 아니라 **정직한 보고**입니다. 파서가 그 서식을 읽지 못했다는 뜻이고, 대개 `.docx`의 본문 글꼴·줄간격에서 나옵니다. 그런 항목은 원본을 직접 열어 확인하는 편이 빠릅니다.

전체 흐름 예시: [examples/example-walkthrough.md](../examples/example-walkthrough.md)

---

## 상세 사용법

샘플 파일과 새 내용을 함께 주면 트리거됩니다.

### 시나리오 1 — 지난번 문서 틀에 이번 내용

```
지난번 주간보고(첨부) 형식 그대로 이번 주 내용 정리해줘.
이번 주 항목은 메모에 있어.
```

가장 흔한 쓰임입니다. 사람이 "지난번 파일 열어 놓고 눈으로 대조"하던 그 작업입니다.

### 시나리오 2 — 발주처가 준 빈 양식 폼 채우기

```
첨부한 서식 파일에 맞춰서 이 초안 내용을 배치해줘.
```

빈 폼이 샘플 역할을 합니다. hwpx 폼필드가 있으면 kordoc `fill_form`으로 채웁니다. 프로파일에 **필수 섹션**이 있는데 새 내용에 대응 항목이 없으면, 비워 두지 않고 무엇을 넣을지 물어봅니다.

### 시나리오 3 — 다른 형식으로 써 둔 초안을 기관 양식으로 이식

```
이 마크다운 초안을 첨부한 기관 서식대로 다시 만들어줘.
내용은 그대로 두고 형식만 맞추면 돼.
```

"내용은 그대로"가 이 스킬의 기본 동작입니다 — 수치·주장을 바꾸지 않고 양식만 이식합니다. 반대로 내용 자체를 새로 쓰거나 조사해야 한다면 그건 이 스킬의 일이 아닙니다.

### ★ Phase 1 프로파일 확인에서 개입하는 법

스킬이 "이 틀로 읽었습니다"라고 보고할 때가 **가장 값싸게 방향을 고칠 수 있는 지점**입니다. 이렇게 말하면 됩니다.

```
하위 항목은 3칸이 아니라 2칸이야
이 문서는 개조식 말고 경어체로 가야 해
「붙임」 절도 필수야 — 빠뜨리지 마
표는 샘플 열 말고 내 데이터 형상에 맞춰서 새로 만들어
```

프로파일이 잘못 읽힌 채로 진행하면 문서 전체를 다시 만들어야 합니다. 여기서 한 줄 고치는 편이 훨씬 쌉니다.

### 충실도 점검에서 ⚠️가 나왔을 때

```
서명자는 ○○○ 팀장이야 — 채워줘
서술체로 남은 3개 항목 개조식으로 고쳐줘
표는 샘플 4열에 맞춰서 다시 짜줘
```

⚠️는 스킬이 결정을 미뤄 둔 자리이므로, 답을 주면 그대로 반영됩니다.

---

## 이 스킬이 하지 않는 것

경계를 분명히 해 두는 편이 서로 편합니다.

- **내용을 만들지 않습니다.** 조사·집필·사실 생성은 이 스킬의 일이 아닙니다. 내용이 부족하면 채워 넣지 않고 요청합니다.
- **수치·주장을 바꾸지 않습니다.** 양식만 이식합니다. 문체 변환이 의미를 건드릴 것 같으면 원문을 유지하고 플래그합니다.
- **양식을 지어내지 않습니다.** 샘플 없이는 시작하지 않고, 파서가 못 읽은 서식은 `UNOBSERVED`로 남깁니다.
- **샘플의 개인정보를 복제하지 않습니다.** 실명 서명·연락처는 새 문서로 옮기지 않고 점검표에 표시합니다.
- **특정 기관의 규정 준수를 보증하지 않습니다.** 제공된 샘플에서 **관측된** 서식을 재현하는 도구입니다. 정확한 규정이 필요하면 해당 기관의 문서 관리 지침·행정업무운영 편람을 확인해야 합니다.

## 한계

- **`.hwp`/`.hwpx`의 온전한 처리는 kordoc에 의존합니다.** 없으면 `.docx` 경로로 우회해야 하고, 그 과정에서 서식이 손실될 수 있습니다.
- **`.docx`는 파서가 읽을 수 있는 것에 한계가 있습니다.** python-docx는 스타일 상속을 해석하지 않아 본문 글꼴·줄간격이 `None`으로 나오는 일이 흔합니다. 여백·용지는 정확히 관측되고, 글꼴·줄간격은 자주 `UNOBSERVED`가 됩니다. 스킬은 이를 숨기지 않고 표기합니다.
- **`.pdf` 샘플은 구조 참고용입니다.** 정밀 서식 재현은 기대하기 어렵습니다.
- **충실도 점검은 프로파일에 담긴 것만 대조합니다.** 관측하지 못한 서식은 점검 대상에도 오르지 못합니다. 점검표의 ✅가 "모든 서식이 완벽하다"는 뜻은 아닙니다.
- **이 저장소에 공개된 검증 기록은 스모크 테스트 한 건입니다.** v0.2.0은 합성 양식으로 4개 Phase를 완주한 적대적 스모크 테스트에서 드러난 결함(.docx 경로 과소명세, 표 스키마 충돌, 머리 정보 결측 처리, 비대화형 폴백)을 보정한 판입니다 — 자세한 내용은 [CHANGELOG.md](../CHANGELOG.md).

---

## 저장소 구성

| 파일 | 내용 |
|---|---|
| [SKILL.md](../SKILL.md) | 스킬 본체 — 4개 Phase 절차와 안전·범위 원칙 |
| [references/profile-schema.md](../references/profile-schema.md) | 양식 프로파일 스키마(YAML) + 파서별 관측 신뢰도 표 |
| [references/korean-form-conventions.md](../references/korean-form-conventions.md) | 한국 공문서 일반 관행 + 명사구→개조식 변환표 |
| [references/fidelity-checklist.md](../references/fidelity-checklist.md) | Phase 4 점검 항목 (hwpx·docx 각각) |
| [examples/example-walkthrough.md](../examples/example-walkthrough.md) | 4개 Phase 전체 흐름 예시 (가상 양식) |

> `korean-form-conventions.md`는 **폴백**입니다. 언제나 **샘플에서 관측된 규칙이 일반 관행보다 우선**합니다. 샘플이 서술체면 서술체를 따릅니다.

---
---

<a name="english"></a>

# Why I built this · Detailed usage

[한국어](#왜-만들었나--상세-사용법) · **English**

Background, the four-phase detail, and usage scenarios trimmed out of the README.

---

## Why I built this

In Korean institutional work, the real cost of a document is **not the content — it's the format.**

The content is usually already in hand. The person writing the report knows what happened, what's next, and what the problem is. The time goes somewhere else:

- One document uses `□` for section headings and `ㅇ` for items; another starts at `Ⅰ.`. Sub-items are indented one space in one house style and three in another.
- One document is written in *gaejosik*; another is in polite declarative form. Within the same institution, an internal report and an outgoing official letter follow different conventions.
- Some places write dates as `'26. 5. 7.(목)`; others don't. Some put the signature as "title + name" below the heading on the right; others place it differently.
- Tables have to match the previous document from column structure down to header cell shading.

And getting these wrong **gets the document sent back** — no matter how correct the content is. So the actual workflow is: open last time's file next to the new one, and push new content in while eyeballing the format.

> **Two terms, if you're not working in Korean.** **HWP/HWPX** is the file format of Hangul Word Processor, the de facto standard for Korean public-sector and institutional paperwork — roughly what `.docx` is elsewhere, but with its own conventions and its own tooling gap. ***Gaejosik*** (개조식) is the terse outline register of Korean official reports: each item is a clipped phrase ending in a specific nominalized verb form rather than a full sentence — `~함` for completed work, `~할 예정임` for planned work, `~이 필요함` for a request. Which ending is correct depends on what kind of section the item sits in, so "just rewrite it in bullet points" does not get you there.
>
> **And if you're not in Korea, this may still be familiar.** Anyone who has filed a government procurement bid, a regulatory submission, a grant report, or a standards-body filing has met the same problem: the receiving institution has already decided what the document must look like, and "close enough" is not a passing grade. The particulars here are Korean. The shape of the problem is not.

### Existing tools solve the opposite direction

There is no shortage of tooling for Korean documents. But most of it solves **"Markdown → hwpx"** — turning *content into a format*, where the format is the one the tool decided on.

What working professionals need more often runs the other way:

> **"Match this institution's format / last quarter's report — exactly."**

The target format is already fixed, and it was fixed by **the commissioning body or the department above you**, not by your tool. It isn't yours to choose.

### But shipping templates wasn't an option

The easy fix is to bundle a pile of per-institution templates. It fails twice:

1. **Coverage never arrives.** Formats differ across institutions, and differ again by document type *within* an institution. A built-in template list is permanently behind.
2. **They aren't mine to ship.** Putting another organization's form files into a public repository and distributing them is something you simply shouldn't do.

So the design goes the other way: **the format is a runtime input, not code** (bring-your-own-template). You hand over your own sample; the skill extracts the shape from it as data. **This repository contains no institutional form assets of any kind.** Even the bundled [examples/example-walkthrough.md](../examples/example-walkthrough.md) is a fictional example, not a real form.

### And generating wasn't enough

Output that's been shaped to a format generally *looks* right. The problem is that **looking right and being right are different things.** Three items left in the wrong sentence register, a sub-item indented two spaces instead of three, a mandatory "Attachments" section missing entirely — none of that jumps out at the eye.

So the skill doesn't stop at generation. It **compares its output against the sample, item by item** (Phase 4): section skeleton, bullet system, sentence endings, date notation, table columns, signature format, each judged and reported. **That comparison, more than the generation, is what this skill is for.**

---

## What makes it different

- 📥 **The format is a runtime input** — works with any institution's format; no institutional assets live in this repo.
- 🔍 **Learn-from-sample** — extracts skeleton, bullet system, register, table conventions and signature format from your sample (using kordoc's format intelligence for hwpx).
- ✅ **Built-in fidelity check** — after generating, it reports item by item whether the format was actually followed.
- 🚫 **Only what was observed** — formatting the parser couldn't read is not invented; it's recorded as `UNOBSERVED`. To assert "body text is 15pt," it has to have actually read 15pt.
- 🔒 **Facts unchanged, personal data not copied** — only the format is transplanted; figures and proper nouns are left alone, and real names in the sample's signature block are not carried into the new document.

---

## The four phases in detail

### Phase 1 — Profile extraction ★ your checkpoint

The sample is parsed into data: document type, section skeleton (hierarchy, order, mandatory sections), the bullet marker and indentation for each level, sentence-ending register, title/date/signature/document-number conventions, table practice, fonts and margins. The parsing path depends on the format:

| Format | Parsing path |
|---|---|
| `.hwp` / `.hwpx` | **kordoc MCP** — `parse_document`, `parse_form`, `extract_profile`, `parse_table`, `detect_format` |
| `.docx` | **python-docx** — bullet level from the leading marker character plus leading-space count; font via a three-step fallback; table shading by reading `w:shd` XML directly; margins in EMU |
| `.pdf` | Text and layout for reference (precise format reproduction is limited) |

**The skill stops here and asks you to confirm:** "Here's the shape I read out of your sample — and here's what I couldn't observe. Shall I match this?" This is the checkpoint that prevents building an entire document against a misread profile and then having to unwind it.

> In non-interactive or single-shot runs (batch jobs, subagents) where there's nobody to answer, it does not block. It writes the profile out as an artifact, proceeds, and collects everything needing confirmation into the Phase 4 checklist as ⚠️.

### Phase 2 — Content mapping

New content is placed into the skeleton and the register rules are applied. Three disciplines matter here:

- **Missing information is not invented.** If the new content has no date or signatory, a placeholder in the correct *format* goes in, flagged ⚠️ for you to confirm.
- **Data is not forced into tables.** If your data's natural column count differs from the sample's table, the skill does not pad blanks with "-" or repeat a value to fill the shape — it asks whether to conform to the sample's columns or build a new table that fits the data. Forcing the fit distorts the data.
- **Figures and proper nouns are untouched.** If converting a sentence ending looks like it would shift the meaning, the original text stays and gets flagged.

When converting noun-phrase notes into *gaejosik*, tense is decided by **which section the item belongs to**. The same note, "call for proposals closed," becomes "closed the call" in a results section and "will close the call" in a plan section (see [references/korean-form-conventions.md](../references/korean-form-conventions.md)).

### Phase 3 — Generation

Output is produced in the profile's format. Bullet markers and indentation are attached **at this stage** — they're deliberately kept out of the Phase 2 text so markers can't end up **double-attached**.

- **hwpx sample → hwpx output** — kordoc `fill_form` (for form-field templates) or a style-donor build, preserving the original's formatting.
- **docx sample → docx output** — generated with python-docx, explicitly re-applying the formatting that was **observed** in the original (heading font/size/alignment, cell shading, margins in EMU).

The original sample is never overwritten; output goes to a new file. A Markdown draft can be emitted alongside for review.

### Phase 4 — Fidelity check ★ how to read the result

The output is compared against the sample profile and reported as a table. For hwpx, kordoc `compare_documents` can compare structure directly.

| Item | Sample | Output | Verdict |
|---|---|---|---|
| Section skeleton and order | Ⅰ□ㅇ- hierarchy | identical | ✅ |
| Bullet system | ㅇ 1 space / - 3 / * 5 | identical | ✅ |
| Double-attached markers | — | none | ✅ |
| Gaejosik register | ~함 / ~임 | 3 items still in declarative form | ⚠️ locations given |
| Date notation | '26. 5. 7.(목) | format matches, value is a placeholder | ⚠️ confirm |
| Table columns | 4 columns | 2 columns, built to fit the data | ⚠️ reason given |
| Mandatory sections | includes Attachments | missing | 🔴 |

**How to read the verdicts:**

- ✅ — matches the sample. Nothing to do.
- ⚠️ — a partial match, or **a decision the skill deliberately left to you**. Almost always one of three things: ① an item where the register conversion was ambiguous, so the original wording was kept; ② a placeholder where information (date, signatory) simply wasn't available; ③ a genuine decision point such as a table-column conflict. **This is the column a human needs to read** — answer, and it gets applied.
- 🔴 — missing or in violation. If even one line is 🔴, the skill does not report the document as format-compliant.

If you see `UNOBSERVED`, that is not a failure — it's an **honest report**. It means the parser couldn't read that piece of formatting, and it most often shows up for body font and line spacing in `.docx`. For those fields it's faster to open the original and look.

A full worked run: [examples/example-walkthrough.md](../examples/example-walkthrough.md)

---

## Detailed usage

Provide the sample file and the new content together.

### Scenario 1 — This week's content in last time's shape

```
Format this week's items exactly like the attached weekly report.
The items are in my notes.
```

The most common use. It's the same job as opening last time's file and eyeballing the format — done for you.

### Scenario 2 — Filling an empty form the commissioning body sent

```
Lay this draft out according to the attached form template.
```

The empty form serves as the sample. If it has hwpx form fields, kordoc `fill_form` populates them. If the profile has a **mandatory section** with no corresponding content in your draft, the skill asks what belongs there rather than leaving it blank.

### Scenario 3 — Porting a draft written in some other format

```
Rebuild this Markdown draft in the attached institutional format.
Keep the content as-is — only the format needs to match.
```

"Keep the content as-is" is the default behavior: figures and claims aren't touched, only the format is transplanted. If the content itself needs to be researched or written, that's a different job.

### ★ Intervening at the Phase 1 profile check

When the skill reports "here's the shape I read," that is **the cheapest point at which to correct course.** Just say so:

```
Sub-items are indented two spaces, not three
This one should be in polite declarative form, not gaejosik
The Attachments section is mandatory — don't drop it
Build the table to fit my data instead of the sample's columns
```

Proceeding on a misread profile means rebuilding the whole document. Fixing one line here is far cheaper.

### When the fidelity check comes back with ⚠️

```
The signatory is the team lead — fill it in
Convert the 3 items still in declarative form
Rebuild the table to the sample's 4 columns
```

⚠️ marks decisions the skill held back on, so answering applies them directly.

---

## What this skill doesn't do

Clear boundaries make this easier for everyone.

- **It doesn't produce content.** Research, writing, and fact generation are not this skill's job. If content is missing, it asks rather than filling in.
- **It doesn't change figures or claims.** Only the format is transplanted. If a register conversion looks like it would shift meaning, the original stays and gets flagged.
- **It doesn't invent formats.** It won't start without a sample, and formatting the parser couldn't read is recorded as `UNOBSERVED`.
- **It doesn't copy personal data out of your sample.** Real names and contact details in the signature block are not carried into the new document; the checklist notes it.
- **It doesn't guarantee compliance with any institution's regulations.** It reproduces the formatting **observed** in the sample you provided. For authoritative rules, check that institution's own document-management guidance.

## Limitations

- **Full `.hwp`/`.hwpx` handling depends on kordoc.** Without it you have to route through `.docx`, and formatting can be lost in transit.
- **`.docx` parsing has a hard ceiling.** python-docx doesn't resolve style inheritance, so body font and line spacing frequently come back `None`. Margins and page size are read reliably; fonts and line spacing often end up `UNOBSERVED`. The skill labels this rather than hiding it.
- **`.pdf` samples are structural reference only.** Don't expect precise format reproduction.
- **The fidelity check only compares what made it into the profile.** Formatting that was never observed can't be checked either — a table full of ✅ does not mean every aspect of the format is perfect.
- **The verification record published in this repo is a single smoke test.** v0.2.0 is the release that fixed the defects it exposed — an under-specified `.docx` path, table schema conflicts, missing header information, and the non-interactive fallback — after an adversarial run completed all four phases against a synthetic form. Details in [CHANGELOG.md](../CHANGELOG.md).

---

## Repository layout

| File | Contents |
|---|---|
| [SKILL.md](../SKILL.md) | The skill itself — the four-phase procedure and its safety/scope principles |
| [references/profile-schema.md](../references/profile-schema.md) | Format profile schema (YAML) + per-parser observability table |
| [references/korean-form-conventions.md](../references/korean-form-conventions.md) | General Korean official-document conventions + noun-phrase → *gaejosik* conversion table |
| [references/fidelity-checklist.md](../references/fidelity-checklist.md) | Phase 4 checklist (separate items for hwpx and docx) |
| [examples/example-walkthrough.md](../examples/example-walkthrough.md) | A full four-phase run (fictional form) |

> `korean-form-conventions.md` is a **fallback**. Rules observed in your sample always take precedence over general convention. If your sample is in declarative form, the skill follows the sample.
