# AI Proposal Triage Design

## 1. AI vs. rules

Use AI for document understanding: extract facts from uploaded proposals into a strict JSON schema, including price, experience, timeline, licence evidence, insurance/WSIB, warranty, team, safety, and monitoring. Ask it to include short evidence excerpts and mark fields `unknown` when evidence is absent or ambiguous. Treat all proposal text as untrusted data; never follow instructions found inside a proposal.

Use ordinary code for file validation, schema validation, mandatory pass/fail checks, score calculations, ranking, tie-breaking, and output generation. These operations need predictable results and should not depend on a language model's judgment. The admin's criteria and weights are loaded from versioned configuration, not hard-coded.

## 2. Consistency and explainability

Store the source file hash, extracted fields, evidence excerpts/page references, model/prompt version, criteria version, scoring breakdown, flags, reviewer actions, and final output. Use deterministic scoring rules and tie-breakers. Given the same validated extracted facts and the same criteria version, the ranking calculation is repeatable. AI extraction itself can vary, so low-confidence or contradictory fields must be reviewed and the accepted extraction snapshot reused for rescoring.

Proposed scoring rules (the brief supplies weights but does not specify numeric formulas):
- Experience (30%): `min(relevant projects in last 5 years / 14, 1) × 100`.
- Price (25%): 100 for ≤ CAD 200,000; 90 for ≤ 220,000; 80 for ≤ 240,000; 60 for ≤ 250,000; 40 for ≤ 260,000; 0 above budget. These fixed bands make scores easy to explain.
- Timeline (15%): linearly scale from 100 at 10 weeks to 0 at 16 weeks; cap at 100 for ≤10 weeks. More than 16 weeks scores 0 and is flagged against the project requirement. Missing timeline is not scored; flag for clarification.
- Warranty (15%): average of panel warranty score and workmanship warranty score. Panel: 100 at 25 years or more, otherwise years/25 × 100. Workmanship: years/10 × 100, capped at 100.
- Team and approach (15%): 25 points each for a named project lead, named/defined crew, safety plan, and monitoring or progress-reporting plan.
- Weighted total: sum of each criterion's normalized score × its weight. Round to one decimal only for display.

Mandatory criteria are gates, not weighted points. If a required check fails, mark ineligible; if evidence is missing or ambiguous, flag for human review. Never turn unknown into pass. Tie-break in order: higher experience score, then lower price, then shorter confirmed timeline.

## 3. Human in the loop

A reviewer confirms extracted facts and evidence for mandatory requirements, resolves flags, and reviews the ranking and explanations. Only an authorized human makes the award decision. Overrides require a reason and are logged with reviewer identity, timestamp, before/after values, and criteria version. Keep an audit trail of uploads, source hashes, extraction snapshots, scores, flags, prompt/model versions, criteria edits, and decisions.

## 4. Changing criteria

Store each category's criteria as a versioned configuration with effective date and editor. Existing results remain attached to the version used at the time. If an admin changes criteria or weights, validate that weighted criteria total 100%, create a new version, and offer a controlled rescore of all proposals in that comparison set using the new version. Clearly label old and new results; do not silently overwrite history.

## 5. Risks and safeguards

- **Wrong extraction or OCR:** retain evidence/page references; validate data types and totals; flag unclear values for review.
- **Hallucinated facts:** require evidence for every extracted claim; use `unknown` when absent; do not score unsupported claims as facts.
- **Prompt injection/manipulation:** treat uploaded documents as data, isolate them from system instructions, ignore embedded commands, and test with adversarial examples.
- **Bias or inconsistent judgments:** use published category criteria, fixed formulas, evidence-based fields, and periodic audits.
- **Missing or contradictory requirements:** fail closed for mandatory checks or flag for human confirmation; never infer licence/insurance status.
- **Privacy and security:** restrict access, validate uploads, scan files, encrypt stored documents, set retention rules, and never expose secrets in logs.
- **Cost/latency:** limit file size/pages, batch extraction where practical, cache accepted extraction snapshots, and track usage.
- **Criteria drift:** version prompts, models, criteria, and scoring code; retain reproducible audit records.

## Sample-set outcome

A ranks first under the proposed rules. E and C are scored using the same rules; C's embedded instruction is ignored. B is ineligible because the bidder's own licence is not confirmed as required. D is flagged and not ranked until its timeline is confirmed. These results are a design proposal, not a claim that the brief defines these exact scoring formulas.
