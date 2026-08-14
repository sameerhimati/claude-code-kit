---
name: oauth-setup
description: Set up OAuth providers step-by-step. Minimizes console-hopping with checklists and code generation.
disable-model-invocation: false
allowed-tools: Task, Bash, Read, Write, Edit, Glob, Grep, WebSearch
---

# OAuth Setup: $ARGUMENTS

## Step 1: Which Provider?

Ask user which auth method they need:
1. **Google OAuth** — web or mobile
2. **Apple Sign In** — iOS or web
3. **Supabase Auth** — phone, email, or social
4. **Custom JWT** — roll your own

---

## Google OAuth

### Console Setup Checklist
Walk the user through these steps (they must do this in browser):

- [ ] Go to [Google Cloud Console](https://console.cloud.google.com)
- [ ] Create project or select existing
- [ ] Enable "Google Identity" API
- [ ] Go to Credentials → Create OAuth 2.0 Client ID
- [ ] Set application type (Web / iOS / Android)
- [ ] Add authorized redirect URIs:
  - Development: `http://localhost:3000/api/auth/callback/google`
  - Production: `https://yourdomain.com/api/auth/callback/google`
- [ ] Copy **Client ID** and **Client Secret**

### Environment Variables
```bash
GOOGLE_CLIENT_ID=your-client-id
GOOGLE_CLIENT_SECRET=your-client-secret
GOOGLE_REDIRECT_URI=http://localhost:3000/api/auth/callback/google
```

### FastAPI Integration
```python
from authlib.integrations.starlette_client import OAuth

oauth = OAuth()
oauth.register(
    name="google",
    client_id=settings.GOOGLE_CLIENT_ID,
    client_secret=settings.GOOGLE_CLIENT_SECRET,
    server_metadata_url="https://accounts.google.com/.well-known/openid-configuration",
    client_kwargs={"scope": "openid email profile"},
)

@router.get("/auth/google")
async def google_login(request: Request):
    redirect_uri = request.url_for("google_callback")
    return await oauth.google.authorize_redirect(request, redirect_uri)

@router.get("/auth/google/callback")
async def google_callback(request: Request):
    token = await oauth.google.authorize_access_token(request)
    user_info = token.get("userinfo")
    # Create or update user in database
    # Generate JWT session token
    # Redirect to frontend
```

### Next.js Integration (if using NextAuth)
```typescript
// app/api/auth/[...nextauth]/route.ts
import NextAuth from "next-auth";
import GoogleProvider from "next-auth/providers/google";

const handler = NextAuth({
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
  ],
});

export { handler as GET, handler as POST };
```

### Cloudflare Workers Integration
```typescript
// For Cloudflare Workers (AdMute pattern)
async function handleGoogleAuth(request: Request, env: Env): Promise<Response> {
  const url = new URL(request.url);
  const code = url.searchParams.get("code");

  const tokenResponse = await fetch("https://oauth2.googleapis.com/token", {
    method: "POST",
    headers: { "Content-Type": "application/x-www-form-urlencoded" },
    body: new URLSearchParams({
      code,
      client_id: env.GOOGLE_CLIENT_ID,
      client_secret: env.GOOGLE_CLIENT_SECRET,
      redirect_uri: env.GOOGLE_REDIRECT_URI,
      grant_type: "authorization_code",
    }),
  });

  const { access_token, id_token } = await tokenResponse.json();
  // Verify id_token, create session, return JWT
}
```

---

## Apple Sign In

### Setup Checklist
- [ ] Go to [Apple Developer Portal](https://developer.apple.com)
- [ ] Register App ID with Sign In with Apple capability
- [ ] Create Services ID (for web) or enable entitlement (for iOS)
- [ ] Create and download private key (.p8 file)
- [ ] Note: Team ID, Key ID, Client ID (Services ID or Bundle ID)

### iOS (SwiftUI)
```swift
import AuthenticationServices

struct AppleSignInButton: View {
    var body: some View {
        SignInWithAppleButton(.signIn) { request in
            request.requestedScopes = [.fullName, .email]
        } onCompletion: { result in
            switch result {
            case .success(let auth):
                guard let credential = auth.credential as? ASAuthorizationAppleIDCredential else { return }
                let identityToken = String(data: credential.identityToken!, encoding: .utf8)!
                // Send identityToken to your backend for verification
            case .failure(let error):
                print("Apple Sign In failed: \(error)")
            }
        }
    }
}
```

### Backend Verification
```python
import jwt
import httpx

async def verify_apple_token(identity_token: str) -> dict:
    # Fetch Apple's public keys
    async with httpx.AsyncClient() as client:
        resp = await client.get("https://appleid.apple.com/auth/keys")
        apple_keys = resp.json()["keys"]

    # Decode and verify the token
    header = jwt.get_unverified_header(identity_token)
    key = next(k for k in apple_keys if k["kid"] == header["kid"])

    decoded = jwt.decode(
        identity_token,
        key,
        algorithms=["RS256"],
        audience="your-client-id",
        issuer="https://appleid.apple.com",
    )
    return decoded  # Contains sub, email, etc.
```

---

## Supabase Auth

### Setup Checklist
- [ ] Go to [Supabase Dashboard](https://app.supabase.com)
- [ ] Project Settings → API → copy URL and anon key
- [ ] Authentication → Providers → enable desired providers
- [ ] For phone auth: enable Phone provider, configure SMS (Twilio)
- [ ] For social auth: add client IDs from respective providers

### Environment Variables
```bash
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_KEY=your-service-key  # Backend only, never expose to client
```

### FastAPI Integration
```python
from supabase import create_client, Client

supabase: Client = create_client(settings.SUPABASE_URL, settings.SUPABASE_SERVICE_KEY)

# Phone auth
async def send_otp(phone: str):
    result = supabase.auth.sign_in_with_otp({"phone": phone})
    return result

async def verify_otp(phone: str, token: str):
    result = supabase.auth.verify_otp({"phone": phone, "token": token, "type": "sms"})
    return result.user

# Middleware to verify Supabase JWT
async def get_current_user(authorization: str = Header(...)):
    token = authorization.replace("Bearer ", "")
    user = supabase.auth.get_user(token)
    return user
```

### Row Level Security (RLS)
```sql
-- Enable RLS on table
ALTER TABLE activities ENABLE ROW LEVEL SECURITY;

-- Users can read all activities
CREATE POLICY "Anyone can view activities"
  ON activities FOR SELECT
  USING (true);

-- Users can only create their own activities
CREATE POLICY "Users can create own activities"
  ON activities FOR INSERT
  WITH CHECK (auth.uid() = creator_id);

-- Users can only update their own activities
CREATE POLICY "Users can update own activities"
  ON activities FOR UPDATE
  USING (auth.uid() = creator_id);
```

---

## Security Checklist (All Providers)

- [ ] **PKCE flow** enabled (not implicit grant)
- [ ] **State parameter** used to prevent CSRF
- [ ] **Tokens stored securely** (httpOnly cookies, not localStorage)
- [ ] **Token refresh** handled (refresh tokens rotated)
- [ ] **Redirect URI validation** — exact match, no wildcards in production
- [ ] **HTTPS only** in production
- [ ] **Scope minimization** — request only needed scopes
- [ ] **ID token verification** — validate signature, issuer, audience, expiry
- [ ] **Rate limiting** on auth endpoints
- [ ] **Secrets not in code** — environment variables only
