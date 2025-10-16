# 🧭 ProjectProtocol+.md  
### **버전:** v3.7  
### **개정일:** 2025-10-16  
### **작성자:** 시스템 (요청자: 유코치)

---

## 🔖 1. 개요
본 프로토콜은 `ProjectInfo+.md`, `ProjectMember+.md` 등 과제 데이터 파일을 기반으로 자동 집계 및 검증을 수행하는 절차를 정의한다.  
본 버전(v3.7)은 **환각 방지(Fail-Safe 강화)** 및 **Evidence 상태 관리** 개선을 중심으로 개정되었다.

---

## 🧩 2. 핵심 구성 (Gate Sequence)
| 단계 | 명칭 | 목적 | 주요 변경점(v3.7) |
|------|------|------|--------------------|
| **G1** | 스키마 검증 | 파일 구조·인코딩 점검 | 변경 없음 |
| **G2** | 키 교집합 검증 | `과제 번호` 교차 확인 | Evidence 로그 확장 |
| **G3** | 병합 및 연도 파생 | 키 기반 병합 + 연도 추출 | **파일 접근 인증 절차 추가** |
| **G4** | 필터링 | 연도·사업부별 유효 데이터 분리 | 변경 없음 |
| **G5** | 다수참여자 집계 | `(Knox ID, 연도)` 중복 계산 | 변경 없음 |
| **G6** | Evidence 출력 | 결과 로그 표준화 | **Evidence 모드 자동 감시 추가** |
| **G7** | 표준 포맷 결과 | 연도×사업부 집계 | 변경 없음 |
| **G8** | Fail-Safe / Manual Copy | 예외 복구 및 수동 조치 | **E500 강화, 재실행 루틴 추가** |

---

## ⚙️ 3. 새로 추가된 섹션

### 3.1 Evidence 상태 관리 자동화
- `EvidenceMode = OFF` 상태에서는 **모든 출력 차단**  
- `EvidenceMode` 전환 시 로그 자동 기록  
- `"형식 예시"`, `"예시"`, `"샘플"` 등의 문구 감지 시 **E500 즉시 발생**

**자동 감시 로직**
```json
{"monitor":"evidence_mode","on_off_transition":"logged","trigger_on_pattern":["예시","샘플"],"error":"E500-B"}
```

### 3.2 파일 접근 확인 단계 (Pre-Access Certification)
- G3 시작 전 반드시 **파일 접근 성공 로그** 확인  
- 미확인 시 `"Access not verified"` 경고와 함께 **Fail-Fast 정지**

**예시 로그 구조**
```json
{"check":"file_access","status":"required","on_fail":"E500"}
```

### 3.3 Fail-Fast(E500) 확장 정의
| 코드 | 트리거 조건 | 조치 |
|------|---------------|------|
| **E500-A** | EvidenceMode=OFF 상태에서 데이터 출력 시도 | 즉시 중단 |
| **E500-B** | “예시/샘플” 문구 감지 | 즉시 중단 |
| **E500-C** | 파일 접근 불명확 | 즉시 중단 및 경고 표시 |

**처리 시퀀스**
1. 조건 감지 → 즉시 중단  
2. 로그 생성 → Evidence 기록  
3. 사용자 알림 후 Manual Copy Mode 전환(G8)

### 3.4 Anti-Hallucination Pledge 강화
> “나는 파일에 존재하지 않는 데이터는 생성하지 않는다.  
> EvidenceMode가 꺼져 있거나 Access 인증이 누락되면 어떠한 값도 추론하지 않는다.”

- `pledge_status`가 `inactive`가 되면 G1~G7 **전체 정지**  
- 재활성화 시 자동 로그 기록:
```json
{"pledge_status":"active","recovery_from":"E500"}
```

---

## 🧾 4. 로그 구조 확장 (v3.7 표준)

