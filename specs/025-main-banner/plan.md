# Implementation Plan: 메인 배너, 관리자 배너 관리, 내 블로그 목록 작게

**Branch**: `main` | **Date**: 2026-10-13 | **Spec**: [spec.md](spec.md)

## Summary

DB: ERD v24 `banners`(광고), `site_settings`(BANNER_POPULAR_COUNT·BANNER_POPULAR_DAYS), `files.purpose` BANNER — `V21__create_banners_and_site_settings.sql`.
서버(`com.oneblog.banner`): `Banner` 엔티티·`BannerRepository`, `SiteSettings`(JDBC, 없으면 기본값), `BannerService`(보여 줄 배너 목록: 인기 글 + 광고 번갈아), `BannerAdminService`(광고 CRUD·순서·설정, ADM-06 기록), `BannerController` `GET /api/banners`, `/api/admin/banners/**`, 사진 업로드 `POST /api/admin/banners/image`(관리자만, purpose BANNER). 인기 글 후보는 `FeedService`와 같은 범위(`SearchService.VISIBLE_POST`·`VISIBLE_BLOG`·차단 필터) + 썸네일 있는 글, 카드는 `PostCardService`. `PurgeTask`가 연결되지 않은 BANNER 사진도 30일 뒤 지움.
화면: `pages/home/banner-carousel.js`(넘김·멈춤·단추)·`pages/home/main-banner.js`(불러오기·탭에 따라 숨김), `index.html` 인사말 빼고 배너 한 자리(내 블로그·메인 피드에서만 보임), `home/home.js` 블로그 그림 작게, `admin.html`·`pages/admin/admin-banners.js` 배너 탭, `app.css` 배너·작은 블로그 목록·관리자 배너 자리.

## Research

- 자동으로 움직이는 내용: WCAG 2.2.2 — 멈출 수단, 마우스·포커스에 멈춤, prefers-reduced-motion이면 자동 넘김 끔. `aria-roledescription="carousel"`, 장마다 `role="group" aria-roledescription="slide"`, 보이지 않는 장은 `inert`.
- 광고 링크: 저장할 때 스킴 검사(https/http 또는 `/`로 시작하고 `//`가 아닌 것). 화면은 `a.href`에 넣고 바깥 주소는 `target=_blank rel="noopener noreferrer sponsored"`.
- 사진은 우리 서버 `/files/`만(CSP img-src 'self' 그대로).

## Constitution Check

| 원칙 | 확인 | 결과 |
|---|---|---|
| III. 보안 기준선 | 관리자 API는 ADMIN만(SecurityConfig), 사진 검사(SEC-08), 링크 스킴 검사, textContent, 활동 기록 | ✅ |
| IV. ERD 우선 | ERD v24를 먼저 고친 뒤 V21 | ✅ |
| V. 단순함 | 인기 글은 테이블 없이 볼 때 계산, 설정은 키-값 한 테이블 | ✅ |
