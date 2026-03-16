# Stack Skills

<p align="right">
  <a href="README.md">English</a> · <a href="README.ko.md">한국어 (Korean)</a>
</p>

**안일한 결정이 나가기 전에 잡아내세요.**

AI는 안전한 기본값을 고르고, 반박 대신 동의하고, 출처를 밝히지 않고, 추론을 숨기고, 모든 게 잘 될 거라 가정합니다. 이것은 여섯 가지 인지 실패입니다. Stack Skills는 각각에 방화벽을 설치합니다.

```
npx skills add whynowlab/stack-skills --all
```

> [Claude Code](https://claude.com/claude-code) 전용 | [Agent Skills](https://agentskills.io) 오픈 표준으로 다른 AI 에이전트 호환
> Created by [@thestack_ai](https://github.com/whynowlab)

---

## 차이를 확인하세요

### Before: 기본 AI 리서치

> **당신:** SQLite가 동시 유저 1000명인 앱에 적합한가요?
>
> **AI:** 아뇨, SQLite는 고동시성 애플리케이션에 적합하지 않습니다. 프로덕션 워크로드에는 PostgreSQL이나 MySQL을 사용하세요.

### After: `cross-verified-research` 적용

> **당신:** /cross-verified-research SQLite가 동시 유저 1000명인 앱에 적합한가요?
>
> **AI:** **가능합니다. 단, 조건이 있습니다.** 동시 유저 1000명 ≠ 동시 쓰기 1000건. 일반적인 웹 앱에서 이 규모면 동시 쓰기 트랜잭션은 ~30건입니다. SQLite는 WAL 모드에서 ~120 쓰기 트랜잭션/초를 처리합니다 (검증: sqlite.org 벤치마크 + Expensify 프로덕션 사례 — SQLite로 1000만+ 유저 서빙). 진짜 제약은 유저 수가 아니라 ~100 동시 writer 이상에서의 쓰기 경합입니다.
>
> 출처: 🏛️ sqlite.org WAL 문서, 🛡️ Expensify 엔지니어링 블로그 (2024), ⚠️ HN 토론 (교차 확인됨)

**기본 응답은 잘못된 답을 자신 있게 내놨습니다. 스킬은 출처 검증을 강제하여 반대 — 그리고 정확한 — 결론에 도달했습니다.**

### Before: 기본 AI 코드 리뷰

> **당신:** 이 JWT 인증 모듈을 리뷰해줘
>
> **AI:** 코드 구조가 잘 되어 있습니다. 주석을 좀 더 추가하고 TypeScript를 쓰면 타입 안전성이 올라갈 것 같습니다. 일부 변수명 개선 여지가 있습니다.

### After: `adversarial-review` 적용

> **당신:** /adversarial-review 이 JWT 인증 모듈을 리뷰해줘
>
> **AI:**
> **Steel-man:** JWT 기반 무상태 인증은 수평 확장을 위해 선택됨. 15분 액세스 + 7일 리프레시 분리는 보안과 UX의 균형.
>
> 🔴 **Critical: 리프레시 토큰 로테이션 없음.** 탈취된 토큰으로 7일간 탐지 불가 상태로 접근 가능. 수정: 토큰 패밀리 체인 로테이션 구현.
>
> 🟠 **Major: Rate limiting이 인메모리 Map() 사용.** 재시작 시 초기화, 멀티 인스턴스 배포에서 우회됨. 수정: Redis + TTL로 이전.
>
> **판정: 조건부 통과** — 토큰 로테이션 수정 후 배포 필수.

**기본 응답은 아무것도 못 찾았습니다. 스킬은 Critical 보안 취약점을 발견했습니다.**

---

## 6가지 실패, 6개 방화벽

| 인지 실패 | 증상 | 방화벽 | 강제하는 것 |
|:---|:---|:---|:---|
| **조기 결론** | AI가 모호한 요청을 즉시 실행 | `scope-clarifier` | 실행 전 구조화된 명확화 |
| **환각** | 검증 없이 주장 | `cross-verified-research` | 출처 추적, 교차 검증, S/A/B/C 소스 등급제 |
| **고착 편향** | 첫 번째 "뻔한" 답에 고정 | `creativity-sampler` | 확률 가중치 5개 옵션 + 비관습적 대안 강제 |
| **확증 편향** | 반론 대신 동의 | `adversarial-review` | Steel-man 후 3벡터 공격. "괜찮다"는 구조적으로 불가 |
| **블랙박스 사고** | 근거 없이 결론만 제시 | `reasoning-tracer` | 가정 목록, 신뢰도 분해, 최약점 분석 |
| **낙관 편향** | 계획이 성공할 거라 가정 | `pre-mortem` | 실패 가정 후 역추적, 5개 시나리오 + 회로 차단기 |

---

## 스킬

### scope-clarifier

모호한 요청에 대한 조기 실행을 방지합니다.

```
요청 → 5W1H 분해 → 모호성 점수 → 명확화 또는 실행
```

- 모든 요청을 6개 차원으로 분해: What, Who, Where, When, Why, How
- 모호성 점수(0-6) — 명확한 요청은 즉시 진행, 모호한 요청은 최대 3개 질문
- 모든 질문에 객관식 옵션과 기본 가정을 명시
- 체인의 **맨 처음**에 실행하도록 설계

```
사용: /scope-clarifier 인증 시스템 만들어줘
```

### cross-verified-research

4단계 검증 리서치 파이프라인. 반환각 방어.

```
분해 → 검색 & 수집 → 교차 검증 → 합성
```

- 모든 주장은 인용 가능한 출처에 추적 가능해야 함 — 아니면 `Unverified` 표시
- 소스 등급: S(학술/스펙), A(공식 문서), B(커뮤니티 — 경고 표시), C(일반)
- 교차 검증: 핵심 주장은 2개 이상 *독립* 출처 필요 (같은 원본 재작성은 불인정)
- 적응적 깊이: 좁은 질문 2-3쿼리, 넓은 조사 8+쿼리

```
사용: /cross-verified-research 모바일 백엔드에 gRPC가 REST보다 나은가?
```

### creativity-sampler

고착 편향 방어. 확률 가중치 옵션 생성기.

- 기본 5개 옵션, 각각 전형성 존 배정 (Conventional → Wild card)
- 최소 1개는 비관습적/와일드카드 존에서 — AI가 평소 억제하는 아이디어
- **Hidden Assumptions** 섹션: "뻔한" 답이 뻔해 보이는 이유를 명시적으로 해부
- 제약 조건 기반 의사결정 매트릭스 + 사용자가 놓친 1개 숨겨진 기준

```
사용: /creativity-sampler 실시간 리더보드에 어떤 데이터베이스?
```

### adversarial-review

확증 편향 방어. 구조화된 악마의 변호인.

- **Steel-man 우선**: 공격 전에 현재 접근법의 최강 정당화를 먼저 명시
- **3개 독립 벡터**: 논리적 건전성 / 엣지 케이스 공격 / 구조적 무결성
- 심각도 분류: Critical(반드시 수정) / Major(수정 권장) / Minor / Note
- Critical/Major는 반드시 트레이드오프 분석 포함 대안 제시
- 명시적 판정 기준: 완화 불가 Critical = FAIL

```
사용: /adversarial-review 3인 스타트업에서 Kubernetes를 선택했다
```

### reasoning-tracer

블랙박스 방어. AI 추론을 투명하게.

- **가정 목록**: 모든 가정 번호 매기기, 중요도/검증가능성 평가
- **결정 트리**: 각 분기에서 어떤 대안을 고려했고 왜 기각했는지
- **신뢰도 분해**: 전체 신뢰도를 하위 구성요소로 분해 + 근거
- **최약점**: 하나의 가정이 틀리면 결론이 가장 크게 바뀌는 지점
- **대안 결론**: "만약 [최약 가정]이 틀리면, 결론은 [X]로 바뀐다"

```
사용: /reasoning-tracer 이 프로젝트에 마이크로서비스를 추천하는 이유는?
```

### pre-mortem

낙관 편향 방어. 전향적 실패 분석.

- "6개월 후, 이 계획은 완전히 실패했다. 무엇이 잘못됐는가?"
- 5개 카테고리별 실패 시나리오: 기술/조직/외부/시간/가정
- 가능성 x 영향도 매트릭스 → 상위 3개 집중 분석
- **선행 지표**: 각 상위 리스크의 측정 가능한 조기 경보 신호
- **회로 차단기**: 멈추고 방향을 바꿔야 할 구체적 트리거 조건

```
사용: /pre-mortem Q3에 모놀리스를 마이크로서비스로 마이그레이션 예정
```

---

## 어떤 스킬을 써야 할까?

### 일상 사용 (주력)

| 상황 | 스킬 | 예시 |
|:---|:---|:---|
| 요청이 모호하거나 암묵적 가정이 있을 때 | `scope-clarifier` | "인증 시스템 만들어줘" → SSO? RBAC? OAuth? 명확화 |
| 기술 조사 또는 사실 검증 | `cross-verified-research` | "모바일에 gRPC가 REST보다 나은가?" |
| 코드, 아키텍처, 결정 리뷰 | `adversarial-review` | "3인 스타트업에서 Kubernetes를 선택했다" |

### 중요한 순간

| 상황 | 스킬 | 예시 |
|:---|:---|:---|
| 옵션 사이에서 선택할 때 | `creativity-sampler` | "실시간 리더보드에 어떤 데이터베이스?" |
| 프로젝트 착수 전 리스크 제거 | `pre-mortem` | "Q3에 마이크로서비스 마이그레이션 예정" |

### 심층 분석

| 상황 | 스킬 | 예시 |
|:---|:---|:---|
| AI가 왜 그렇게 추천하는지 알고 싶을 때 | `reasoning-tracer` | "이 프로젝트에 왜 마이크로서비스?" |
| 전체 감사 추적이 필요할 때 | `reasoning-tracer --full` | 아키텍처 결정 기록 |

### 추천 체인

- **모든 작업**: `scope-clarifier` → [다른 스킬] (항상 명확화 먼저)
- **기술 의사결정**: `scope-clarifier` → `creativity-sampler` → `cross-verified-research` → `adversarial-review`
- **아키텍처 리뷰**: `reasoning-tracer` → `adversarial-review`
- **프로젝트 킥오프**: `pre-mortem` → `creativity-sampler` (완화책용)

---

## 작동 구조

```
AI의 기본 추론:

  [질문] ──→ [첫 번째 그럴듯한 답] ──→ [그대로 사용]


Stack Skills 적용 시:

  [요청]
       │
       ├──→ scope-clarifier ──→ "정확히 뭘 원하시나요?"  (조기 결론 방지)
       │
       ├──→ creativity-sampler ──→ "선택지가 전부 뭐야?"  (고착 방어)
       │
       ├──→ cross-verified-research ──→ "이거 진짜야?"  (환각 차단)
       │
       ├──→ reasoning-tracer ──→ "왜 이걸 믿어?"  (가정 노출)
       │
       ├──→ adversarial-review ──→ "뭐가 잘못됐어?"  (확증 편향 제거)
       │
       └──→ pre-mortem ──→ "어떻게 실패해?"  (낙관 편향 방어)
```

각 방화벽은 독립적 — 하나만 쓰거나, 체이닝 가능.

---

## 왜 스킬인가?

이 지시를 CLAUDE.md나 시스템 프롬프트에 붙여넣을 수도 있습니다. 하지만:

1. **컨텍스트 효율성** — 스킬은 트리거될 때만 로드됩니다 (~100 토큰 스캔, 호출 시에만 전체 지시). 시스템 프롬프트는 매 메시지마다 토큰 비용을 지불합니다.
2. **조합 가능성** — 스킬은 체이닝됩니다: `scope-clarifier` → `cross-verified-research` → `adversarial-review`. 단일 프롬프트는 선택적 호출이 불가능합니다.
3. **이식성** — `npx skills add`는 Claude Code, Cursor, Copilot에서 동작합니다. 시스템 프롬프트는 하나의 도구에 종속됩니다.
4. **커뮤니티** — 스킬은 공유, 포크, 개선할 수 있습니다. dotfiles의 프롬프트는 본인만 도움됩니다.

---

## 설치

### 빠른 설치 (npx)

```bash
npx skills add whynowlab/stack-skills --all
```

### 개별 설치

```bash
npx skills add whynowlab/stack-skills/scope-clarifier
npx skills add whynowlab/stack-skills/cross-verified-research
npx skills add whynowlab/stack-skills/adversarial-review
npx skills add whynowlab/stack-skills/creativity-sampler
npx skills add whynowlab/stack-skills/reasoning-tracer
npx skills add whynowlab/stack-skills/pre-mortem
```

### 수동 설치

```bash
git clone https://github.com/whynowlab/stack-skills.git
cp -r stack-skills/skills/* ~/.claude/skills/
```

### 프로젝트별 설치

```bash
cp -r stack-skills/skills/adversarial-review .claude/skills/
```

---

## 호환성

| 플랫폼 | 상태 |
|:--------|:-----|
| Claude Code | 완전 지원 |
| Cursor | 호환 (Agent Skills 표준) |
| GitHub Copilot | 호환 (Agent Skills 표준) |
| Codex CLI | 호환 (Agent Skills 표준) |

---

## 라이선스

MIT License. [LICENSE](LICENSE) 참조.

---

**Stack Skills** by [@thestack_ai](https://github.com/whynowlab) — AI를 위한 6개의 인지 방화벽.
