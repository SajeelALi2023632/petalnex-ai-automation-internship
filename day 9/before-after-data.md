# Day 9 — Before / After Sample Data

## Before (raw input — 15 records, inconsistent casing, mixed string/number scores)

```json
[
  { "name": "john SMITH", "email": "JOHN.smith@Example.com", "score": "78" },
  { "name": "maria Garcia", "email": "maria.garcia@EXAMPLE.com", "score": 92 },
  { "name": "DAVID lee", "email": "David.Lee@example.COM", "score": "65" },
  { "name": "priya Patel", "email": "priya.patel@Example.com", "score": 88 },
  { "name": "ahmed HASSAN", "email": "Ahmed.Hassan@example.com", "score": "45" },
  { "name": "linda chen", "email": "LINDA.CHEN@example.com", "score": 71 },
  { "name": "MICHAEL brown", "email": "michael.brown@Example.COM", "score": "99" },
  { "name": "sara Kim", "email": "Sara.KIM@example.com", "score": 58 },
  { "name": "carlos RUIZ", "email": "carlos.ruiz@EXAMPLE.COM", "score": "83" },
  { "name": "emma Wilson", "email": "EMMA.wilson@example.com", "score": 67 },
  { "name": "yusuf IBRAHIM", "email": "yusuf.ibrahim@Example.com", "score": "34" },
  { "name": "olga PETROVA", "email": "olga.petrova@EXAMPLE.com", "score": 95 },
  { "name": "raj Sharma", "email": "Raj.Sharma@example.COM", "score": "61" },
  { "name": "amina YUSUF", "email": "amina.yusuf@Example.com", "score": 79 },
  { "name": "tom ANDERSON", "email": "tom.anderson@EXAMPLE.COM", "score": "50" }
]
```

## After transform (all 15 records — normalized + graded, before filtering)

```json
[
  { "name": "John Smith", "email": "john.smith@example.com", "score": 78, "grade": "C" },
  { "name": "Maria Garcia", "email": "maria.garcia@example.com", "score": 92, "grade": "A" },
  { "name": "David Lee", "email": "david.lee@example.com", "score": 65, "grade": "D" },
  { "name": "Priya Patel", "email": "priya.patel@example.com", "score": 88, "grade": "B" },
  { "name": "Ahmed Hassan", "email": "ahmed.hassan@example.com", "score": 45, "grade": "F" },
  { "name": "Linda Chen", "email": "linda.chen@example.com", "score": 71, "grade": "C" },
  { "name": "Michael Brown", "email": "michael.brown@example.com", "score": 99, "grade": "A" },
  { "name": "Sara Kim", "email": "sara.kim@example.com", "score": 58, "grade": "F" },
  { "name": "Carlos Ruiz", "email": "carlos.ruiz@example.com", "score": 83, "grade": "B" },
  { "name": "Emma Wilson", "email": "emma.wilson@example.com", "score": 67, "grade": "D" },
  { "name": "Yusuf Ibrahim", "email": "yusuf.ibrahim@example.com", "score": 34, "grade": "F" },
  { "name": "Olga Petrova", "email": "olga.petrova@example.com", "score": 95, "grade": "A" },
  { "name": "Raj Sharma", "email": "raj.sharma@example.com", "score": 61, "grade": "D" },
  { "name": "Amina Yusuf", "email": "amina.yusuf@example.com", "score": 79, "grade": "C" },
  { "name": "Tom Anderson", "email": "tom.anderson@example.com", "score": 50, "grade": "F" }
]
```

## Final output (after `.filter()` — grade A or B only)

This is the actual node output confirmed working in n8n — 5 of 15 records:

```json
[
  { "name": "Maria Garcia", "email": "maria.garcia@example.com", "score": 92, "grade": "A" },
  { "name": "Priya Patel", "email": "priya.patel@example.com", "score": 88, "grade": "B" },
  { "name": "Michael Brown", "email": "michael.brown@example.com", "score": 99, "grade": "A" },
  { "name": "Carlos Ruiz", "email": "carlos.ruiz@example.com", "score": 83, "grade": "B" },
  { "name": "Olga Petrova", "email": "olga.petrova@example.com", "score": 95, "grade": "A" }
]
```

| name | email | score | grade |
|---|---|---|---|
| Maria Garcia | maria.garcia@example.com | 92 | A |
| Priya Patel | priya.patel@example.com | 88 | B |
| Michael Brown | michael.brown@example.com | 99 | A |
| Carlos Ruiz | carlos.ruiz@example.com | 83 | B |
| Olga Petrova | olga.petrova@example.com | 95 | A |
