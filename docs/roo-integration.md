Roobet Game Statistics & Bet History Integration Proposal
============

#### Overview ####

>This document outlines the necessary features and enhancements required from Roobet's development team to facilitate the api integration with a game statistics and analysis application, akin to [StakeStats.net](https://stakestats.net/) for [Stake.com](https://stake.com/). The primary focus is on enabling users to access and download their betting history efficiently.


Authentication & Account Linking
------------

### Discord OAuth 2.0 Integration
#### 1. Discord OAuth 2.0 Integration
*  **Purpose**: Utilize Discord OAuth 2.0 for user authentication to leverage the strong community engagement and ease of communication within the Discord platform.
*  **Benefits**:
    * Enhanced user engagement through community features.
    * Simplified user onboarding process.
    * Maintains user privacy, as Discord shares minimal personal data compared to platforms like Facebook.

#### 2. Linking Roobet Account
*  **Method**: Users link their Roobet account by sending a chat message in the following format:
~~~text
 @RooStatsAccount link-id: uniqueRandomGeneratedString
~~~

*  **Process**:
    * The backend generates a unique identifier (`link-id`) for the user.
    * The user sends the linking message in Roobet's chat.
    * The Application(*`RooStats`*) verifies the message and associates the Roobet account with the user's profile.

Roobet API Analysis
--------------

#### 1. Endpoint: `/_api/history`

- **Description**: Retrieves user transaction history, including bets.

- **Query Parameters**:
  - `type`: Specifies the transaction type (e.g., `bet`).
  - `page`: Page number for pagination.
  - `order`: Sorting order (`asc` or `desc`).
  - `limit`: Number of records per page.

- **Example Request**:
  ```http
  GET /_api/history?type=bet&page=1&order=desc&limit=100
  Host: roobet.com
  ```

- **Observations**:
  - Fetching large datasets (e.g., 10,000 records) can result in timeouts (~30-60 seconds).
  - The response size is approximately 500KB, with a resource size of ~9-10MB.

#### 2. Endpoint: `/_api/history/bets`

- **Description**: Provides detailed bet history.

- **Query Parameters**:
  - `page`: Page number for pagination.
  - `order`: Sorting order (`asc` or `desc`).
  - `limit`: Number of records per page.

- **Example Request**:
  ```http
  GET /_api/history/bets?page=1&order=desc&limit=100
  Host: roobet.com
  ```

- **Observations**:
  - More efficient than `/_api/history` for retrieving bet data.
  - Fetching 10,000 records takes approximately 3-10 seconds.
  - Response size is around 676KB, with a resource size of ~9MB.

**Recommendation**: Prefer using `/_api/history/bets` for bet history retrieval due to better performance and detailed data.

Download Functionality
--------------

#### Overview

>Roobet's current endpoint—`GET /_api/history/bets?page=<n>&order=desc&limit=<xx>`—works but is undocumented and limited to paged JSON responses that can time out on large datasets. To support robust exports, three download variants are proposed.

### Variant A: Direct Download Button

>#### Flow
>1. **User Action**: Clicks "Download All Bets."
>2. **Backend**: 
>   - Streams the full dataset in one response (e.g., `application/json` or `text/csv`)
>3. **Client**: Browser triggers file save.
>
>#### Pros
>- **Simplicity**: Minimal infrastructure changes.
>- **Immediate**: Users get data in real time.
>
>#### Cons
>- **Resource-Intensive**: Large JSON/CSV responses block threads and may time out
>- **Poor UX on Slow Networks**: Users may see long loading or browser crashes.
>- **Hard Rate-Limit Control**: Difficult to throttle without impacting all users

### Variant A-1: Part 1 - Pre‑computed `Archive` Endpoint

#### Objective
`Provide users with only the dates on which they placed bets—avoiding zero‑record downloads and expensive on‑the‑fly counts.`

