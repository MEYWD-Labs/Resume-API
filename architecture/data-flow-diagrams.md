# Data Flow Diagrams

## 1. User Registration Flow

```
User → Resume-UI → Resume-SSO → Resume-Account-API → D1 Database
                        │              │
                        │              └→ KV (session)
                        └→ Email Worker (verification)

Sequence:
1. User submits registration form (Resume-UI)
2. Resume-UI sends POST /auth/register to Resume-SSO
3. Resume-SSO validates and creates user in D1
4. Resume-SSO triggers Resume-Account-API to create account profile
5. Resume-Account-API stores profile in D1, session in KV
6. Resume-SSO sends verification email via Email Worker
7. JWT tokens returned to Resume-UI
```

## 2. Resume Creation & Editing Flow

```
┌─────────┐     ┌──────────┐     ┌────────────┐     ┌────────────┐
│  User   │────▶│Resume-UI │────▶│Resume-API  │────▶│D1 Database │
└─────────┘     └────┬─────┘     └─────┬──────┘     └────────────┘
                     │                  │
                     │                  ├──────────▶ R2 (images)
                     │                  │
                     │                  └──────────▶ Durable Objects
                     │                               (real-time collab)
                     └─────────────────────────────▶ WebSocket

Real-time Collaboration Flow:
1. User A opens resume for editing
2. Resume-UI establishes WebSocket to Durable Object via Resume-API
3. User B joins same resume
4. Both users connected to same Durable Object instance
5. Changes broadcast to all connected clients in real-time
6. Periodic saves to D1 database
```

## 3. PDF Generation Flow

```
User Request → Resume-UI → Resume-API → Queue → Resume-Processor
                              │                        │
                              └→ Job ID                ├→ D1 (template)
                                                       ├→ Workers AI
                                                       └→ R2 (PDF storage)
                                                           │
                                                           ▼
                              Email Worker ← Webhook ← Complete

Steps:
1. User clicks "Export PDF" in Resume-UI
2. Resume-API creates job in Cloudflare Queue
3. Job ID returned immediately to user
4. Resume-Processor picks up job from queue
5. Processor fetches resume data from D1
6. Generates PDF using template engine
7. Stores PDF in R2 storage
8. Updates job status in D1
9. Sends completion notification via Email Worker
10. Resume-UI polls or receives WebSocket notification
```

## 4. Admin Dashboard Flow

```
┌─────────────┐     ┌──────────────┐     ┌──────────────────┐
│Admin User   │────▶│Resume-Admin  │────▶│Resume-Admin-API  │
└─────────────┘     └──────────────┘     └────────┬─────────┘
                                                   │
                            ┌──────────────────────┼──────────────────────┐
                            ▼                      ▼                      ▼
                        D1 Database            KV Store            Workers Analytics
                        (user data)            (metrics)              (usage)

Admin Operations:
- View all users and resumes
- Manage subscriptions and billing
- Access analytics and metrics
- Moderate content
- System configuration
```

## 5. Mobile Offline Sync Flow

```
┌─────────────────┐     Local Storage     ┌─────────────────┐
│Resume-Mobile-UI │◄───────────────────────│  AsyncStorage   │
└────────┬────────┘                        └─────────────────┘
         │                                           ▲
         │ Online                                    │
         ▼                                           │
   ┌──────────┐     Sync Queue     ┌────────────────┤
   │Resume-API│◄────────────────────│  Sync Manager  │
   └────┬─────┘                     └────────────────┘
        │
        ▼
   D1 Database

Offline Sync Process:
1. User edits resume offline in mobile app
2. Changes stored in AsyncStorage with timestamps
3. App maintains sync queue of pending operations
4. When online, Sync Manager processes queue
5. Conflict resolution based on timestamps
6. Server acknowledges successful sync
7. Local storage updated with server state
```
