# 랠리 인터뷰 — 비주얼 방향 재설정 (구현 가능성 조사)

> 작성: 2026-09-16. 현행 로우폴리 3D(빈 캐릭터 + 표준 PBR)를 폐기하고 방향을 다시 잡기 위한 조사.
> **결론 먼저: 만화/스케치는 구현 가능하다. 단 "어떻게 그리느냐"보다 "무엇을 그리느냐"가 먼저다.**

## 지금 무엇이 문제인가 — 원인을 분리한다

두 불만은 원인이 다르고, 해법도 다르다. 섞으면 잘못 고친다.

| 불만 | 진짜 원인 | 셰이더로 고쳐지나 |
|---|---|---|
| **빈 캐릭터가 극혐** | **형상 문제.** 캡슐+구 프리미티브를 사람이라 부른 것 | ❌ **안 된다.** 셀셰이딩한 캡슐은 그냥 "만화풍 캡슐"이다 |
| **코트가 옛날 3D 느낌** | **렌더링 문제.** 평평한 색면 + 표준 PBR + 그림자 = 2000년대 게임 룩 | ✅ **된다.** 선으로 그리면 즉시 사라진다 |

→ 코트는 **렌더링을 바꾸면 살릴 수 있고**, 캐릭터는 **무엇으로 대체할지 결정해야 한다.**
이 둘을 한 번에 해결하려다 실패하는 게 가장 흔한 함정이다.

### 하이큐급이 왜 어려운가 (근거)

Guilty Gear Xrd 는 실시간 3D로 손그림 애니메이션을 재현한 사실상 유일한 기준점이다.
기술감독 Junya Motomura 의 GDC 발표가 그 비용을 그대로 말해준다 — **정점을 "나중에 외곽선이 필요할 자리"에
손으로 배치하고, 법선을 일일이 보정하고, 스키닝 변형까지 통제하면서 캐릭터당 약 20k 폴리곤 예산**을 지킨다.
즉 그 룩은 셰이더가 아니라 **모델링·리깅 단계의 수작업**에서 나온다. 셰이더만 얹어서는 재현되지 않는다.

