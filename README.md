# form-tailor

[![Version](https://img.shields.io/badge/version-0.2.0-blue.svg)](CHANGELOG.md)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-purple.svg)

**기관 양식 샘플을 넣으면 그 틀대로 새 문서를 만들어 주는** [Claude Code](https://claude.com/claude-code) 스킬입니다.

고정 템플릿을 내장하지 않습니다. 사용자가 제공한 **샘플(.hwp/.hwpx/.docx)에서 구조·글머리 체계·문체·서식을 학습**하고, 새 내용을 그 양식에 맞춰 생성한 뒤, **샘플 대비 서식 충실도를 점검**해 보고합니다.

> **English**: A Claude Code skill that reproduces *your* document format. Instead of shipping a fixed template, it parses a sample you provide (Korean HWP/HWPX or DOCX), extracts a **style profile** (section skeleton, bullet hierarchy □ㅇ-*①, 개조식 sentence endings, date/signature conventions, table layout, fonts), maps your new content into that skeleton, generates a document in the same format, and then runs a **fidelity check** against the sample. The skill carries no institutional template — the form is always your runtime input, so nothing proprietary lives in this repo.

## 왜 다른가

한글(HWP) 변환·생성 도구는 이미 많습니다. 하지만 대부분 "마크다운 → hwpx"처럼 **내용을 형식으로 바꾸는** 데 그칩니다. form-tailor는 반대 방향의 어려운 문제를 풉니다:

> **"이 기관 양식 / 지난번 보고서 틀에 정확히 맞춰라."**

- 📥 **양식은 런타임 입력** — 어느 기관 양식에도 적용, 저장소엔 어떤 기관 자산도 없음
- 🔍 **learn-from-sample** — 샘플에서 골격·기호·문체·표·서명 형식을 추출(kordoc 양식 지능 활용)
- ✅ **충실도 점검 내장** — 생성 후 "정말 그 양식을 지켰는가"를 항목별로 대조 보고 (이 스킬의 핵심 값)
- 🔒 **사실 불변·개인정보 보호** — 양식만 이식하고 수치·주장은 그대로, 샘플의 실명 서명은 복제하지 않음

## 작동 방식

```
[샘플 양식] ──파싱──▶ [양식 프로파일] ──┐
                                       ├──▶ [양식에 맞춘 새 문서] ──▶ [충실도 점검]
[새 내용/원고] ────정리──────────────────┘
```

1. **Phase 1 — 프로파일 추출**: 샘플을 파싱해 틀을 데이터로. 요약 보고 후 확인(경량 승인 게이트).
2. **Phase 2 — 내용 매핑**: 새 원고를 골격에 배치, 문체·기호 규칙 적용.
3. **Phase 3 — 생성**: 같은 형식으로 출력(hwpx는 원본 스타일 보존, docx는 스타일 적용).
4. **Phase 4 — 충실도 점검**: 샘플과 대조해 섹션·기호·문체·표·서명 일치를 항목별 판정.

예시: [examples/example-walkthrough.md](examples/example-walkthrough.md)

## 설치

```bash
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/parkjui92-tech/form-tailor.git
```

### 권장 의존성

- **[kordoc](https://github.com/chrisryugj/kordoc) MCP** — HWP/HWPX 파싱·폼필드 채움·구조 비교의 핵심. `parse_form`·`extract_profile`·`fill_form`·`compare_documents` 사용. 미설치 시 python-docx/LibreOffice로 폴백하나 hwpx 서식 보존은 제한됩니다.
- `.docx`는 python-docx 또는 docx 스킬로 처리.

## 사용법

Claude Code에서 샘플 파일과 내용을 주고 요청하면 트리거됩니다.

```
이 샘플 양식(부서보고.hwpx)에 이 내용 맞춰서 만들어줘
지난번 계획서 형식 그대로 이번 과제 계획서 작성
첨부한 기관 서식대로 정리해줘
```

## 범위

- 이 스킬은 **양식 이식**에 집중합니다. 내용의 조사·집필·사실 생성은 하지 않습니다(부족하면 지어내지 않고 요청).
- 특정 기관 규정 준수를 **보증하지 않습니다** — 제공된 샘플의 관측 서식을 재현하는 도구입니다.

## 연구자용 스킬 시리즈

- **form-tailor** (이 저장소) — 기관 양식 맞춤 제작
- **[paper-proofread](https://github.com/parkjui92-tech/paper-proofread)** — 한국어 학술 원고 교정교열
- **[fact-verify](https://github.com/parkjui92-tech/fact-verify)** — 출처 신뢰도 검증 (한국 학술·정책 문헌 포함)

## 라이선스

[Apache License 2.0](LICENSE)
