# korean-labor-calc

한국 노동법·급여 계산 엔진입니다. 사이트 「일의 무늬」(https://yellowenvelope.kr/tools)의 계산 도구가 쓰는 순수 TypeScript 함수와 시험을 그대로 떼어 냈습니다.

- 외부 패키지 의존 없음(시험에만 vitest). 날짜는 `'YYYY-MM-DD'` 문자열, 금액은 정수 원 단위로 셉니다.
- 법정 숫자(최저임금·보험료율·세율·공제표)는 `src/content/rates.ts` 에 **날짜 축**으로 두고, 값마다 근거 조문·고시와 확인일을 적었습니다. 고시가 없는 해는 다른 해 값으로 대신하지 않고 계산을 멈춥니다.
- 끝수 처리(원 미만·10원 미만 버림 등)마다 조문인지, 국세청 프로그램 방식인지, 실무 관행인지 주석으로 밝혔습니다.

## 들어 있는 것

| 계산 | 파일 | 근거 |
|---|---|---|
| 퇴직금·평균임금 | `src/lib/calc/severance.ts` | 근로자퇴직급여 보장법 §8, 근로기준법 §2①6 |
| 퇴직소득세 | `src/lib/calc/retirement-tax.ts` | 소득세법 §48·§55, 시행령 §105. 근속월수는 국세청 퇴직소득 세액계산 프로그램 식 `DATEDIF(입사일, 마지막 근무일, "M") + 1` |
| 실수령액(4대보험·근로소득 간이세액) | `src/lib/calc/take-home-pay.ts` | 소득세법 시행령 [별표2] <개정 2026. 2. 27.> — 원천 CSV 와 sha256 은 `data/tax/` |
| 중도퇴사 연말정산(고정형) | `src/lib/calc/midyear-settlement.ts` | 소득세법 §47·§52⑧·§55·§59·§59의4⑨. 보험료 소득공제(A)·표준세액공제(B) 두 경로를 고르지 않고 함께 냅니다 |
| 최저임금 확인 | `src/lib/calc/min-wage.ts` | 최저임금법 §6, 시행령 §5 |
| 주휴수당 | `src/lib/calc/weekly-holiday.ts` | 근로기준법 §55①, 시행령 §30 |
| 해고예고수당 | `src/lib/calc/notice-pay.ts` | 근로기준법 §26 |
| 연차수당 | `src/lib/calc/leave-pay.ts`, `src/lib/leave/` | 근로기준법 §60·§61, 시행령 §6 |
| 통상임금 점검 | `src/lib/calc/ordinary-wage-check.ts` | 대법원 2024. 12. 19. 선고 2020다247190 전원합의체 판결 등(인용은 판결 원문 그대로) |
| 임금 지연이자 | `src/lib/calc/wage-delay-interest.ts` | 근로기준법 §37, 시행령 §17·§18 |
| 상시 근로자 수 | `src/lib/headcount.ts`, `src/lib/headcount-plan.ts` | 근로기준법 시행령 §7의2 |
| 월급 대조 | `src/lib/pay-check/` | 일할 방식별 대조(회사 기준 확인 필요) |

`src/app/**/copy.ts` 는 각 도구가 결과를 설명하는 문장(근거·그래서 생기는 일·산식)입니다. 시험이 이 문장까지 봅니다.

## 쓰는 법

```ts
import { calcRetirementTax } from './src/lib/calc/retirement-tax';

const r = calcRetirementTax({ pay: 30_000_000, hireDate: '2016-03-02', lastWorkDate: '2026-06-30' });
console.log(r.years, r.incomeTax, r.localTax);
```

```bash
npm install
npm test          # vitest
npm run typecheck # tsc --noEmit
```

## 주의

- 계산 결과는 **예상값**입니다. 회사의 취업규칙·근로계약·실제 공제 내역에 따라 달라질 수 있고, 법률 판단이나 세무 신고를 대신하지 않습니다.
- 법령은 바뀝니다. `rates.ts` 의 값과 확인일을 보고, 개정되면 원문으로 다시 확인해 날짜 축에 줄을 더하세요. 시험(`tests/unit/rates.test.ts`)은 적용 범위가 곧 끝나는 값을 경고합니다.
- 시험 사례는 모두 합성 입력입니다.

## 라이선스

MIT
