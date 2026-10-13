# Implementation Plan: 배너 미리보기와 사진 위치 정하기

**Branch**: `main` | **Date**: 2026-10-13 | **Spec**: [spec.md](spec.md)

## Summary

DB: ERD v28 `banners.image_pos_x`·`image_pos_y`(TINYINT 0~100, 기본 50, `ck_banners_image_pos`) — `V23__add_banner_image_position.sql`.
서버: `Banner.positionImage`, `BannerRequest.imagePosX·imagePosY`(빼면 50, 범위 밖 400), `BannerRow`·`BannerSlide`에 위치.
화면: `pages/admin/banner-preview.js`(실제 크기로 그려 `transform: scale`로 줄임, pointer 끌기, range 막대), `admin-banners.js` 연결, `banner-carousel.js`가 `style.objectPosition` 적용(CSSOM이라 CSP 그대로), `app.css` 관리자 자리.

## Constitution Check

| 원칙 | 확인 | 결과 |
|---|---|---|
| III. 보안 기준선 | 값은 서버가 0~100 정수로 검사, 화면은 숫자만 style 속성에 넣음, 글자는 textContent | ✅ |
| IV. ERD 우선 | ERD v28을 먼저 고친 뒤 V23 | ✅ |
| V. 단순함 | 확대·자르기 없이 위치만, 미리보기는 메인 배너 CSS를 그대로 씀 | ✅ |
