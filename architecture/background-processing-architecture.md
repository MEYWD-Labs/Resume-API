# Background Processing Architecture

## Queue-Based Job Processing

```
┌─────────────────────────────────────────────────────────────┐
│              Background Job Processing Flow                  │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│ Job Types & Queues                                          │
│ ─────────────────                                          │
│                                                              │
│ PDF Generation Queue                                        │
│ ├─ Priority: High                                          │
│ ├─ Retry: 3 attempts                                       │
│ ├─ Timeout: 30 seconds                                     │
│ └─ Dead Letter Queue: pdf_generation_dlq                   │
│                                                             │
│ AI Enhancement Queue                                       │
│ ├─ Priority: Medium                                        │
│ ├─ Retry: 2 attempts                                      │
│ ├─ Timeout: 60 seconds                                    │
│ └─ Rate Limited: 10 req/second                            │
│                                                            │
│ Email Notification Queue                                  │
│ ├─ Priority: Low                                          │
│ ├─ Retry: 5 attempts (exponential backoff)               │
│ ├─ Timeout: 10 seconds                                    │
│ └─ Batch Processing: up to 100 emails                    │
│                                                           │
│ Data Sync Queue                                          │
│ ├─ Priority: Low                                         │
│ ├─ Retry: Infinite (with backoff)                       │
│ ├─ Timeout: 120 seconds                                  │
│ └─ Deduplication: Based on entity ID                     │
│                                                           │
│ Analytics Processing Queue                               │
│ ├─ Priority: Background                                  │
│ ├─ Schedule: Every 5 minutes                             │
│ ├─ Batch Size: 1000 events                               │
│ └─ Storage: Workers Analytics API                        │
│                                                           │
│ Job Processing Pipeline                                  │
│ ───────────────────────                                 │
│                                                           │
│     Resume-API                                           │
│         │                                                 │
│         ▼                                                 │
│   Create Job ──────► Cloudflare Queue                   │
│         │                    │                           │
│    Return Job ID            │                           │
│         │                    ▼                           │
│         │            Resume-Processor                    │
│         │           (Queue Consumer)                     │
│         │                    │                           │
│         │            ┌───────┴────────┐                 │
│         │            │                 │                 │
│         │      Process Job      Handle Error            │
│         │            │                 │                 │
│         │            ▼                 ▼                 │
│         │      Update Status      Retry/DLQ             │
│         │            │                                   │
│         │            ▼                                   │
│         └────── Notification                            │
│                (Email/WebSocket)                         │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

## Job Status Tracking

```sql
-- Job Status Table Structure
CREATE TABLE job_status (
  id TEXT PRIMARY KEY,
  type TEXT NOT NULL,
  status TEXT NOT NULL, -- pending, processing, completed, failed
  user_id TEXT NOT NULL,
  payload JSON,
  result JSON,
  error TEXT,
  retry_count INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  completed_at TIMESTAMP
);

-- Job Status Flow
pending → processing → completed
                    ↘
                      failed → retry → processing
                            ↘
                              dead_letter_queue
```