#### Endpoint
```http
GET /_api/history/bets/archive
Host: roobet.com
Accept: application/json
```

#### Response
```json
[
  { "date": "2025-04-18", "totalBets": 125 },
  { "date": "2025-04-15", "totalBets": 98 }, 
  { "date": "2025-04-14", "totalBets": 1225 },
  { "date": "2025-04-09", "totalBets": 198 },
  ...
]
```

#### Notes
- **Pre‑computation:** Avoids recalculating aggregates on every request.
- **Storage:** Store `archive` records in a lightweight table or cache, updated nightly or incrementally.

### A.1: Part 2 - Date‑Scoped Download Endpoint

#### Objective
>Allow per‑day bet exports, keeping each payload bounded and timeout‑safe.

#### Endpoint
```http
GET /_api/history/bets?date=2025-04-01
Host: roobet.com
Accept: application/json
```

#### Behavior
- Returns all bets for the specified date in a single JSON or CSV response.
- Streaming the response via HTTP chunked transfer and Node.js streams prevents high memory usage.
- Limits and timeouts can be tuned per endpoint.

### Front‑end UI Flow
1. Fetch `archive`
   - Call `/bets/archive` to render a paginated table (10 records/page):

| Date       | Total Bets | Download Link                     |
|------------|-------------|-----------------------------------|
| 2025-04-18 | 125         | [Download](/bets?date=2025-04-18) |
| 2025-04-15 | 98          | [Download](/bets?date=2025-04-15) |
| 2025-04-14 | 1225        | [Download](/bets?date=2025-04-14) |
| 2025-04-09 | 198         | [Download](/bets?date=2025-04-09) |
| ...        | ...         | ...                               |

2. Date‑Scoped Download
   - User clicks the download link.
   - Browser triggers file download immediately.

### Pros & Cons
| Pros | Cons |
|------|------|
| Efficient: Avoids zero‑record fetches | Data Lag: May be up to 24h stale |
| Simple Integration | Requires daily storage for `archive` records |
| Predictable Performance | |

### Implementation Considerations
- **Batch Job for `archive`:** Nightly CRON job or Lambda to aggregate user bet counts.
- **Streaming & Rate-Limiting:** Use Node.js streams for file responses. Apply per‑user and per‑endpoint rate limits.
- **API Documentation:** Document all endpoints in OpenAPI spec with examples and headers.

---

## Variant B: Asynchronous Batch Processing

### Flow
- **User Action:** Clicks "Request Bet History."
- **Backend:**
  - Enqueues a batch job.
  - Returns `202 Accepted` with `Location` header for polling.
- **Processing:**
  - Worker fetches data in pages and writes to files (e.g., `bets-<user>-<date>-<part>.json`).
- **Completion:**
  - Sends an email with a download link to a zipped archive.

### Pros
- **Scalable:** Offloads to background jobs.
- **Resilient:** Supports retries and partial failure handling.
- **User Feedback:** Polling endpoint provides status updates.

### Cons
- **Complexity:** Requires queue, job status tracking, and file storage.
- **Latency:** Users wait for job completion.

---

## Variant C: Pre‑generated Daily Archives (Stake‑Style)

### Flow
- **Pre‑generation:**
  - Daily CRON job archives bets for each user into files (e.g., ZIP or CSV).
- **User Action:**
  - Clicks “Download Archive” →
  - Calls `GET /_api/history/bets/archive?date=YYYY-MM-DD`
- **Server:**
  - Serves the pre‑generated file immediately.

### Pros
- **Instant Downloads:** Millisecond response times.
- **Predictable Load:** Workload is off‑peak.

### Cons
- **Storage Overhead:** Archive files consume space; requires cleanup.
- **Data Freshness:** Only works for full days; not ideal for real-time needs.

---

By implementing and iterating through these variants, Roobet can ensure users are empowered to access their data efficiently, while maintaining infrastructure reliability and performance at scale.