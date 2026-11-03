# Campaign Events — Evaluator Verification Walkthrough

This guide provides a structured, step-by-step checklist for evaluating the implementation against all requirements in the assignment prompt.

---

## 📋 Evaluation Checklist

- [x] **Part 1**: `NOTES.md` completed (Problem interpretation, assumptions, ambiguities, PM questions, priorities).
- [x] **Part 2**: HTTP Service implemented with `POST /events`, `GET /campaigns/{id}/stats`, and optional `GET /campaigns/{id}/events`.
- [x] **Part 2**: Concurrency safety, input validation, deduplication, and resilient batch parsing verified.
- [x] **Part 3**: All 4 bugs in `debugging/` fixed; output matches `expected_output.txt` byte-for-byte.
- [x] **Part 3**: `BUGS.md` documents what was wrong, why it happened, the minimal fix, and how verified.
- [x] **Part 4**: Scale Memo completed in `NOTES.md` (100M events/day, queueing, duplicates, monitoring).
- [x] **Part 5**: Angry Marketer scenario forensic analysis completed in `NOTES.md`.
- [x] **Wrap-up**: "What I completed / what I skipped" and "If I had another day" added to `NOTES.md`.
- [x] **AI Disclosure**: `AI_USAGE.md` completed.

---

## 🔍 Verification Steps

### Step 1: Verify Part 3 (Debugging Exercise)
Run the fixed program against the 20,000-line dataset and compare with ground truth:
```bash
cd debugging
go run . events.jsonl > actual.txt
fc actual.txt expected_output.txt
```
**Verification Outcome**: `FC: no differences encountered` (100% byte-for-byte match).

---

### Step 2: Run the Automated Backend Test Suite
Run the test suite covering edge cases, race conditions, and real-world seed data:
```bash
cd backend
go test -v ./...
```
**Key Tests to Observe:**
- `TestPostEvents_Deduplication`: Submitting identical event IDs does not double-count stats.
- `TestPostEvents_UniqueOpensPerCampaign`: Unique contacts are partitioned per campaign.
- `TestPostEvents_PartialBatchFaultTolerance`: Malformed items (bad timestamps, missing IDs) are rejected without failing the valid events in the batch.
- `TestPostEvents_Concurrency`: Multiple concurrent worker goroutines write simultaneously without race conditions or counter corruption.
- `TestSeedDataFile_FullSurvival`: Ingests the 235-line `seed/events.json` file. All messy records (unparseable dates, missing contact IDs, unknown types like `spam_report`) are isolated, while 211 valid events are accepted and 18 duplicates are detected.

---

### Step 3: Run the Live HTTP Server & Probe Endpoints
Start the server:
```bash
cd backend
go run .
```

In a separate terminal, test the endpoints:

#### Test Ingestion:
```bash
curl -X POST http://localhost:8080/events \
  -H "Content-Type: application/json" \
  -d '[
    {"event_id":"e1","campaign_id":"cmp_demo","contact_id":"ct_1","type":"sent","timestamp":"2026-08-10T10:00:00Z"},
    {"event_id":"e2","campaign_id":"cmp_demo","contact_id":"ct_1","type":"delivered","timestamp":"2026-08-10T10:01:00Z"},
    {"event_id":"e3","campaign_id":"cmp_demo","contact_id":"ct_1","type":"opened","timestamp":"2026-08-10T10:05:00Z"}
  ]'
```

#### Test Stats Reporting:
```bash
curl http://localhost:8080/campaigns/cmp_demo/stats
```
**Expected Response:**
```json
{
  "campaign_id": "cmp_demo",
  "sent": 1,
  "delivered": 1,
  "opened": 1,
  "clicked": 0,
  "unique_opens": 1
}
```

#### Test 404 for Unknown Campaign:
```bash
curl -i http://localhost:8080/campaigns/non_existent/stats
```
**Expected Response:** HTTP 404 Not Found.

---

## 📚 Deliverable Files
- [NOTES.md](file:///C:/Users/haree/Downloads/TWW%20Main%20Task/NOTES.md)
- [BUGS.md](file:///C:/Users/haree/Downloads/TWW%20Main%20Task/BUGS.md)
- [AI_USAGE.md](file:///C:/Users/haree/Downloads/TWW%20Main%20Task/AI_USAGE.md)
- [backend/](file:///C:/Users/haree/Downloads/TWW%20Main%20Task/backend)
- [debugging/](file:///C:/Users/haree/Downloads/TWW%20Main%20Task/debugging)

