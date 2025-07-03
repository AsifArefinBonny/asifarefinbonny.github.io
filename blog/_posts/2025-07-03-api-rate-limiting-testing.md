---
layout: post
title: "My Journey Testing API Rate Limiting: From Confused to Confident"
date: 2025-07-03
image: /images/ratelimit.png
---

![Rate Limiting Hero Image](/images/blog/rate-limiting-cover.png)

So there I was, staring at a JIRA ticket that said "implement rate limiting" and honestly? I had no clue where to start...

_Spoiler alert: it turned into one of the most interesting testing challenges I’ve tackled._

---

## The "Wait, What Even Is Rate Limiting?" Moment

Imagine you're at a restaurant: “Max 4 people per table. Orders every 10 minutes.” That’s rate limiting — for humans. Now translate that into API land, and it gets trickier.

Our system was getting **1000+ requests per minute**, overloading the servers. The devs built a **sliding window rate limiter**:

- `RATE_LIMITER_PERMIT_LIMIT`: Requests allowed
- `RATE_LIMITER_WINDOW_SECONDS`: Time window
- `RATE_LIMITER_WINDOW_SEGMENTS`: How granular the window shifts

After drawing diagrams and wrestling with metaphors, I understood: it's like a train window — old requests slide out, new ones come in.

![QuickDial Screenshot 1](/images/ratelimit.png)

---

## Crafting the Test Plan (AKA "How Many Ways Can This Break?")

☕ Coffee-fueled thoughts:

### ✅ The Basics
- Does it block after the limit?
- Returns `429 Too Many Requests`?
- "Retry-After" header correct?

### ❓ What-Ifs
- Does Customer A affect Customer B?
- What if no auth headers?
- Malformed tokens?
- Edge-timed requests?

### ⚠️ Nightmare Fuel
- Server restarts?
- Clock drift?
- Bypass via multiple endpoints?

---

## The Great Environment Variable Hunt

Everyone says: “Just set the env vars.”

But with **Kubernetes**, multiple **pods**, **replica sets**, and **load balancers** — I needed to check them all:

```bash
kubectl exec -it deployment/api-deployment -- env | grep RATE_LIMITER
```

![Kubernetes Environment](https://via.placeholder.com/500x250/27AE60/FFFFFF?text=Kubernetes+Environment+Variables)

---

## Building the Postman Collection

My Postman suite tested:

- Basic success path
- 429 triggering
- Retry header values
- Customer isolation
- Recovery after timeout
- Edge auth/token errors

Validation scripts auto-checked everything — my robot army of assertions.

---

## The Replica Set Plot Twist

Just when I thought I’d nailed it…

Turns out, each app instance kept its **own limiter state**.  
2 requests to A + 2 to B ≠ 4 total. I was being tricked.

Fixes:
- Use **sticky sessions**
- Or a **shared limiter state** via Redis

![Load Balancer Issue](https://via.placeholder.com/600x300/E74C3C/FFFFFF?text=Load+Balancer+Rate+Limiting+Issue)

---

## Finding the Right Numbers (The Math Part)

Example:

```
Permit Limit = Peak Usage × Safety × (Window / 60)
             = 50 × 2.0 × (60/60)
             = 100 requests/minute
```

Start **conservative**, monitor, then scale.

---

## Scripting the Stress Tests

```python
# The "Overly Eager Customer"
def stress_test_single_customer():
    for i in range(100):
        res = requests.get(API_URL, headers=auth_headers)
        if res.status_code == 429:
            print(f"Rate limited after {i+1} requests")
            break
        time.sleep(0.1)
```

Also tested:
- “Good Citizen” (paced calls)
- “Malicious User” (bombardment)

![Testing Scripts](https://via.placeholder.com/550x275/9B59B6/FFFFFF?text=Automated+Testing+Scripts)

---

## The Unexpected Challenges

### ⏱ Timing Issues
- Network jitter messes with precision
- Buffer zones are your friend

### ❗ Flaky Tests
- Load balancer ≠ test predictability
- Needed sticky sessions or shared state

### 🔀 Env Differences
- Test vs Prod ≠ same behavior

### 💻 “Works on My Machine”
- CI/CD runs ≠ Local test reliability

---

## Conquering the Chaos

- ✅ Retry logic & tolerance buffers
- 🧪 Environment-specific configs
- 🧰 Setup & teardown for reliable state
- 🔍 Added logging to debug flaky cases

---

## Final Strategy

- ✅ Unit + Integration + Load tests
- ✅ Monitoring scripts for real-time alerting
- ✅ Config formulas based on real-world usage
- ✅ Postman + Python + CI = Testing Power Trio

![Success](https://via.placeholder.com/400x200/2ECC71/FFFFFF?text=Testing+Success!)

---

## Lessons Learned

- Understand the algorithm first
- Distributed systems make everything harder
- Build for real usage, not ideal cases
- Monitor, adjust, repeat

---

## Would I Do It Again?

✅ Yes — but with:
- Shared state from day one
- Early load test strategy
- Ops team involved early
- Documentation for the future me

---

## In Closing...

Testing rate limiting =  
💻 Load testing + 🔒 Security + 📐 Math + 🔍 Debugging

And now, I’m proud to say:  
**I don’t just test if it works — I test if it holds up under pressure.**

Have you faced similar challenges testing rate limiting?  
👉 Drop your story — I’d love to learn from you too.
