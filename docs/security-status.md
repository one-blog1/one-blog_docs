# 보안 진행 현황

최종 확인 2026-10-12 · 기준: 코드 [one-blog1/one_blog](https://github.com/one-blog1/one_blog) `main`(54f3ba4), 운영 `https://one-blog.java21.net`(`/api/health` 0.1.0)

요구사항 [4장 보안(SEC-01~13)](requirements/04-non-functional-requirements.md)과 관련 결정(D-80, D-102)의 진행 상황입니다. 코드와 운영 서버 둘 다에서 동작하면 **구현**, 운영 설정이나 확인·결정이 남았으면 **미구현**으로 봅니다.

## 요약

| 구분 | 항목 수 | 항목 |
|---|---|---|
| 구현 | 13 | SEC-01, 02, 03(계정 잠금), 04, 05, 06, 07, 08, 09, 10, 11, 13, D-102 |
| 미구현 | 5 | SEC-03(IP 제한), SEC-12(HSTS), 8430 직접 접속 차단, 운영·개발 DB 분리, SEC-11 문서 정리 |

## 구현

| ID | 내용 | 구현 위치 (`src/main/java/com/oneblog/` 기준) | 테스트 |
|---|---|---|---|
| SEC-01 | 비밀번호 bcrypt 저장 | `common/config/PasswordConfig.java` (`BCryptPasswordEncoder(10)`) | 전용 테스트 없음 |
| SEC-02 | 비밀번호 8~15자, 영문·숫자·특수문자 | `member/SignupPolicy.java`, `SignupService`·`AccountService`·`PasswordResetService`에서 검사 | `SignupIntegrationTest` |
| SEC-03 | 로그인 5번 실패 시 5분 잠금 (관리자 제외, D-105) | `auth/AuthService.java` | `AccountSecurityIntegrationTest` |
| SEC-04 | 토큰 쿠키 HttpOnly·Secure, 로그인 유지 14일 / 30분 무활동 | `common/security/AuthCookies.java`, `auth/RefreshToken.java`. 운영은 GitHub Secrets `COOKIE_SECURE=true`. 2026-10-12 브라우저에서 HttpOnly·Secure 확인 | `LoginLogoutIntegrationTest`, `RememberMeIntegrationTest` |
| SEC-05 | 재설정 인증번호 30분·1회·5번 틀리면 무효 | `account/PasswordResetService.java` | `AccountSecurityIntegrationTest` |
| SEC-06 | XSS·SQL Injection 방지 | `common/text/MarkdownRenderer.java`(jsoup), 화면은 `post.js`·`notice.js` 본문만 `innerHTML`, SQL은 JPA | `PostIntegrationTest` |
| SEC-07 | 블로그별 역할을 매 요청 서버가 확인 | `blog/BlogAccessService`, `BlogManageService.requireOwner`, `CategoryService.requireManager` 등 | `BlogAccessIntegrationTest` |
| SEC-08 | 업로드 확장자·MIME·파일 서명·크기 검사 | `file/FileUploadService.java`, `file/ImageTypeDetector.java` | `FileUploadIntegrationTest`, `ImageTypeDetectorTest` |
| SEC-09 | 관리자 계정 분리, 전용 로그인, 활동 기록 | `auth/AuthService.adminLogin`, `admin/AdminActionLogger.java` | `AdminIntegrationTest` |
| SEC-10 | HTTPS + CSRF(`X-XSRF-TOKEN`) | `common/config/SecurityConfig.java`(`csrf.spa()`), `static/js/api.js`. HTTPS는 서버 nginx(Certbot), http→https 301 확인 | 기능별 통합 테스트 |
| SEC-11 | Spring Security 인증·인가 | `SecurityConfig` 주소별 권한 + `anyRequest().denyAll()` | `BlogAccessIntegrationTest` |
| SEC-13 | 로그인 3번 실패 후 Cloudflare Turnstile | `common/security/TurnstileVerifier.java`, 화면 `static/js/captcha.js`·`login.js`. 운영 키는 서버 `~/one-blog/secrets.env`. 2026-10-12 운영 `/api/auth/captcha-config` `"enabled":true` 확인 | 키 없이 꺼진 상태로만 테스트 |
| D-102 | 회원정보 수정 전 비밀번호 재확인 10분 | `account/ReauthService.java` | `AccountIntegrationTest` |

## 미구현

코드는 들어가 있고, 남은 일은 운영 설정과 확인·결정입니다. 설정값 위치: `secrets.env`는 서버 `~/one-blog/secrets.env`(배포 때 컨테이너 환경변수로 합쳐짐), Secrets는 GitHub `one_blog` 저장소 Actions Secrets. 반영은 Actions → Deploy → Run workflow.

| ID | 내용 | 지금 상태 | 남은 일 |
|---|---|---|---|
| SEC-03 | 같은 IP 로그인 10분 30번 (D-80) | `FORWARD_HEADERS_STRATEGY`가 기본 `none`이라 nginx 뒤에서 모든 요청이 127.0.0.1로 보임. 제한이 사이트 전체에 하나로 걸림 | nginx에 `X-Forwarded-For`·`X-Forwarded-Proto` 확인 → `secrets.env`에 `FORWARD_HEADERS_STRATEGY=framework` → 재배포 |
| SEC-12 | 보안 헤더 5종 | CSP·X-Frame-Options·nosniff·Referrer-Policy는 코드에 있음. HSTS는 앱이 요청을 https로 알아야 붙음 | SEC-03과 같은 설정으로 해결. 이후 `curl -sI --resolve one-blog.java21.net:443:127.0.0.1 https://one-blog.java21.net`로 5종 확인 |
| - | 8430 직접 접속 차단 | 앱이 호스트 네트워크로 8430을 직접 엶. SEC-03 설정을 켜면 8430으로 IP를 속일 수 있음 | nginx `proxy_pass`가 127.0.0.1인지 확인 → `secrets.env`에 `SERVER_ADDRESS=127.0.0.1` (SEC-03과 함께) |
| - | 운영·개발 DB 분리 | 확인 전. `DEPLOY.md`상 로컬과 같은 Crowfoot DB(`cf_u36_d1`)를 쓸 가능성 | 서버 `deploy.env`의 `DB_HOST`·`DB_NAME` 확인 → 같으면 운영용 DB 발급 후 Secrets 교체 |
| SEC-11 | 블로그별 역할은 `@PreAuthorize`로 확인 (4장 문구) | 코드는 서비스 메서드(`requireOwner` 등)로 확인. 보안 효과는 같음 | 4장 문구를 구현에 맞출지 팀 결정 |

### 재배포 메모

- `secrets.env`의 값은 배포할 때마다 컨테이너 환경변수에 합쳐집니다(`scripts/deploy.sh`). 같은 이름이 있으면 GitHub Secrets 값이 이깁니다.
- `COOKIE_SECURE`는 `deploy.yml`이 항상 써 넣으므로 GitHub Secrets에서만 바꿀 수 있습니다.
- 컨테이너 재시작만으로는 바뀌지 않습니다. 고치기 전에 `cp secrets.env secrets.env.bak`로 백업합니다.
