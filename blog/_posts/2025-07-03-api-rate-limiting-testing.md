---
layout: post
title: "My Journey Testing API Rate Limiting"
subtitle: "(And Why I Now Smile at 429 Errors)"
date: 2025-07-03
image: /images/ratelimit.png
---

When I was assigned to test our application’s new rate limiting feature, I’ll admit, my first reaction was, “Rate limiting? That’s just a ‘Too Many Requests’ error, right?” But a quick dive into research revealed a fascinating world of control mechanisms designed to protect APIs from overuse.
Here’s my note, packed with lessons and tips for anyone testing rate limiting on their next project.

![Rate Limiting Hero Image](/images/ratelimit.png)

<!--more-->

## Understanding the concept
My journey began with a simple Google search: "What is API rate limiting?" 
What I found was fascinating! Turns out, there are several flavors of rate limiting: 
- Fixed Window: Like a bouncer at a club - "You can enter 10 times per hour, period." 
- Sliding Window: More like a moving train - "You can be on this train for 10 minutes, but people get off as time passes." 
- Token Bucket: Imagine a bucket that fills with tokens at a steady rate, and each request consumes a token. 
- Leaky Bucket: Like a bucket with a hole - requests pour in, but they leak out at a controlled rate. 

The more I read, the more I realized that rate limiting is literally EVERYWHERE! Twitter limits how many tweets you can post, GitHub limits API calls, even your favorite food delivery app probably limits how many orders you can place per minute (though honestly, who tests that limit? 😅 ).

The implementation used a sliding window approach, and let me tell you, wrapping my head around this concept was like trying to understand time travel in a sci-fi movie. 
The breakthrough came when I started thinking about it like a subway train:
- The train has 4 seats (our permit limit) 
- The journey takes 8 seconds (our window) 
- Every 2 seconds, some passengers get off and new ones can board (our segments) 
Unlike a fixed window where everyone gets kicked off at once and has to wait for the next "train," a sliding window is more civilized - people leave gradually, making room for others. 

## Crafting the Test Plan
Now that I understood WHAT I was testing, I needed to figure out HOW to test it.
I structured my testing approach around these key areas:

- Functional Testing: Does the rate limiting actually work as designed?
- Configuration Testing: Do environment variables behave correctly?
- Customer Isolation: Can one user's behavior affect another's limits?
- Performance Testing: What's the overhead of rate limiting?
- Recovery Testing: Can users resume normal API usage after being rate limited?

## Building Test Cases: From Sunshine to Storm
I developed comprehensive test cases covering multiple scenarios:

Sunshine Scenarios: Normal usage within limits (e.g., 10 requests in 10 seconds)
Boundary Testing: Requests at the exact boundary of the limit or just after the window resets.
Edge Cases: Rapid bursts, timing variations, and concurrent requests
Error Scenarios: Invalid tokens, malformed requests
Recovery Scenarios: Behavior after rate limit expires
Special Cases: Testing concurrent clients and system behavior under high load.

I used Postman to simulate requests and validate responses, ensuring each case was repeatable and measurable.

## Starting Simple, Then Scaling Up
I began testing with straightforward environment variables:
```
{
  "RATE_LIMITER_PERMIT_LIMIT": "10",
  "RATE_LIMITER_WINDOW_SECONDS": "10",
  "RATE_LIMITER_WINDOW_SEGMENTS": "2"
}
```
This setup allowed 10 requests every 10 seconds. I sent 10 requests (all passed), then an 11th immediately (got a 429—success!). As confidence grew, I tested more complex configurations, like a 60-second window with 6 segments. This required patience, as waiting for windows to reset was tedious, teaching me the value of test-friendly configurations.

## The Kubernetes Curveball: Replica Sets and Sticky Sessions
Testing in our staging environment revealed inconsistent behavior. Sometimes, I could make more requests than expected before hitting the limit. The culprit? Our Kubernetes setup used multiple replicas, each tracking rate limits independently due to load balancing. A client’s requests could spread across replicas, diluting the limit enforcement.
For example, with a 10-request limit across three replicas, a client could make 30 requests if evenly distributed. 
Discussed with the team and come up with the sticky sessions solution to ensure requests from the same client hit the same replica within the rate-limiting window. 

## Setting Production Values
"How strict should limits be?" Too loose = risk abuse; too tight = frustrate users. Here’s a formula:

Production Limit = (Peak Requests/Min × Safety Multiplier) × (Window Seconds ÷ 60)  
Example: 50 requests/min peak × 2.0 multiplier × 120s window → 200 requests/2 minutes.

## Automating with Postman
Manual testing was getting tedious, so I also built a comprehensive Postman collection. 
The collection included:  
- Basic functionality tests: "Does it work?"  
- Rate limiting verification: "Does it actually block me?" 
- Customer isolation tests: "Does Alice affect Bob?" 
- Edge case testing: "What about invalid tokens?" 
- Recovery testing: "Can I use the API again later?" 

I wrote JavaScript that would: Automatically validate response codes, check for proper headers (like Retry-After), generate summary reports, and handle timing-sensitive scenarios. Here's my favorite test script snippet:

```javascript
if (pm.response.code === 429) {
    pm.test('🛑 Rate limit triggered correctly', function () {
        pm.response.to.have.status(429);
        pm.expect(pm.response.headers.get('Retry-After')).to.exist;
    });
    console.log('🎉 Rate limiting is working!');
} else {
    pm.test('✅ Request accepted', function () {
        pm.expect([200, 201, 202, 404]).to.include(pm.response.code);
    });
}
```

To simulate real-world conditions, I enhanced my Postman collection to mimic 50 clients making requests.

## Final Thoughts
If you're about to embark on your own rate limiting testing journey, remember:
- It's more complex than it initially appears
- But also more interesting than you might think
- The key is understanding the "why" before jumping into the "how"
- And yes, you will become oddly excited about HTTP 429 errors

Now, whenever I see a "Too Many Requests" error in the wild, I smile a little. Because I know there's probably a tester somewhere who spent way too much time making sure that error message shows up at exactly the right moment. Happy testing! 🚀
