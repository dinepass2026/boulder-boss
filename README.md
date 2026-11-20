# Boulder Boss ยิมปีนผา — Phase 1 (frontend, mock data)

Mobile-first web game, Thai UI, Bai Jamjuree (self-hosted via `@fontsource`). React 18 + TypeScript + Vite, HashRouter (deploys to Vercel as a static site, no rewrites needed).

```bash
npm install
npm run dev          # http://localhost:5173  → open /#/dev for the screen index
npm run build        # dist/ (Vercel: framework "Vite", output "dist")
npm run build:single # dist-single/index.html — one self-contained file for previews
```

## Layout model
Every screen is the exact 390×844 artboard from the design canvas. `src/app/Stage.tsx` scales that artboard to fit the viewport (scale = 1 on a 390×844 phone, letterboxed on paper colour elsewhere, framed on desktop). Positions, sizes, colours and type match the handoff 1:1.

## Structure
```
src/
  components/   Chibi · Skyline · Wall · BottomNav   ← reusable, typed, hand-written
  screens/      A01…J59 (32 Phase-1 screens)        ← generated from handoff, see below
  app/          App (router) · Stage · Toast · routes.tsx (route table) · DevIndex (/dev)
  lib/          tokens.ts (design tokens + grade colours) · dc.ts (useDC, useGo, toast)
  styles.css    CSS variables, pressed state (translateY 4px, shadow 0), focus ring
tools/design-import/   dc2tsx.py converter + nav maps + patches
```

### Components
- `<Chibi pose skin hair shirt pants shoes bag band w />` — pose: stand | climb | wave | cheer | fall; height = w × 4/3.
- `<Skyline time="auto|dawn|day|dusk|night" w h />` — `auto` follows the device clock (dawn 5–8, day 8–16, dusk 16–19, night) and re-checks every minute.
- `<Wall mode="climb|preview|plain" climberTop pose route others reach onHoldTap />` — 390×620; route/others are `Hold[]` (`MOCK_ROUTE`, `MOCK_OTHER_HOLDS` exported). In climb mode reachable holds get the dashed yellow ring; with `onHoldTap` they become tappable.
- `<BottomNav active="map|gym|club|shop|profile" />` — player role only. Shop is shown disabled (revenue hidden); Club shows a "Phase 3" toast.

### Screens & routes
| Route | Screen |
|---|---|
| `/` → auto `/welcome` | A01 Splash |
| `/welcome` `/login` `/forgot` `/register` `/verify` | A02–A06 |
| `/player-info` `/avatar` `/tutorial` `/link-guest` | A07–A10 |
| `/map` `/zone` `/gym` `/wall` `/routes` `/route` `/beta` | B11, B12, B13, B13b, B14, B15, B15b |
| `/climb` `/move/:move` (dyno, heelhook) `/pause` `/fall` `/result/:result` (send, flash, project, speed) | C16–C20 |
| `/profile` `/profile/edit` `/logbook` `/wardrobe` `/achievements` `/player` | D21–D26 |
| `/notifications` `/settings` `/legal` `/system/:state` (offline, maintenance, update) | J56–J59 |
| `/dev` | screen index for the team |

Mock flow: Welcome → Guest → Player info → Avatar → Tutorial → Map → Sukhumvit → Crimp Factory → routes → ข้าวเหนียวมะม่วง → เริ่มปีน → tap a ringed hold → Dyno → กระโดด! (70% Send / 30% Fall).
Buttons for later-phase features (Lobby, Boss Wall, Speed Run, DMs, friends, club) show a "เร็วๆ นี้" toast.

Not in Phase 1: B13c gym floor, B15c/C16b speed run, E/F/G/I sections, and everything in `_hidden-revenue/` (H46–H52).

## Regenerating screens from a new handoff
```bash
sh tools/design-import/import.sh ../boulder-boss-handoff
```
`dc2tsx.py` converts `.dc.html` markup to JSX (inline styles → style objects, `<sc-for>` → `.map`, `<sc-if>` → conditional, `<dc-import>` → component) and ports the `DCLogic.renderVals` view-model as a plain function (`useDC` provides props/state/setState). Button wiring lives in `nav/<screen>.json`; hand edits live in `patches.py` so they survive regeneration.

## Next (per HANDOFF build order)
1. Climb gameplay: Canvas 2D wall, stamina/grip, dyno timing + heel-hook drag mini-games (replace C16/C17 mock taps).
2. Supabase: auth (email/LINE/Google/Apple, guest → link), routes, logbook, realtime presence (gym floor, lobby chat), friends, DMs, speed-run records. Move mock data out of each screen's `renderVals` into a data layer at that point.
