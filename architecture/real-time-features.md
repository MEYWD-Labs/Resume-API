# Real-time Features

## WebSocket & Durable Objects Architecture

```
┌─────────────────────────────────────────────────────────────┐
│            Real-time Collaboration Architecture              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│ Durable Object Classes                                      │
│ ───────────────────────                                    │
│                                                              │
│ ResumeCollaborationRoom                                    │
│ ├─ Manages WebSocket connections for resume editing        │
│ ├─ Broadcasts changes to all connected clients             │
│ ├─ Maintains operation history                             │
│ ├─ Handles conflict resolution (CRDT-based)               │
│ └─ Periodic snapshot to D1                                 │
│                                                             │
│ UserPresenceTracker                                       │
│ ├─ Tracks online users per resume                          │
│ ├─ Manages cursor positions                                │
│ ├─ Handles user avatar/color assignment                    │
│ └─ Cleanup on disconnect                                   │
│                                                             │
│ NotificationHub                                           │
│ ├─ Real-time notification delivery                         │
│ ├─ Manages user subscription to events                     │
│ ├─ Priority queue for notifications                        │
│ └─ Fallback to email for offline users                     │
│                                                             │
│ WebSocket Connection Flow                                 │
│ ──────────────────────────                               │
│                                                            │
│ Client                Resume-API            Durable Object│
│   │                        │                      │        │
│   ├──WS Connect──────────► │                      │        │
│   │                        ├──Get/Create DO────► │        │
│   │                        │                      │        │
│   │◄──WS Upgrade────────── │◄──DO Instance────── │        │
│   │                        │                      │        │
│   ├──Send Operation──────────────────────────────►│        │
│   │                                               │        │
│   │◄──Broadcast Change─────────────────────────── │        │
│   │                                               │        │
│   ├──Heartbeat─────────────────────────────────► │        │
│   │                                               │        │
│   │◄──Presence Update───────────────────────────  │        │
│                                                            │
│ Real-time Event Types                                     │
│ ──────────────────                                       │
│                                                            │
│ Document Events:                                          │
│ - document.update                                         │
│ - document.save                                           │
│ - document.delete                                         │
│                                                            │
│ Collaboration Events:                                     │
│ - user.join                                               │
│ - user.leave                                              │
│ - cursor.move                                             │
│ - selection.change                                        │
│                                                            │
│ System Events:                                            │
│ - connection.established                                  │
│ - connection.lost                                         │
│ - sync.required                                           │
│ - conflict.detected                                       │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

## Conflict Resolution Strategy

```
Conflict Resolution Algorithm (CRDT-based):
─────────────────────────────────────────

1. Each operation has a unique ID (user_id + timestamp + counter)
2. Operations are ordered by timestamp, then by user_id
3. Concurrent operations are transformed against each other
4. Result is deterministic across all clients

Example:
User A: Insert "Hello" at position 0 at time T1
User B: Insert "World" at position 0 at time T2

Resolution:
- If T1 < T2: Result is "HelloWorld"
- If T2 < T1: Result is "WorldHello"
- If T1 == T2: Order by user_id
```
