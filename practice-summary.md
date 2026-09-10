# 온톨로지 매니저 실습 정리

> 공개 공유본: 배치 ID·측정값·일부 수량·경로는 설명용 예시로 바꿨습니다. 예시 값은 원본 행 또는 실제 운영 결과의 증거가 아닙니다. 원본 CSV·책 본문·계정 정보는 포함하지 않습니다.

> 처음부터 다시 실습할 때: [순서대로 따라하기](practice-guide.md) — 설정값·저장·완료 확인을 단계별로 정리.

기준: 2026-09-08 대화에서 직접 확인한 결과. 실습 브랜치: `review-practice`.

## 목적

배치의 품질검사를 확인하고, 사람이 출하 검토 결과를 기록하는 흐름으로 Object·Property·Link·Action을 익힌다.

## 1. Object Type — 관리할 대상

| 타입 | 데이터 |
|---|---|
| 배치출하검토 | 생산 배치 30건 |
| 품질검사 | 검사 기록 60건 |

Object Type은 대상의 종류이고, `DEMO-01-02` 같은 배치 하나가 Object다.

## 2. Property — 정보를 담는 칸

배치출하검토에 8개 속성을 구성했다.

- 기본 정보: Batch Id, Manufacture Date, Product Code, Recipe Id
- 검토 정보: 검토 상태, 보류 사유, 판정자, 판정 시각

## 3. Link — 배치와 검사 연결

배치 하나에서 연결된 품질검사 여러 건을 조회한다. `DEMO-01-02`에서 다음 두 건을 직접 확인했다.

| 검사 | 결과 | 측정값 |
|---|---|---:|
| 내용량 | 불합격 | 98.20 |
| 점도 | 합격 | 5100 |

## 4. Action — 출하 검토 기록

처음 생성된 이름 `Modify 배치출하검토`를 `출하 검토 기록`으로 변경했다.

| 수정할 속성 | 기록하는 값 |
|---|---|
| 검토 상태 | 사람이 선택한 검토중·보류·승인 |
| 보류 사유 | 사람이 입력한 이유 |
| 판정자 | Current User: 실행한 사용자 |
| 판정 시각 | Current Time: 실행 시각 |

Action을 실행해서 위 속성에 값을 기록한다. 기록 후 별도의 Action을 실행하는 구조가 아니다.

## 5. 보류 사유의 조건부 필수 입력

1. Parameters → 보류 사유 → General로 이동한다.
2. 기본 Required는 끈다.
3. Required 아래 Add override를 누른다.
4. IF 영역의 `condition`을 누른다.
5. Based on → Parameter → 검토 상태를 선택한다.
6. 비교 조건은 Is equal to를 선택한다.
7. Compare against → Static value → String 입력칸에 `보류`를 입력한다.
8. THEN의 Is Required를 켠다.
9. Done → Save to branch → 변경사항 확인 화면의 Save to branch로 저장한다.

의미: 검토 상태가 보류일 때만 보류 사유가 필수다. Required 아래 `1 override`가 표시된다. 검토중·승인에서는 필수 표시가 없고 보류에서는 나타나는 것을 확인했다.

## 6. Test run과 실제 실행

Test run은 변경될 값을 보여주는 모의 실행이며 실제 객체에 저장하지 않는다. 결과 화면의 `Unknown Resource`는 Object Type 정보를 불러오지 못한 표시였다. 상세 원인은 미확정이며 모의 실행 자체는 성공했다.

실제 기록은 다음 경로로 확인했다.

1. Ontology Manager → 배치출하검토 → Overview
2. Open in → Insight
3. 브랜치가 `review-practice`인지 확인
4. 표의 Title 열에서 `DEMO-01-02` 더블클릭
5. 객체 상세의 Linked objects에서 품질검사 2건 확인
6. 객체 상세 위쪽 Actions → 출하 검토 기록
7. 자동 지정된 배치를 확인하고 검토 상태와 사유 입력
8. Submit으로 실제 기록
9. Properties에서 결과 확인

| 속성 | 확인된 실제 기록 |
|---|---|
| 검토 상태 | 보류 |
| 보류 사유 | [실습] 내용량 검사 불합격으로 재검사 필요 |
| 판정자 | 실행한 사용자 계정 ID |
| 판정 시각 | 실행 당시 시각 화면 표시 |

`Edits successfully applied.` 메시지와 객체 속성 반영을 확인했다. 같은 객체에 다시 실행하면 검토 값과 판정 시각이 갱신된다.

## 완료 범위와 남은 작업

완료: Object 생성 → Property 구성 → Link 연결 → Action 설정 → 조건부 입력 검증 → 실제 기록 확인.

