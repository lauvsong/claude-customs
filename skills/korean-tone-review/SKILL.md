---
name: korean-tone-review
description: '한국어 산출물(README·문서·공유 메시지·긴 응답)의 AI 직역체·기계 번역체를 mechanical 패턴 매칭으로 검수한다. 출력 직전 또는 사용자가 명시 요청 시 호출. Use when finishing a long Korean deliverable (README, doc, message draft, structured long response), or when user requests review with phrases: 톤 봐줘, 어색한 거 잡아줘, 직역체 체크, korean-tone-review, /korean-tone-review.'
---

# korean-tone-review

긴 한국어 산출물을 출력 직전에 self-check 가 아닌 mechanical 패턴 매칭으로 검수한다. 자기 작성 컨텍스트 안의 self-check 는 직역체를 잡지 못한다는 것이 이번 도입 배경.

## 언제 트리거하나

- README · 기술 문서 · 공유 메시지 · 보고서 같은 **긴 한국어 산출물** 작성 직후
- 사용자 명시 요청: "톤 봐줘", "어색한 거 잡아줘"

짧은 응답·코드·영어 텍스트는 면제.

## 검수 절차 (5단계)

### 1. 텍스트 수집

검수 대상 텍스트를 모은다. 코드 블록(```)·인용·링크 텍스트는 제외하고 자연어 본문만.

### 2. S1 패턴 — mechanical 매칭, 한 건도 통과 X

[`references/ai-tell-taxonomy.md`](references/ai-tell-taxonomy.md) 의 S1 카테고리를 정규식 수준으로 매칭한다. 발견 시 무조건 패치.

핵심 매칭 룰 (전체는 taxonomy 참고):

| 패턴 | 정규식 (개념) | 예 |
|---|---|---|
| Rhetorical 헤딩 | `^#+\s.+(인가|하는가|보이나|되나)\??$` | `## 왜 X인가`, `## 차단되면 어떻게 보이나` |
| 콜론 헤딩 | `^#+\s[^#:]+:\s\S` | `### 빠르게: /guardrail-config 스킬` |
| Slash 헤딩 | `^#+\s.+\s/\s.+$` | `### 시크릿 스캔 끄기 / 패턴 추가` |
| 영어 직역 단어 | `(레시피\|알아둘 것\|한 번에 보기\|~인지 여부)` | "자주 쓰는 설정 레시피" |
| "default" 한국어 본문 혼용 | `default\s(로|으로|값|ON|OFF)` | "default 로 ON 입니다" |
| 이중 피동 | `\w되어진다\b` | "판단되어진다" |
| "~에 대해/~를 통해" 남발 | `에\s대(해|하여)\b`, `를\s통(해|하여)\b` | "AI 에 대해 논의" |
| "가지고 있다" | `가지고\s있다\b` | "경쟁력을 가지고 있다" |

### 3. S2 패턴 — 밀도 검사 (3회 이상이면 패치)

문서 전체에서 카운트하고 임계값 넘으면 패치.

| 패턴 | 임계값 | 예 |
|---|---|---|
| `~구조입니다 / ~식으로 설계되어 있어요` 어미 | 한 문서 1회 이하 | "~한 줄로 적용하는 구조입니다" |
| `것이다` 종결 | 한 문서 2회 이하 | "~한 것이다" |
| 정도부사 (`매우`, `정말`, `극히`) | 한 문서 2회 이하 | "매우 중요합니다" |
| `또한`·`따라서`·`즉` 문두 접속사 | 70% 이상 삭제 | "또한 ~. 또한 ~." |
| `~적·~성·~화` 추상 접미사 | 한 문서 12회 이하 | "기술적·구조적·근본적" |
| Hype 어휘 (`혁신적`, `획기적`, `압도적`) | 0 회 | "혁신적인 솔루션" |

### 4. 변경률 가드 (과윤문 방지)

원본 대비 변경률 추정:
- **5~30%**: 정상 범위
- **30% 초과**: 과윤문 의심 — 원본 톤·의미 훼손 위험. 패치 범위 축소
- **5% 미만**: 저윤문 — S1 패턴 재확인

### 5. 판정 출력

다음 중 하나로 결론:

```
verdict: accept           # S1 0건, S2 임계값 이내
verdict: patch_needed     # S1 또는 S2 임계초과. findings 와 함께 patch 안 제시
verdict: rewrite          # S1 5+ 건 또는 다수 카테고리 동시 위반. 섹션 단위 재작성 권고
verdict: hold_and_report  # 자연어 판단이 모호한 경우 사용자에게 결정 위임
```

`findings` 형식:

```
- [S1] 카테고리: Rhetorical 헤딩
  위치: "## 차단되면 어떻게 보이나"
  대안: "## 차단 예시"

- [S2] 카테고리: 어미 어색 ('구조입니다')
  위치: "...적용하는 구조입니다."
  대안: "...적용합니다."
```

## 핵심 원칙

1. **mechanical 매칭 우선** — 주관 판단은 마지막에만. taxonomy 의 정규식 룰부터 적용.
2. **국소성** — 전체 재작성 X, S1/S2 발견 구간만 수정 안 제시.
3. **의미·톤 보존** — 변경률 30% 초과 시 의심.
4. **누적 학습** — 새로 발견한 어색 패턴은 [`references/local-additions.md`](references/local-additions.md) 에 누적.

## 출력 가이드

검수 결과는 drafter (메인 Claude) 가 바로 patch 할 수 있는 형태로:
- BAD/GOOD 페어 명시
- 위치는 헤딩 텍스트나 짧은 quote 로 식별 가능하게
- 한 번에 다 보여주고 drafter 가 일괄 패치하게 (왔다갔다 X)

## 참고

- [`references/ai-tell-taxonomy.md`](references/ai-tell-taxonomy.md) — 분류 체계 (im-not-ai 차용 + 우리 케이스 압축)
- [`references/local-additions.md`](references/local-additions.md) — 누적 사례 (사용자 지적받은 패턴들)
