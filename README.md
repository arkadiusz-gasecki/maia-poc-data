## Emails
Folder __emails__ contain 200 emails with dummy settlement cases for further investigation


## API
Interactive docs available at: `https://crm-api-265362761937.europe-west6.run.app/docs`

### Authentication

Every request must include the header:

```
X-API-Key: <key>
```

### Rate limiting

Each API key is limited to **10 requests per minute** by default.

### Endpoints

#### `GET /trades` — search trade records

Exactly **one** of the two search modes must be used.
Returning the full dataset is not permitted.

**Mode A — by reference number**

```
GET /trades?reference_number=AB12345678
```

**Mode B — by five-field combination**

```
GET /trades?settlement_date=2025-03-14&security=US0378331005&quantity=42000&amount=1250000&currency=USD
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `reference_number` | string | Exact reference, e.g. `AB12345678` |
| `settlement_date` | string | ISO date, e.g. `2025-03-14` |
| `security` | string | Security ISIN, e.g. `US0378331005` |
| `quantity` | integer | Exact quantity |
| `amount` | float | Exact amount (tolerance ± 0.005) |
| `currency` | string | `USD`, `CHF`, or `EUR` |

**Response**

```json
{
  "count": 1,
  "trades": [ { ... } ]
}
```

| Status | Meaning |
|--------|---------|
| `200` | One or more matching trades found |
| `404` | No matching trade |
| `422` | Wrong combination of parameters |
| `429` | Rate limit exceeded |
| `500` | Simulated server error |

---

#### `GET /security` — look up a security

Provide **either** `isin` or `name`, not both.

```
GET /security?isin=US0378331005
GET /security?name=Apple Inc.
```

**Response**

```json
{ "isin": "US0378331005", "name": "Apple Inc." }
```

Name matching is case-insensitive.

---

#### `GET /counterparty` — look up a counterparty

Provide **either** `lei` or `name`, not both.

```
GET /counterparty?lei=W22LROWP2IHZNBB6K528
GET /counterparty?name=Goldman Sachs
```

**Response**

```json
{ "lei": "W22LROWP2IHZNBB6K528", "name": "Goldman Sachs" }
```

Name matching is case-insensitive.

---

#### `POST /answers` — submit a solution for one email

Each API consumer submits exactly one answer per email file.
All five fields are mandatory; only `email_file` is validated for format.

```
POST /answers
Content-Type: application/json
X-API-Key: <key>
```

**Request body**

```json
{
  "email_file":                  "email_007.eml",
  "trade_status_classification": "trade-related",
  "core_issue_classification":   "wrong_price",
  "solution_classification":     "reject",
  "answer":                      "Amount does not match our records for ref AB12345678."
}
```

| Field | Type | Validation |
|-------|------|------------|
| `email_file` | string | Must match `email_NNN.eml`; NNN between `001` and `MAX_EMAIL_NUMBER` |
| `trade_status_classification` | string | Free text, no validation |
| `core_issue_classification` | string | Free text, no validation |
| `solution_classification` | string | Free text, no validation |
| `answer` | string | Free text, no validation |

**Response (201 Created)**

```json
{ "status": "accepted", "consumer": "Consumer Alpha", "email_file": "email_007.eml" }
```