- Main 반영은 하지 않았다.
- 실제 물류 시스템의 출하를 차단하는 연동은 만들지 않았다.
- 별도의 출하 승인 Action은 제안만 했다.
- 다음 실습은 실행 조건(Submission Criteria): 현재 상태와 새 상태가 모두 보류일 때 중복 보류 실행을 막는 규칙이다. 이 문서의 기존 실습 완료 범위에는 포함하지 않는다.

핵심: **배치(Object)를 열고 → 검사(Link)를 확인하고 → 검토 기록(Action)을 실행해 → 속성(Property)에 판단을 남긴다.**

## 설정값 찾아보기

기존 실습 기록을 바탕으로 정리했다. 표시 이름은 화면용, API 이름은 코드에서 식별하는 용도다. 새 브랜치에서는 선택한 데이터셋과 저장된 정의를 확인한다.


### Object Type 설정값

| 설정 항목 | 쉬운 뜻 | 이번 실습에서 선택·확인할 값 |
| --- | --- | --- |
| Display name · 표시 이름 | 사람이 화면에서 읽는 이름 | 배치출하검토 / 품질검사 |
| API name | 코드에서 타입을 구분하는 이름 | 기존 이름과 충돌하지 않게 지정. 표시 이름과 역할이 다름 |
| Datasource · 데이터 소스 | 객체 정보를 가져올 데이터셋 | 배치: batch_master / 검사: quality_inspection_for_ontology |
| Primary key · 기본키 | 객체 한 건을 구분하는 고유한 값 | 배치: batch_id / 검사: inspection_id |
| Title · 대표 표시 속성 | 목록·상세에서 객체 이름처럼 보일 값 | 배치: batch_id / 검사: inspection_id |
| Allow edits · 편집 허용 | 액션으로 객체를 수정할 수 있도록 준비 | 검토 결과를 저장할 배치출하검토는 Edits Enabled 확인 |

기본키는 비어 있거나 중복되면 안 된다. Title은 화면 표시용이고, 기본키는 객체 식별용이다.

성공 확인: 저장 후 배치 30건·검사 60건이 실제 조회되는지 확인한다. 편집 허용과 사용자 실행 권한은 별도 설정이다.


### Property 설정값

| 설정 항목 | 쉬운 뜻 | 이번 실습에서 선택·확인할 값 |
| --- | --- | --- |
| 배치 기본 정보 4개 | 원천 컬럼과 속성을 연결 | Batch Id, Manufacture Date, Product Code, Recipe Id → 각각 원천 컬럼 매핑 |
| 검사 정보 5개 | 검사 데이터의 컬럼을 연결 | inspection_id, batch_id, test_item, result, measured_value |
| String · 문자열 | 이름·상태·이유 같은 글자 값 | 검토 상태, 보류 사유, 판정자 |
| Timestamp · 날짜와 시각 | 언제 실행했는지 저장할 값 | 판정 시각 → Timestamp |
| Double · 소수 포함 숫자 | 검사 측정값을 숫자로 저장 | measured_value → Double |
| Edit-only · 편집 전용 속성 | 원천 컬럼 대신 액션으로 값을 기록하는 칸 | 검토 상태·보류 사유·판정자·판정 시각 |

판정 시각을 String으로 만들면 시간값 연결에서 막힐 수 있다. 이번 실습은 Timestamp에 Current Time을 연결한다.

Edit-only 설정에서 권한 기준 데이터셋 선택을 요구하면 해당 객체의 권한 기준을 확인한다. 기존 원천 컬럼을 임의로 검토 결과 저장용으로 바꾸지 않는다.


### Link Type 설정값

| 설정 항목 | 쉬운 뜻 | 이번 실습에서 선택·확인할 값 |
| --- | --- | --- |
| 연결할 두 Object Type | 어떤 대상끼리 연결할지 선택 | 배치출하검토 ↔ 품질검사 |
| Object type foreign keys | 한쪽 속성값으로 상대 객체를 찾는 방식 | 검사에 있는 batch_id로 배치를 찾는다 |
| Many 쪽 객체 | 여러 건이 속하는 쪽 | 품질검사 · 한 배치에 여러 검사 |
| Foreign key · 외래키 속성 | 상대 객체를 찾기 위해 가진 값 | 품질검사의 batch_id |
| One 쪽 객체와 기본키 | 연결되는 하나의 대상 | 배치출하검토의 batch_id |
| 양방향 표시 이름 | 각 객체에서 연결 목록을 부를 이름 | 배치에서 보는 목록: 품질검사 / 검사에서 보는 대상: 배치출하검토 |

검사.batch_id = 배치.batch_id → 해당 배치의 검사로 연결된다.

