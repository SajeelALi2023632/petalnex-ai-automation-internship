# Error Handling — AI-Assisted Recruitment Automation

## 1. Purpose

The recruitment workflow is designed to continue safely when invalid input, AI output errors, or conflicting assessments occur.

The main principle is:

> **Errors and uncertain cases must not result in an automatic hiring decision.**

The workflow uses deterministic validation, AI-output validation, rule-based checks, fallback handling, retries, and human review.

---

## 2. Candidate Form Validation

### Node: `Extract Candidate Information`

Before the candidate is sent to the AI system, the submitted form data is normalized and validated.

The workflow checks:

- Full name is present
- Email address has a valid format
- Position being applied for is present
- Resume text is long enough to analyze

The node generates:

```text
validation_errors
is_valid
```

If one or more checks fail, `is_valid` becomes `false`.

### Why this is important

Invalid applications are stopped before unnecessary AI processing takes place. This reduces incorrect processing and avoids spending an AI request on unusable data.

---

## 3. Invalid Form Submission Handling

### Node: `valid submission`

The IF node checks whether the candidate data is valid.

### Valid path

```text
is_valid = true
        ↓
Insert candidate into PostgreSQL
        ↓
AI Analysis
```

### Invalid path

```text
is_valid = false
        ↓
Prepare Reject Email1
        ↓
Send an Email
```

For invalid submissions, the workflow now directly sends an email to the candidate instead of continuing to database storage and AI analysis.

The email informs the candidate that the submitted application data is invalid and that they should correct the information and submit the application again.

This prevents malformed application data from entering the recruitment assessment pipeline.

---

## 4. AI Output Validation

### Node: `Parse & Validate AI Output`

AI-generated output is treated as untrusted data.

The workflow attempts to parse the AI response as JSON and checks that the required fields are available.

The expected AI output contains:

```json
{
  "skills": [],
  "experience_years": 0,
  "score": 0,
  "recommendation": "Shortlist | Reject | Manual Review",
  "reason": "..."
}
```

The recommendation is also checked against the allowed values.

### AI parsing/validation failure

If the AI response cannot be parsed or does not contain valid information, the workflow does not allow the malformed result to continue as a normal recommendation.

Instead, it creates a safe fallback:

```text
Recommendation: Manual Review
Score: 0
```

This ensures that an AI formatting or response error cannot directly produce an automated hiring decision.

---

## 5. Rule-Based and AI Disagreement

### Node: `Rule-Based + AI Assessment`

The workflow does not rely only on the AI recommendation.

A deterministic rule-based assessment independently evaluates the candidate using:

- Required skills
- Years of experience
- Rule-based score

The rule score is calculated from the number of matched required skills.

The workflow then calculates a combined score using the AI score and rule score.

If the AI recommendation and the rule-based recommendation disagree, the candidate is escalated to:

```text
Manual Review
```

### Example

```text
AI Recommendation:       Shortlist
Rule Recommendation:     Reject

Result:
        ↓
Manual Review
```

This prevents an unresolved disagreement between automated assessment methods from becoming a final hiring decision.

---

## 6. Human-in-the-Loop Error/Safety Handling

### Nodes:

```text
Notify Approver
      ↓
Human Review
      ↓
Route by Decision
```

The workflow requires an authorized human reviewer to make the final decision.

The reviewer can select:

- `Shortlist`
- `Reject`
- `Hold for Manual Review`

The reviewer can also provide their name and decision notes.

This is the main safety mechanism of the workflow:

> **The AI provides an assessment, but the human reviewer makes the final recruitment decision.**

---

## 7. Decision Routing

### Node: `Route by Decision`

After human review, a Switch node routes the application according to the human's decision.

```text
                 ┌── Shortlist ──→ Prepare Shortlist Email ──┐
Human Review ────┼── Reject ──────→ Prepare Reject Email ─────┼→ Candidate Email
                 └── Hold ────────→ Prepare Hold Notice ─────┘
```

The candidate therefore receives an email based on the human review decision rather than directly on the AI recommendation.

---

## 8. Email Handling

The workflow prepares different messages for the possible outcomes:

### Invalid application

The candidate is notified that their submitted information is invalid and should be corrected.

### Shortlist

The candidate receives a message informing them that they have been shortlisted for the next stage.

### Reject

The candidate receives a professional rejection message.

### Hold / Manual Review

The candidate receives a message informing them that their application is still under review.

---

## 9. AI/API Failure Handling

The Google Gemini model has retry handling configured.

If a temporary AI/API error occurs, the configured retry mechanism gives the operation another opportunity to complete.

The workflow also validates the AI response after it is received, so a successful API response is not automatically assumed to be valid.

These two mechanisms address different failure types:

```text
Temporary API/Model Failure
        ↓
Retry

Malformed/Unexpected AI Output
        ↓
Parse & Validate
        ↓
Manual Review fallback
```

---

## 10. Database Safety

Candidate information is stored in PostgreSQL only after the initial application validation succeeds.

The final database update records the recruitment assessment and review information, including relevant scores, recommendations, reviewer information, notes, and decision timestamp.

The database provides a persistent record of the recruitment process.

---

## 11. Error-Handling Summary

| Error / Situation | Handling |
|---|---|
| Missing candidate name | Application marked invalid |
| Invalid email | Application marked invalid |
| Missing position | Application marked invalid |
| Resume too short | Application marked invalid |
| Invalid form submission | Candidate receives rejection/correction email |
| Temporary Gemini/API failure | Retry mechanism |
| Invalid AI JSON | Manual Review fallback |
| Missing/invalid AI fields | Manual Review fallback |
| AI/rule recommendation disagreement | Manual Review |
| Human chooses Shortlist | Shortlist email |
| Human chooses Reject | Rejection email |
| Human chooses Hold | Hold/manual-review email |

---

## 12. Overall Error-Handling Flow

```text
Candidate Form
      ↓
Extract & Validate
      ↓
   Valid?
   /     No      Yes
 ↓        ↓
Invalid   PostgreSQL
Email       ↓
           AI Analysis
              ↓
       Validate AI Output
          /                Invalid        Valid
         ↓             ↓
   Manual Review   Rule + AI
                       ↓
                Recommendations
                  agree?
                 /                      No         Yes
               ↓           ↓
        Manual Review   Human Review
                           ↓
                    Final Decision
                           ↓
                    Email + Database
```

---

## 13. Security and Privacy Notes

Do not commit the following to the GitHub repository:

- PostgreSQL/Supabase passwords
- Gemini API keys
- Email credentials
- n8n credentials
- Real candidate resumes or personal information

Use credentials stored in n8n rather than hard-coding secrets in workflow code.

For demonstration and testing, use synthetic candidate information rather than real applicant data.

---

## 14. Conclusion

The error-handling design ensures that the workflow does not blindly trust either user input or AI output.

The system follows a layered approach:

```text
Input Validation
      ↓
AI Output Validation
      ↓
Rule-Based Verification
      ↓
Disagreement Escalation
      ↓
Human Approval
      ↓
Final Communication & Database Update
```

The result is an automated recruitment workflow with controlled failure paths, explicit escalation, and human oversight.
