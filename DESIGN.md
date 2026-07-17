# DESIGN.md — FOMO.EXE (WebGL 태양계)

## 0. Research Log

| Lane | Deliverable |
|---|---|
| Embedded references | Loaded Layer A `gpt-tasteskill.md` (Awwwards-tier cinematic scroll) — picked for "개FOMO / 쌈뽕 / 압도적" brief. Layer B brand skipped: brief is a WebGL art piece, not a brand site. |
| Static visual reference | **User-supplied**: `~/Pictures/Screenshots/Screenshot From 2026-07-17 21-25-55.png` — purple emission-nebula photo. This is the **visual contract** for the space backdrop: violet→magenta→pink cloud ramp, near-black dust lanes, cyan-white star cores with cross diffraction spikes, warm orange pin-stars. |
| Imagen concept drafts | Skipped — user supplied the reference directly; generating drafts would dilute the contract. |
| Lazyweb screens | Skipped — reference already fixes the direction. |

**Direction lock**: "허블 망원경이 본 복숭아빛 없는 딥 퍼플 성운 위에, HDR로 타는 태양." 시그니처 머티리얼 = **HDR light** (ACES tonemap + tight bloom + anamorphic streak). 기억에 남는 순간 = 스크롤로 소행성 벨트 한가울레를 관통할 때 60,000 인스턴스가 사방에서 스쳐가는 것.

## 1. Color Tokens (reference-extracted)

| Token | Hex | Use |
|---|---|---|
| `space-void` | `#05010d` | scene background, dust-lane shadow |
| `nebula-deep` | `#1a0533` | nebula base |
| `nebula-violet` | `#3b0a5e` | cloud mid |
| `nebula-magenta` | `#a13ec2` | cloud bright |
| `nebula-pink` | `#e08ae0` | cloud rim / highlights |
| `star-cyan` | `#c8fbff` | hot star cores (HDR push → bloom) |
| `star-warm` | `#ffd9a0` | warm pin-stars |
| `sun-core` | `#fff3c4` | sun HDR hot spots |
| `sun-edge` | `#ff7a1a` | sun rim / corona |
| `ink` | `#eef2ff` | primary text |
| `accent` | `#ffb347` | kickers, HUD accents |
| `accent2` | `#6ee7ff` | stats, links, tech highlights |

Gradient rule: nebula ramp는 `deep→violet→magenta→pink` 4-stop perceptual ramp. 한 개 복숭아/주황 성운 금지 — 참고 이미지는 별만 웜톤.

## 2. Typography

- Display: **Unbounded** 900 (hero mega, stats) — never Inter.
- Body: **Noto Sans KR** 300/500/700.
- Mono (HUD/kicker): **JetBrains Mono** 400/700, letter-spacing .2em+.
- H1 ≤ 3 lines at `clamp(44px, 11vw, 150px)`, ultra-wide measure.

## 3. Materials / Primitives

- `nebula-dome`: BackSide sphere, domain-warped 4-octave FBM + ridged dust lanes + tilted milky-way band. Linear HDR, values < 0.8 (bloom 아래).
- `sun-plasma`: 5-octave FBM ×2 domain warp, HDR ×2.2 → bloom이 물 대상.
- `star-field`: 150k GPU points, 트윙클 + 상위 3% 별에만 십자 회절 스파이크(참고 이미지의 시안 코어).
- `atmo-fresnel`: BackSide additive fresnel glow. **음수 밑 pow 금지(NaN 깜빡임)** — clamp 필수.
- `god-ray`: 스크린스페이스 56탭 래디얼 블러. 태양 스크린 투영 좌표 + facing² × 거리 감쇠 가중치. lum>0.78 픽셀만 발광 → 행성 차광 자연 발생.
- `cursor-lens`: 파이널 패스에서 커서 주변 UV를 중력렌즈처럼 휨 (`exp(-md²·14)`). 마우스 속도로 강도.
- `warp`: 스크롤 속도 → 줌 블러 8탭 + FOV 58→74 킥.
- `planet-label`: 3D 프로젝션 추적 DOM 라벨, mono 10px, 거리/시야각 페이드.
- `audio`: 프로시저럴 WebAudio — 디튠 쏘우 드론 + 브라운 노이즈 밴드패스(스크롤 스윕) + 섹션 전환 서브베이스 쿵. 제스처 게이트(포인터다운/키), SOUND 토글.
- `hud`: mono 텍스트 + 스파크라인, `rgba(238,242,255,.75)` on transparent.

## 4. Post Pipeline (order matters)

RenderPass(Linear HDR) → UnrealBloom(strength .6, radius .5, **threshold 1.0** — HDR만 번진다) → GodRay(56탭 스크린스페이스 샤프트) → **OutputPass(ACES + sRGB)** → FinalPass(CA + vignette + film grain + grade: sat ×1.14, contrast ×1.06, violet shadow lift). 톤맵은 OutputPass에서만 — 이중 적용 금지.

## 5. Motion

- Scroll choreography: 8 camera keyframes, smoothstep easing, spring smoothing (`1-exp(-dt*4.2)`).
- Mouse parallax: camera offset ≤ ±3.2, spring `dt*5`. GPU-composited CSS only (`transform`, `opacity`).
- Warp: scroll velocity → zoom blur + FOV kick (가속 페달). Lens: mouse velocity → 중력 렌징 강도.
- Panels: fade+translateY 40px + 글자 단위 키네틱 스태거(26ms 간격, cubic-bezier(.19,1,.22,1)), progress-bar width. Slop animation 금지 — 모든 움직임은 스크롤/포인터 상태의 함수.

## 6. Accessibility / Accepted Debt

- `cursor: none`는 `@media (hover: none)`에서 자동 해제.
- Adaptive DPR(0.85–2.0)로 47fps 미만 시 해상도 방어.
- **Debt**: Lighthouse 감사 생략 — 풀스크린 WebGL 아트 페이지에서 카테고리 점수는 무의미. 퍼프 예산은 draw calls < 40, tris < 2.5M으로 HUD가 자체 표시.
- **Debt**: WebGL1 폰백 없음 — WebGL2 없으면 fallback 메시지 표시(의도된 거절).
