# 핸드오프 — 개인 PR 사이트 디벨롭 (새 세션 시작점)

> 작성: 2026-08-05, 도메인 전환 세션에서 넘김. **이 레포(`nulloongzi/nulloongzi.github.io`)의
> 사이트를 발전시키는 세션은 이 문서부터 읽으면 된다.**

## 현재 상태

- **https://nulloongzi.com 라이브.** 이 레포 main → `Deploy static content to Pages` 워크플로(Actions)로 자동 배포. Enforce HTTPS 켜짐.
- `index.html` 단일 페이지, 바닐라 JS(빌드 없음). 섹션: 히어로 / 소개 / 프로젝트(누룽지도·AI심판) / 연락 / 푸터. KO/EN 토글은 `data-ko`/`data-en` 속성 방식.
- 디자인 토큰은 누룽지도와 공유: 옐로 `#fac710` · 브라운 `#8d6e63` · 다크 `#4e342e` · 크림 `#fff8e1`, Pretendard. (근거: 웹 레포 `css/main.css`, `docs/design-system.md`)
- 톤 근거는 웹 레포 `docs/PHILOSOPHY.md` — "누룽지에게 DM하는 느낌", 따뜻함, 개인 브랜드가 뿌리.

## 채워야 할 플레이스홀더 (사용자에게 물어볼 것)

1. **인스타 핸들** — `index.html` 하단 `CONTACT.instagram`. 값이 비면 연락 섹션 자체가 숨겨진다.
2. **프로필 사진** — 지금은 🏐 이모지 원형(`.avatar`). 실사진으로 교체 시 `<img>`로.
3. 이메일(`CONTACT.email`) — `hello@nulloongzi.com`을 쓰려면 Cloudflare Email Routing(무료) 설정 필요.

## ⚠️ 건드리면 안 되는 것 (연쇄 장애 이력 있음)

| 대상 | 이유 |
|---|---|
| `null_oongzi-do/index.html` (리다이렉트 스텁) | 옛 `github.io/null_oongzi-do` 딥링크·구버전 앱(2.1.0+7) 공유 링크가 apex 301을 타고 여기로 착지 → 쿼리 보존해 `do.nulloongzi.com`으로 넘긴다. **지우면 기배포 링크 전멸.** |
| `.well-known/assetlinks.json` | 앱 App Links 검증용 (지문은 웹 레포 사본과 동일해야 함) |
| Settings → Pages의 **Custom domain** | 지웠다 다시 넣는 행위 금지. 2026-08-04~05에 "제거 후에도 GitHub 엣지에 301 잔존 → 지도 전체 접속 불가" 장애를 겪었다. 상세: 웹 레포 `docs/handoff-custom-domain.md` |
| `.github/workflows/static.yml` | 전체 레포를 그대로 Pages에 올린다. 빌드 도구 도입 시 이 전제가 깨지니 신중히. |

- 서브도메인 구조: apex = 이 사이트, `do.nulloongzi.com` = 지도(웹 레포 소유), 새 프로젝트는 서브도메인 추가(Cloudflare CNAME `nulloongzi.github.io` + 해당 레포 CNAME 파일). **이 레포에 다른 프로젝트 경로를 만들지 말 것** — 스텁(`null_oongzi-do/`)은 유일한 예외.

## 디벨롭 아이디어 (검토 후보, 확정 아님)

- AI 심판 프로젝트 카드 구체화 (지금은 "준비 중" 배지만)
- 누룽지도 실적 노출(클럽 수·픽업 크루 수 등) — 단, PHILOSOPHY 가치필터 #1(랭킹·위상 금지)에 걸리지 않는 방식으로
- OG 이미지(지금은 텍스트만), 파비콘 정식화(지금은 이모지 SVG)
- 인스타 피드/릴스 임베드 — 웹 레포 `js/insta-embed.js`에 선례 있음

## 관련 문서

- 웹 레포 `docs/handoff-custom-domain.md` — 도메인 구조·콘솔 등록·사고 전체 기록
- 웹 레포 `docs/PHILOSOPHY.md` / `docs/design-system.md` — 톤과 시각 언어
