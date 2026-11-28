# 프리소마 harness

프리소마 팀의 공용 문서 레포입니다. 코드는 없고, 모든 레포가 따르는 규칙과 명세를 모읍니다.

**서비스**: 원룸 계약서를 OCR로 읽고, 임차인이 알아야 할 점과 집주인과 상의할 점을 AI로 정리해 주는 웹 서비스

## 레포 구성

| 레포 | 내용 | 담당 |
|---|---|---|
| [frontend](https://github.com/professioNULL-presoma/frontend) | React + Vite (TypeScript) | 석현 |
| [backend](https://github.com/professioNULL-presoma/backend) | Spring Boot API, DB, 배포 | 채민, 예준 |
| ai (생성 예정) | FastAPI OCR·마스킹·AI 검토 | 소희 |
| harness | 공용 문서 (여기) | 전원 |

## 문서

| 문서 | 내용 |
|---|---|
| [CONVENTION.md](CONVENTION.md) | 브랜치, 커밋, 이슈, PR, 리뷰, 배포, 보안 규칙 |
| [docs/api.md](docs/api.md) | API 명세 |
| [docs/findings.md](docs/findings.md) | 분석 결과(findings)·정답셋 형식, 일치율 계산 초안 |

## 일정

| 날짜 | 마일스톤 |
|---|---|
| 12/9 | 1차 배포 |
| 12/26 | 기능 동결 |
| 12/28 | 최종 발표 |

## 팀

| 이름 | 역할 |
|---|---|
| 안소희 | 팀장, AI 파이프라인 |
| 김채민 | 백엔드 (회원·조회 API, ERD) |
| 김예준 | 백엔드 (업로드·암호화·배포, 개인정보보호법) |
| 손석현 | 프론트엔드, UX |

## 정기 일정

- 기술 멘토링: 화·목 19시
- 팀 회의: 수·목·금 19~23시 중
- 멘토링 때 공유: 한 일 / 할 일 / 막힌 점, 배포 IP
