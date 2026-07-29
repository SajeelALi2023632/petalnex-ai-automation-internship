# Sample API Requests — Lead Management System

All requests: `POST` to your n8n webhook URL, `Content-Type: application/json`.

---

## 1. High Priority Lead

**Request**
```json
{
  "name": "Sarah Chen",
  "email": "sarah.chen@acmecorp.com",
  "company": "Acme Corporation",
  "service": "Enterprise Automation Package",
  "budget": 25000
}
```

**Expected behavior:** Passes validation → scores ≥ 50 → routes to Switch output "High" → Gmail notification sent to sales team → acknowledgment email sent to lead.

**Expected response**
```json
{
  "status": "received",
  "priority": "High"
}
```

---

## 2. Medium Priority Lead

**Request**
```json
{
  "name": "Raj Patel",
  "email": "raj.patel@smallbiz.com",
  "company": "Smallbiz Consulting",
  "service": "Standard Automation",
  "budget": 5000
}
```

**Expected behavior:** Passes validation → scores 25–49 → routes to Switch output "Medium" → row appended to "Medium Priority Leads" Google Sheet tab → acknowledgment email sent to lead.

**Expected response**
```json
{
  "status": "received",
  "priority": "Medium"
}
```

---

## 3. Low Priority Lead

**Request**
```json
{
  "name": "Jamie Lee",
  "email": "jamielee@gmail.com",
  "company": "Freelance",
  "service": "Basic Support",
  "budget": 800
}
```

**Expected behavior:** Passes validation → scores < 25 → routes to Switch output "Low" → row appended to "Low Priority - Nurture List" Google Sheet tab → acknowledgment email sent to lead.

**Expected response**
```json
{
  "status": "received",
  "priority": "Low"
}
```

---

## 4. Validation Failure — Missing Required Field

**Request**
```json
{
  "name": "Alex Kim",
  "email": "alex.kim@test.com",
  "company": "Test Co",
  "budget": 3000
}
```
*(missing `service`)*

**Expected behavior:** Fails validation → routes to error response, no downstream processing occurs.

**Expected response** (HTTP 400)
```json
{
  "status": "error",
  "message": "Validation failed",
  "errors": ["Missing required field: service"]
}
```

---

## 5. Validation Failure — Invalid Email Format

**Request**
```json
{
  "name": "Morgan Diaz",
  "email": "morgan.diaz-at-test.com",
  "company": "Diaz Designs",
  "service": "Consulting",
  "budget": 4000
}
```

**Expected behavior:** Fails validation on email format check.

**Expected response** (HTTP 400)
```json
{
  "status": "error",
  "message": "Validation failed",
  "errors": ["Invalid email format"]
}
```

---

## 6. Edge Case — Malformed Budget (currency symbols)

**Request**
```json
{
  "name": "Casey Wu",
  "email": "casey.wu@example.com",
  "company": "Wu Enterprises",
  "service": "Premium Package",
  "budget": "$10,000"
}
```

**Expected behavior:** Passes validation (budget is treated as valid input); Lead Processing node strips `$` and `,`, converting to the number `10000`. Since this exceeds the $10,000 threshold plus a high-value service match, this should score High.

**Expected response**
```json
{
  "status": "received",
  "priority": "High"
}
```

---

## 7. Boundary Test — Just Under High Threshold

**Request**
```json
{
  "name": "Priya Sharma",
  "email": "priya@brightpathsolutions.com",
  "company": "Bright Path Solutions",
  "service": "Premium Support",
  "budget": 11000
}
```

**Expected behavior:** Budget > $10,000 (+30) + high-value service keyword "premium" (+20) + company (+10) + business email (+10) = 70 → High.

**Expected response**
```json
{
  "status": "received",
  "priority": "High"
}
```

---

## 8. Empty Payload

**Request**
```json
{}
```

**Expected behavior:** Fails validation on every required field.

**Expected response** (HTTP 400)
```json
{
  "status": "error",
  "message": "Validation failed",
  "errors": [
    "Missing required field: name",
    "Missing required field: email",
    "Missing required field: company",
    "Missing required field: service",
    "Missing required field: budget"
  ]
}
```
