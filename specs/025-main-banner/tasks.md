# Tasks: 메인 배너, 관리자 배너 관리, 내 블로그 목록 작게

- [X] T001 요구사항 BRD-12·ADM-08·6.8·D-132, ERD v24(REQ-035) `banners`·`site_settings`·`files.purpose` BANNER, `V21__create_banners_and_site_settings.sql`
- [X] T002 `com.oneblog.banner`: 엔티티·저장소·설정·보여 줄 배너 목록(인기 글 + 광고)
- [X] T003 관리자 API: 광고 CRUD·순서·노출·사진 업로드·인기 글 설정·미리보기, 활동 기록, 남은 BANNER 사진 정리
- [X] T004 화면: `pages/home/banner-carousel.js`·`pages/home/main-banner.js`, `index.html`·`home.js`(인사말 빼기, 내 블로그 목록 작게), `app.css`
- [X] T005 관리자 화면 "배너" 탭 `admin-banners.js`
- [X] T006 테스트 `BannerIntegrationTest`, `BannerAdminIntegrationTest`
- [X] T007 Chromium으로 메인 두 탭·휴대폰·관리자 배너 탭 확인(가짜 API: 5초 넘김, 마우스 올리면 멈춤, 참여 신청 탭에서 숨김, 움직임 줄이기면 멈춤, 비었을 때 숨김), 로컬 MySQL 8.0으로 V21·정리 쿼리 확인
- [X] T008 CI `./gradlew test` 통과
- [X] T009 D-133: ERD v26 `banners.kind`, `V22__add_banner_kind.sql`, 새 블로그·인기 블로그 자동(`BannerBlogQueries`), 종류별 설정, 관리자 종류 고르기, 인기 글 좋아요 조건 빼기, 테스트 보강
- [ ] T010 CI 통과
