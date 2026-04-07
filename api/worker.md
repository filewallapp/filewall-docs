# 0. Mental Model (READ THIS FIRST)

This is a **distributed job processing system**:

```
Client / LocalTest
        ↓
   enqueueFile()
        ↓
   Redis (metadata + queue)
        ↓
   BullMQ Queues
        ↓
   Workers (parallel)
        ↓
   Processing (FFmpeg / Sharp)
        ↓
   Upload (local / R2)
        ↓
   Redis updated → Admin API → UI
```

---

# 1. CORE CONCEPTS (CRITICAL)

## 1.1 TTL (Time-To-Live)

TTL = expiry time in Redis.

Used in:

* `job metadata`
* `logs`
* `locks`

### Why?

Without TTL:

* Redis grows forever → memory leak
* stale jobs remain forever

### Example

```ts
await connection.expire(jobKey, 86400)
```

→ delete job after 24h

---

## 1.2 Lock (Idempotency Lock)

```
lock:{jobId}
```

### Purpose:

Prevent **duplicate processing**

### Why needed?

BullMQ can:

* retry jobs
* crash + restart workers

→ same job can run twice

### Fix:

```ts
SET lockKey WORKER_ID PX TTL NX
```

* NX = only set if not exists
* PX = expiry

If lock exists → skip job

---

## 1.3 Heartbeat

Runs every 60s:

```ts
setInterval(() => {
  extend lock TTL
}, 60s)
```

### Why?

If job runs long:

* lock expires
* another worker starts same job → duplication

Heartbeat prevents that.

---

## 1.4 Queue System (BullMQ)

Queues:

* small-files
* medium-files
* large-files
* image-files

Each has:

* different concurrency
* different cost

---

## 1.5 DLQ (Dead Letter Queue)

```
dead-letter-queue
```

Stores:

* permanently failed jobs

Why?

* debugging
* retry manually

---

# 2. FILE-BY-FILE EXPLANATION

---

# src/config.ts

### Purpose:

Central config system

---

### validateEnv()

```ts
if (value === undefined || value === '')
```

Why:

* prevents silent misconfig
* fail fast

---

### config object

Key parts:

#### mode

```ts
mode: process.env.MODE
```

* `local` → test mode
* `server` → production

---

#### redis

```ts
host + port
```

Used by BullMQ + metadata

---

#### ffmpegPath

Allows:

* system ffmpeg
* custom binary

---

#### tempDir

```ts
os.tmpdir()
```

Used for:

* downloads
* processing

---

#### concurrency

Controls:

* parallel jobs per worker

---

#### rateLimit

Currently unused, but for API throttling

---

#### disk thresholds

```ts
minFreeBytes
targetFreeBytes
```

Used to:

* prevent disk crash

---

### Directory creation

```ts
if (!exists) mkdir
```

Why:
Avoid runtime crash when writing files

---

---

# src/constants.ts

Pure constants.

---

### JOB_STATUS

Backend state machine:

```
queued → processing → uploading → completed
```

---

### JOB_STAGE

More granular:

```
downloading, processing, uploading
```

Used for UI.

---

### REDIS_KEYS

Important pattern:

```ts
job:{id}
job:{id}:logs
lock:{id}
```

---

---

# src/index.ts (ENTRY POINT)

---

### dotenv

```ts
dotenv.config()
```

Loads `.env.local`

---

### Import workers

```ts
import './worker/fastWorker';
```

Why:
Just importing → starts workers

---

### SESSION_ID

```ts
process.env.SESSION_ID = Date.now()
```

Used to isolate runs.

---

### Mode switch

```ts
if server → start API
else → localTest
```

---

### Background jobs

#### 1. Stuck job recovery

Runs every 60s:

```ts
recoverStuckJobs()
```

---

#### 2. Disk cleanup

```ts
if free < threshold → cleanup
```

---

---

# src/localTest.ts

Simulates real usage.

---

### initialScan()

Reads test folder → enqueue files

---

### startPolling()

Every 3s:

* detects new files

---

### enqueueSafe()

Steps:

1. detect file type
2. build job object
3. call enqueueFile()

---

---

# src/queue/enqueue.ts (VERY IMPORTANT)

This is the **entry to system**

---

### classify()

```ts
<100MB → small
<500MB → medium
else → large
```

---

### basePriority()

Combines:

* user tier
* file size

Lower number = higher priority

---

### getPriority()

Adds **aging**

```ts
priority = base - waitingTime
```

Why:
Prevents starvation

---

### enqueueFile()

#### Step 1: read existing job

```ts
hgetall(job)
```

---

#### Step 2: store metadata

```ts
hset(job:{id}, {...})
```

This is **source of truth for UI**

---

#### Step 3: TTL

```ts
expire 24h
```

---

