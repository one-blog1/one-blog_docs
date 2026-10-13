# 보안 진행 현황

최종 확인 2026-10-13 · 기준: 운영 `https://one-blog.java21.net` 0.17, 코드 [one-blog1/one_blog](https://github.com/one-blog1/one_blog) `main`(fd38a11)

요구사항 [4장 보안(SEC-01~13)](requirements/04-non-functional-requirements.md)과 보안에 관련된 결정(D-80, D-102, D-125, D-126, D-128, D-135)의 진행 상황입니다. 코드와 운영 서버 둘 다에서 동작하면 **구현**, 운영 설정이나 확인이 남았으면 **미구현**으로 봅니다.

## 요약

| 구분 | 항목 수 | 항목 |
|---|---|---|
| 구현 | 19 | SEC-01, 02, 03(계정 잠금), 04, 05, 06, 07, 08, 09, 10, 11, 12(HSTS 빼고), 13, D-102, 글 안 영상(D-125), 소셜 로그인(D-126), 썸네일 받기(D-128), 배너 링크(6.8), 무중단 배포(D-135) |
| 미구현 | 4 | SEC-03(같은 IP 제한), SEC-12(HSTS), 앱 포트 직접 접속 막기, 운영·개발 DB 분리 |

## 구현

경로는 `src/main/java/com/oneblog/`, 화면은 `src/main/resources/static/` 기준입니다.

| ID | 내용 | 구현 위치 | 테스트 |
|---|---|---|---|
| SEC-01 | 비밀번호 bcrypt 저장 | `common/config/PasswordConfig.java` (`BCryptPasswordEncoder(10)`) | 전용 테스트 없음 |
| SEC-02 | 비밀번호 8~15자, 영문·숫자·특수문자 | `member/SignupPolicy.java`, `SignupService`·`AccountService`·`PasswordResetService`에서 검사 | `SignupIntegrationTest` |
| SEC-03 | 로그인 5번 실패 시 5분 잠금, 관리자 제외(D-105) | `auth/AuthService.java` (`MAX_LOGIN_FAILURES`, `LOCK_DURATION`) | `AccountSecurityIntegrationTest` |
| SEC-04 | 토큰 쿠키 HttpOnly·Secure, 로그인 유지 14일 / 30분 무활동 | `common/security/AuthCookies.java`, `auth/RefreshToken.java`. 운영은 GitHub Secrets `COOKIE_SECURE=true`(`deploy.yml`이 넘김) | `LoginLogoutIntegrationTest`, `RememberMeIntegrationTest` |
| SEC-05 | 재설정 인증번호 30분·1회·5번 틀리면 무효 | `account/PasswordResetService.java` | `AccountSecurityIntegrationTest` |
| SEC-06 | XSS·SQL Injection 방지 | `common/text/MarkdownRenderer.java`(jsoup). 화면은 글·공지 본문만 `innerHTML`(`js/pages/post/post.js`, `js/pages/notice.js`), 약관 전문은 DOMParser(`js/pages/signup.js`). SQL은 JPA와 이름 붙인 파라미터 | `PostIntegrationTest` |
| SEC-07 | 블로그별 역할을 매 요청 서버가 DB로 확인 | `blog/BlogAccessService.java`, `blog/ops/BlogManageService.requireOwner`, `category/CategoryService.requireManager` 등 | `BlogAccessIntegrationTest` |
| SEC-08 | 업로드 확장자·MIME·파일 서명·크기 검사 | `file/FileUploadService.java`, `file/ImageTypeDetector.java` | `FileUploadIntegrationTest`, `ImageTypeDetectorTest` |
| SEC-09 | 관리자 계정 분리, 전용 로그인, 활동 기록 | `auth/AuthService.java`(관리자 로그인), `admin/AdminActionLogger.java` | `AdminIntegrationTest` |
| SEC-10 | HTTPS + CSRF(`X-XSRF-TOKEN`) | `common/config/SecurityConfig.java`(`csrf.spa()`), `js/lib/api.js`. HTTPS는 서버 nginx(Certbot), http→https 301 | 기능별 통합 테스트 |
| SEC-11 | Spring Security 인증·인가 | `SecurityConfig` 주소별 권한(정한 주소 외 `anyRequest().denyAll()`, `/api/admin/**`은 `hasRole("ADMIN")`) + 서비스 메서드에서 블로그별 역할 확인(SEC-07 행). `@PreAuthorize`는 쓰지 않음. 4장 문구를 코드에 맞춤(D-138) | `BlogAccessIntegrationTest`, `AdminIntegrationTest` |
| SEC-12 | 보안 헤더 (HSTS는 미구현 표) | `SecurityConfig`: CSP(`CONTENT_SECURITY_POLICY`), Referrer-Policy `strict-origin-when-cross-origin`, X-Frame-Options DENY·nosniff(Spring Security 기본값) | `AccountSecurityIntegrationTest` |
| SEC-13 | 로그인 3번 실패 후 Cloudflare Turnstile | `common/security/TurnstileVerifier.java`, `auth/AuthService.java`(`CAPTCHA_AFTER_FAILURES`), 화면 `js/components/captcha.js`·`js/pages/login.js`. 운영 키는 서버 `~/one-blog/secrets.env` | 키 없이 꺼진 상태로만 테스트 |
| D-102 | 회원정보 수정 전 비밀번호 재확인 10분 | `account/ReauthService.java`, `auth/AccessTokenService.java`(`REAUTH_TTL`) | `AccountIntegrationTest` |
| D-125 | 글 안 영상: 재생기 사이트 세 곳만 | `common/text/VideoEmbeds.java`(링크에서 영상 번호만 뽑음, `FRAME_ORIGINS`), `MarkdownRenderer`(iframe `sandbox`, 글쓴이가 쓴 iframe은 지움). CSP `frame-src`는 Turnstile과 `www.youtube-nocookie.com`·`chzzk.naver.com`·`vod.sooplive.com`만 | `VideoEmbedTest`, `VideoFrameCspTest` |
| D-126 | 소셜 로그인: 세션 없이 서명한 HttpOnly 쿠키, 사이트 토큰 저장 안 함, 자동 연결 안 함 | `auth/social/SocialAuthorizationRequestCookies.java`(state 쿠키 10분, 고치면 서명이 안 맞아 버림), `SocialTickets.java`(서명한 JWT, 가입 대기 15분·연결 표시 10분), `NoTokenStorage.java`, `SocialLoginService.java`(같은 이메일 기존 회원은 비밀번호로 로그인해야 연결), `SocialCookies.java`. 내 정보의 연결하기는 CSRF 검사를 거친 요청에서만 | `SocialLoginIntegrationTest`, `SocialProfilesTest` |
| D-128 | 영상 썸네일 받기(SSRF 대비) | `post/HttpVideoThumbnailFetcher.java`: API 주소는 코드에 정한 것만, 사진은 정한 사이트의 https(443)만(사용자 정보·다른 포트 거부), API는 redirect를 따르지 않고 사진은 허용한 사이트 안에서만 3번까지, 연결 2초·응답 3초·2MB까지. `post/VideoThumbnailService.java`가 받은 바이트의 형식(`ImageTypeDetector`)을 확인하고 위치 정보를 지움. 읽는 사람은 우리 서버 `/files/`만 받음(`img-src 'self'`) | `HttpVideoThumbnailFetcherTest`, `VideoThumbnailIntegrationTest` |
| 6.8 | 배너 링크 검사 | `banner/BannerLinks.java`: `https://`·`http://`(호스트 필수, 사용자 정보 금지) 또는 `/`로 시작하는 우리 주소만. `javascript:`·`//다른 사이트` 거부. 배너 사진은 관리자만 올림(SEC-08 검사) | `BannerIntegrationTest` |
| D-135 | 무중단 배포 | `scripts/deploy.sh`: 바깥 nginx → 8430 프록시 컨테이너(`one-blog-proxy`, nginx:1.27-alpine) → 앱 `one-blog-blue`(8431) 또는 `one-blog-green`(8432), 모두 `--network host`. 새 버전이 `/api/health`를 통과하지 못하면 새 것만 `one-blog-<색>-failed`로 남기고 지금 버전은 그대로 요청을 받음. `deploy.env`·`secrets.env`·`run.env`는 600, 실패 요약에 남기는 로그 줄에서 password·secret이 든 줄은 뺌 | CI의 jar·Docker 실행 확인 |

## 미구현

남은 일은 운영 설정과 확인입니다. 설정값 위치: `secrets.env`는 서버 `~/one-blog/secrets.env`(배포 때 컨테이너 환경변수로 합쳐짐), Secrets는 GitHub `one_blog` 저장소 Actions Secrets. 반영은 Actions → Deploy → Run workflow.

| ID | 내용 | 지금 상태 | 남은 일 |
|---|---|---|---|
| SEC-03 | 같은 IP 로그인 10분 30번 (D-80) | 코드는 있음(`AuthService.LOGIN_PER_IP`). 운영은 `FORWARD_HEADERS_STRATEGY`가 기본 `none`이고 `deploy.yml`도 넘기지 않음. 앱은 모든 요청을 프록시 주소(127.0.0.1)로 봐서 제한이 사이트 전체에 하나로 걸림. 프록시는 `X-Forwarded-For`·`X-Forwarded-Proto`를 넘기지만 앱이 읽지 않음 | 아래 "앱 포트 직접 접속 막기"를 먼저 → `secrets.env`에 `FORWARD_HEADERS_STRATEGY=framework` → 재배포 |
| SEC-12 | HSTS | Spring Security 기본값은 HTTPS 요청에만 HSTS를 붙임. 앱이 요청을 http로 보므로 붙지 않음. 나머지 4종(CSP·X-Frame-Options·nosniff·Referrer-Policy)은 구현 | SEC-03과 같은 설정으로 해결. 이후 운영 응답 헤더로 5종 확인 |
| - | 앱 포트 직접 접속 막기 | D-135 구조에서 프록시(8430)와 앱(8431·8432)이 모두 호스트 네트워크로 열림. 프록시는 `listen 8430`(주소 지정 없음), 앱은 `SERVER_ADDRESS`가 없어 모든 주소에서 받음. 프록시는 들어온 `X-Forwarded-Proto`를 그대로 넘기고 `X-Forwarded-For`에 이어 붙임. SEC-03 설정을 켠 뒤 이 포트들에 바로 닿을 수 있으면 접속 IP·https 여부를 속일 수 있음 | 바깥에서 8430~8432로 바로 닿는지 확인 → 앱은 `secrets.env`에 `SERVER_ADDRESS=127.0.0.1`(프록시가 `127.0.0.1:<포트>`로 보내므로 그대로 동작) → 프록시는 바깥 nginx 위치에 맞춰 `listen 127.0.0.1:8430`으로(`deploy.sh` 수정) |
| - | 운영·개발 DB 분리 | 확인 전. `DEPLOY.md`에 "로컬과 같은 DB를 쓰는 동안"이라는 안내가 있고, 0.17 노트에 로컬 서버를 운영 DB에 연결하는 경우가 적혀 있음 | 서버 `deploy.env`의 `DB_HOST`·`DB_NAME`을 로컬 `.env`와 비교 → 같으면 운영용 DB 발급 후 Secrets 교체 |

### 재배포 메모

- `secrets.env`의 값은 배포할 때마다 컨테이너 환경변수에 합쳐집니다(`scripts/deploy.sh`). 같은 이름이 있으면 GitHub Secrets(`deploy.env`) 값이 이깁니다.
- `COOKIE_SECURE`는 `deploy.yml`이 항상 써 넣으므로 GitHub Secrets에서만 바꿀 수 있습니다.
- 컨테이너 재시작만으로는 바뀌지 않습니다. 고치기 전에 `cp secrets.env secrets.env.bak`로 백업합니다.
- `deploy.sh`를 고치면 다음 배포 때 Actions가 새 파일을 올립니다. 프록시 설정(`~/one-blog/proxy/nginx.conf`)은 `deploy.sh`가 배포마다 다시 쓰고, 떠 있는 프록시가 `nginx -s reload`로 다시 읽습니다.
