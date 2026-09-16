# 랠리 인터뷰 — 레퍼런스 (타이포 + 선 방향)

> 작성: 2026-09-16. **이번에는 레퍼런스부터.** 직전 커밋(`3ce61d0`)은 레퍼런스 없이 바로 구현했고,
> 그 과정에서 카메라 연출을 내가 임의로 폐기했다. 둘 다 잘못이다. 다시 만들기 전에 근거를 모은다.

## 직전 구현에서 뭘 잘못했나

1. **레퍼런스를 안 찾았다.** 방향(타이포 + 선)을 받자마자 내 머리에서 바로 만들었다.
   빈 캐릭터 → 스케치 실루엣이 연달아 반려된 원인이 정확히 그건데 또 같은 방식으로 갔다.
2. **카메라 연출을 임의로 버렸다.** "선으로만 미니멀"을 **평면 SVG**로 단정했다.
   그런데 *선만 쓰는 것*과 *2D인 것*은 다른 문제다. 3D 씬을 **선만으로 렌더**하면
   국면별 카메라 키프레임·시네마틱 무빙을 **그대로 유지**한 채 선 미니멀이 된다.
   그 선택지를 꺼내지도 않고 혼자 접었다 — 그동안 쌓은 카메라 작업이 통째로 날아갔다.

---

## A. 선만 쓰는 3D — 카메라 무빙을 살리는 길

핵심: 면을 지우고 **모서리만 그린다.** 기존 `CAMS` 키프레임·스크롤 보간·공 궤적을 손대지 않고
렌더링만 바꾸는 것이므로, 카메라 연출이 보존된다.

