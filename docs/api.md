# API 명세

백엔드가 API를 만들거나 바꿀 때 이 문서를 먼저 고치는 PR을 올립니다. 프론트는 이 문서를 기준으로 연동합니다.

## 공통

- Base URL: `http://<배포 IP>:8080/api` (도메인 정해지면 변경)
- 인증: `Authorization: Bearer <accessToken>`
- 날짜: ISO 8601 (`2026-12-09T19:00:00+09:00`)

### 응답 형식

```json
{ "success": true, "data": { }, "error": null }
```

```json
{ "success": false, "data": null, "error": { "code": "CONTRACT_NOT_FOUND", "message": "계약서를 찾을 수 없습니다." } }
```

## 엔드포인트 목록

| 기능 | Method | URL | 인증 | 담당 | 상태 |
|---|---|---|---|---|---|
| 회원가입 | POST | `/auth/signup` | X | 채민 | 개발 중 |
| 로그인 | POST | `/auth/login` | X | 채민 | 개발 중 |
| 현재 회원 | GET | `/auth/me` | O | 채민 | 개발 중 |
| 현재 동의 문서 | GET | `/consent-documents/{type}` | X | 채민 | 개발 중 |
| 계약서 업로드 | POST | `/contracts` | O | 예준 | 설계 중 |
| 계약서 목록 | GET | `/contracts` | O | 채민 | 개발 중 |
| 계약서 분석 결과 | GET | `/contracts/{id}/analysis` | O | 소희 | 개발 중 |

상태: `설계 중` → `개발 중` → `완료`

## 인증과 동의 문서

### GET `/consent-documents/{type}`

현재 시행 중인 활성 문서 중 최신 버전을 반환한다. `type`은 `SIGNUP_PRIVACY`, `CONTRACT_PROCESSING`, `EXPERT_SHARING`, `RESEARCH_USE` 중 하나다.

| HTTP | 응답 |
|---|---|
| 200 | `data: { id, documentType, version, title, content, isRequired, effectiveAt }` |
| 400 | `INVALID_INPUT` |
| 404 | `CONSENT_DOCUMENT_NOT_FOUND` |

### POST `/auth/signup`

`Content-Type: application/json`으로 보낸다.

| 필드 | 형식 |
|---|---|
| `email` | 이메일 주소. 서버에서 소문자로 정규화한다. |
| `password` | 영문과 숫자를 포함해 8자 이상, UTF-8 기준 최대 72바이트 |
| `nickname` | 앞뒤 공백을 제외하고 2~20자 |
| `agreedConsentDocumentIds` | 현재 시행 중인 `SIGNUP_PRIVACY` 문서 ID 하나가 담긴 UUID 배열 |

| HTTP | 응답 |
|---|---|
| 201 | `data: { userId }` |
| 400 | `INVALID_INPUT` 또는 `INVALID_CONSENT` |
| 404 | `CONSENT_DOCUMENT_NOT_FOUND` |
| 409 | `EMAIL_ALREADY_EXISTS` |

### POST `/auth/login`

JSON으로 `email`과 `password`를 보낸다. 발급된 JWT는 1시간 동안 유효하다.

| HTTP | 응답 |
|---|---|
| 200 | `data: { accessToken, user: { id, nickname } }` |
| 400 | `INVALID_INPUT` |
| 401 | `INVALID_CREDENTIALS` |
| 503 | `AUTH_UNAVAILABLE` |

### GET `/auth/me`

`Authorization: Bearer <accessToken>`을 보낸다.

| HTTP | 응답 |
|---|---|
| 200 | `data: { id, nickname }` |
| 401 | `INVALID_ACCESS_TOKEN` |
| 503 | `AUTH_UNAVAILABLE` |

## 계약서 내역과 분석 결과

두 API에 `Authorization: Bearer <accessToken>`을 보낸다.

### GET `/contracts`

로그인한 회원의 삭제되지 않은 계약서를 최신순으로 반환한다. `data`는 배열이며 항목은 `{ id, title, processingStatus, createdAt }`이다. 내역이 없으면 빈 배열을 반환한다. 제목이 없는 계약서는 `title`이 `null`이다.

`processingStatus` 값은 `UPLOADED`, `OCR_PROCESSING`, `RETAKE_REQUIRED`, `EXTRACTING`, `REVIEWING`, `COMPLETED`, `FAILED`다.

### GET `/contracts/{id}/analysis`

현재 공개된 성공 결과를 반환한다. 재분석이 실패한 뒤에도 이전 공개 결과가 있으면 읽을 수 있다. `processingStatus`는 현재 계약서 상태이며, `completedAt`은 공개된 리뷰의 완료 시각이다.

| HTTP | 응답 |
|---|---|
| 200 | `data: { contractId, title, processingStatus, reviewId, summary, findings, completedAt, conditions }` |
| 400 | `INVALID_INPUT` |
| 401 | `INVALID_ACCESS_TOKEN` |
| 404 | `CONTRACT_NOT_FOUND`. 없는 계약서, 삭제된 계약서, 다른 회원 계약서는 같은 코드로 응답한다. |
| 409 | `ANALYSIS_NOT_READY`. 현재 공개된 분석 결과가 없다. |

`conditions`에는 `propertyType`, `propertyAddressMasked`, `depositAmount`, `monthlyRentAmount`, `maintenanceFeeAmount`가 들어 있다. 추출되지 않은 필드는 `null`이다. 금액은 원 단위다. `findings`는 저장된 검토 항목 배열이며 항목 내부 필드는 AI와 프론트 연동 작업에서 확정한다.

## 상세 (작성 예시)

### POST `/contracts` 계약서 업로드

**Request** `multipart/form-data`

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| file | File | O | 계약서 이미지 또는 PDF |

**Response** `201`

```json
{ "success": true, "data": { "contractId": 1, "status": "PROCESSING" }, "error": null }
```

**Errors**

| 코드 | HTTP | 언제 |
|---|---|---|
| `INVALID_FILE_TYPE` | 400 | 지원하지 않는 형식 |
| `FILE_TOO_LARGE` | 400 | 용량 초과 |
