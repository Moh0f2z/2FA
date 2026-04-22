
# 2FA Bypass
  [PortSwigger - Two-factor authentication](https://portswigger.net/web-security/authentication/multi-factor)
  
  [7 Advanced ways of bypassing insecure 2-FA implementations](https://www.intigriti.com/researchers/blog/hacking-tools/broken-authentication-7-advanced-ways-of-bypassing-insecure-2-fa-implementations)
### Practice:
  [PortSwigger Labs: Authentication](https://portswigger.net/web-security/all-labs#authentication)
  
### Writeups:
  [Pentester.land: Bug Bounty Writeups](https://pentester.land/writeups/)

  [good post 15 techinques](https://x.com/hetmehtaa/status/1726670754631299419)
  
  [Awesome Bug bounty Writeups](https://github.com/devanshbatham/Awesome-Bugbounty-Writeups?tab=readme-ov-file#2fa-related-issues)


### Methodology
  1. Bruteforcing
  2. You can re-use the same old token or provide none at all
  3. You can re-use any of the backup tokens
  4. The token is not tied to the session
  5. The token is anywhere exposed in the HTTP response
  6. Any tokens for testing and development purposes such as "0000" or "123456" are still accepted in production
  7. Re-usable 2FA tokens: Check if the token that you've used previously is still accepted when you log in the second time
  8. you can chain between csrf and 2fa : use csrf to disable 2fa
  9. Password reset
  10. add host header like ``` X-Forwarded-For``` to bypass rate limit
  11. try auoth to bypass 2fa
  12. response manipulation
  13. Backup Code Abuse
      <h3>What is “Backup Code Abuse”?</h3>
        . Any flaw that lets an attacker misuse backup codes to bypass 2FA:
        
        . Reusing the same code multiple times.
      
        . Guessing codes due to weak format or no rate-limit.
      
        . Generating or viewing someone’s codes without re-auth/CSRF.
      
        . Using codes from account A on account B (binding/integrity bug).
      
        . Old codes still valid after “Regenerate”.
      
        . Codes leaked in responses, JS, emails, or logs.
      
        . Racing two requests so the same code is accepted twice.
      <h3>How to Test (Practical Checks)</h3>
      
        1. <h5>Reuse test</h5>
        
      ```
          . Generate codes, use one to log in, then try using the exact same code again.  
          . If it works twice → vulnerability.
      ```
      
        2. <h5>Regenerate invalidation</h5>
        
        ```
          . Save an old code → click Regenerate → try the old code.
          . If old code still works → vulnerability.
        ```
      
      3. <h5>View/Regenerate without re-auth</h5>
      
      ```
          . Hit GET /settings/mfa/backup-codes or POST /.../regenerate.
          . If you can view/regenerate without entering password/OTP and there’s no CSRF token → high-risk vuln.
      ```

      4. <h5>Brute-force feasibility</h5>

         ```
         . Identify format/length (UI or error messages).
         . Send a few spaced attempts (respect scope/legality).
         . No throttling/lockout and small key-space → likely exploitable.
         ```

      5. <h5>Cross-account integrity</h5>

      ```
        .Create two test accounts (A,B), generate codes, try A’s code on B.
        . If accepted → severe integrity bug.
      ```

      6. <h5>Leakage</h5>
      ```
        . Inspect responses that “generate” or “download” codes (Network tab/Burp).
        . Search public JS bundles for backup, codes, mfa.
        . Check emails/logs for full, unmasked code 
      ```
      
      7. <h5>Race condition</h5>
      
      ```
        .Submit the same backup code twice in parallel (Repeater/Intruder with concurrent requests).
        . If both succeed → mark-as-used isn’t atomic.
      ```

14. <h3> Google Authenticator Bypass </h3>
```
  Steps:-
    1) Set-up Google Authenticator for 2FA
    2) Now, 2FA is enabled
    3) Go on the password reset page and change your password
    4) If your website redirects you to your dashboard then 2FA (Google Authenticator) is bypassed
```

15. 2FA Bypass
```
    Enter the victim’s username and password.
    The system correctly prompts for the 2FA code.
    Click Back in the browser to return to the login page.
    Enter the same credentials again (username + password).
    You are logged in successfully without entering the 2FA code, confirming a working 2FA bypass.
```

16. <h3>try if invation link lead to 2fa bypass</h3>
