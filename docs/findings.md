# 분석 결과 형식 (findings)

> **초안 v0.1** · 작성: 소희(ai) · 의견 받는 중
> 이 형식이 정해져야 backend 저장, frontend 결과 화면, 자문단 정답셋 작성을 시작할 수 있습니다.

| 누가 | 어떻게 쓰나 |
|---|---|
| ai | `/review` 응답으로 만든다 |
| backend | `contract_reviews.findings`, `contract_questions`에 저장한다 |
| frontend | 결과 화면(요약, 위험도, 꼭 알아야 할 점, 상의할 점, 원문 근거), 질문 목록을 그린다 |
| 연구 | 자문단 정답셋(`expert_answers.answer_items`)을 **같은 형태**로 쓴다 → AI 결과와 자동 비교 |

## 1. 한눈에 보기

`/review` 응답 예시입니다. 내용은 가상 계약서 기준이며, 판단 문구는 형식을 보여 주기 위한 예시입니다.

```json
{
  "summary": "보증금 500만 원, 월세 45만 원 계약입니다. 퇴실 청소비 공제와 중도 퇴실 시 중개수수료 부담 조건이 있어 미리 확인이 필요합니다.",
  "counts": { "CAUTION": 1, "CHECK": 2, "GOOD": 1 },
  "findings": [
    {
      "id": "f1",
      "category": "MUST_KNOW",
      "risk_level": "CAUTION",
      "topic": "EARLY_TERMINATION",
      "title": "중도 퇴실하면 새 세입자와 중개수수료를 내가 책임져요",
      "explanation": "계약 기간 중에 나가려면 다음 세입자를 직접 구하고, 그 중개수수료도 내야 한다는 조건입니다.",
      "actions": [
        "중도 퇴실 가능성이 있다면 계약 전에 부담 범위를 줄일 수 있는지 상의하기",
        "새 세입자를 못 구하면 보증금은 언제 돌려받는지 확인하기"
      ],
      "evidence": [
        { "page": 2, "clause_label": "특약 3항", "text": "계약기간 내 중도 퇴실 시 임차인이 새 임차인을 구하고 중개수수료를 부담한다.", "bbox": null }
      ],
      "question": "중도 퇴실할 때 새 세입자를 못 구하면 보증금은 언제 돌려주시나요?",
      "law_refs": []
    },
    {
      "id": "f2",
      "category": "DISCUSS",
      "risk_level": "CHECK",
      "topic": "DEPOSIT_DEDUCTION",
      "title": "퇴실 청소비 15만 원이 보증금에서 빠져요",
      "explanation": "나갈 때 청소 상태와 관계없이 15만 원을 공제하는 조건입니다.",
      "actions": ["청소를 직접 하면 공제하지 않는지 상의하기"],
      "evidence": [
        { "page": 2, "clause_label": "특약 2항", "text": "퇴실 시 청소비 15만원은 보증금에서 공제한다.", "bbox": null }
      ],
      "question": "직접 청소하고 나가도 청소비 15만 원을 공제하시나요?",
      "law_refs": []
    },
    {
      "id": "f3",
      "category": "MISSING",
      "risk_level": "CHECK",
      "topic": "MAINTENANCE_FEE",
      "title": "관리비에 무엇이 포함되는지 적혀 있지 않아요",
      "explanation": "관리비 5만 원의 포함 항목이 '별도 협의'로 되어 있어, 수도·인터넷 등을 따로 내야 하는지 알 수 없습니다.",
      "actions": ["포함 항목을 계약서 특약에 적어 달라고 요청하기"],
      "evidence": [
        { "page": 1, "clause_label": "제1조", "text": "관리비 금 오만원정 (₩50,000) (정액, 포함 항목: 별도 협의)", "bbox": null }
      ],
      "question": "관리비 5만 원에 수도, 인터넷, 전기 중 무엇이 포함되나요?",
      "law_refs": []
    },
    {
      "id": "f4",
      "category": "MUST_KNOW",
      "risk_level": "GOOD",
      "topic": "LEASE_TERM",
      "title": "계약 기간은 2년이에요",
      "explanation": "2027년 1월 1일부터 2028년 12월 31일까지 24개월입니다.",
      "actions": [],
      "evidence": [
        { "page": 1, "clause_label": "제2조", "text": "임대차 기간은 인도일로부터 2028년 12월 31일까지(24개월)로 한다.", "bbox": null }
      ],
      "question": null,
      "law_refs": []
    }
  ],
  "meta": {
    "model_name": "(미정)",
    "model_version": "(미정)",
    "prompt_version": 1
  }
}
```

