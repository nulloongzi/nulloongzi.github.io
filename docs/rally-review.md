# 랠리 인터뷰(`rally3d.html`) 냉정 평가 + 보완 레퍼런스

> 작성: 2026-09-16. **결정: `rally3d.html`을 `index.html`로 승격(2번).** 지금 메인(서비스 소개)은
> 그 아래 섹션으로 흡수한다. 이 문서는 승격 전에 무엇을 다듬을지의 브리프다.
> 평가는 코드 전체(623줄)를 읽고 썼다. 레퍼런스는 이 환경에서 본문 접근이 막혀 검색 요약 기준.

## 총평

**컨셉 A · 카피 A- · 실행 C+.** 프로토타입으로는 훌륭하고, 메인으로 내보내기엔 프로토타입답게 보인다.

### 살릴 것

- **형식이 내용을 말한다.** 서브(시작)→리시브(받은 것)→토스(올려주는 것)→스파이크(하고 싶은 말)→착지(밥그릇).
  흔한 "스크롤하면 3D가 돈다"가 아니라 랠리 국면이 곧 인생 국면이다. 이게 이 페이지의 전부고, 손대지 말 것.
- **카피에 목소리가 있다.** "받은 환대는 플레이가 아니라 밥상에서 왔어요" · "그걸로 충분해요" ·
  "널? 눌? 눌웅지?" — 구체적이고 자기 말이다. 다듬되 매끈하게 갈지 말 것.
- **밥그릇 엔딩**이 브랜드(누룽지·밥·도시락)를 회수한다.
- 기술적으로 성실하다: 외부 에셋 0, Three.js 로컬 번들(r168), 카메라 lerp 스무딩, ACES 톤매핑,
  WebGL 실패 try/catch, reduced-motion 일부 대응.

### 약한 것 (냉정하게)

| # | 문제 | 근거(코드) | 왜 중요한가 |
|---|---|---|---|
| A | **랠리가 일어나지 않는다.** 선수 4명 전부 정지 포즈. 공과 카메라만 움직인다. | `server/receiver/setter/spiker` 모두 초기 rotation만 설정, 프레임 루프에서 갱신 없음 (415~447행) | "랠리 인터뷰"인데 랠리는 디오라마고 카메라가 그 사이를 날 뿐. 공이 얼어 있는 스파이커의 손을 통과한다. 은유의 핵심(움직임)이 비어 있다. |
| B | **텍스트와 3D가 분리돼 있다.** 유리 카드가 장면 위에 떠 있다. | `.inner { background: rgba(255,248,225,.86); backdrop-filter: blur(7px) }` (31~42행) | 크림 카드 위 크림 바닥 → 뭉개짐. 텍스트가 장면 *속*이 아니라 장면을 *가리는* 구조. 카메라가 텍스트 자리를 비워주는 프레이밍이 없다. 2019년식 패턴. |
| C | **폰트를 안 불러온다.** | `font-family: "Pretendard Variable", Pretendard, …` 만 있고 `@font-face`·`<link>` 없음 | 전부 시스템 폴백. Windows는 맑은 고딕. 의도한 인상이 나오지 않는다. |
| D | **메인이 될 준비가 안 됐다.** | `<title>…(프로토타입)</title>`, OG 메타 0, 파비콘 0, 영문 0, 프로젝트·채널 링크 없음 | index엔 KO/EN 토글이 있고 PHILOSOPHY상 외국인이 교두보인데 "나를 소개하는" 페이지가 한국어만. 누룽지도·안치기 카드, 유튜브·링크트리도 없다. |
| E | **모바일이 위험하다.** | shadow map 2048 + PCFSoft + ACES + 풀스크린 캔버스 + DPR 2 (195~199행). `reduced`는 lerp만 끔(594행). 로딩 상태 없음 | 중급 폰에서 뜨겁고 끊긴다. 2D 폴백(`rally.html`)은 계획만 있고 연결이 없다. Three.js 모듈 받는 동안 크림색 빈 화면. |
| F | **방향감이 없다.** | 7섹션, 진행 표시·국면 네비 없음. 힌트는 히어로에 한 번 | 얼마나 남았는지, 지금이 어느 국면인지 모른다. 긴 스크롤 내러티브의 기본 장치가 빠졌다. |
| G | **인터뷰어가 없다.** | `.q::before { content: "Q. " }` 뿐 | 누가 묻는지 정체가 없다. 장치 하나면 톤이 서는데(공이 묻는다 / 독자가 묻는다 / 형이 묻는다) 지금은 FAQ처럼 읽힌다. |
| H | **AI 심판 모순.** | 155행 "AI 심판이요 … 만들고 있어요" | 2026-09-16에 메인의 AI 심판 카드를 "코드 흔적 0"으로 내렸다. 인터뷰가 메인이 되면 카드는 없는데 본인이 준비 중이라 말하는 상태. 문구를 살릴지 카드를 되살릴지 정해야 한다. |
| I | 잔재 | 532행 `Math.max(...) * 0 +` 죽은 코드 · 64행 `.hint { margin-top: 46vh }` 하드코딩 · 카드 페이드가 한 방향(위로 스크롤하면 전부 켜진 채) · 캔버스 `aria-hidden` 없음 · 국면 제목이 `div`(h2 아님) | 스크린리더엔 평면 문서. 잔재는 승격 때 같이 치운다. |