#### Step 4: choose queue

```ts
image → imageQueue
small → smallQueue
```

---

#### Step 5: add to BullMQ

```ts
queue.add()
```

Options:

* priority
* retries
* exponential backoff

---

---

# src/worker/handler.ts (CORE ENGINE)

This is the **most critical file**

---

## FLOW:

---

### STEP 1: LOCK

```ts
SET lock NX
```

If fail → exit

---

### STEP 2: CREATE TEMP DIR

```ts
/tmp/{jobId}
```

---

### STEP 3: DOWNLOAD

```ts
download(input → local file)
```

---

### STEP 4: COMPUTE TTL

Based on:

* file size
* video duration

---

### STEP 5: HEARTBEAT

Keeps lock alive

---

### STEP 6: VALIDATE DISK

```ts
if free < required → fail
```

---

### STEP 7: PROCESS

#### IMAGE:

```ts
sharp → resize + watermark
```

#### VIDEO:

Uses FFmpeg → HLS streaming

---

## VIDEO PIPELINE (IMPORTANT)

Instead of single file:

```
.m3u8 (playlist)
.ts files (chunks)
```

Why?

* progressive playback
* streaming support

---

### WATCHER

```ts
fs.watch(hlsDir)
```

Uploads files as they appear

---

### FALLBACK SCAN

Why:
fs.watch is unreliable

---

### PROGRESS TRACKING

From FFmpeg:

```
out_time_ms
```

Converted to %

---

---

### STEP 8: UPLOAD

* images → upload file
* video → already uploaded via chunks

---

### STEP 9: COMPLETE

```ts
status = completed
```

---

### STEP 10: ERROR HANDLING

If fail:

#### Case 1: retryable

→ BullMQ retry

#### Case 2: permanent

→ DLQ

---

### Poison job detection

```ts
same error repeated → stop retry
```

---

### STEP 11: CLEANUP

* success → delete temp
* fail → delay delete

---

### STEP 12: RELEASE LOCK

```ts
DEL lockKey
```

---

---

# Workers

---

## fastWorker.ts

Small files:

```ts
concurrency: 5
```

---

## standardWorker.ts

Medium:

```ts
concurrency: 3
```

---

## heavyWorker.ts

Large:

```ts
concurrency: 1
```

---

## imageWorker.ts

Images:

```ts
concurrency: 15
```

---

# Why separate workers?

Prevents:

* large jobs blocking small jobs

---

---

# src/server/admin.ts

Provides **system visibility**

---

### getSystemSnapshot()

Does:

1. scan Redis
2. fetch all jobs
3. merge:

   * Redis state
   * BullMQ state

---

### queue position

```ts
queue.getWaiting()
```

---

### ETA calculation

```ts
eta = remaining / speed
```

---

---

# src/server/http.ts

Simple HTTP server

---

### endpoints

#### /admin

→ system snapshot

#### /admin/job/:id

→ job detail

#### /preview/:id

→ video preview URL

#### /admin/dlq

→ failed jobs

#### /admin/retry/:id

→ retry job

---

### static serving

Serves local video files

---

---

# src/server/ws.ts

WebSocket server

Every 1s:

```ts
send system snapshot
```

Used for real-time UI

---

---

# src/utils

---

## cleanup.ts

Deletes old temp folders

---

## disk.ts

Uses:

```ts
statfs
```

to get disk space

---

## r2.ts

Handles:

* download
* upload

---

---

# FINAL FLOW (COMPLETE)

```
localTest → enqueueFile
           ↓
        Redis metadata
           ↓
        BullMQ queue
           ↓
        Worker picks job
           ↓
        Lock acquired
           ↓
        Download file
           ↓
        Process (image/video)
           ↓
        Upload result
           ↓
        Update Redis
           ↓
        UI reads via /admin or WS
```

---

# CRITICAL DESIGN DECISIONS (WHY THIS WAY)

---

## 1. Redis = source of truth

Not BullMQ

Why:

* BullMQ state is limited
* Redis allows custom metadata

---

## 2. HLS instead of MP4

Why:

* streaming
* partial playback
* large file support

---

## 3. Separate queues

Why:

* fairness
* avoids blocking

---

## 4. Idempotency lock

Why:

* prevents double processing

---

## 5. Heartbeat

Why:

* prevents lock expiry mid-job

---

# WEAK AREAS (Which SHOULD QUESTION)

## Assumption: Redis never fails

Reality:

* if Redis goes down → system breaks

Mitigation:

* retry layer
* fallback persistence

---

## Assumption: fs.watch is reliable

Already patched with scanner

---

## Assumption: disk always enough

Currently we check, but:

* concurrent jobs can still exceed disk

---

## Assumption: lock TTL always correct

Edge:

* job takes longer than TTL even with heartbeat delay

---
