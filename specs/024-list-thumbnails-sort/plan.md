# Implementation Plan: 글 목록 새 모양·영상 썸네일, 블로그 글 최신순·인기순

**Branch**: `main` | **Date**: 2026-10-13 | **Spec**: [spec.md](spec.md)

## Summary

DB: `files.purpose`에 `VIDEO_THUMB`(ERD "One Blog 메인블로그" v21, `V20__add_video_thumb_file_purpose.sql`). 영상 구분(youtube:번호 등)은 `files.original_name`에 둔다.
서버: `VideoEmbeds.Embed.key`, `MarkdownRenderer.leadingVideo`, `VideoThumbnailFetcher`(`HttpVideoThumbnailFetcher`), `VideoThumbnailService`(커밋 뒤 새 트랜잭션), `VideoThumbnailExtension`(afterSave·describeList), `PostSummaryExtension`(미리보기·작성자 사진), `PostListItem`·`PostCard`에 `authorImageUrl`·`thumbnailVideo`·`excerpt`, `PostSort`와 `GET /api/blogs/{slug}/posts?sort=`.
화면: `components/post-item.js` 새 모양, `app.css` 글(004) 자리, `blog.html`·`blog-posts.js`·`blog-categories.js` 정렬·주소, `home/home.js` 뒤로 가기 자리, `home/blog-list.js` 주소를 함부로 바꾸지 않기.

## Research

### R1. 영상 썸네일을 어디서 보여 줄까

- **Decision**: 서버가 받아 저장(사용자 선택). 읽는 사람 IP가 영상 사이트로 가지 않고 CSP `img-src 'self'` 그대로, 사이트 사진 주소가 바뀌거나 만료돼도 안 깨짐. 글당 수십 KB.
- **Alternatives**: 영상 사이트 사진을 바로 띄우기 — 저장 공간 0이지만 위 세 가지 문제, 치지직·SOOP은 어차피 서버가 API를 불러야 함.

### R2. 썸네일 주소

- 유튜브: `https://i.ytimg.com/vi/{번호}/mqdefault.jpg`(320×180, 검은 띠 없음).
- 치지직: `api.chzzk.naver.com/service/v1/clips/{번호}/detail`, `/service/v2/videos/{번호}`의 `content.thumbnailImageUrl`(공식이 아닌 API, 비공식 라이브러리 jaesung9507/chzzk·streamlink 참고).
- SOOP: `POST api.m.sooplive.com/station/video/a/view`(nTitleNo, nApiLevel=10, Referer)의 `data.thumb`(yt-dlp afreecatv 추출기 참고). 플레이어 화면의 og:image는 비어 있음.
- 배포 뒤 실제 링크로 확인 필요(이 작업 환경에서는 치지직·SOOP에 접속할 수 없었음).

### R3. 언제 받을까

- **Decision**: 글 저장 트랜잭션이 커밋된 뒤(afterCommit) 같은 요청에서. 연결 2초·응답 3초라 길어야 몇 초, 보통 수백 ms. DB는 묻는 동안 잡지 않는다.
- **Alternatives**: 비동기(@Async) — 저장 응답은 빠르지만 테스트·실패 처리가 복잡.

### R4. 남의 주소로 요청(SSRF)

- API 주소는 코드에 정한 것만, 사진 주소는 `ytimg.com`·`pstatic.net`·`naver.net`·`sooplive.com`·`sooplive.co.kr`·`afreecatv.com`·`livecloud-thumb.akamaized.net`의 https(443)만, redirect를 따르지 않음, 2MB까지, 받은 바이트는 앞부분으로 사진인지 확인하고 위치 정보 제거.

## Constitution Check

| 원칙 | 확인 | 결과 |
|---|---|---|
| III. 보안 기준선 | SSRF 제한, 사진 검사(SEC-08), CSP 그대로, 사용자 입력은 textContent | ✅ |
| IV. ERD 우선 | ERD v21을 먼저 고친 뒤 V20 | ✅ |
| V. 단순함 | 썸네일은 files 재사용, 정렬은 Pageable Sort만 바꿈 | ✅ |