## 보완 방향 + 레퍼런스

### A. 랠리를 실제로 일으키기

최소: 국면 진입 시 해당 선수의 팔 스윙 한 번(pivot 회전 트윈), 공이 닿는 순간 임팩트(스케일 펄스 + 짧은 카메라 셰이크).
카메라가 "치는 순간"에 도착하도록 키프레임을 재배치.

- [Codrops — How to Build Cinematic 3D Scroll Experiences with GSAP](https://tympanus.net/codrops/2025/11/19/how-to-build-cinematic-3d-scroll-experiences-with-gsap/) (2025-11) — 스크롤을 카메라 패스·라이팅·셰이더에 연결해 정지 장면을 시퀀스로.
- [Lusion 모의 제품 런치 — Awwwards SOTM 2026-04](https://www.hontran.dev/blog/best-award-winning-websites-2026) — **무게와 관성이 있는 히어로 오브젝트 하나**, 스크롤이 2D 레이어를 미는 게 아니라 진짜 Z축 깊이로 카메라를 움직임. 우리 공이 이래야 한다.
- [Bruno Simon 포트폴리오](https://bruno-simon.com/) · [케이스 스터디](https://medium.com/@bruno_simon/bruno-simon-portfolio-case-study-960402cc259b) — 캐릭터가 세계와 *상호작용*하는 3D 개인 사이트의 원형. 2025판은 Blender + TSL.
- [Codrops — Scroll-Driven 3D Gallery Using a Blender Camera Path](https://tympanus.net/codrops/2026/07/07/building-a-scroll-driven-3d-gallery-using-a-blender-camera-path-with-three-js-and-gsap/) (2026-07) — 지금 하드코딩된 `CAMS` 키프레임 8개를 Blender 스플라인으로 그려 넣는 방법. 프레이밍 재설계(B)와 같이 하면 좋다.

### B. 텍스트를 장면 안에 넣기

유리 카드를 없애거나 크게 줄이고, **카메라 키프레임마다 화면 한쪽 40%를 비워 텍스트를 거기 놓는다.**
정지 프레임 하나하나가 포스터로 보이게. 국면 제목(Serve/Receive…)은 장면 속 3D 텍스트나 코트 라인에 얹는 것도 후보.

- [Obys Agency](https://www.awwwards.com/awwwards/collections/webgl/) — 편집 아트디렉션·타이포 모션의 기준. **정지 프레임이 포스터처럼 보인다.**
- [Five Years of -99 — Awwwards](https://www.awwwards.com/inspiration/webgl-fullscreen-horizontal-scroll-navigation-five-years-of-99) — WebGL + 실험 타이포 + GSAP.
- [Awwwards WebGL 컬렉션](https://www.awwwards.com/awwwards/collections/webgl/) · [30 Experimental WebGL Websites](https://www.awwwards.com/30-experimental-webgl-websites.html) — "3D를 스펙터클이 아니라 **분위기와 프레이밍**에 쓴다"는 게 공통 평.

### C. 폰트 — 한 줄이면 끝

```html
<link rel="stylesheet" as="style" crossorigin
  href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/variable/pretendardvariable-dynamic-subset.min.css">
```

- [Pretendard README](https://github.com/orioncactus/pretendard/blob/main/packages/pretendard/docs/en/README.md) — dynamic-subset은 페이지에 쓰인 글자만 내려받는다(전체 variable CSS 대비 ≈2MB 절감).
  `index.html`도 같은 상태(폰트 미로드)라 같이 고친다.

### D. 메인 승격 체크리스트

`index.html`에서 가져올 것: `<title>` · OG(og.png 1200×630 이미 있음) · 파비콘 4종 · KO/EN 토글(`data-ko/data-en`) · 프로젝트 카드(누룽지도·안치기) · 채널(인스타·유튜브·링크트리).
착지 섹션 뒤에 "지금 만드는 것" 섹션으로 붙인다 — 인터뷰가 먼저, 서비스가 뒤.

### E. 성능 티어 + 폴백

- 데스크톱: 지금 설정 유지.
- 모바일: shadow map 1024, PCF(soft 아님), DPR ≤1.5, fog 유지. `InstancedMesh`는 지금 오브젝트 수(≈40)에선 불필요.
- 저사양·`prefers-reduced-motion`·WebGL 실패: **`rally.html`(2D) 또는 정적 포스터로 분기.** 지금은 `reduced`가 lerp만 끄고 렌더 부하는 그대로다.
- 로딩: Three.js 모듈 도착 전까지 히어로 텍스트는 먼저 보이게(캔버스만 늦게).

- [Utsubo — 100 Three.js Tips That Actually Improve Performance (2026)](https://www.utsubo.com/blog/threejs-best-practices-100-tips) — 데스크톱 풀/모바일 축소/저사양 정적의 3티어를 WebGL 확장·GPU 문자열·벤치 렌더로 판별.
- [Digital Strategy Force — Three.js 모바일 최적화](https://digitalstrategyforce.com/journal/how-do-you-optimize-threejs-performance-for-mobile-devices/) — 텍스처 50%/25% 변형, 드로우콜 <50, 블룸·톤매핑은 전 티어 / DOF는 상위 티어만.
- [AppScale — Three.js in Production 2026](https://appscale.blog/en/blog/threejs-production-3d-web-2026-webgpu-realtime-standards) — WebGPU → WebGL → 포스터 폴백 사슬 + reduced-motion.
- [DEV — Improving UX in Three.js "Scroll Me" Websites](https://dev.to/maxgeris/improving-ux-in-threejs-scroll-me-websites-addressing-common-pitfalls-for-better-user-engagement-5d7e) — 스크롤 하이재킹·진행 표시 부재·무거운 초기 로드·폴백 없음이 흔한 함정.

### F. 진행 표시 + 국면 네비

화면 가장자리에 세로 진행 바 + 국면 마커 5개(Serve·Receive·Toss·Spike·Land) 고정. 클릭하면 해당 섹션으로.
`scroll-snap`으로 챕터를 맞출지는 시험 후 결정 — 카메라 lerp와 충돌할 수 있다.

- [Vev — Scrollytelling 가이드](https://www.vev.design/guides/scrollytelling/) · [Scrollytelling Tools](https://www.vev.design/blog/scrollytelling-tools/) — 진행 표시·챕터 마커가 완주율을 올린다.
- [Lovable — Scrolling Designs: 8 Patterns (2026)](https://lovable.dev/guides/scrolling-designs-patterns-when-to-use) — 어느 패턴을 언제.
- [NYT Snow Fall](https://thecityjournal.net/innovation/technology-reflections-2022/the-art-of-scrollytelling/) (2012) — 장르의 기준점. 챕터 구분과 미디어 배치.

### 형식 자체의 레퍼런스 (스포츠 × 스크롤 내러티브)

- [NYT — 월드컵 공인구의 역사](https://thecityjournal.net/innovation/technology-reflections-2022/the-art-of-scrollytelling/) — **공 하나**를 스크롤로 해부·변천. 우리와 가장 가까운 구조.
- [The Pudding](https://pudding.cool/) — 농구 코트 59,507개 위성 모자이크, 야구 라인업 에세이. 데이터가 아니라 **한 사물·한 사람을 끝까지 따라가는 집요함**을 참고.
- [The Pudding — Making Internet Things pt.3: Storytelling](https://pudding.cool/process/how-to-make-dope-shit-part-3/) — 그들의 제작 원칙.
- 2026 표준 스택: **Lenis(스무스 스크롤) + GSAP ScrollTrigger + Three.js**, 챕터엔 `scroll-snap`, CSS `view-timeline`
  ([Svilenković — Scrollytelling Trends 2026](https://svilenkovic.com/3d/scrollytelling-trends-2026), [Utsubo — Best Three.js Websites 2026](https://www.utsubo.com/blog/best-threejs-websites-2026)).
  지금은 순수 rAF + lerp인데 그대로도 된다 — 라이브러리 도입은 A·F를 하다 필요해질 때.

### G. 인터뷰어 장치 (선택)

세 후보: **공이 묻는다**(랠리 은유와 맞물림) / 독자가 묻는다("여기까지 스크롤한 사람에게"가 이미 이 톤) / 그 형이 묻는다(토스 국면의 "우연히 한 형").
[Chris Guillebeau — An Interview With Yourself](https://chrisguillebeau.com/an-interview-with-yourself) — 자문 형식이 왜 자기소개보다 강한가.

### J. 소리 (선택, 기본 무음)

랠리는 소리가 있는 스포츠다 — 서브 휘슬, 타격음, 바닥 튐, 그릇에 담기는 소리. 국면 임팩트(A)에 붙이면 효과가 크다.
단 **기본 무음 + 마스터 토글**, 스크롤마다 나지 않게 국면 전환 순간 한 번만.
[Supadark — 5 Best Practices for Web Sound Effects](https://supadark.com/notes/5-best-practices-for-designing-web-sound-effects) — 목적 없는 소리는 뺀다, 호버·스크롤마다 금지, 마스터 뮤트.

## 승격 작업 순서

**P0 — 승격 전제(이거 없이 메인에 못 올림)**
1. D: 메타·OG·파비콘·타이틀, 프로젝트·채널 섹션, KO/EN
2. C: Pretendard 로드
3. E: 저사양·reduced-motion → `rally.html` 분기, 로딩 상태
4. H: AI 심판 문구 결정
5. I: 잔재 정리(죽은 코드, 하드코딩 vh, h2, aria-hidden)
6. 기존 `index.html` 내용 → 인터뷰 뒤 섹션으로 이관, 파일 교체

**P1 — 완성도**
7. F: 진행 바 + 국면 네비
8. A: 국면 임팩트(팔 스윙·공 펄스·셰이크)
9. B: 카메라 프레이밍 재설계, 유리 카드 축소
10. E: 모바일 티어 분기

**P2 — 있으면 좋은 것**
11. G: 인터뷰어 장치
12. J: 소리
13. Blender 카메라 패스로 `CAMS` 교체

## 하지 말 것

- 카피를 "매끈하게" 고치기. 지금 목소리가 자산이다.
- 랠리 구조 바꾸기. 국면 5개는 확정.
- 라이브러리부터 넣기. Lenis·GSAP은 A·F를 손으로 하다 한계가 오면.
- 데스크톱만 보고 판단하기. 트래픽의 2/3는 폰이다.