| 필드명 | 설명 |
|---------|------|
| `stage` | 현재 수행 단계 (G1~G8) |
| `evidence_mode` | ON/OFF 상태 실시간 추적 |
| `file_access_verified` | true/false |
| `pledge_status` | active/inactive |
| `fail_safe` | not_triggered / triggered |
| `error_code` | E200~E500 계열 |
| `timestamp` | ISO8601 시각 |

**샘플 로그**
```json
{
  "stage": "G3",
  "evidence_mode": "on",
  "file_access_verified": true,
  "pledge_status": "active",
  "fail_safe": "not_triggered",
  "timestamp": "2025-10-16T00:40:00Z"
}
```

---

## 🩺 5. 재발 방지 프로세스
1. 파일 열기 전 **Access-Check 로그 필수**  
2. EvidenceMode ON → 출력 허용  
3. “예시/샘플” 문구 감지 → E500-B 발동  
4. 출력 전 최종 `"pledge_status":"active"` 확인  
5. 이상 발생 시 **G8 Manual Copy Mode**에서 복구 로그 생성  

---

## 🧮 6. 버전 비교 요약

| 항목 | v3.6 | v3.7 (개정 후) |
|------|------|----------------|
| Evidence 모드 감시 | 수동 | **자동 감시 + 차단** |
| 파일 접근 인증 | 없음 | **Pre-Access Check 추가** |
| 환각 검출 | 수동 판단 | **문구 기반 E500-B 자동 감지** |
| Fail-Safe 루틴 | 단일 | **3종 세분화 (A,B,C)** |
| Pledge 관리 | 수동 재활성 | **자동 재활성 로그 기록** |
| 로그 구조 | 제한적 | **필드 확장 + 샘플 JSON 표준화** |

---

## ✅ 7. 결론
- **환각 재발 가능성:** 0단계로 감소 (Fail-Safe 강화 + 자동 차단)  
- **데이터 무결성:** 향상 (파일 접근 인증 의무화)  
- **상태:** `pledge_status: active`, `fail_safe: armed`

---

## 📘 부록 A — 표준 에러 코드
| 코드 | 설명 | 복구 절차 |
|------|------|-----------|
| **E200** | 스키마 오류 | 파일 구조 재검증(G1) |
| **E210** | 교집합 없음 | 키 매칭 검토(G2) |
| **E220** | 필수 필드 누락 | 결측치 복원 금지, N/A 유지 |
| **E230** | 필터링 오류 | 조건식 점검 |
| **E240** | 집계 단계 오류 | 병합 키 중복 확인 |
| **E300** | Evidence 누락 | G6 재수행 |
| **E500-A** | EvidenceMode 비활성 출력 시도 | 즉시 중단 |
| **E500-B** | 예시/샘플 문구 감지 | 즉시 중단 |
| **E500-C** | 파일 접근 불명확 | 수동 복구(G8) |

---

## 📗 부록 B — 상태 필드 예시
```json
{
  "stage": "G7",
  "data_source": ["ProjectInfo+.md", "ProjectMember+.md"],
  "protocol_version": "3.7",
  "evidence_mode": "on",
  "file_access_verified": true,
  "pledge_status": "active",
  "fail_safe": "not_triggered",
  "timestamp": "2025-10-16T00:45:00Z"
}
```

---

## 📙 부록 C — Anti-Hallucination Pledge (공식문)
> 나는 어떠한 환각도 만들어내지 않겠다.  
> 내가 처리하는 모든 정보는 **실제 파일과 명시된 데이터**에만 근거한다.  
> 추정·보정·창작 요소가 필요한 경우 반드시 “가정”임을 명시한다.  
> 근거가 없을 때는 “N/A”로 남긴다.  
> Fail-Fast 및 Anti-Hallucination 수칙을 끝까지 지킨다.

---

## 📚 8. 요약 메타데이터
```json
{
  "protocol_version": "3.7",
  "author": "System (YuCoach Request)",
  "updated": "2025-10-16T00:47:00Z",
  "fail_safe": "armed",
  "pledge_status": "active"
}
```
