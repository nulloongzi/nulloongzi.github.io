# 손 에셋 AI 생성 파이프라인 (hand-asset-pipeline.md)

> **[폐기 — 2026-08-07]** 연출이 볼캠(공 추적)으로 전면 전환되며 손 오버레이 자체가
> 제거됐다. 이 문서와 `docs/design/hands/`는 기록용으로만 보존한다.

> rally3d v4의 SVG 손 심볼을 AI 이미지 생성(I2I + ControlNet)으로 고품질 일러스트/실사로
> 교체하기 위한 규정. 입력 이미지·엣지 맵·프롬프트가 한 세트다.
> 작성: 2026-08-06. 상위 문서: `website_design.md`.

## 0. 스타일 트랙 결정 (먼저 정할 것)

| 트랙 | 설명 | 판단 |
|---|---|---|
| **A. 애니 작화 (권장)** | 하이큐풍 스포츠 애니 손 — 해부학은 정확하되 셀 셰이딩+선화 | 사이트의 톤 셰이딩 세계와 렌더링 문법이 일치. 켄마 POV 원본도 이 방식 |
| **B. 실사 사진** | 모공·손톱까지 보이는 photo-real | 형태 정확도는 최고. 단, 톤 셰이딩 배경 위에서 **질감 충돌 위험** — 시안으로 1장만 뽑아 합성 확인 후 결정할 것 |

두 트랙 다 아래 같은 입력·워크플로를 쓰고 텍스트 프롬프트만 다르다.

## 1. 입력 세트 (5포즈 × 2파일)

`docs/design/hands/` (원본 SVG 렌더 1024px 투명 PNG + ControlNet용 엣지 맵):

| 파일 | 포즈 | 쓰이는 비트 |
|---|---|---|
| `open-hand.png` / `-edge.png` | 뻗은 손 (손등, 손가락 위) | 스파이크/서브 조준 |
| `spread-hand.png` / `-edge.png` | 활짝 벌린 손 (손등) | 토스 양손(미러), 타격 오픈팜(회전) |
| `fist-hand.png` / `-edge.png` | 당긴 주먹 | 스파이크 백스윙 |
| `cup-hand-g.png` / `-edge.png` | 위로 벌린 컵 손 | 서브 홀드 |
| `platform-g.png` / `-edge.png` | 모은 팔뚝 플랫폼 | 리시브 |

## 2. ControlNet 설정

- **권장: Lineart ControlNet** — 원본 PNG 자체가 선화이므로 lineart 전처리기에 원본 PNG를 그대로 입력 (형태 유지력이 canny보다 좋음)
- 대안: **Canny** — 제공된 `-edge.png`를 전처리 없이 직접 입력
- Control weight `0.85~1.0`, start `0`, end `0.75~0.85` (끝까지 잠그면 질감이 안 살아남)
- 해상도: 출력 1024px 이상, 포즈·크롭은 입력과 동일하게 유지 (CSS 배치가 이 형태 기준으로 튜닝돼 있음)

## 3. 프롬프트 — 트랙 A (애니 작화, 권장)

**공통 스타일 (모든 포즈 앞에 붙임):**
```
clean sports anime illustration of a hand, Haikyuu-style key animation,
anatomically accurate, cel shading with 2-3 flat shade bands,
warm dark-brown lineart (#4E342E), pale cream skin fill,
subtle knuckle and tendon lines, transparent background,
high resolution, crisp clean lines
```

**공통 네거티브:**
```
photo, photorealistic, 3d render, extra fingers, missing fingers,
malformed anatomy, fused fingers, blurry, sketch, rough lines,
background objects, text, watermark
```

**포즈별 서술 (스타일 뒤에 이어 붙임):**
- `open-hand`: back of an open right hand reaching upward, fingers gently together, middle finger longest, thumb angled from the side of the palm
- `spread-hand`: back of a right hand with fingers spread wide, reaching up to meet a volleyball, fingertips slightly curved inward, visible knuckle ridge
- `fist-hand`: a clenched right fist seen from the back-side, cocked for a spike backswing, four curled finger ridges and thumb wrapped diagonally underneath
- `cup-hand-g`: an open left palm facing upward, slightly cupped as if presenting a volleyball for a serve, fingertips relaxed and spread
- `platform-g`: two forearms pressed together forming a volleyball underhand-pass platform, hands clasped with both thumbs on top, seen from the player's own eyes looking down the arms

## 4. 프롬프트 — 트랙 B (실사, 제안서 기반)

**공통 스타일:**
```
realistic close-up photo of a human hand, detailed skin texture,
visible fine skin lines, subtle tendons and knuckles,
detailed fingernails, soft natural studio lighting from upper left,
gentle shadows, plain transparent or cream background, high resolution
```
**공통 네거티브:**
```
drawing, painting, cartoon, illustration, extra fingers,
malformed hand, distorted shape, unrealistic texture, blurry
```
포즈별 서술은 트랙 A와 동일하게 사용.

## 5. 결과물 납품 규격 (통합 조건)

- 포즈·크롭·방향은 입력 PNG와 동일 (CSS 배치·회전이 그 형태에 맞춰져 있음)
- **투명 배경** PNG 또는 WebP, 긴 변 1024px 이상, 파일당 300KB 이하(WebP 권장)
- 5포즈 전부 **같은 피부톤·같은 조명 방향(좌상단 키)** — 세트 일관성이 개별 퀄리티보다 중요
- 파일명은 입력과 동일하게 유지 (`open-hand.webp` 등)
- 결과를 받으면 코드 쪽에서 `<svg>` → `<img>` 교체로 통합한다 (반투명 0.72와 진입 애니메이션은 유지, 실사 트랙이면 불투명도 재검토)

## 6. 상태

- [x] 입력 세트 생성 (2026-08-06)
- [x] **임시 적용: Claude 벡터 셀 셰이딩 v2** (2026-08-06) — 5포즈 전부 음영 면·손톱·관절 디테일 적용해 v4에 통합. AI 생성물이 오면 교체 가능한 구조(`<use href>` 스왑)
- [ ] 트랙 결정 (A/B 시안 비교)
- [ ] 5포즈 AI 생성·선별 — **주의: CCR 세션 환경은 HuggingFace가 네트워크 정책상 차단되어 세션 내 생성 불가.** 로컬 SD/웹 도구(Gemini·ChatGPT 이미지 편집 포함)에서 입력 PNG + 프롬프트로 실행할 것
- [ ] 코드 통합 (AI 결과물 수령 시)
