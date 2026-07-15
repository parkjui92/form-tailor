# 양식 프로파일 스키마

form-tailor가 샘플에서 추출해 기록하는 "양식 = 데이터" 형식. 고정 템플릿이 아니라 **샘플마다 새로 추출**한다. Phase 2에서 이 스키마의 `content`를 채워 생성기에 넘긴다.

## 프로파일 (샘플에서 추출)

```yaml
profile:
  doc_type: "보고 | 계획 | 계획서 | 공문 | 제안서 | 기타(자유기술)"   # 감지값
  source_format: "hwpx | docx | pdf"
  header:
    title: { position: "상단중앙", max_len: 20, style: "동작성 종결" }
    subtitle: { present: false }
    date_format: "'YY. M. D.(요일)"        # 샘플에서 관측된 실제 패턴
    signature: { format: "직책 + 이름", position: "제목 하단 우측" }
    doc_number: { present: true|false, pattern: "..." }
  skeleton:                                  # 섹션 골격 (순서 있음)
    - { level: 0, marker: "Ⅰ. Ⅱ.", label_example: "개요" }
    - { level: 1, marker: "□",       label_example: "추진배경" }
    - { level: 2, marker: "ㅇ",      indent_spaces: 1 }
    - { level: 3, marker: "-",       indent_spaces: 3 }
    - { level: 4, marker: "*",       indent_spaces: 5 }
    - { level: 2, marker: "① ②",    note: "번호 열거 시" }
  required_sections: ["요약(◇)", "붙임"]     # 반드시 존재해야 하는 섹션
  style:
    tense_ending: "개조식(~함/~임/~할 예정임) | 서술체(~한다) | 경어체"
    sentence_max_lines: 2
    one_focus_per_item: true
    emphasis: "핵심 명사 키워드 볼드 위주"
  tables:
    present: true|false
    conventions: "헤더 음영, 가운데 정렬, 4열 등 관측값"
  fonts:                                     # 가능한 경우만 (hwpx 스타일 공여)
    preserve_from_sample: true               # 원본 스타일을 그대로 보존
    observed: "제목 HY헤드라인M / 본문 휴먼명조 15pt 등(있으면)"
  margins_layout: "용지·여백·줄간격은 원본 보존"
```

## 채워진 콘텐츠 spec (Phase 2 산출)

프로파일의 `skeleton` 레벨에 맞춰 내용을 배치. **글머리 기호는 넣지 않는다**(생성기가 부착).

```yaml
content:
  header:
    title: "..."                 # 프로파일 규칙에 맞춰 정규화
    date: "..."                  # date_format 적용
    signature: "..."             # 새 내용 기준 (샘플 실명 복제 금지)
  blocks:
    - { level: 1, marker: "□", text: "추진배경" }
    - { level: 2, marker: "ㅇ", text: "..." }     # 기호 없이 텍스트만
    - { level: 3, marker: "-", text: "..." }
    - { level: 2, marker: "①", num: 1, text: "..." }
  tables:
    - { after_block: 3, header: ["구분","현황","계획","비고"], rows: [[...]] }
  attachment: "붙임 본문 또는 표준 문구"
```

## 원칙

- 프로파일은 **관측 기반**이다. 샘플에 없는 규칙을 지어내지 않는다(불명확하면 "미관측"으로 두고 일반 관행 폴백을 표시).
- `preserve_from_sample: true`인 항목(글꼴·여백·표 스타일)은 새로 정의하지 말고 **원본을 스타일 공여자로 보존**한다.
- 이 스키마는 특정 기관 전용이 아니다. `doc_type`·`skeleton`은 샘플마다 달라진다.
