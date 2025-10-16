# ProjectProtocol+.md (v3.6)
> Revision: 2025-10-16  
> 목적: `ProjectInfo+.md`, `ProjectMember+.md` 기반의 참가자 및 성과 자동 집계 시  
> **환각(Hallucination)**, **자동보정(Auto-Imputation)**, **파싱 불일치**를 방지하고  
> 데이터 신뢰성과 근거(Evidence) 기반 출력을 강화한다.

## ⚙️ System Mode
| 항목 | 설정값 | 설명 |
|------|---------|------|
| Temperature | **0.1 (고정)** | 언어적 표현만 제어, 환각 방지와는 무관 |
| Data Source | ProjectInfo+.md / ProjectMember+.md | 두 파일만 유일한 근거로 사용 |
| Evidence Mode | 선택적 (G1~G2 완료 후 실행) | 구조 및 교집합 검증만 수행, 수치 계산 보류 |
| Fail-Fast | 활성화 | 오류 감지 시 즉시 중단 (`E200`, `E210`, `E220`) |
| Auto-Imputation | 차단 | 비어있는 셀을 추론값으로 채우는 행위 금지 |
| Missing Cells | `block` | 누락된 셀은 Null이 아닌 에러로 처리 |
| Random Fill | `disabled` | 형식 보정을 위한 자동 수 생성 금지 |

## 🧩 Gate Sequence (실행 단계)

### G1. 파일 로드 및 스키마 검증
- 입력 파일은 반드시 **Markdown 표 형식(`|` 구분자, 헤더 포함)**이어야 한다.  
- 첫 행에는 반드시 `과제 번호`가 존재해야 하며, 누락 시 `E200:HEADER_MISSING` 발생.  
- 헤더/셀 수 불일치 시 3회 자동 재시도 후 중단.  
- 인코딩(EUC-KR, UTF-8) 불일치 시 자동 정규화 수행.

### G2. 키 교집합 검증 (`과제 번호`)
- 두 파일의 `과제 번호` 교집합 및 여집합을 비교.  
- 여집합 존재 시 해당 항목은 제외하고 로그에 기록.  
- 교집합이 0이면 `E210:NO_KEY_MATCH`로 중단.  
- 교집합 통계(`total`, `intersection`, `diff`)를 Evidence로 남김.

### G3. 병합 및 연도 파생
- 병합 키: `과제 번호`  
- 연도 기준: `과제 시작일`의 연도(YYYY)  
- 필수 필드 누락 시 `E220:NULL_FIELD_BLOCK`  
- 병합 결과는 Evidence Mode에서도 확인 가능.

### G4. 사업부 기준 필터 및 집계
- 사업부 구분: `ProjectInfo+.md`의 `과제 사업부`  
- 필터링 조건:
  - `과제 상태`가 `진행` 또는 `완료`
  - 연도별 필터 가능 (`연도=2021` 등)
- 집계 규칙:
  - 과제별 참여자 합산
  - 동일 Knox ID 중복 시 `다수참여`로 별도 표시
  - 표기 형식: `총원(다수참여)`

### G5. 다수참여자 계산
- 동일 Knox ID가 같은 연도에 2회 이상 참여하면  
  - 첫 번째 등장: 기본 카운트  
  - 두 번째 이후: 괄호 내 숫자 증가  
- 예시: `81(5)` → 총 81명 중 5명은 중복참여자  
- 중복 판정 로직: `(Knox ID, 연도)` 조합 기준

### G6. Evidence 출력
- 모든 결과에는 **근거 로그(Evidence)** 를 포함해야 함.  
- Evidence 누락 시 `E300:EVIDENCE_MISSING` 발생.

### G7. 최종 출력 및 표준 포맷
- 연도별·사업부별 집계 표 형식 유지.  
- 하단 표기 필수:  
  > Data Source: ProjectInfo+.md + ProjectMember+.md (v3.6)  
  > Evidence Mode: ON/OFF  
  > Fail-Safe: Triggered / Not Triggered

### G8. Fail-Safe 및 수동 복구
- 다음 상황에서는 자동 중단 후 **Manual Copy Mode**로 전환:
  - 헤더 누락 (`E200`)
  - 키 불일치 (`E210`)
  - Null 필드 (`E220`)
  - 환각 감지 (`E500:HALLUCINATION_DETECTED`)
- Manual Copy Mode에서는 수치 계산 불가, 구조만 표시.

## 🧠 Anti-Hallucination Logic (신규 추가)
| 항목 | 동작 |
|------|------|
| Empty Cell Handling | “N/A”로 표시, 값 추론 금지 |
| Unverified Numbers | 출력 차단 (`E500`) |
| Fabricated Knox ID | 생성 차단 (`E510`) |
| Total Count Overflow | 총합이 `Member.count()` 초과 시 중단 |
| Evidence Requirement | 최소 1개 이상 실제 행 병렬 출력 |

## 🧾 실행 로그 예시
```json
{
  "stage": "G5",
  "rows_joined": 879,
  "duplicates_found": 27,
  "fail_safe": "not_triggered",
  "timestamp": "2025-10-16T12:00Z"
}
```

## 📘 Lessons Learned (교훈 반영)
1. **낮은 Temperature는 환각 방지책이 아니다.**  
   → 데이터 기반 근거 검증(Evidence Gate)이 필수적이다.  
2. **파일 파싱 구조가 불안정하면 근거 없는 자동보정이 발생한다.**  
   → Markdown 파서 검증(G1)을 강화한다.  
3. **Fail-Safe는 경고가 아닌 중단 조건이다.**  
   → 모든 Gate는 `raise` 모드로 동작시킨다.  
4. **Evidence Mode를 먼저 실행해 신뢰성을 확보한 후 집계를 수행한다.**  
   → 단계적 실행 절차를 기본 워크플로로 설정한다.  

## 🪪 버전 이력
| 버전 | 날짜 | 주요 변경 내용 |
|------|------|----------------|
| v3.5 | 2025-10-15 | 기본 Gate Sequence 및 집계 규칙 정의 |
| **v3.6** | **2025-10-16** | 환각 차단, Evidence Mode 추가, Fail-Fast 강화, Auto-Imputation 차단, Lessons Learned 반영 |
