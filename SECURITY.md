# 보안 운영 가이드 — Metanet AI Hub

> 임원진 피드백 반영 (2026-04) — 공개 범위 결정 전까지 본 문서의 체크리스트를 기준으로 운영합니다.

## 1. 적용된 방어 조치

| 영역 | 조치 | 위치 |
|---|---|---|
| Firestore 규칙 | 길이 제한, 필드 검증, downloads 조회 차단 | `firestore.rules` |
| 클라이언트 검증 | 본문 4000자 / 제목 200자 / 닉네임 40자 / 8초 쿨다운 | `index.html` (addPost) |
| 인덱싱 차단 | `noindex, nofollow` 메타태그 | `index.html` head |
| 사외비 표기 | 사이드바 푸터 © 2026 Metanet · 외부 공유 금지 | `index.html` 사이드바 |
| 라이선스 | All rights reserved + 외부 재배포 금지 명시 | `LICENSE` |

## 2. ⚠️ 운영자가 직접 해야 하는 작업

### 2.1 Firestore 규칙 배포 (필수, 즉시)
1. https://console.firebase.google.com → `claude-retooling-hub` 프로젝트
2. **Firestore Database → 규칙** 탭
3. `firestore.rules` 내용 복사·붙여넣기 후 **게시**
4. 규칙 시뮬레이터에서 다음 케이스 검증:
   - `posts` 4001자 본문 → 거부 ✓
   - `downloads` get 요청 → 거부 ✓
   - 빈 텍스트 게시 → 거부 ✓

### 2.2 Firebase 콘솔 설정 점검
- **Authentication 활성화 여부 확인** (현재 미사용 → 향후 도입 권장)
- **승인된 도메인** 목록에서 불필요 도메인 제거
- **Firestore 사용량 알림** 설정 (일일 read 40K, write 15K 도달 시 알림)

### 2.3 GitHub 저장소 설정
- 저장소 **public/private 결정** (Case A/B 결정 후)
- public 유지 시 README에 사외비 안내 + 라이선스 링크 추가
- branch protection: main 직접 push 금지

## 3. Case A (내부 전용) 추가 조치

GitHub Pages는 IP 제한 불가. 하나 선택:

- **(권장) Cloudflare Access** — 무료, 사내 SSO/이메일 도메인 인증
- **Firebase Hosting + Auth** — 회사 이메일(@metanet.* 등) 화이트리스트
- **임시: 비밀번호 게이트** — `auth-gate.html` 스니펫 참조 (별도 제공)

## 4. Case B (외부 공개) 추가 조치

- 호스팅을 **Cloudflare Pages 또는 Vercel** 로 이전 (대역폭 제한 회피)
- `firestore.rules` 의 `delete` 를 `false` 로 변경 → 운영자 콘솔에서만 삭제
- 사내 명단/조직도/일정 등 회사 정보 분리(별도 비공개 페이지 또는 제거)
- 모더레이션 도구(신고, 차단어 필터) 추가 개발

## 5. 정기 점검 체크리스트 (월 1회)

- [ ] Firestore 일일 read/write 사용량 확인
- [ ] `posts` 컬렉션 스팸/부적절 게시물 검토
- [ ] Firebase 콘솔 의심 IP 접근 로그 확인
- [ ] 외부 인용/스크린샷 발생 여부 모니터링
