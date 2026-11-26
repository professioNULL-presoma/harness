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
| 계약서 목록 | GET | `/contracts` | O | 채민 | 설계 중 |
| 계약서 분석 결과 | GET | `/contracts/{id}/analysis` | O | 소희 | 설계 중 |

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
