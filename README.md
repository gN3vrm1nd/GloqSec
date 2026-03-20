# Gloqsec API

Fraud detection for developers. One API call scores users by email reputation, IP history, and encrypted behavioral biometrics — returning a trust score from 0 to 100 in under a second.

> **Early access** — Starter and Growth available now. Business and Enterprise Q3 2026. Gloqsec Network Q4 2026. [Join the waitlist →](mailto:n3vrm1nd.decoy@gmail.com)

---

## Table of Contents
- [What makes Gloqsec different](#what-makes-gloqsec-different)
- [By the numbers](#by-the-numbers)
- [Try it now](#try-it-now)
- [Detection example](#detection-example)
- [How it works](#how-it-works)
- [Quick start](#quick-start)
- [Use cases](#use-cases)
- [API reference](#api-reference)
- [Verdict levels](#verdict-levels)
- [Flags reference](#flags-reference)
- [Plans & bundles](#plans--bundles)
- [Security model](#security-model)
- [Roadmap](#roadmap)
- [Errors](#errors)
- [Support](#support)

---

## What makes Gloqsec different

Most fraud APIs check email and IP. Gloqsec adds a third layer — behavioral biometrics encrypted in the browser before transmission.

- Behavioral data is **AES-256-GCM encrypted** client-side. Only the Gloqsec server can decrypt it.
- Tokens are **HMAC-SHA256 signed**. Any tampering triggers an automatic block at score ≤15.
- Your **API key never appears in browser code**. All scoring happens server-to-server.

No other affordable fraud API does this.

---

## By the numbers

| Metric | Value |
|---|---|
| Response time | ~900ms cold · <5ms cached |
| Signals per check | Up to 12 |
| Verdict levels | 4 |
| Encryption | AES-256-GCM |
| Signing | HMAC-SHA256 |
| Key rotation | Every 10 minutes |
| Token expiry | 5 minutes |
| Cache TTL | 1 hour per IP and email |

---

## Try it now

No signup needed. A shared demo key is available with **500 requests/day**, reset at midnight UTC.

```
x-api-key: gs_shared_demo_key
```

Check remaining quota:

```bash
curl https://gloqsec.com/shared/usage
```

```json
{
  "calls_used_today": 12,
  "calls_remaining_today": 488,
  "daily_limit": 500,
  "resets": "every 24 hours at midnight UTC"
}
```

> Shared key is rate limited across all users and restricted to the Identity bundle. [Get a dedicated key →](mailto:n3vrm1nd.decoy@gmail.com)

---

## Detection example

**Fraudulent attempt — caught**

```json
{
  "trust_score": 38,
  "verdict": "high_risk",
  "recommendation": "challenge",
  "flags": [
    "disposable_email",
    "reported_263_times",
    "tor_exit_node",
    "bot_like_typing_speed",
    "robotic_mouse_movement",
    "copy_paste_detected",
    "timezone_language_mismatch"
  ]
}
```

Signals: disposable email · IP reported 263 times · Tor exit node · typing at 12ms/keystroke (human avg: 180ms) · robotic mouse · copy-pasted credentials · timezone mismatch.

**Legitimate user — correctly allowed**

```json
{
  "trust_score": 87,
  "verdict": "trusted",
  "recommendation": "allow",
  "flags": ["free_email_provider"]
}
```

Signals: free email provider only · clean residential IP · natural typing at 210ms · natural mouse curves · matching timezone.

---

## How it works

```
Browser (gloqsec.js)        Your server           Gloqsec API
        |                        |                      |
        | collect behavior       |                      |
        | encrypt → tsd -------> |                      |
        |                        | POST /checkUser      |
        |                        | x-api-key: ------->  |
        |                        |                      | verify + score
        |                        | <---- verdict -----  |
        | <--- allow / block --- |                      |
```

Your API key never touches the browser. Ever.

---

## Quick start

### Step 1 — Add the snippet

```html
<script src="https://gloqsec.com/static/gloqsec.js"></script>
<script>
  Gloqsec.init('https://YOUR-SITE');
  Gloqsec.protect('#your-form');

  document.getElementById('your-form').addEventListener('gs-ready', async function() {
    const resp = await fetch('/your-backend/endpoint', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        email: document.getElementById('email').value,
        tsd: document.getElementById('_tsd').value
      })
    });
  });
</script>
```

The snippet collects behavioral signals silently from page load — typing speed, mouse movement, scroll, copy-paste — and injects an encrypted `_tsd` token on submit. No user interaction required.

For non-form use cases:

```javascript
const tsd = await Gloqsec.token();
fetch('/api/submit', {
  method: 'POST',
  body: JSON.stringify({ data: yourData, tsd })
});
```

### Step 2 — Score from your backend

**Python**
```python
import httpx

def check_trust(email, ip, tsd):
    return httpx.post(
        "https://gloqsec.com/checkUser",
        headers={"x-api-key": "gs_your_key_here"},
        json={"email": email, "ip": ip, "tsd": tsd}
    ).json()

result = check_trust(email, request.client.host, tsd)
if result["recommendation"] == "block":
    return {"error": "Request denied"}
elif result["recommendation"] == "challenge":
    return {"error": "Please verify your identity"}
```

**Node.js**
```javascript
async function checkTrust(email, ip, tsd) {
  const { data } = await axios.post(
    'https://gloqsec.com/checkUser',
    { email, ip, tsd },
    { headers: { 'x-api-key': 'gs_your_key_here' } }
  );
  return data;
}

app.post('/endpoint', async (req, res) => {
  const result = await checkTrust(req.body.email, req.ip, req.body.tsd);
  if (result.recommendation === 'block')
    return res.status(403).json({ error: 'Request denied' });
});
```

**PHP**
```php
function checkTrust($email, $ip, $tsd) {
    $ch = curl_init('https://gloqsec.com/checkUser');
    curl_setopt_array($ch, [
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_POST => true,
        CURLOPT_HTTPHEADER => ['x-api-key: gs_your_key_here', 'Content-Type: application/json'],
        CURLOPT_POSTFIELDS => json_encode(['email'=>$email,'ip'=>$ip,'tsd'=>$tsd])
    ]);
    return json_decode(curl_exec($ch), true);
}
```

---

## Use cases

Gloqsec works on any user action that bots or fraudsters can abuse — not just signup forms.

| Use case | What it catches |
|---|---|
| Signup forms | Fake accounts, disposable emails, bot registrations |
| Login forms | Credential stuffing, brute force attempts |
| Password reset | Account takeover attempts from suspicious IPs |
| Checkout / payment | Stolen card usage, robotic form filling |
| Contact forms | Spam bots, automated outreach campaigns |
| Reviews / comments | Coordinated fake review campaigns |
| Voting / polls | Vote manipulation, repeated device fingerprints |
| Referral / promo codes | Fraud ring abuse of reward systems |
| Any resource-creating endpoint | Automated post, order, or booking abuse |

---

## API reference

### POST /checkUser

**Headers**

| Header | Required | Value |
|---|---|---|
| `x-api-key` | Yes | Your key or `gs_shared_demo_key` |
| `Content-Type` | Yes | `application/json` |

**Body**

| Field | Type | Required |
|---|---|---|
| `email` | string | Yes |
| `ip` | string | No — auto-detected if omitted |
| `tsd` | string | Yes |

**Response**
```json
{
  "trust_score": 87,
  "verdict": "trusted",
  "recommendation": "allow",
  "confidence": 94,
  "flags": ["free_email_provider"],
  "bundles_active": ["identity", "network", "behavioral"],
  "breakdown": {
    "identity": {
      "email": { "score": 90, "flags": ["free_email_provider"] },
      "ip":    { "score": 100, "flags": [] }
    },
    "network":    { "score": 100, "flags": [] },
    "behavioral": { "score": 85,  "flags": [] }
  },
  "response_time_ms": 312,
  "plan": "growth"
}
```

### GET /shared/usage
Returns shared demo key quota. No authentication required.

### GET /usage
Returns personal key usage. Requires `x-api-key` header.

```json
{
  "plan": "growth",
  "calls_used": 1240,
  "calls_limit": 25000,
  "calls_remaining": 23760,
  "bundles_active": ["identity", "network", "behavioral"]
}
```

---

## Verdict levels

| Verdict | Score | Recommendation | Action |
|---|---|---|---|
| `trusted` | 75–100 | `allow` | Proceed normally |
| `suspicious` | 50–74 | `review` | Allow, monitor activity |
| `high_risk` | 25–49 | `challenge` | Require additional verification |
| `block` | 0–24 | `block` | Reject the request |

---

## Flags reference

### Identity
| Flag | Meaning |
|---|---|
| `disposable_email` | Known throwaway domain |
| `free_email_provider` | Free consumer provider |
| `found_in_N_breaches` | Email in N known breach databases |
| `suspicious_username_length` | Username under 3 characters |
| `random_looking_username` | Auto-generated username pattern |
| `high_abuse_score_N` | IP abuse confidence score N/100 |
| `moderate_abuse_score_N` | Moderate IP abuse history |
| `reported_N_times` | IP reported N times for abuse |
| `tor_exit_node` | Known Tor exit node |
| `vpn_or_proxy_detected` | VPN, proxy, or datacenter IP |
| `suspicious_hosting` | Suspicious infrastructure |

### Network
| Flag | Meaning |
|---|---|
| `timezone_language_mismatch` | Timezone and language regions don't match |

### Behavioral
| Flag | Meaning |
|---|---|
| `bot_like_typing_speed` | Under 50ms per keystroke — impossible for humans |
| `suspiciously_fast_typing` | 50–100ms per keystroke |
| `robotic_mouse_movement` | Straight line mouse paths — bot pattern |
| `no_mouse_movement` | No mouse activity on desktop |
| `copy_paste_detected` | Credentials pasted from clipboard |
| `no_scroll_detected` | No scroll on desktop — bot behavior |
| `mobile_user` | Mobile detected — mouse signals not applicable |
| `touch_gestures_detected` | Touch confirmed — positive human signal |

> **Mobile:** Gloqsec detects mobile devices automatically. Mouse flags are suppressed and replaced with touch gesture analysis.

### Social
| Flag | Meaning |
|---|---|
| `headless_browser_detected` | Puppeteer, Selenium, or similar framework |

### Network intelligence
| Flag | Meaning |
|---|---|
| `gloqsec_network_flagged` | Previously flagged across the Gloqsec network |
| `known_fraud_pattern` | Matches a known network fraud pattern |

### Security
| Flag | Meaning |
|---|---|
| `behavioral_data_tampered` | Token modified — automatic block |
| `missing_behavioral_data` | No token submitted — automatic block |

---

## Plans & bundles

| Plan | Price | Calls | Bundles | Status |
|---|---|---|---|---|
| Shared demo | Free | 500/day shared | Identity | Live |
| Starter | $10/mo | 5,000 | Identity | Live |
| Growth | $29/mo | 25,000 | Identity + Network + Behavioral | Live |
| Business | $79/mo | 100,000 | + Social | Q3 2026 |
| Enterprise | $199/mo | Unlimited | + Gloqsec Network | Q3 2026 |

> Overage: $0.003/call above plan limit.

### Bundle 1 — Identity Core · all plans
| Signal | Detects |
|---|---|
| Email reputation | Disposable domains, breach history, username patterns |
| IP reputation | Abuse score, report count, country |
| VPN / proxy | Flags VPN, proxy, and datacenter IPs |
| Tor exit node | Known Tor exit nodes |

### Bundle 2 — Network Intelligence · Growth+
| Signal | Detects |
|---|---|
| Timezone vs language | Region mismatch indicating spoofed location |

### Bundle 3 — Behavioral Biometrics · Growth+
| Signal | Detects |
|---|---|
| Typing speed | Sub-human keystroke timing |
| Mouse movement | Robotic straight-line patterns |
| Copy-paste | Pasted credentials |
| Scroll | No scroll on desktop pages |

### Bundle 4 — Social Signals · Business+ · Q3 2026
| Signal | Detects |
|---|---|
| Headless browser | Automation frameworks |
| Device consistency | Anomalous screen and color depth |

> [Join the waitlist →](mailto:n3vrm1nd.decoy@gmail.com)

### Bundle 5 — Gloqsec Network · Enterprise · Q4 2026

Every Gloqsec API call contributes to a shared intelligence layer. An IP that defrauded one customer is flagged for all — automatically.

| Signal | Detects |
|---|---|
| Cross-site IP intelligence | IPs flagged anywhere on the network |
| Behavioral pattern matching | Fraud patterns across multiple products |
| Digital identity graph | Persistent cross-site identity profiles |
| Fraud ring detection | Coordinated attack signatures |

> The more customers on Gloqsec, the stronger the network for everyone. [Join the waitlist →](mailto:n3vrm1nd.decoy@gmail.com)

---

## Security model

| Property | Implementation |
|---|---|
| Behavioral encryption | AES-256-GCM — browser encrypts, only Gloqsec decrypts |
| Tamper detection | HMAC-SHA256 — modification detected instantly |
| Key rotation | Every 10 minutes automatically |
| Token expiry | 5 minutes — replay attacks impossible |
| API key exposure | Never in browser code |
| Transport | HTTPS required |

---

## Roadmap

| Feature | Status |
|---|---|
| Email + IP scoring | Live |
| Behavioral biometrics | Live |
| Network signals | Live |
| Encrypted behavioral tokens | Live |
| Shared demo key | Live |
| Dashboard and analytics | Q2 2026 |
| Social signals bundle | Q3 2026 |
| Gloqsec Network | Q4 2026 |
| Accuracy benchmarks | With first 100 customers |
| False positive rates | With first 100 customers |

> Accuracy and false positive metrics require real-world data to be meaningful. Published transparently as the network grows.

---

## Errors

| Status | Error | Meaning |
|---|---|---|
| 401 | `Invalid or inactive API key` | Wrong or revoked key |
| 429 | `Call limit reached` | Daily or monthly limit hit |
| 422 | `Validation error` | Malformed request body |

---

## Support

- Interactive docs: `https://gloqsec.com/docs`
- Shared key quota: `GET /shared/usage`
- Personal usage: `GET /usage` with your API key
- Email: `n3vrm1nd.decoy@gmail.com`
