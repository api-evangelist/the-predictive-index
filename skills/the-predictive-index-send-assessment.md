---
name: Send a PI assessment to a candidate
description: >-
  Create or update a candidate in The Predictive Index and send them a behavioral
  and/or cognitive assessment in a single request, then retrieve the completed
  results and Candidate Insights packet.
api: openapi/the-predictive-index-integrations-openapi.yml
generated: '2026-07-21'
method: generated
source: https://developers.predictiveindex.com/integration-examples
operations:
  - createCandidate
  - listCandidates
  - listJobs
  - getCandidateInsightsPacket
---

# Send a PI assessment to a candidate

Operating instructions for using The Predictive Index Integration API to send an
assessment and collect results. Every step below maps to a real operationId in
`openapi/the-predictive-index-integrations-openapi.yml`.

## Authentication
- Send your key in the `api-key` request header: `api-key: YOUR_APIKEY_GOES_HERE`.
- The key must be generated in the PI Software under an active user with
  organization-admin permissions and cognitive access enabled. A key under a
  disabled or under-permissioned user returns HTTP 401
  `{"Message":"Authorization has been denied for this request."}`.
- Base URL: `https://integrations.predictiveindex.com`, version prefix `/api/v1`.

## Steps
1. **Find the target job (optional).** Call `listJobs` (`GET /api/v1/jobs/`) to get
   the job the candidate is being assessed for, so you can pass its identifier.
2. **Check for an existing candidate (optional).** Call `listCandidates`
   (`GET /api/v1/candidates/`) to see whether the person already exists and has a
   completed assessment (a candidate with a completed assessment will not be
   re-sent one).
3. **Create the candidate and send the assessment.** Call `createCandidate`
   (`POST /api/v1/candidates/`). This single endpoint handles candidate
   creation-or-update, job assignment, and assessment delivery. Set
   `notifyAssessmentTakerUsingEmail: true` so the candidate receives the
   assessment email. The response includes `isExistingCandidate` telling you
   whether a new record was created or an existing one reused.
4. **Wait for completion via webhook.** Register a webhook listener (see
   `asyncapi/the-predictive-index-webhooks.yml`). PI issues an HTTP POST on
   status 40 (Completed) — and status 60 (Failed) for cognitive assessments.
   Do not poll; the webhook is the completion signal. Respond 200/201 or PI
   retries up to 9 times over 24 hours.
5. **Retrieve results.** Call `getCandidateInsightsPacket`
   (`GET /api/v1/candidates/{personId}/insightspacket`) to generate the Candidate
   Insights package for the completed candidate.

## Rules
- **Rate limits:** requests are capped per the `X-RateLimit-Limit` response header
  over a rolling 60-second window. Back off and retry on HTTP 429.
- **Errors:** responses in the 200 range are success; 400+ is an error, returned
  as `{"Message":"..."}`. See `errors/the-predictive-index-problem-types.yml`.
- **No idempotency key:** there is no idempotency-key header. `createCandidate` is
  upsert-like — inspect `isExistingCandidate` rather than assuming a fresh record.
