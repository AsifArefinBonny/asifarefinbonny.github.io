---
layout: post
title: "Decoding JWT Tokens in Postman"
date: 2025-05-13
image: /images/jwt.png
---

As a Software Quality Assurance (SQA) engineer, ensuring the integrity and accuracy of user data in API responses is a critical part of our testing process. One common task is verifying user details—such as customer information, roles, and permissions—embedded in JSON Web Tokens (JWTs). While websites like jwt.io are useful for decoding JWTs, copying tokens from Postman and navigating to an external site is cumbersome and disrupts our workflow. Fortunately, Postman provides a powerful way to decode JWTs directly within its environment, streamlining the process and keeping everything in one place.

<!--more-->

In this blog post, I’ll share a simple yet effective script to decode JWT tokens inside Postman and display the header and payload in a clean, JSON-formatted view under the Visualization tab. This approach saves time, enhances test efficiency, and ensures we can quickly validate user details during API testing.

## Why Decode JWTs in Postman?

![JWT Hero Image](/images/jwt.png)

JWTs are widely used for authentication and authorization in APIs. They consist of three parts: Header, Payload, and Signature, encoded in Base64 and separated by dots (.). As QA engineers, we often need to inspect the payload to verify user-specific claims (e.g., user_id, roles, or permissions) or the header to check metadata (e.g., algorithm used). Doing this manually via external tools like jwt.io requires extra steps, which can slow down our testing cycles, especially when dealing with multiple tokens or environments.

By decoding JWTs directly in Postman, we can:

- **Save time:** No need to switch between tools.
- **Improve accuracy:** View token details in context with the API response.
- **Enhance debugging:** Quickly validate token contents during test execution.

## The Solution: A Postman Script for JWT Decoding

Below is a step-by-step guide to decode a JWT token in Postman using a pre-built script. This script extracts the token from the API response, decodes the header and payload, and displays them in a formatted, user-friendly view.

### Step 1: Add the Script to Postman

To decode the JWT, place the following script in the Tests tab (also known as the post-response script) of your Postman request. This script assumes your API response contains an accessToken field with the JWT.

```javascript
var jsonData = pm.response.json();
pm.environment.set("jwt", jsonData.accessToken);

function parseJwt(token, part) {
    var base64Url = token.split('.')[part];
    var words = CryptoJS.enc.Base64.parse(base64Url);
    var jsonPayload = CryptoJS.enc.Utf8.stringify(words);
    return JSON.stringify(JSON.parse(jsonPayload), null, 2); // Pretty-print JSON
}
var jwtInfo = {};
jwtInfo.header = parseJwt(jsonData.accessToken, 0);
jwtInfo.payload = parseJwt(jsonData.accessToken, 1);
var template = `
<style>
body { font-family: monospace; background: #1e1e1e; color: #dcdcdc; padding: 10px; font-size: 12px; }
pre { background: #252526; padding: 10px; border-radius: 5px; white-space: pre-wrap; word-wrap: break-word; overflow-x: auto; }
</style>
<h3>JWT Header</h3>
<pre>{{response.header}}</pre>
<h3>JWT Payload</h3>
<pre>{{response.payload}}</pre>
`;
pm.visualizer.set(template, { response: jwtInfo });
```

## How the Script Works

Let’s break down the script from a QA perspective to understand its functionality:

- **Extract the Token:** `pm.response.json()` retrieves the API response as a JSON object, and `pm.environment.set("jwt", jsonData.accessToken)` stores the accessToken in an environment variable for potential reuse.
- **Parse the JWT:** The `parseJwt` function takes the token and a part index (0 for header, 1 for payload). It splits the token at the dots, decodes the specified part using `CryptoJS.enc.Base64.parse`, converts it to a UTF-8 string, and formats it as pretty-printed JSON.
- **Store Decoded Data:** The decoded header and payload are stored in the `jwtInfo` object.
- **Render Visualization:** The template defines an HTML structure with CSS styling for a clean, dark-themed display. The `pm.visualizer.set` method renders the decoded header and payload in Postman’s Visualization tab.

### Step 2: Test the Request

- Send your API request (e.g., a login request that returns an accessToken).
- After the response is received, navigate to the Visualization tab in Postman.
- You’ll see the JWT header and payload displayed as formatted JSON, similar to this:

**Example Output:**

JWT Header
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

JWT Payload
```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "roles": ["admin", "user"],
  "permissions": ["read", "write"],
  "iat": 1516239022
}
```

### Step 3: Validate the Output

As QA engineers, validation is key. Check the following:

- **Header:** Confirm the algorithm (`alg`) and token type (`typ`) match the expected values for your application.
- **Payload:** Verify user-specific claims (e.g., `sub`, `roles`, `permissions`) align with the test case requirements.
- **Errors:** If the token is malformed or the API response doesn’t include an `accessToken`, the script may throw an error. Add error handling if needed for robustness.

## Why This Matters for QA

This approach is a game-changer for SQA engineers because it:

- **Streamlines Testing:** No need to leave Postman to decode tokens, reducing context-switching.
- **Improves Traceability:** The decoded token is tied directly to the API response, making it easier to document and report issues.
- **Supports Automation:** You can integrate this script into Postman collections for automated API tests, ensuring user details are validated consistently.

## Tips for QA Engineers

- **Error Handling:** Enhance the script to handle cases where `accessToken` is missing or invalid. For example, add a try-catch block around the JSON parsing.
- **Environment Variables:** Store the decoded payload in environment variables if you need to use specific claims (e.g., `user_id`) in subsequent requests.
- **Test Coverage:** Use this script to validate token contents across different user roles or permissions as part of your test scenarios.
- **Security:** Ensure sensitive token data is handled securely and not exposed in shared Postman collections.

## Conclusion

Decoding JWTs directly in Postman is a simple yet powerful technique that aligns perfectly with a QA engineer’s goal of efficient, accurate testing. By embedding this script in your Postman requests, you can quickly inspect token details without relying on external tools, making your API testing workflow smoother and more reliable. Try it out in your next test cycle and see how it simplifies validating user data!

Happy testing! 