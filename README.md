# One Blog 문서

One Blog의 요구사항과 기능 명세를 모은 저장소입니다. 소스 코드와 배포 방법은 [one-blog1/one_blog](https://github.com/one-blog1/one_blog)에 있습니다.

## 구성

| 폴더 | 내용 |
|---|---|
| [docs/requirements](docs/requirements/README.md) | 요구사항 분석서 1~8장. 문서끼리 다르면 [8장 결정 기록](docs/requirements/08-decision-log.md)의 최신 결정이 우선합니다 |
| [docs/roadmap.md](docs/roadmap.md) | 기능 로드맵(개발 순서) |
| docs/*.pdf | 처음 받은 요구사항 분석서 원본 |
| [specs](specs) | 기능별 Spec Kit 결과물 (명세, 계획, 작업, 데이터 모델, API 계약, quickstart). 001~016 |

## 규칙

- 요구사항을 바꾸면 해당 장과 8장 결정 기록을 함께 고칩니다.
- 기능 명세는 코드 저장소에서 Spec Kit(`/speckit-specify` → `/speckit-plan` → `/speckit-tasks` → `/speckit-implement`)으로 만들고, 만든 `specs/<번호>-<기능명>/` 폴더를 이 저장소의 `specs/`로 옮깁니다.
- DB 구조는 Crowfoot ERD "One Blog 메인블로그"를 따릅니다.
- 2026-10-08에 코드 저장소에서 분리했습니다. 그전의 변경 기록은 이 저장소에 그대로 남아 있습니다.