- [GDC Vault — GuiltyGearXrd's Art Style: The X Factor Between 2D and 3D](https://www.gdcvault.com/play/1022031/GuiltyGearXrd-s-Art-Style-The) · [영상](https://www.youtube.com/watch?v=yhGjCzxJV3E) · [슬라이드 PDF](https://www.ggxrd.com/Motomura_Junya_GuiltyGearXrd.pdf)
- [GGXrdShading — 기법 재현 데모(오픈소스)](https://github.com/galloscript/GGXrdShading)
- [기술감독 인터뷰 정리](https://forums.easyallies.com/topic/4187/how-the-3d-anime-style-is-made-explained-by-junya-motomura-guilty-gear-xrd-technical-director-more)

**정직한 결론: 리깅된 캐릭터를 손으로 다듬는 작업 없이는 "하이큐 같은 사람"은 안 나온다.**
그러니 선택지는 *사람을 잘 그리는 법*이 아니라 **사람을 어떻게 다룰 것인가**다.

---

## 방향 A — 연필 스케치 3D (코트는 살리고, 사람은 뺀다) ★추천

3D 지오메트리는 유지하되 **후처리로 연필선·해칭·스크린톤**을 입힌다. 빈 캐릭터는 **삭제**하고
**공을 주인공**으로 둔다.

**왜 이게 맞나**
- "옛날 3D 느낌"이 렌더링 원인이므로 **정확히 그 원인을 제거**한다
- 극혐 대상(빈)을 없애면서, 이미 만든 **카메라 키프레임·공 궤적·스크롤 시스템을 그대로 재사용**한다
- 외부 에셋 0 유지 가능. 그림을 그릴 사람이 없어도 된다
- 앞서 조사한 Lusion 사례의 원칙 — **무게와 관성을 가진 히어로 오브젝트 하나** — 와 정확히 일치한다.
  NYT 월드컵 공인구 기사도 "공 하나를 끝까지 따라가는" 같은 구조다

**구현 가능성: 높음.** 튜토리얼과 오픈소스가 다 있다.

- [Codrops — Sketchy Pencil Effect with Three.js Post-Processing](https://tympanus.net/codrops/2022/11/29/sketchy-pencil-effect-with-three-js-post-processing/)
  커스텀 렌더패스, 법선 버퍼 재렌더, 엣지 검출, 텍스처로 질감 — **필요한 전 과정이 한 편에 다 있다.**
- [spite/sketch](https://github.com/spite/sketch) ([데모](https://spite.github.io/sketch/))
  three.js + WebGL2 NPR 탐구 모음. `lines-i~vi`, `cross-hatch-i~viii`, `scribble-i~iii`,
  cartoon 필터, **blueprint**, **CMYK 하프톤**까지. 모듈형 셰이더라 기법 단위로 가져다 쓸 수 있다
- [CBRE Build — Implementing a "sketch" style of rendering in WebGL](https://medium.com/cbrebuild/implementing-a-sketch-style-of-rendering-in-webgl-d6f0e4685a17)
  Microsoft Research 의 Real-Time Hatching 논문 기반. 톤에 따라 6단계 해칭 텍스처를 갈아끼우는 방식
- 외곽선 품질이 핵심인데, 깊이 기반은 그림자까지 따라 그려 지저분해진다 →
  [Omar Shehata — Surface ID 기반 외곽선](https://omar-shehata.medium.com/better-outline-rendering-using-surface-ids-with-webgl-e13cdab1fd94) ·
  [WebGL 외곽선 렌더링 기초](https://omar-shehata.medium.com/how-to-render-outlines-in-webgl-8253c14724f9)
- 손맛: **UV 를 미세하게 흔들어** 선을 떨리게 한다(중앙이 가장 강하고 가장자리로 갈수록 약하게).
  프레임마다 난수를 새로 뽑으면 선이 "끓어" 보이므로 시드를 고정할 것

**만화 요소 얹기 (이게 '스케치'와 '만화'를 가른다)**
- **스크린톤**: 연속 계조를 점 패턴으로 — [glslify/glsl-halftone](https://github.com/glslify/glsl-halftone) ·
  [적용 예](https://medium.com/@weasert/stop-making-boring-3d-games-add-a-manga-halftone-filter-in-5-minutes-237103acabc3) ·
  [스크린톤이란](https://gootaku.com/blog/what-is-screentone)
- **집중선/속도선**: 임팩트 순간에만. 점 패턴이 차분하다면 선 패턴은 극적·행동적이다
  ([패턴 차이](https://makelineart.com/en/blog/manga-screentone-effects-guide))
- 국면 전환을 **만화 컷 분할**처럼 처리하는 것도 후보 ([패널 흐름 원칙](https://animecx.com/manga-panels/))

**사람을 완전히 버리기 아깝다면** — 인터뷰의 정서적 핵심이 사람(밥상·형·초대)이라는 점은 유효하다. 대안:
1. **흔적만 남긴다** — 벤치 위 가방·신발·물병·수건. "사람이 있었다"를 사물로
2. **실루엣** — 실제 배구 자세를 딴 납작한 실루엣(SVG 패스). 3D 모델링 불필요, 만화 톤과도 맞는다
3. **손그림 2D 스프라이트를 3D 위에 합성** — 가장 하이큐에 가깝지만 **그릴 사람이 필요하다**

---

## 방향 B — 순수 선 미니멀 (3D 없이 SVG)

WebGL 을 걷어내고 **SVG 선화 + 스크롤 드로잉**으로 간다. 사용자가 말한 "선으로만 하는 미니멀".

**구현 가능성: 매우 높음.** 기술적으로 가장 안전하다. GPU·WebGL 걱정 없음, 파일 작음,
reduced-motion 대응이 깔끔, 접근성 좋음.

- 원리: `stroke-dasharray`/`stroke-dashoffset` 로 경로를 감췄다가 스크롤에 맞춰 0 으로
  ([CSS-Tricks 원리](https://css-tricks.com/svg-line-animation-works/) ·
  [스크롤 연동](https://codefronts.com/motion/css-scroll-animations/scroll-driven-svg-stroke-draw/) ·
  [ScrollMagic 예제](https://scrollmagic.io/examples/advanced/svg_drawing.html))
- 실무 팁: **`pathLength="1"`** 을 선언하면 실제 곡선 길이와 무관하게 dash 계산이 0~1 로 정규화된다.
  `IntersectionObserver` 로 진입 시 시작, reduced-motion 이면 완성 상태를 정적으로
- 한계: **누군가 그려야 한다.** 코드가 아니라 그림이 결과물을 결정한다.
  손그림 웹 사례 모음 — [25 Websites with Hand Drawn Illustrations](https://line25.com/articles/25-websites-featuring-cool-hand-drawn-illustrations/) ·
  [Creative Market 정리](https://creativemarket.com/blog/the-internets-best-hand-drawn-websites) ·
  [스타일 개괄](https://medium.com/oceanize-geeks/sketchy-looks-hand-drawing-style-in-modern-web-ui-design-e090b6d560fa)

**A 와의 차이**: A 는 코드가 그림을 만들어 준다(그릴 사람 불필요). B 는 그림 품질이 곧 사이트 품질이다.

---

## 방향 C — 3D를 SVG 연필선으로 사전 렌더 (야심작)

[**Krbn**](https://github.com/vpalos/Krbn) — "이 픽셀은 무슨 색인가"가 아니라 **"작가라면 어떤 선을 그릴까"**
를 푸는 엔진. 실루엣을 메시 샘플이 아니라 **정확한 원뿔곡선으로** 뽑고, 은선 제거를 z-buffer 가 아니라
**해석적으로** 처리한다. 크로스해칭, 필압 테이퍼링, **시드 고정 손떨림(프레임 간 "끓음" 없음)** 지원.

**단 오프라인/배치 전용이다** (TypeScript + bun). 브라우저 실시간이 아니다.
→ 쓰는 법: **랠리를 미리 SVG 프레임 시퀀스로 렌더해 두고, 스크롤로 프레임을 넘긴다.**
손으로 안 그리고도 연필 품질의 선화를 얻는 우회로.

- 프레임 재생은 [frame-scroll-animation](https://github.com/kerimcharfi/frame-scroll-animation) 같은
  구현 참고 (ImageBitmap + 캐싱, 스프라이트시트 서브프레임)
- 비용: 빌드 파이프라인 + 에셋 용량. 품질: 높음. **인터랙티브 카메라는 포기**(정해진 프레임만 재생)

---

## 방향 D — 셀셰이딩(툰 셰이딩) 단독 · **비추천**

`MeshToonMaterial` 은 three.js 내장이고 튜토리얼도 많다
([MeshToonMaterial](https://sbcode.net/threejs/meshtoonmaterial/) ·
[커스텀 툰 셰이더](https://www.maya-ndljk.com/blog/threejs-basic-toon-shader) ·
[외곽선 후처리](https://medium.com/@coderfromnineteen/three-js-post-processing-outline-effect-6dff6a2fe3c0) ·
[애니 셰이딩 데모](https://zaneatega.github.io/Three-js-Anime-Shader/)).

**하지만 지금 문제를 못 고친다.** 셀셰이딩은 계조를 2~4단으로 계단화하는 기법이라
*형상이 좋을 때* 애니메이션처럼 보인다. 캡슐 빈을 셀셰이딩하면 **"만화풍 캡슐"**이 될 뿐이고,
GG Xrd 가 보여주듯 진짜 품질은 모델링 수작업에서 온다. A 의 일부로는 쓸 수 있어도 단독 해법은 아니다.

---

## 곁가지 — 참고할 만한 다른 NPR

- [Codrops — Susurrus: Crafting a Cozy Watercolor World with Three.js and Shaders](https://tympanus.net/codrops/2026/04/24/susurrus-crafting-a-cozy-watercolor-world-with-three-js-and-shaders/) (2026-04)
  수채화 방향. "따뜻함"이라는 브랜드 가치(누룽지·밥·환대)와 의외로 잘 맞는다. 만화는 아니지만 후보로 둘 만하다
- spite/sketch 의 **blueprint** · **CMYK 하프톤** 프리셋 — 만화가 아닌 제3의 톤

---

## 추천 — A(스케치 3D + 사람 제거) 를 기본으로, B 를 백업으로

| 방향 | 구현 | 그림 필요 | 기존 자산 재사용 | "만화" 충족 | "옛날 3D" 해소 |
|---|---|---|---|---|---|
| **A 스케치 3D + 공 주인공** | 높음 | 불필요 | 카메라·궤적·스크롤 전부 | ◐ 스크린톤·집중선 얹으면 ○ | ○ |
| B 순수 SVG 선화 | 매우 높음 | **필수** | 스크롤 구조만 | ○ | ○ |
| C Krbn 사전 렌더 | 중간 | 불필요 | 씬 데이터 | ○ | ○ |
| D 셀셰이딩 단독 | 높음 | — | 전부 | ✕ | △ |

**A 를 권하는 이유**: 그릴 사람 없이 오늘 시작할 수 있고, 이미 만든 카메라·공 궤적·스크롤 시스템이
그대로 살아 있으며, 두 불만(빈 캐릭터·옛날 3D)을 **각각의 원인에 맞게** 없앤다.
"공 하나가 서브에서 밥그릇까지 가는 연필 스케치"는 오히려 지금보다 컨셉이 선명하다.

**먼저 정할 것 — 사람을 어떻게 할 것인가.** 이게 정해져야 A 안에서도 작업이 갈린다:
① 완전 제거(공만) ② 흔적만(가방·신발·벤치) ③ 실루엣 ④ 손그림 스프라이트(그릴 사람 필요)

## 다음 단계 제안

방향이 정해지면 **전체를 갈아엎기 전에 한 국면(서브)만 프로토타입**으로 만들어 실제 화면을 보고 판단한다.
지금 방식(문서 → 전면 구현 → 실측에서 문제 발견)보다 싸다.
