# Design Decisions (Worker Node)

# Overview

## 1. Redis Instead of DB (for now)

Assumption:

> Phase 1 doesn’t need permanent storage

Tradeoff:

* Fast
* Not persistent

---

## 2. Admission Control

You added this early — good, but:

* Not strictly required for MVP
* Prevents future scaling issues

---

## 3. Worker Separation

You split:

* fast / standard / heavy

Reality:

* Overkill for very small scale
* But good for future scaling

---

## 4. Polling vs WebSockets

Current:

* Polling (simple)

Future:

* WebSockets / SSE for real-time updates

---

# Risks / Limitations

* Queue position is approximate (not exact ordering)
* Redis TTL = 24h (data loss after expiry)
* FFmpeg progress is not % (time-based)
* No cancellation support yet
* No job deduplication

---

# What You Should Do Next

## Must (for Phase 1 completion)

* Add API endpoint for job status
* Integrate frontend polling
* Replace mock upload with real R2

---

## Should

* Add job cancellation
* Add progress normalization (%)
* Improve queue position tracking

---

## Later (Phase 2+)

* Move Redis → DB (Postgres)
* Add WebSockets
* Add multi-worker autoscaling
* Add file-type processors (PDF, images, etc.)

---