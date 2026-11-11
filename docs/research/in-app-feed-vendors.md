# Research: in-app notification feed vendors

**Ticket:** [#3 Vendor landscape: in-app feed services (Knock vs alternatives)](https://github.com/Framstag/matt-pocock-software-factory/issues/3)
**Map:** [#1 In-app notifications for Cadence](https://github.com/Framstag/matt-pocock-software-factory/issues/1)
**Date:** 2026-11-11

This surfaces facts for the vendor *decision* (ticket #4). It does **not** pick a winner. Scope is the **in-app feed** only (bell + notification center) — Cadence keeps email on its own Postmark mailer, so email/SMS channel quality is not weighed here.

## Evaluation axes

1. **Free tier** — exists, and limits at Cadence's (small) scale.
2. **React Router v8 / SSR fit** — official React SDK, SSR-safe.
3. **Headless / fully-styleable** — can we fully own the feed UI with our own shadcn/ui components, or are we handed a fixed widget? (We require headless.)
4. **EU / GDPR data residency** — EU hosting option.

Plus: feed exposure (component vs headless hooks), real-time mechanism, pricing past the free tier, self-host option.

## Comparison

| Vendor | Free tier | Headless React hooks | SSR fit (RR v8) | EU data residency | Real-time | Paid entry | Self-host |
|---|---|---|---|---|---|---|---|
| **Knock** | ✅ 10,000 messages/mo | ✅ `@knocklabs/react-core` (`useNotifications`, `useKnockFeed`) | ⚠️ feed is client-only; no RR SSR docs | ❓ not stated on public pricing | WebSocket | $250/mo (50k msgs, then $0.005) | ❌ |
| **Courier** | ✅ 10,000 sends/mo | ✅ `@trycourier/react-hooks` `useInbox` + render props | ⚠️ "handles SSR gracefully" per SDK; no RR specifics | ✅ Business tier+ | (inbox SDK) | pay-as-you-go $0.005/send | ❌ |
| **Novu** | ✅ 10,000 workflow runs/mo (5 workflows) | ✅ `@novu/react` `useNotifications`, `useCounts` (+ `@novu/nextjs`) | ⚠️ `@novu/nextjs` handles SSR; plain `@novu/react` for other React | ✅ EU region (cloud) | (inbox) | Pro from $30/mo (30k runs) | ✅ MIT, self-hostable free |
| **MagicBell** | ✅ 1,000 deliveries/mo | ⚠️ inbox SDK; headless not clearly documented on pricing page | ❓ not documented here | ❓ not stated on public pricing | (inbox) | $249/mo (50k deliveries) | ❌ |

Legend: ✅ confirmed · ⚠️ partial / caveated · ❓ not found in sources consulted · ❌ not available.

## Per-vendor notes

### Knock
- **Free tier:** $0/mo, up to **10,000 messages/mo**, no card required. Note Knock's billing counts *each channel send as a message* — an in-app + email fan-out for one event is two messages. Since Cadence sends email via Postmark (not Knock), only the in-app sends count here.
- **Headless:** yes. `@knocklabs/react-core` (~33 KB) exposes `useNotifications` + `useNotificationStore` / `useKnockFeed()` → `{ feedClient, useFeedStore }`, giving items, counts and `mark*`/`fetch*` methods. You can build the list/popover/paging with your own shadcn/ui components. `@knocklabs/react` ships pre-built components if wanted (CSS ~13 KB).
- **SSR:** the browser reads the feed directly from Knock over a **WebSocket** with a public key; the server triggers workflows and (in enhanced-security mode) signs a per-user token. The feed never fetches during SSR — `KnockProvider` builds the client and authenticates during render, so you render the Knock subtree **client-only**. **Server rendering is not documented by Knock for React Router** — client-only mount is the known-good pattern.
- **EU residency:** not mentioned on the public pricing page (SSO/SCIM are Enterprise-only). Needs direct confirmation with Knock if residency is a hard gate.
- **Paid:** Starter **$250/mo** (50k messages, then $0.005/ea); Enterprise custom.

### Courier
- **Free tier:** Developer plan $0/mo, **10,000 sends/mo**.
- **Headless:** yes. **Courier Inbox** ships `@trycourier/courier-react` with a `useInbox` hook that works **without any of the React components** (true headless), plus render props to override individual sub-components. Explicitly marketed as "built to be customized."
- **SSR:** SDK described as handling SSR "gracefully" with no DOM access during server render; no React-Router-specific guidance found.
- **EU residency:** ✅ listed as a feature on **Business tier and above**.
- **Paid:** Business is pay-as-you-go **$0.005/send** beyond the free tier (no $250 floor); Enterprise custom.

### Novu
- **Free tier:** $0/mo, **10,000 workflow runs/mo**, but capped at **5 workflows / 2 environments / 24-hour activity retention / 3 team members**. Watch the 5-workflow cap against a larger notification catalogue.
- **Headless:** yes. `@novu/react` ships both a prebuilt `<Inbox />` *and* headless hooks (`useNotifications`, `useCounts`, `usePreferences`); `@novu/js` is framework-agnostic.
- **SSR:** `@novu/nextjs` handles SSR edge cases; `@novu/react` is for other React apps. React Router isn't called out, but the headless hooks are client-side and mount cleanly.
- **EU residency:** ✅ **EU region** offered (alongside US, UK, Singapore, Australia, Japan, South Korea).
- **Self-host:** ✅ core is **MIT-licensed and self-hostable for free** (API, workflow engine, React Inbox). Self-hosting needs MongoDB + Redis + S3; Community edition drops branding removal and email tracking.
- **Paid:** Pro from **$30/mo** (30k runs, +$1.20/1k); Team from $250/mo.

### MagicBell
- **Free tier:** Builder plan $0/mo, **1,000 deliveries/mo** (the smallest free allowance here), 1 project, unlimited users.
- **Headless:** inbox SDK exists; the pricing page does not document headless/full customization. Needs a docs check before relying on it.
- **EU residency:** not mentioned on the pricing page.
- **Sunset risk:** **No** — despite chatter that users migrate off it because "in-app inbox alone isn't enough," MagicBell explicitly states no sunset, and continues to operate. (The "going away Oct 2026" noise refers to **AWS Pinpoint**, a different product.)
- **Paid:** Startup **$249/mo** (50k deliveries, +$0.0025/ea); Enterprise custom.

## Cross-cutting observations (not decisions)

- **All four clear the headless + free-tier bars** in principle, except MagicBell where headless is undocumented and the free allowance (1k) is an order of magnitude smaller than the others' (10k).
- **EU/GDPR residency is a real differentiator:** only **Novu** (EU region) and **Courier** (Business tier+) publicly offer it. Knock and MagicBell don't state it publicly — if residency is a **hard gate**, those two need direct confirmation or drop out.
- **SSR is a non-issue for the feed itself** across all vendors: the feed is a client-side, real-time (WebSocket/poll) surface that shouldn't render during SSR. The React Router v8 pattern is to mount the provider/feed **client-only**. None document React Router explicitly.
- **Novu is the only self-host option** — relevant if data control or avoiding per-message pricing matters, at the cost of running MongoDB/Redis/S3.
- **Pricing shape differs:** Courier and Knock bill per send ($0.005); Courier has no monthly floor while Knock's next tier jumps to $250/mo. Novu bills per workflow *run* (cheaper entry at $30/mo) but the free tier's 5-workflow cap is the binding constraint for a broad catalogue.

## Sources

- Knock pricing — https://knock.app/pricing
- Knock in-app channel — https://knock.app/channels/in-app-notifications
- Knock React (`@knocklabs/react-core`) — https://github.com/knocklabs/javascript/tree/main/packages/react
- Novu pricing — https://novu.co/pricing
- Novu headless mode — https://docs.novu.co/platform/inbox/headless-mode
- Novu platform overview — https://docs.novu.co/platform/overview
- Novu inbox — https://novu.co/inbox/
- Courier pricing — https://www.courier.com/pricing
- Courier Inbox overview — https://www.courier.com/docs/platform/inbox/inbox-overview
- Courier react-inbox README — https://github.com/trycourier/courier-react/blob/main/packages/react-inbox/README.md
- MagicBell pricing — https://www.magicbell.com/pricing
- MagicBell alternatives / status context — https://www.suprsend.com/post/magicbell-alternatives
