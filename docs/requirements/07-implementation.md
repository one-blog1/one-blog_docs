# 7. 구현 방식

3~5장을 구현할 때 팀이 고른 방식입니다. 모두 확정됐고, 선택지를 비교한 내용과 고른 이유는 오른쪽 열의 번호로 [8장 결정 기록](08-decision-log.md)에서 찾을 수 있습니다.

| 항목 | 결정 | 관련 요구사항 | 결정 기록 |
|---|---|---|---|
| 로그인 방식 | JWT + HttpOnly 쿠키, "로그인 유지" 체크박스, Refresh Token은 DB에 저장 | SEC-04 | D-53, D-61, D-62, D-82 |
| 보안 프레임워크 | Spring Security (보안 헤더, CSRF 토큰, 인증·인가) | SEC-10~12 | D-09 |
| 비밀번호 암호화 | bcrypt | SEC-01 | D-81 |
| 매크로 방지 | 계정별 잠금(5번 실패 시 5분) + IP별 요청 제한(로그인은 같은 IP 10분에 30번) | SEC-03 | D-80 |
| 사람 확인 (CAPTCHA) | Cloudflare Turnstile | SEC-13 | D-79 |
| XSS 걸러내는 위치 | 서버 (jsoup) | SEC-06 | D-64 |
| 화면 방식 | 정적 HTML + JS (글 상세의 og 태그만 서버가 채움) | SEC-10, BRD-07 | D-66 |
| 이메일 인증 | 6자리 인증번호, Spring에서 직접 발송·검증 | USR-02 | D-10 |
| 메일 발송 | Gmail SMTP | USR-02, USR-06, USR-08 | D-69 |
| 블로그 주소 | 경로 방식 `/blog/{주소}` | BLG-01 | D-70 |
| 부블로그장 권한 | 권한별 체크박스 | BLG-13, 2장 | D-71 |
| 폐쇄 처리 | `@Scheduled` 매일 04:00 (Asia/Seoul) | BLG-09 | D-12 |
| 폐쇄 예정 중 상태 | 평소처럼 운영 | BLG-09 | D-67 |
| 개수 제한값 | 설정 파일 + 환경변수, 만들 때 DB 잠금 | BLG-10 | D-68 |
| 알림 전달 | 30초마다 확인(폴링). 자동 확인은 로그인 연장에서 제외 | SOC-04 | D-72 |
| 메인 피드 | 볼 때마다 모아오기, 팔로우·구독은 탭 나누기 | BRD-09 | D-73, D-65 |
| 에디터 | Toast UI Editor (마크다운) | BRD-01 | D-74 |
| 이미지 저장 | 서버 디스크(1차), 이중화 때 S3 | BRD-05, SCL-02 | D-75 |
| 소셜 로그인 | Spring Security OAuth2 Client(카카오·네이버·구글, 키가 있는 사이트만), 사이트 토큰은 저장하지 않음 | USR-09 | D-126 |
| 목록 넘기기 | 번호 페이지 | BRD-02 | D-76 |
| 조회수 | DB 조회 기록 테이블 | BRD-11 | D-77 |
| 대댓글 | 1단계 + 답글 앞에 @닉네임 표시 | BRD-06 | D-78 |
| 배포 | GitHub Actions: `build.gradle.kts`의 version을 올려 main에 반영했을 때만 CI → Docker 이미지 → SSH로 서버에서 실행. 버전이 그대로인 push는 CI만. 버전 그대로 다시 배포는 Actions → Deploy → Run workflow. 배포에 성공하면 Actions가 코드 저장소에 태그 `v버전`을 자동으로 붙임 | 4장 배포 자동화 | D-118, D-120 |
| 버전 규칙 | 배포할 때마다 0.01씩 올림(점 뒤 두 자리, 0.21 → 0.22). 릴리즈 노트는 문서 저장소 `releases/`, 사용자 공지는 관리자가 메인 공지로 올림. 버전은 사용자가 정할 때만 올림 | 7장 배포 | D-119, D-121 |
| 브랜치·개발 흐름 | 기능 하나를 스펙 단위로(Spec Kit specify → plan → tasks → implement, 단계마다 사용자 확인). `feature/<영어 기능이름>` → 확인 후 `develop`에 merge → 버전을 올릴 때 `develop`을 `main`에 merge해 배포. 운영 버그는 `hotfix/<이름>`을 main에서 만들어 main·develop 둘 다에 merge. CI는 main·develop·feature/**·hotfix/**에서, 배포는 main에서만. push·merge는 사용자 확인 뒤 | 전체 | D-136, D-137 |
| 배포 준비 | 실행 jar 하나(`one-blog.jar`), 서버·DB 시간대 Asia/Seoul 고정, 상태 확인 `/api/health`(지금 버전도 알려 줌), GitHub Actions CI(테스트·jar 실행·Docker 실행), Dockerfile, DB 변경은 Flyway 마이그레이션(앱은 `ddl-auto: validate`). 절차는 [배포 안내](https://github.com/one-blog1/one_blog/blob/main/DEPLOY.md) | SEC-10, 4장 무중단 배포 | D-117, D-99 |
| CI 단계·정적 분석 | 1단계 `unit`(feature·hotfix push와 PR, DB 없는 단위 테스트 + 커버리지, PR merge 필수 검사) → 2단계 `full`(develop·main push, 전체 테스트 + E2E + 커버리지 + SonarQube 품질 게이트 + jar·Docker 실행 확인). E2E는 Playwright Java로 테스트 안에서 서버·DB를 띄워 핵심 흐름 7개를 실제 브라우저로 확인(CI 안의 일회용 스테이징, `...E2ETest`). SonarQube는 받은 Community 서버(`http://s4.java21.net:9000`, 프로젝트 `one-blog`)라 PR 분석 없이 merge 뒤 게이트를 보고, 실패하면 CI 실패 → main이면 배포 안 됨. 테스트 이름 `...IntegrationTest` = DB 필요. 배포는 전환 직후 운영 확인(smoke-check), 실패하면 프록시를 옛 버전으로 자동 되돌림. 스테이징·카나리 없음 | 4장 배포 자동화, 4장 무중단 배포 | D-140 |
| 무중단 배포 | 블루-그린: 8430은 nginx 프록시 컨테이너, 앱은 블루(8431)·그린(8432)을 번갈아 씀. 쉬는 쪽에 새 버전 → `/api/health` 확인 → nginx reload로 전환 → 옛 컨테이너 정상 종료(30초까지). 새 버전이 상태 확인에 실패하면 새 것만 실패 처리하고 지금 버전이 그대로 요청을 받음. DB 변경은 옛·새 버전이 함께 돌 수 있게(추가 먼저, 삭제는 다음 배포) | 4장 무중단 배포 | D-135 |

> **팀 결정: 정적 HTML + JS** (D-66, 잠시 정했던 전부 Thymeleaf(D-63)에서 변경)
>
> 화면은 HTML을 내려주고 JS가 API로 데이터를 받아 채운다. 이 방식에서 지킬 것:
> - **XSS**: 글 본문은 서버가 마크다운을 HTML로 바꾼 뒤 jsoup으로 걸러서 API로 보내고(SEC-06), 화면은 그 본문에만 `innerHTML`을 쓴다. 닉네임·댓글·태그 등 나머지는 모두 `textContent`로 넣는다.
> - **CSRF**: 모든 JS 요청에 `X-XSRF-TOKEN` 헤더를 붙인다 (SEC-10). 공통 `api.js` 한 곳에서 처리하면 빠뜨리지 않음.
> - **SNS 미리보기**: 카톡·X는 JS를 실행하지 않으므로, 글 상세 주소에 한해서는 서버가 HTML의 og 태그(제목·요약·이미지)만 채워서 내려줘야 함 (BRD-07).

> **팀 결정: 설정 파일 + 환경변수** (D-68)
>
> 값은 `blog.limit.public=${BLOG_LIMIT_PUBLIC:3}`처럼 적어 환경변수로 덮어쓸 수 있게 한다. 이중화 때는 배포 환경 설정에서 값 하나만 바꾸면 모든 서버에 같은 값이 적용되고, 바꾼 뒤에는 모든 서버를 다시 시작한다. 블로그를 만들 때는 개수를 세고 만드는 과정을 DB에서 잠가(예: 회원 행 `SELECT ... FOR UPDATE`) 동시에 여러 번 눌러도 제한을 넘지 않게 한다.
