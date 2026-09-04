---
name: realtime-refresh
description: "How Darwin pushes live data-refresh signals to open pages (Channels-based, no polling)"
metadata: 
  node_type: memory
  type: project
  originSessionId: 3bbfd9a8-7623-44d0-a149-43aa9734e9a1
  modified: 2026-09-01T17:29:23.313Z
---

Added 2026-09-01. Pages that showed stale data until F5 (kanban boards, calendário,
boards list) now refresh themselves via a lightweight change-signal system built on
the existing Django Channels socket.

**Backend**
- `core/realtime.py` — `notify(topics, resource, *, id, action, actor_id, on_commit=True)`
  fans out `{type:"resource_changed", scope, resource, id, action, actor_id}` to
  Channels groups. Topics: `"user:<id>"` → `events_user_<id>`, `"board:<id>"` →
  `events_board_<id>`, `"global"` → `events_global`. `scope` = the board topic if
  present, else the resource name (that's the key the client refetches on).
- `BriefingConsumer` (`apps/chat/consumers.py`, `/ws/briefing/`) is the one always-on
  socket. Joins `events_user_<id>` + `events_global` on connect; client sends
  `{action:"subscribe"|"unsubscribe", topic:"board:<id>"}` to (un)watch a board,
  membership-checked. Handler `resource_changed`.
- Signals emit the notifications: `apps/followups/signals.py` (FollowUp, Board,
  BoardMember, FollowupTag, CardImagem, m2m), `apps/calendario/signals.py`
  (CalendarEvent → `global`/`calendario`).

**Frontend**
- `store/realtimeStore.js` — `{connected, ticks:{[scope]:n}}`, `bump(scope)`.
- `hooks/useGlobalWebSocket.js` — extended: routes `resource_changed` → `bump`,
  exports `subscribeTopic`/`unsubscribeTopic` (refcounted) for the shared socket.
- `hooks/useAutoRefresh(scope, refetch, {throttleMs, canRefetch, enabled, onFocus})`
  — refetches when the scope ticks; `canRefetch` defers (mid-drag / modal open),
  returned `flush()` releases it. Wired into `pages/Followups.jsx` (`board:<id>`),
  `pages/BoardsList.jsx` (`boards`), `pages/Calendario.jsx` (`calendario`).

Admin usage/costs deliberately NOT on realtime (high-volume rows) — left to
focus-refresh only. Dev `runserver` serves the WS and autoreloads.

Related: [[playwright_verification_notes]]
