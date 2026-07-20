# form-tailor

[![Version](https://img.shields.io/badge/version-0.2.0-blue.svg)](CHANGELOG.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-purple.svg)

**한국어** · [English](README.en.md)

기관 양식 샘플을 넣으면 **그 틀대로 새 문서를 만들어 주는** [Claude Code](https://claude.com/claude-code) 스킬입니다.
고정 템플릿을 내장하지 않습니다 — 양식은 언제나 **런타임 입력**이고, 생성 후에는 "정말 그 양식을 지켰는가"를 샘플과 항목별로 대조해 스스로 보고합니다.

<!-- 데모 GIF 자리 -->

## 왜 템플릿을 안 넣나

한글(HWP) 문서를 다루는 도구는 이미 많습니다. 그런데 대부분은 **"마크다운 → hwpx"**, 즉 *내용을 형식으로 바꾸는* 방향을 풉니다 — 이때 형식은 도구가 정해 둔 표준 형식입니다. 실무에서 자주 필요한 건 반대 방향입니다. 목표 형식이 이미 정해져 있고, 그건 도구가 아니라 **발주처나 상급 부서가 정한 것**입니다.

그렇다고 기관별 템플릿을 내장할 수는 없습니다. 양식은 기관마다 다르고 같은 기관 안에서도 문서 종류마다 달라 **내장 목록은 언제나 뒤처지고**, 남의 기관 서식 파일을 저장소에 넣어 배포하는 것은 **애초에 해서는 안 되는 일**입니다. 그래서 반대로 설계했습니다 — **양식을 코드가 아니라 런타임 입력으로 받습니다**(BYO-template). 이 저장소에는 어떤 기관의 서식 자산도 들어 있지 않고, 동봉된 예시조차 실제 양식이 아닌 가상의 예시입니다.

→ [왜 만들었나·상세 사용법](docs/why.md)

## 작동 방식

```
[샘플 양식] ──파싱──▶ [양식 프로파일] ──┐
                                       ├──▶ [양식에 맞춘 새 문서] ──▶ [충실도 점검]
[새 내용/원고] ────정리──────────────────┘
```

입력은 **두 가지**이고 둘 다 필수입니다 — 따라 할 **양식 샘플**(`.hwp`/`.hwpx`/`.docx`, `.pdf`는 구조 참고용)과 **채울 내용**(원고·메모·다른 형식으로 써 둔 초안). 샘플 없이 "기관 양식대로"만 요청하면 **양식을 지어내지 않고 샘플을 요청합니다.**

1. **프로파일 추출 ★** — 섹션 골격, 레벨별 글머리 기호와 들여쓰기, 종결 어미, 제목·날짜·서명 형식, 표 관행을 데이터로 뽑고 **여기서 한 번 멈춰 확인을 받습니다.**
2. **내용 매핑** — 새 원고를 골격에 배치하고 문체 규칙을 적용합니다. 없는 정보는 지어내지 않고, 표에 데이터를 욱여넣지 않고, 수치·고유명사는 건드리지 않습니다.
3. **생성** — 프로파일에 맞는 형식으로 출력합니다(hwpx는 kordoc, docx는 python-docx로 **관측한** 서식을 재적용). 원본 샘플은 덮어쓰지 않습니다.
4. **충실도 점검** — 생성물을 샘플 프로파일과 항목별로 대조해 표로 보고합니다([전체 흐름 예시](examples/example-walkthrough.md)).

## 충실도 점검

양식에 맞춰 뽑아 놓고 보면 대체로 그럴듯해 보입니다. 문제는 **그럴듯해 보이는 것과 실제로 맞는 것이 다르다**는 데 있습니다 — 항목 세 개만 서술체로 남아 있어도, 하위 항목 들여쓰기가 3칸이 아니라 2칸이어도, 필수 「붙임」 절이 통째로 빠져 있어도 눈으로는 잘 안 걸립니다. **생성 자체보다 이 대조가 이 스킬의 값입니다.**

✅ 샘플과 일치 · ⚠️ 부분 일치이거나 **스킬이 임의로 결정하지 않고 남겨 둔 자리**(문체 변환이 애매한 항목, 정보가 없어 넣은 placeholder, 표 열 충돌 같은 결정 지점) — **여기가 사람이 봐야 하는 곳**이고 답을 주면 그대로 반영합니다 · 🔴 누락·위반. 한 줄이라도 🔴이면 스킬은 "양식 준수"라고 보고하지 않습니다.

## 설치

```bash
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/parkjui92/form-tailor.git
```

설치 후 Claude Code를 재시작하면 관련 요청에 자동으로 반응합니다.

## 쓰는 법

```
지난번 주간보고(첨부) 형식 그대로 이번 주 내용 정리해줘   ← 지난번 문서 틀 재사용
첨부한 서식 파일에 맞춰서 이 초안 내용을 배치해줘         ← 빈 양식 폼 채우기
이 마크다운 초안을 첨부한 기관 서식대로 다시 만들어줘     ← 다른 형식 초안 이식
하위 항목은 3칸이 아니라 2칸이야                          ← ★프로파일 확인에서
서술체로 남은 3개 항목 개조식으로 고쳐줘                  ← 점검표에 ⚠️가 나왔을 때
```

프로파일 확인에서 한 번 멈춰 섭니다. 잘못 읽은 틀로 문서 전체를 만들어 놓고 되돌리는 것보다, 여기서 한 줄 고치는 편이 훨씬 쌉니다.

## 범위·한계

- `.hwp`/`.hwpx` 파싱·폼필드 채움·구조 비교에는 [kordoc](https://github.com/chrisryugj/kordoc) MCP가 필요합니다. 미설치 시 python-docx / LibreOffice로 폴백하지만 **hwpx 서식 보존은 제한**됩니다
- **내용을 만들지도, 양식을 지어내지도 않습니다.** 조사·집필은 이 스킬의 일이 아니고 수치·주장도 바꾸지 않습니다 — 양식만 이식합니다. 샘플의 실명 서명·연락처도 새 문서로 복제하지 않고 점검표에 표시합니다
- **특정 기관의 규정 준수를 보증하지 않습니다.** 제공된 샘플에서 **관측된** 서식을 재현하는 도구입니다
- **점검표의 ✅가 "모든 서식이 완벽하다"는 뜻은 아닙니다.** `.docx`는 python-docx가 스타일 상속을 해석하지 않아 본문 글꼴·줄간격이 자주 `UNOBSERVED`가 되고, 관측하지 못한 서식은 점검 대상에도 오르지 못합니다. `.pdf` 샘플은 구조 참고용입니다
- 공개된 검증 기록은 합성 양식으로 4개 Phase를 완주한 스모크 테스트 한 건입니다 ([CHANGELOG.md](CHANGELOG.md))
- 세부 규칙: [SKILL.md](SKILL.md) · [프로파일 스키마](references/profile-schema.md) · [공문서 관행·개조식 변환표](references/korean-form-conventions.md) · [점검 항목](references/fidelity-checklist.md)

## 시리즈

**에이전트 팀 킷** — [policy-research-kit](https://github.com/parkjui92/policy-research-kit) (정책연구보고서) · [rnd-proposal-kit](https://github.com/parkjui92/rnd-proposal-kit) (정부 R&D 제안서) · [socsci-paper-kit](https://github.com/parkjui92/socsci-paper-kit) (사회과학 논문)

**제작·편집 킷** — [lecture-deck-kit](https://github.com/parkjui92/lecture-deck-kit) (강의자료 HTML 덱 · 브라우저 라이브 편집)

**단독 스킬** — **form-tailor** (이 저장소, 기관 양식 맞춤) · [fact-verify](https://github.com/parkjui92/fact-verify) (출처 검증) · [paper-proofread](https://github.com/parkjui92/paper-proofread) (한국어 학술 교정교열) · [report-to-brief](https://github.com/parkjui92/report-to-brief) (보고서 압축)

## 라이선스

[MIT](LICENSE). 독점 기관 양식·서식 파일은 포함하지 않습니다(BYO-template 원칙).
