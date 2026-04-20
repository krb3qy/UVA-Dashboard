# Overnight session notes — 2026-04-20

## What's live

### Repos
- [krb3qy/UVA-Dashboard](https://github.com/krb3qy/UVA-Dashboard) — public (needed for GH Pages on current plan), main branch
- [krb3qy/Alert-Extension-v2](https://github.com/krb3qy/Alert-Extension-v2) — private, forked from Alert-Extension, master branch
- Original `Alert-Extension` untouched

### GitHub Pages
- URL: https://krb3qy.github.io/UVA-Dashboard/
- Enabled after this commit; may take 30–90s for first build to finish
- Check status: `gh api repos/krb3qy/UVA-Dashboard/pages --jq .status`

### Genesys Cloud config (new — nothing existing modified)
- **OAuth client** `UVA Dashboard` — ID `acd9ac35-984e-4f30-bb46-426160f498e0`
  - Grant: CODE (PKCE), scopes: routing conversations user-basic-info notifications analytics:readonly presence users:readonly
  - Redirect URIs: `https://krb3qy.github.io/UVA-Dashboard/` and `/index.html`
  - Secret NOT used client-side (PKCE). Secret is stored in `C:\Users\krb3qy\Documents\projects\_work\oauth_client_response.json` if you need to rotate.
- **Client App integration** `UVA Dashboard` — ID `2e36ec87-b83b-49d1-913d-6f89784c1d44`
  - Type: `embedded-client-app`, displayType: `standalone`
  - URL: `https://krb3qy.github.io/UVA-Dashboard/`
  - **State: DISABLED** on purpose — enable via GC UI or `gc integrations update` once you've sanity-checked
  - Enable command: `gc integrations update 2e36ec87-b83b-49d1-913d-6f89784c1d44` with body `{"intendedState":"ENABLED"}`

## Decisions I made without you

1. **Both new repos created PRIVATE initially, UVA-Dashboard flipped to PUBLIC.** GH Pages on private needs paid plan. Your IW-Merge-Widget / UVA-Bridge are already public and have the same shape of content. If you want private, you'll need to either pay for GitHub Pro or move hosting to Cloudflare Pages / Netlify (both have free private hosting).
2. **Alert-Extension-v2 is PRIVATE.** It's not GH Pages hosted — it's a Chrome extension. No reason to expose.
3. **Client App integration left in DISABLED state.** I didn't want to push an untested URL into the live GC UI for your users.
4. **PKCE client created without PKCE-specific flag.** The OAuthClientRequest enum only had CODE / TOKEN / etc. — no "PKCE-CODE" option. `CODE` grant accepts PKCE-style `code_challenge` + `code_verifier` per GC's docs and this matches how IW-Merge works. If it turns out your tenant enforces confidential CODE clients, we'd need to either use `TOKEN` (implicit) or switch to a PKCE-specific flow if GC exposes one.

## What works today

Phase 1 + 2 from the mapping doc:
- OAuth PKCE flow (copied verbatim from IW-Merge, new clientId)
- Pop-out window with BroadcastChannel handoff (same pattern as IW-Merge)
- WebSocket notification channel + live subscriptions (no polling)
- Two-panel layout: My Interactions (top) + Queue Calls (bottom)
- Queue tab bar; user picks queues from Settings
- Action toolbar + right-click context menu with: Answer, Hold/Unhold, Mute, Disconnect, Transfer, Pick Up, Listen, Coach, Join
- Transfer modal (phone number or user email)
- Duration tick on all rows
- State pills colored by state

## What's NOT done yet (Phase 3+)

- **Workgroup stats sidebar** (the right-side numbers panel from the PureConnect view): needs `v2.analytics.queues.{id}.observations` subscription + renderer
- **Directory/presence panel**: `v2.users.{id}.presence` + `v2.users.{id}.routingStatus`
- **Drag-drop merge** between rows
- **Merge button** (IW-Merge logic not yet ported into this dashboard — still separate widget)
- **Column customization** (PureConnect lets users pick which columns to show)
- **Multi-queue simultaneous view** (split-panel like the PC "horizontal tab" thing) — today it's tabs only
- **"Steal" interaction** (reassign an alerting call from another agent) — the `replace` endpoint supports it but no UI affordance yet

## Before the demo — checklist

1. **Wait for GH Pages build to finish.** Check: `curl -sI https://krb3qy.github.io/UVA-Dashboard/ | head -1` should return `200`.
2. **Enable the Client App integration** in Genesys (UI or CLI). Until enabled, the iframe won't appear in the GC app menu.
3. **Install Alert-Extension-v2 locally** for WebRTC pickup. In `chrome://extensions`:
   - Toggle Developer Mode
   - Load unpacked → `C:\Users\krb3qy\Documents\projects\Alert-Extension-v2\`
   - Copy the extension ID (bottom of the card)
4. **In the dashboard Settings modal**, paste the extension ID. This is what `chrome.runtime.sendMessage` targets. Blank = no WebRTC click injection (desk phone agents work without it).
5. **Pick queues** in Settings → they open as tabs in the bottom panel.
6. **Sanity test**: make a test call into a monitored queue, confirm it appears live, click the row, click Pick Up.

## Known risks / fragile spots

- **PKCE + CODE grant without secret**: if token exchange returns 401 without a client_secret, we may need to switch to `authorizedGrantType=TOKEN` (implicit) or reconfigure the client. IW-Merge works with the same pattern, so this *should* be fine.
- **Scope `users:readonly`** may or may not grant read of other users for the presence/assignment resolver. If the assigned-agent name doesn't render, try adding `users` (without :readonly) — `PUT /api/v2/oauth/clients/{id}` with updated scopes.
- **GH Pages content is public.** OAuth client ID is exposed — that's normal and not a secret. No keys or PHI are in the repo.
- **`displayType: standalone`** makes the dashboard take over the full app area in Genesys. If you wanted a widget/sidebar instead, change via `gc integrations config current update` with `displayType: widget`.
- **Seeding `My Interactions` pulls `/api/v2/conversations`** once on start. After that it's pure WebSocket. If the initial fetch fails (permissions?), the top panel will be empty until a new call comes in.

## If demo day hits and nothing works

Fallback: the original Alert-Extension is still installed and working for the team. That's the safety net you asked for.

## Files created this session
- `C:\Users\krb3qy\Documents\projects\UVA-Dashboard\index.html`
- `C:\Users\krb3qy\Documents\projects\UVA-Dashboard\README.md`
- `C:\Users\krb3qy\Documents\projects\UVA-Dashboard\SESSION_NOTES.md` (this file)
- `C:\Users\krb3qy\Documents\projects\Alert-Extension-v2\` (clone + v2.1.0 bridge)
- `C:\Users\krb3qy\Documents\projects\_work\oauth_client_body.json`, `oauth_client_response.json`, `integration_body.json`, `integration_config.json`, `integration_response.json`, `integration_types.json`
- `C:\Users\krb3qy\Documents\pureconnect-reference\mapping_pureconnect_to_gc.md`, `mapping_sources.md`, `Interaction_Desktop_User_Guide_2023R3.pdf`, `_existing-widgets/` (copies of your public widget repos for reference)
- `C:\Users\krb3qy\Documents\genesys-cloud-reference\platform-client-sdk-cli\` (cloned — your CLAUDE.md referenced this path but it didn't exist before)
