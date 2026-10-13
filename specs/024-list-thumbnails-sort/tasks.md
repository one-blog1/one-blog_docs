# Tasks: 글 목록 새 모양·영상 썸네일, 블로그 글 최신순·인기순

- [X] T001 ERD v21 `files.purpose` VIDEO_THUMB·`original_name` 설명, REQ-022, `V20__add_video_thumb_file_purpose.sql` (로컬 MySQL 8.0에서 V1~V20 적용 확인)
- [X] T002 `VideoEmbeds.Embed.key`, `MarkdownRenderer.leadingVideo`
- [X] T003 `VideoThumbnailFetcher`·`HttpVideoThumbnailFetcher`(유튜브·치지직·SOOP, 허용 주소, 시간 제한), `app.video-thumbnail.enabled`(테스트는 끔)
- [X] T004 `VideoThumbnailService`, `VideoThumbnailExtension`, `PostImageExtension`은 POST 파일만 정리, `PostPageRenderer` og:image
- [X] T005 `PostSummaryExtension`, `UserDisplayService.profileImages`, `PostListItem`·`PostCard` 필드
- [X] T006 `PostSort`, `GET /api/blogs/{slug}/posts?sort=`, `PostPageResponse.sort`
- [X] T007 `post-item.js`·`app.css` 새 목록, `blog.html`·`blog-posts.js`·`blog-categories.js` 정렬·주소, `home.js`·`blog-list.js` 뒤로 가기
- [X] T008 테스트 `VideoThumbnailIntegrationTest`, `HttpVideoThumbnailFetcherTest`, `VideoEmbedTest`(key·leadingVideo), `PostSortIntegrationTest`
- [X] T009 Chromium으로 블로그 정렬·카테고리·뒤로 가기, 메인 뒤로 가기, 휴대폰 목록 확인(가짜 API)
- [ ] T010 CI `./gradlew test` 통과
- [ ] T011 배포 뒤 치지직·SOOP 실제 링크로 썸네일 확인