## 2. 항목 (finding)

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `id` | string | O | 응답 안에서만 고유 (`f1`, `f2` …). 질문이 어느 항목에서 나왔는지 연결할 때 씀 |
| `category` | enum | O | 화면 구역. [코드표](#4-코드표) 참고 |
| `risk_level` | enum | O | `CAUTION`(주의) / `CHECK`(확인) / `GOOD`(양호) |
| `topic` | enum | O | 무엇에 관한 항목인지. 정답셋과 비교할 때 기준이 됨 |
| `title` | string | O | 한 줄 요약. 임차인 눈높이의 쉬운 말 |
| `explanation` | string | O | 쉬운 설명. 법률 용어는 풀어서 |
| `actions` | string[] | O | 임차인이 할 수 있는 행동. 없으면 `[]` |
| `evidence` | Evidence[] | O | 원문 근거. `MISSING`은 관련 조항이 없으면 `[]` |
| `question` | string \| null | O | 집주인에게 물어볼 질문. `CAUTION`·`CHECK`는 필수, `GOOD`은 `null` |
| `law_refs` | LawRef[] | O | 관련 법 조항. RAG 붙기 전(4주차)까지는 `[]`. 이후 유료 기능 후보 |

### Evidence (원문 근거)

`contract_details`의 항목별 근거에도 같은 형태를 씁니다.

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `page` | int | O | 1부터. 업로드 순서대로 이어 붙인 페이지 번호 (`/ocr` 응답과 같음) |
| `clause_label` | string \| null | O | 사람이 읽는 위치. `제1조`, `특약 3항` 등. 못 정하면 `null` |
| `text` | string | O | 계약서 원문 문장 (프론트에서 가린 이미지를 읽은 것이라 마스킹된 상태) |
| `bbox` | number[4] \| null | O | `[x1, y1, x2, y2]`, 페이지 크기 대비 0~1 비율. 원본 사진은 분석 후 삭제되므로 화면은 **텍스트 근거 위주**로 보여 주고, bbox는 원본이 있을 때만 씀 |

### LawRef (법령 근거 · 4주차)

| 필드 | 타입 | 설명 |
|---|---|---|
| `law_name` | string | `주택임대차보호법` |
| `article` | string | `제6조의3` |
| `text` | string | 조문 요약 또는 인용 |
| `effective_date` | date | 인용한 조문의 시행일 (연혁 구분용) |

## 3. 요약·개수·질문

- `summary`: 결과 화면 맨 위 요약. → `contract_reviews.summary`
- `counts`: `risk_level`별 개수. 홈 목록의 "주의 N · 확인 N" 표시용. → `contract_reviews`에 컬럼으로 둘지 backend와 결정
- 질문 목록: `findings[].question`이 있는 항목마다 `contract_questions` 한 행으로 저장

| `contract_questions` 컬럼 | 값 |
|---|---|
| `field` | 해당 항목의 `topic` |
| `question_text` | `question` |
| `evidence` | `evidence[0].text` (없으면 NULL) |
| `display_order` | `risk_level` 순(CAUTION → CHECK), 같으면 `findings` 순서 |
| `source` (추가 요청) | `AI`. 사용자가 직접 추가한 질문은 `USER` |

## 4. 코드표

### `category` (화면 구역)

| 코드 | 화면 이름 | 언제 |
|---|---|---|
| `MUST_KNOW` | 꼭 알아야 할 점 | 임차인이 부담하거나 제한받는 조건, 계약의 핵심 조건 |
| `DISCUSS` | 집주인과 상의할 점 | 계약 전에 조정하거나 확인하면 좋은 조건 |
| `MISSING` | 계약서에 빠진 내용 | 있어야 할 내용이 없거나 "별도 협의"처럼 비어 있음 |

### `risk_level`

| 코드 | 화면 | 의미 |
|---|---|---|
| `CAUTION` | 주의 | 임차인에게 불리할 수 있어 계약 전에 꼭 확인해야 함 |
| `CHECK` | 확인 | 문제는 아니지만 미리 확인하면 좋음 |
| `GOOD` | 양호 | 일반적인 조건. 안심 요소로 보여 줌 |

### `topic` (항목 주제)

| 코드 | 주제 |
|---|---|
| `DEPOSIT` | 보증금 금액·지급 일정·반환 |
| `DEPOSIT_DEDUCTION` | 보증금에서 공제하는 비용 (청소비 등) |
| `RENT` | 월세 금액·지급일·연체 |
| `MAINTENANCE_FEE` | 관리비 금액·포함 항목·산정 방식 |
| `LEASE_TERM` | 계약 기간·갱신 |
| `EARLY_TERMINATION` | 중도 해지·퇴실 |
| `RESTORATION` | 원상복구 범위·비용 |
| `REPAIR` | 수리·하자 책임 |
| `BROKERAGE_FEE` | 중개수수료 |
| `USE_RESTRICTION` | 용도 변경·전대·반려동물·흡연 등 생활 제한 |
| `PROPERTY_RIGHTS` | 등기·근저당·선순위 권리 |
| `LANDLORD_IDENTITY` | 임대인 본인·대리인 확인 |
| `MOVE_IN_REGISTRATION` | 전입신고·확정일자 |
| `OTHER` | 그 밖의 특약 |

`topic`을 추가·변경할 때는 이 문서를 먼저 고치는 PR을 올립니다 (정답셋과 비교 기준이 바뀌기 때문).

## 5. 정답셋 (`expert_answers.answer_items`)

자문단 답변은 **같은 finding 형태**로 입력합니다. 다른 점만 적습니다.

| 필드 | 정답셋에서는 |
|---|---|
| `id` | `e1`, `e2` … |
| `evidence` | `page`와 `clause_label`, `text`만 있으면 됨. `bbox`는 `null` |
| `question`, `actions` | 선택. 있으면 질문·행동 품질 비교에 씀 |
| `title`, `explanation` | 자문자 표현 그대로. 비교에는 쓰지 않고 참고용 |
| `note` (추가) | 자문자 메모. 판단 근거, 관행 설명 등 |

입력은 자문단이 스프레드시트에 `topic / risk_level / 조항 / 원문 / 설명`을 쓰면 운영자(연구 담당)가 이 형태로 옮기는 방식을 생각하고 있습니다.

## 6. 일치율 계산 (초안)

연구 목표(전문가 검토 대체율 50~80%)를 재는 기준입니다. 숫자는 `evaluation_results.score`, 항목별 결과는 `evaluation_details`에 저장합니다.

1. **짝 맞추기**: 정답 항목과 AI 항목은 `topic`이 같고 근거 조항이 같으면 짝으로 봅니다 (`clause_label`이 같거나 `text`가 겹침). `MISSING`은 `topic`만 같으면 짝으로 봅니다.
2. **지표**
   - **찾아낸 비율 (재현율)**: 정답 항목 중 AI가 짝을 찾은 비율 ← 대체율의 기본값
   - **위험도 일치율**: 짝지어진 항목 중 `risk_level`이 같은 비율
   - **과잉 지적 비율**: AI 항목 중 정답에 없는 비율 (`GOOD` 제외)
3. 위험도는 `CAUTION`과 `GOOD`을 바꿔 말한 경우를 가장 큰 오류로 봅니다.

이 기준은 ai 레포에서 오프라인으로 먼저 돌려 보고, 숫자가 안정되면 `evaluation_results`에 기록합니다.

## 7. 정할 것

- [ ] `counts`를 `contract_reviews` 컬럼으로 둘지, `findings`에서 계산할지 (backend)
- [ ] `contract_questions.source` 컬럼 추가 (backend)
- [ ] `topic` 목록이 자문단이 보는 관점과 맞는지 (자문단 1차 검토)
- [ ] 결과 화면에서 `GOOD` 항목을 어디에, 몇 개까지 보여 줄지 (frontend)
- [ ] 일치율 짝 맞추기 기준과 대체율 목표에 쓸 지표 (연구)