- **`EdgesGeometry` + `LineSegments`** — 지오메트리의 *모서리만* 뽑는다. 인접 면의 법선 각도가
  임계값(기본 1°)을 넘을 때만 선을 그려서, 와이어프레임처럼 삼각형 대각선이 지저분하게 남지 않는다.
  ([EdgesGeometry 문서](https://threejs.org/docs/pages/EdgesGeometry.html) ·
  [LineSegments](https://threejs.org/docs/pages/LineSegments.html) ·
  [실전 정리](https://dustinpfister.github.io/2021/05/31/threejs-edges-geometry/) ·
  [포럼: 모서리만 보이게](https://discourse.threejs.org/t/how-to-show-only-edge-lines/9724))
- **굵기 있는 선이 필요하면** `LineBasicMaterial` 로는 안 된다(대부분 플랫폼에서 1px 고정).
  `LineMaterial` + `WireframeGeometry2` 를 쓰고 **`resolution` 을 반드시 설정**할 것 —
  안 하면 선이 화면을 덮어버린다. ([LineMaterial 문서](https://threejs.org/docs/pages/LineMaterial.html) ·
  [주의점 스레드](https://discourse.threejs.org/t/wireframegeometry2-linematerial-remove-diagonal-lines-on-box/61746))
- 참고: `WireframeGeometry` 는 삼각형 분할선까지 다 보여준다 → **코트·네트에는 `EdgesGeometry` 가 맞다.**

### 미감 근거 — 2026 '블루프린트' 트렌드

선만 쓰는 화면이 지금 촌스럽지 않은 이유. **가는 선 연결자, 모노스페이스 활자, 2색 팔레트**로
차분하고 의도적으로 읽히는 스타일이 2026 트렌드로 돌아왔다. "구조와 과정을 일부러 드러내는"
시스템 중심 언어 — 코트 라인·궤적·측정선이 곧 그림이 되는 우리 구조와 맞는다.

- [Kittl — Blueprint graphic design: the utilitarian aesthetic returns in 2026](https://www.kittl.com/blogs/blueprint-graphic-design-trend-stl/)
- [Zeka Design — Blueprint Graphic Design: structure and process as visual language](https://www.zekagraphic.com/blueprint-graphic-design/)

### 스크롤 × 카메라

- [Codrops — More Than a Portfolio: Building a Scroll-Driven 3D World with Something to Say](https://tympanus.net/codrops/2026/04/28/more-than-a-portfolio-building-a-scroll-driven-3d-world-with-something-to-say/) (2026-04)
- [Master.dev — Virtual Scroll-Driven 3D Scenes](https://master.dev/blog/virtual-scroll-driven-3d-scenes/) — 스크롤이 카메라 패스·조명·디테일 단계를 동시에 제어
- [Awwwards — 3D camera scroll (Pixelynx Musicverse)](https://www.awwwards.com/inspiration/3d-camera-scroll-submission-6364dde2a2f86187118961)
- [Awwwards — Scroll 3D Animation 모음](https://www.awwwards.com/inspiration/scroll-3d-animation)

---

## B. 타이포그래피가 주인공인 사이트 — 실제 수상작

내가 만든 타이포는 근거 없이 짠 것이다. 아래가 기준점이 되어야 한다.

- **By-Kin** (영국 스튜디오) — Awwwards **SOTD + Developer Award** + FWA + CSSDA.
  Next.js·GSAP·Strapi. 평가가 정확히 우리가 노리는 지점이다: **"절제의 마스터클래스 —
  자신 있는 에디토리얼 타이포, 무게감 있는 스무스 스크롤, 스스로를 드러내지 않으면서
  전체를 하나의 연속된 면처럼 느끼게 하는 전환."** 내비게이션·리빌·페이싱이 곧 이야기다.
  ([케이스 스터디](https://www.hontran.dev/blog/by-kin-case-study-award-winning-website) ·
  [Awwwards](https://www.awwwards.com/sites/kin-2))
- **sakazuki** — 2026-05 타이포그래피 아너, 2026-06-14 SOTD. 일본의 숨은 것들을 잇는 멤버십.
  **문화적 서사를 활자로 푸는** 사례라 인터뷰 구조와 가깝다. ([Awwwards](https://www.awwwards.com/sites/sakazuki))
- **日暮里ゼミナール / Nippori Seminar** — 2026-01 타이포그래피 아너.
  **"실제 목소리들이 모이는 라디오"** — 2막 커리어와 자기 발견을 다룬다. **인터뷰/구술 콘텐츠를
  활자로만 푸는 가장 가까운 레퍼런스.** ([CSS Design Awards](https://www.cssdesignawards.com/sites/nippori-seminar/48668/))
- **Locomotive** — 큰 그로테스크 활자를 모듈러 그리드에 앉히고, **스크롤 속도에 반응**하는 읽기 흐름.
- **Obys Agency** — 에디토리얼 아트디렉션·타이포 모션의 기준점. 정지 프레임이 포스터처럼 선다.
- 모음: [Awwwards 타이포그래피 수상작](https://www.awwwards.com/websites/winner_category_typography/) ·
  [에디토리얼 폰트 사이트](https://www.awwwards.com/websites/editorial/) ·
  [Simple Scroll Only Typography Site](https://www.awwwards.com/inspiration/simple-scroll-only-typography-site)

### 키네틱 타이포 (과하지 않게 쓸 것)

스크롤 위치에 **가변폰트 축(굵기·너비)** 을 매핑해 글자가 눌리고 펴지게 하는 기법이 2026 표준에 가깝다.
Pretendard Variable 을 이미 쓰고 있으니 추가 비용이 거의 없다.

- [Threestudio — Kinetic Typography in Web Design: 2026 실무 가이드](https://www.3str.net/blog/kinetic-typography-in-web-design)
- [StudioMeyer — 키네틱 타이포 2026: 예시 + GSAP 코드](https://studiomeyer.io/en/blog/kinetische-typografie)
- [Raw.Studio — Kinetic Typography Is Redefining UX](https://raw.studio/blog/stop-scrolling-kinetic-typography-is-redefining-ux/)
- NYT 장문 피처: 큰 제목 + 우아한 서체로 복잡한 이야기를 접근 가능하게 — **활자가 시선을 인도한다**

---

## C. 선이 그려지는 모션 — 지금 구현보다 나은 방법

현재는 `stroke-dashoffset` + `getPointAtLength` 를 직접 굴린다. 동작은 하지만 업계 표준 도구가 따로 있다.

- **GSAP MotionPath + ScrollTrigger** — 곡선 위 위치를 계산해 스크롤로 스크럽한다.
  `autoRotate` 로 진행 방향에 맞춰 회전까지. 반응형으로 곡선을 다시 계산하는 게 관건인데
  Codrops 글이 **드래그로 제어점을 조정하는 비주얼 컨피규레이터**까지 포함한다.
  ([Codrops — Responsive, Scroll-Triggered Curved Path Animations with GSAP](https://tympanus.net/codrops/2025/12/17/building-responsive-scroll-triggered-curved-path-animations-with-gsap/) (2025-12) ·
  [Webflow 예제](https://webflow.com/made-in-webflow/website/motionpath-with-scrolltrigger) ·
  [GSAP 포럼 — 반응형 이슈](https://gsap.com/community/forums/topic/25696-responsive-animation-of-image-along-path-scrolltrigger-motionpath/))
- **실제 사례**: [EternaCloud — Scroll-based line animation (Awwwards)](https://www.awwwards.com/inspiration/scroll-based-line-animation-eternacloud)
- 스포츠 궤적을 선으로 보여주는 관용구: 애니메이션 트레일 라인으로 움직임의 흐름을 드러낸다
  ([Flourish — 스포츠 데이터 시각화](https://flourish.studio/resources/sports/))

---

## 다시 제안하는 방향

**A + B 를 합친다. C 는 구현 도구.**

3D 씬을 유지하되 `EdgesGeometry` 로 **코트·네트·공을 선만으로** 렌더한다. 사람은 없다.
**국면별 카메라 키프레임과 시네마틱 무빙은 그대로 살아난다** — 서브에서 카메라가 서버 뒤에 있다가,
공을 따라 네트를 넘고, 스파이크에서 낙하를 따라가고, 마지막에 밥그릇으로 내려앉는다.
활자는 By-Kin·Nippori Seminar 기준으로 다시 짜고, 카메라가 비워둔 쪽에 앉힌다.

즉 **버린 카메라 작업을 되살리고, 거기에 레퍼런스 기반 타이포를 얹는 것.**

### 확인할 것

- 이 방향이 맞는지 (3D 선 렌더 + 카메라 복원 + 타이포)
- 아니면 평면 유지하되 타이포만 레퍼런스 기준으로 다시 짤지
- GSAP 도입 여부 — 현재는 무의존. MotionPath 가 필요해지는 시점에만
