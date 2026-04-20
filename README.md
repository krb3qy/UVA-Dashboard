# UVA Dashboard

Genesys Cloud queue-monitoring dashboard that replicates the PureConnect
Interaction Desktop view used by the transfer center.

## What it is

- Single-file HTML app, hosted on GitHub Pages
- Iframed into Genesys via a Client App integration (ID `2e36ec87-b83b-49d1-913d-6f89784c1d44`)
- Pop-out window supported (BroadcastChannel-coordinated, same pattern as IW-Merge widget)
- Auth: OAuth PKCE against the `UVA Dashboard` client (ID `acd9ac35-984e-4f30-bb46-426160f498e0`)
- Data: WebSocket notifications (no polling) — subscribes to:
  - `v2.users.{me}.conversations.calls`
  - `v2.routing.queues.{queueId}.conversations.calls` per monitored queue

## Panels

- **Top — My Interactions**: calls currently assigned to the signed-in user
- **Bottom — Queue Calls**: tabs for each monitored queue; one click shows that queue's active calls

## Actions

| Action | Where | Endpoint |
|---|---|---|
| Answer | My | `PATCH participants state=connected` + extension bridge for WebRTC |
| Hold / Unhold | My | `PATCH participants held=true/false` |
| Mute | My | `PATCH participants muted=true` |
| Disconnect | My | `PATCH participants state=disconnected` |
| Transfer | My | `POST participants/{id}/replace` |
| Pick Up | Queue | `POST participants/{id}/replace` with `userId=me` + extension bridge |
| Listen | Queue | `POST participants/{id}/monitor` |
| Coach | Queue | `POST participants/{id}/coach` |
| Join (Barge) | Queue | `POST participants/{id}/barge` |

## WebRTC pickup bridge

For agents using the in-browser GC softphone, pickup needs a DOM-level click on the
native Genesys answer button (browser WebRTC gesture requirement — cross-origin
iframes can't do this themselves). This dashboard proxies those clicks through
`Alert-Extension-v2` via `chrome.runtime.sendMessage`. Set the extension ID in
Settings; leave blank if all agents use desk phones.

## Configuring queues

Click the ⚙ (gear) icon in the header → pick queues to monitor. Selections persist
in localStorage and open as tabs in the bottom panel.

## Related repos

- [Alert-Extension](https://github.com/krb3qy/Alert-Extension) — original alert extension (unchanged)
- [Alert-Extension-v2](https://github.com/krb3qy/Alert-Extension-v2) — same as v1 plus an `externally_connectable` bridge so this dashboard can trigger the WebRTC answer click
- [IW-Merge-Widget](https://github.com/krb3qy/IW-Merge-Widget) — standalone merge widget (scope precursor to this dashboard)
