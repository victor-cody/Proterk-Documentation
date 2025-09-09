# Add Passwordless Login with Magic Links Using Protekt

Passwordless authentication is a secure and user-friendly alternative to traditional passwords. Instead of remembering complex passwords, users can sign in with a **magic link** sent directly to their email inbox. With Protekt, implementing passwordless login is straightforward and developer-friendly.

In this guide, you’ll learn how to add **Passwordless Login with Magic Links** to your application using Protekt.

---

## Prerequisites

Before you begin, make sure you have:

- A **Protekt account** — [Sign up here](#) if you don’t have one.
- Your **Protekt API Key** from the [Dashboard](#).
- A basic web application setup (React, Node.js, or your preferred framework).
- An email provider configured (SMTP or supported transactional service such as SendGrid).

---

## Step 1: Enable Passwordless in Protekt Dashboard

1. Log in to your [Protekt Dashboard](#).
2. Navigate to **Authentication Methods → Passwordless**.
3. Select **Magic Links** as the delivery method.
4. Configure your **email provider** under **Email Settings**.
   - Provide SMTP credentials or API key for your transactional email service.
5. Save the configuration.

---

## Step 2: Request a Magic Link

Protekt exposes an endpoint to generate and send magic links. Here’s a sample request:

```bash
POST https://api.protekt.io/v1/passwordless/start
Content-Type: application/json
Authorization: Bearer <YOUR_API_KEY>


{
  "email": "user@example.com",
  "redirect_url": "https://yourapp.com/callback"
}
```

## Step 3: Handle Magic Link Redirect

When a user clicks the magic link, they’ll be redirected to your application with a one-time token parameter.

Example redirect URL:

```arduino
https://yourapp.com/callback?token=<MAGIC_TOKEN>
```

On your server, you’ll need to exchange this token for a Protekt session (JWT).

```bash
POST https://api.protekt.io/v1/passwordless/verify
Content-Type: application/json
Authorization: Bearer <YOUR_API_KEY>

{
  "token": "<MAGIC_TOKEN>"
}
```


Response:

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR...",
  "id_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6...",
  "expires_in": 3600,
  "token_type": "Bearer"
}
```

**access_token** — Use this for API calls.
**id_token** — Contains user identity claims (email, user ID).
**expires_in** — Expiration time in seconds.

## Step 4: Implement in a React Application (Example)

Below is a minimal React flow:

```jsx
import { useState } from "react";

function MagicLinkLogin() {
  const [email, setEmail] = useState("");

  const requestMagicLink = async () => {
    await fetch("https://api.protekt.io/v1/passwordless/start", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: "Bearer <YOUR_API_KEY>",
      },
      body: JSON.stringify({
        email,
        redirect_url: "http://localhost:3000/callback",
      }),
    });
    alert("Check your email for the magic link!");
  };

  return (
    <div>
      <h2>Login with Magic Link</h2>
      <input
        type="email"
        placeholder="Enter your email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <button onClick={requestMagicLink}>Send Magic Link</button>
    </div>
  );
}

export default MagicLinkLogin;
```

## Step 5: Secure the Session

Once you’ve verified the magic link token and received an `access_token`, store it securely:

- In memory or HTTP-only cookies (recommended).
- Avoid storing tokens in `localStorage` or `sessionStorage` due to XSS risks.

Step 6: Verify the User Session

On the backend, validate the token using Protekt’s public keys:

```bash
GET https://api.protekt.io/v1/userinfo
Authorization: Bearer <ACCESS_TOKEN>
```

This endpoint returns the authenticated user’s profile:

```json
{
  "sub": "user_123",
  "email": "user@example.com",
  "email_verified": true
}
```

## Best Practices

- Always configure a short expiry for magic links (default: 15 minutes).
- Use HTTPS for all redirect URLs.
- Provide clear UI/UX feedback to users (e.g., “Check your email for a link”).
- Monitor email deliverability and handle bounce/blocked cases gracefully.

## What’s Next?

- Try adding [Multi-Factor Authentication (MFA)](#) for stronger security.
- Explore [Protekt Guides](#) for integrating with social providers like Google or GitHub.
- Read the [API Reference](#) for more advanced use cases.
