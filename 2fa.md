# 🔐 2FA Bypass Testing Checklist

> Methodology for testing Two-Factor Authentication implementations in bug bounty engagements.
> **Always test within authorized scope only.**

---

## Phase 1: Quick Wins (Low Effort, High Yield)

- [ ] **Direct endpoint access** — After password step, try navigating directly to the post-2FA endpoint (dashboard/profile) without completing verification
- [ ] **Back button bypass** — Login → 2FA prompt appears → click Back → re-enter same credentials → check if logged in without OTP
- [ ] **Response manipulation** — Intercept response (`{"success":false}` / non-200) and modify to `true`/200 via Burp Match & Replace; observe frontend behavior
- [ ] **Token exposure in response** — Check network responses, HTML source, and JS bundles for leaked OTP/token values

---

## Phase 2: Session & Token Logic

- [ ] **Token not tied to session** — Extract a token from Account A, attempt to use it in Account B's session
- [ ] **Token reuse** — Use the same token twice for the same account
- [ ] **Parallel session manipulation** — Start sessions for attacker + victim accounts simultaneously; complete 2FA step for attacker's account without finalizing, then attempt to advance victim's session using the same step
- [ ] **CSRF on "disable 2FA"** — Check if the disable-2FA endpoint lacks CSRF protection

---

## Phase 3: Brute Force on OTP

- [ ] **Rate limit absence** — Attempt multiple OTP submissions; check if any limit exists at all
- [ ] **Valid OTP after limit triggers** — Even if rate limit returns 401 after N attempts, test whether submitting the *correct* OTP still returns 200 (limit may be cosmetic)
- [ ] **Slow brute force** — If only a speed-based limit exists (not a total cap), space out attempts
- [ ] **Resend resets rate limit** — Check if clicking "resend code" resets the attempt counter
- [ ] **Client-side rate limit bypass** — If limit is enforced only in JS, send requests directly via Burp/Repeater
- [ ] **Infinite OTP regeneration + weak code space** — If codes are short (e.g. 4 digits) and regeneration is unlimited, test feasibility of brute forcing a small code space

---

## Phase 4: Password Reset as a Bypass Path

- [ ] **Password reset disables 2FA** — Enable 2FA → trigger password reset → log in with new password → check if 2FA is still required
- [ ] **Reset link reusability** — Test if the same reset link can be used more than once

---

## Phase 5: OAuth & Alternative Auth

- [ ] **OAuth account compromise** — If "Login with Google/Facebook" exists, check if it fully bypasses 2FA
- [ ] **Email verification link at signup** — Check if the account-activation link (sent on registration) grants profile access without 2FA

---

## Phase 6: Infrastructure & Versioning

- [ ] **Legacy subdomains** — Check `old.`, `legacy.`, `staging.`, `beta.` subdomains for missing/weaker 2FA
- [ ] **Old API versions** — Test `/v1/` vs `/v2/`/`/v3/` for inconsistent 2FA enforcement
- [ ] **Predictable "Remember Me" cookie** — Check if cookie value is guessable (e.g. base64-encoded user_id)
- [ ] **X-Forwarded-For / IP spoofing** — Test if 2FA is skipped based on a trusted/internal IP header

---

## Phase 7: Backup Codes

- [ ] **Reuse test** — Use the same backup code twice; should fail on second use
- [ ] **Regenerate invalidation** — Save old code → regenerate → try old code again
- [ ] **View/regenerate without re-auth** — Test `GET /settings/mfa/backup-codes` or `POST /.../regenerate` without password/OTP re-verification and without CSRF protection
- [ ] **Cross-account integrity** — Generate codes on Account A, try using one on Account B
- [ ] **Brute-force feasibility** — Identify code format/length; test for missing throttling with a small key-space
- [ ] **Leakage** — Inspect "generate"/"download" responses, public JS bundles, emails, and logs for unmasked codes
- [ ] **Race condition** — Submit the same backup code twice in parallel (Turbo Intruder); check if "mark-as-used" is atomic

---

## Gaps Often Missed

- [ ] **Missing 2FA enforcement per sensitive endpoint** — Check if actions like change-password, change-email, or add-payment-method require re-verification, especially on stale sessions
- [ ] **JWT/token claim manipulation** — If a claim like `"mfa_verified": false` exists, test tampering (works only if signature isn't validated properly, e.g. `alg: none`)
- [ ] **Downgrade attack** — If multiple 2FA methods exist (SMS/TOTP/Email), test switching to the weakest method regardless of what the user has enabled
- [ ] **Missing re-auth on MFA settings change** — Test disabling/changing the phone number tied to 2FA without password/OTP confirmation
- [ ] **Concurrent session limit bypass** — Check if 2FA verification on one session affects/bypasses other concurrent sessions
- [ ] **TOTP replay window size** — Test how large the time-drift acceptance window is; overly generous windows weaken security
- [ ] **"Trusted device" mechanism weaknesses** — Check if the trust cookie/token is guessable or usable across different devices without proper fingerprinting

---

## Advanced Techniques

- [ ] **Multi-step form state confusion** — If the flow is password → send-OTP → verify-OTP, try jumping directly to verify-OTP without ever triggering send-OTP
- [ ] **HTTP request smuggling** — If a WAF sits in front of the auth endpoint, test smuggling to reach the backend while bypassing WAF-level rate limiting
- [ ] **GraphQL batching attacks** — If GraphQL is used, test batched queries in a single HTTP request to try multiple OTPs while evading per-request rate limits
- [ ] **Timing attack on OTP comparison** — If comparison isn't constant-time (e.g. `==` instead of `hmac.compare_digest`), measure response time differences to infer correct digits
- [ ] **Chained vulnerability** — If a separate bug (e.g. IDOR) grants partial account access, use it to reach the disable-2FA endpoint directly instead of bypassing verification itself

---

## References

- [PortSwigger — Two-factor authentication](https://portswigger.net/web-security/authentication/multi-factor)
- [Intigriti — 7 Advanced ways of bypassing insecure 2FA implementations](https://www.intigriti.com/researchers/blog/hacking-tools/broken-authentication-7-advanced-ways-of-bypassing-insecure-2-fa-implementations)
- [PortSwigger Labs — Authentication](https://portswigger.net/web-security/all-labs#authentication)
- [Pentester.land — Bug Bounty Writeups](https://pentester.land/writeups/)
- [Awesome Bug Bounty Writeups — 2FA section](https://github.com/devanshbatham/Awesome-Bugbounty-Writeups?tab=readme-ov-file#2fa-related-issues)

---

*Test only within authorized bug bounty scope.*