저장 후 DEMO-01-02의 Linked objects에서 내용량·점도 두 검사가 보이는지 확인한다. inspection_id는 검사 자체를 구분하는 키이고, 배치를 찾는 키는 batch_id다.


### Action Type 설정값

| 설정 항목 | 쉬운 뜻 | 이번 실습에서 선택·확인할 값 |
| --- | --- | --- |
| 표시 이름 | 실행할 때 선택하는 액션 이름 | 출하 검토 기록 |
| Modify object 규칙 | 이미 있는 객체의 정보를 수정 | 배치출하검토 객체 수정 |
| 대상 객체 Parameter | 어느 배치에 기록할지 전달받는 값 | 배치출하검토 객체 1개 · 객체 상세에서 실행 시 대상 확인 |
| 검토 상태 Parameter | 사람이 선택하는 값 | String · 검토중 / 보류 / 승인 · Required 켜기 |
| 보류 사유 Parameter | 사람이 입력하는 이유 | String · 기본 Required 끄기 · 보류일 때만 override로 필수 |
| Rules · 속성에 값 연결 | 입력값과 자동 값을 저장할 칸 지정 | 검토 상태 ← 입력 / 보류 사유 ← 입력 / 판정자 ← Current User / 판정 시각 ← Current Time |

입력 파라미터는 액션이 받는 값, 속성은 배치에 저장할 칸이다. 두 항목을 규칙에서 연결한다.

Required는 입력이 필요한지, Submission Criteria는 액션을 실행해도 되는지 정한다. 실행 권한은 누가 실행할 수 있는지 정하며 기존 환경의 설정을 확인한다. 조건부 Required 상세 순서는 부록 3, 실제 실행은 부록 4를 참고한다.

## 이름·Description 작성과 속성별 입력 예시

아래 설명 문구는 새로 작성한 제안이며 기존에 저장된 문구의 인용이 아니다. Description은 사람이 읽는 안내문으로, 실행 조건을 자동 설정하지 않는다.

| 표시 이름 | 데이터 타입 | 원천 컬럼 / 방식 | Description 예시 |
| --- | --- | --- | --- |
| Batch Id | String | batch_id | 생산 배치 한 건을 구분하는 고유 식별자. 검사 데이터와 연결하는 기준이다. |
| Manufacture Date | String · 기존 실습 기준 | manufacture_date | 배치의 제조일. 원천 데이터의 날짜 표기를 사용한다. |
| Product Code | String | product_code | 이 배치에서 생산한 제품의 코드. |
| Recipe Id | String | recipe_id | 배치 생산에 사용한 배합 또는 제조 레시피의 식별자. |
| 검토 상태 | String | Edit-only | 사람이 선택한 출하 검토 결과. 검토중, 보류, 승인 중 하나를 기록한다. |
| 보류 사유 | String | Edit-only | 배치를 보류한 이유. 검토 상태가 보류일 때 반드시 입력한다. |
| 판정자 | String | Edit-only | 출하 검토 기록 액션을 실행한 사용자. Current User로 기록한다. |
| 판정 시각 | Timestamp | Edit-only | 출하 검토 기록 액션을 실행한 날짜와 시각. Current Time으로 기록한다. |

### 보류 사유 속성의 세부 설정 예시

- Display name: 보류 사유.
- API name: 새로 만들 때 `holdReason` 같은 고유 이름을 사용한다. 제안 이름이며 기존 속성의 API 이름은 유지한다.
- Property ID가 표시되면 자동 생성된 식별자를 확인한다. 표시 이름과 구분한다.
- Description: 배치를 보류한 이유. 검토 상태가 보류일 때 반드시 입력한다.
- Type: String.
- Data: Edit-only property. 원천 CSV 컬럼을 임의로 연결하지 않는다.
- 권한 기준 Dataset을 요구하면 배치 객체의 권한 기준을 확인한다. 이 선택은 보류 사유 값을 읽어올 컬럼 선택과 다르다.
- Save to branch 후 속성 목록과 액션 규칙에서 연결을 확인한다.
- 보류일 때 필수 입력은 액션 Parameters의 Required override로 별도 설정한다.

### 타입 설명 문구 예시

- 배치출하검토: 배치별 품질검사 근거를 확인하고 출하 검토 상태와 판단 사유를 기록한다.
- 품질검사: 배치별 검사 항목, 측정값, 합격·불합격 결과를 조회한다.
- 링크의 설명란이 있는 경우: 배치에 해당하는 품질검사를 batch_id로 연결해 검토 근거를 확인한다.
- 출하 검토 기록 액션: 선택한 배치의 검토 상태와 보류 사유를 저장하고, 실행한 사용자와 시각을 함께 기록한다.
