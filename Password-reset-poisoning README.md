# HTTP Host Header Vulnerability — Password Reset Poisoning
 
**Lab:** Basic Password Reset Poisoning (PortSwigger Web Security Academy)
**Assignment:** Web Application Exploitation — HTTP Host Header Vulnerabilities
 
## Overview
 
While working through this lab, I found that the password reset feature didn't properly check where requests were actually coming from. It just trusted whatever Host value was sent in the request. So I swapped the Host header to point to a server I controlled, and set the username to my target's username. The app generated a reset link using my fake host meaning if the victim's reset email went out, the token inside it would point back to my server instead of the real site.
 
<img width="700" alt="PortSwigger lab page showing Solved status" src="https://github.com/user-attachments/assets/1aeda224-39ab-49be-8c88-8c3ce49f4b8a">

*Caption: PortSwigger "Basic password reset poisoning" lab marked as solved.*
 
## Affected Functionality
 
The "Forgot Password" feature on the website is affected.
 
## Exploitation Outcome
 
Someone trying to take over another person's account could trick the system into sending that person's password reset link to themselves instead.
 
## Potential Security Impact
 
This can let an attacker reset a victim's password and gain full access to their account, without the real user ever knowing it happened.
 
## Explanation of HTTP Host Header Injection (In My Own Words)
 
When you visit a website, your browser tells the server which site it thinks it's talking to. Normally, the server just trusts this without checking if it's actually true. The server believes whatever it's told about where it is, instead of double-checking the truth for itself, and that's what makes this exploit possible.
 
## Explanation of Password Reset Poisoning (In My Own Words)
 
Password Reset Poisoning is when an attacker tricks an application's "forgot password" feature into sending the reset link to them instead of the real user. If the victim ends up using that link, the attacker gets hold of the reset token and can take over their account.
 
## The Relationship Between the Host Header and Password Reset Functionality
 
The vulnerability exists because the website lets the visitor decide part of a trusted link instead of deciding that for itself, and that's the root cause of the whole vulnerability.
 
## The Security Weakness
 
The weakness is that the app never double-checks the Host header before using it. That one missing check turns into a vulnerability an attacker can take advantage of.
 
## The Application's Incorrect Assumption
 
The application assumed the Host header was being honest about where it was coming from, instead of confirming accuracy.
 
## Documentation on How I Approached the Lab
 
**How I identified the relevant functionality:**
Initially, I intercepted the password reset request from my own account using Burp Suite and tested whether I could manipulate the Host header to any value I wanted before forwarding the request. It worked the value I input was sent back to me in the password reset email.
 
<img width="700" alt="Burp Suite Proxy HTTP History showing the intercepted forgot-password request" src="https://github.com/user-attachments/assets/42df1531-dd9a-467d-88e8-d6803afae0b7">

*Caption: Burp Suite's HTTP History showing the intercepted POST /forgot-password request, with the original Host header still intact.*
 
**How I investigated HTTP requests and responses:**
After confirming the Host header was reflected in the email, I inspected the raw HTTP request closely in Burp Suite to see exactly where the Host value appeared, and checked the response and email content to confirm the reset link itself was built using that same value.
 
**How I formed my hypothesis:**
Based on this, I hypothesized that if the application wasn't checking the Host header before using it to build the reset link, I could redirect a password reset token to a server I controlled and if I also changed the username to a different account, I could capture another user's reset token instead of my own.
 
**How I tested it:**
I intercepted a new password reset request, changed the Host header to point to my exploit server, changed the username field to my target account, then forwarded the request. I then checked my exploit server's access log to confirm whether the reset token request had actually been sent there.
 
<img width="700" alt="Burp Suite Repeater showing Host header changed to exploit server" src="https://github.com/user-attachments/assets/5172370e-fe2f-47b0-8ab5-fd3eb2d32b66">

*Caption: Burp Suite Repeater showing the request with the Host header changed to my exploit server's domain, before forwarding it.*
 
**Why I performed each important step:**
I started by testing on my own account first to observe normal behavior of the request. I inspected the request/response to confirm my assumption. I formed a hypothesis based on evidence rather than testing randomly. Finally, I tested using the provided lab test account (carlos).
 
## Steps to Reproduce
 
1. Logged into my own test account (wiener) and clicked the "Forgot Password" functionality.
2. Intercepted the password reset request using Burp Suite before it reached the server.
3. Changed the Host header in the intercepted request to my own exploit server's domain.
4. Changed the username field in the request body to the target account's username (carlos).
5. Forwarded the modified request to the server.
6. Checked the exploit server's access log and confirmed a request had been captured, containing a leaked `temp-forgot-password-token` — proving the poisoned reset link had reached my server when the simulated victim "clicked" it.

<img width="700" alt="Exploit server access log showing the captured token request" src="https://github.com/user-attachments/assets/db74d71e-801e-44b0-b237-5ceecff08cc7">

*Caption: The exploit server's access log showing the victim's request containing the leaked password reset token.*
 
7. Took the captured token and used it to construct the genuine password reset link on the real target domain, then set a new password for carlos's account.
8. Logged in using the username "carlos" and the new password I had just set.

<img width="700" alt="Login page with username carlos entered" src="https://github.com/user-attachments/assets/b489cdad-d0ac-42c1-ab55-1e64d09b0422">

*Caption: Logging in as carlos using the new password set with the captured reset token.*
 
9. Confirmed the lab's objective was achieved successfully logged into carlos's account, with the lab marked "Solved."

<img width="700" alt="My Account page confirming successful login as carlos, lab marked Solved" src="https://github.com/user-attachments/assets/35053b35-ab5a-4a5f-b399-ce95c1292474">

*Caption: Successfully logged in as carlos, confirming full account takeover. Lab status shows "Solved."*
 
## Explanation on Every Important Step Taken While Pentesting the Application
 
**What I was trying to achieve:** I wanted to confirm whether the application's password reset functionality properly checked where its reset links were being generated from.
 
**What I observed:** When I intercepted the password reset request and changed the Host header to a server I controlled, the application generated the reset link using my domain instead of its own legitimate domain. The reset email that followed contained a link pointing to my exploit server.
 
**Why the observation mattered:** This showed that the application had no server-side check to confirm the Host header was legitimate before using it. A password reset functionality was relying entirely on client-supplied, unverified input.
 
**How it contributed to demonstrating the vulnerability:** By combining this with changing the username to a target account, I was able to prove that it could be used to redirect another user's password reset token to an attacker-controlled server, capture that token, and use it to take over their account which is what makes it a genuine vulnerability.
 
## What an Attacker Could Achieve if This Vulnerability Existed in a Real Application
 
- **Steal User Accounts (Account Takeover):** The attacker intercepts the victim's secret reset token, changes their password, and locks them out.
- **Gain Admin Control:** If the attacker targets a site administrator or manager, they can take over the entire platform, accessing sensitive company records, customer databases, and system controls.
- **Attack Thousands of Users at Once:** Because the request can be automated, an attacker could poison reset passwords for an entire list of customers at once.
- **Cause Financial and Reputational Harm:** Beyond stolen data, the business could suffer direct financial theft, fraud, regulatory fines, and major loss of customer trust.
## Confidentiality, Integrity, and Availability Implications
 
- **Confidentiality:** An attacker can break into a victim's account and read private information, personal messages, saved addresses, payment details, or sensitive company files.
- **Integrity:** The attacker can change account settings and passwords without permission, and trick the website into sending fake, malicious reset links to real users.
- **Availability:** Once the attacker changes the password, the real owner gets completely locked out of their own account. If an admin account is stolen, the attacker could even disrupt parts of the website for everyone.
## Conditions Required for Successful Exploitation
 
- **The website doesn't verify authenticity:** The server blindly trusts the Host header without checking it against an approved list of trusted domains.
- **The system sends an email:** The application automatically sends the poisoned link to the attacker-controlled server.
- **The password reset:** The attacker checks their access logs, retrieves the password reset token, and uses it to take over the target's account.
## Severity Rating
 
**Severity: High**
 
**Justification:**
- **High Impact (Complete Account Takeover):** If exploited, the attacker achieves full account takeover without needing the victim's current password. If an administrative account is targeted, this escalates to full compromise of the entire application and its underlying data.
- **No Authentication Required to Initiate:** An attacker does not need an existing account or special privileges to launch the attack anyone who knows a target's email address or username can trigger the poisoned password reset.
**Justification of Technical Impact:**
The technical impact is significant because the vulnerability allows an attacker to redirect a password reset token meant for the account owner to a server they control. Since password reset tokens are used to set a new password without requiring the old one, capturing this token grants full control over the targeted account a direct path to account takeover.
 
**Justification of Exploitation Requirements:**
This vulnerability requires minimal effort to exploit. The attacker needs only basic tools (such as Burp Suite) to intercept an HTTP request, modify two fields (the Host header and the username parameter), and have access to a server they control to receive the redirected token. No advanced technical skill is needed.
 
**Justification of the Sensitivity of the Affected Functionality:**
The password reset function is one of the most sensitive features in any application, since it directly controls access to a user's account. Any flaw in this functionality carries high risk compared to flaws in less critical areas, because successful exploitation doesn't just expose data — it hands over complete account access to the attacker.
 
## How Developers Can Fix or Prevent This Vulnerability
 
The core fix is to never let request data decide something as sensitive as where a password reset link is sent. Everything else (token expiration, single-use tokens, extra verification) exists as additional security on top. The fix isn't about catching bad requests it's about removing the need to trust unverified input in the first place. A system that no longer needs to trust the Host header can't be exploited through it, regardless of what an attacker sends.
 
## Summary of the Vulnerability and What I Learned
 
In this lab, I found that the app's password reset feature trusted the Host header without checking if it was real. By changing that header to my own server and targeting another account, I was able to redirect a password reset link away from the real user, capture the reset token on my own server, and use it to take over their account.
 
It didn't take advanced skills, just a basic understanding of how requests work and attention to detail. That's what made it dangerous.
 
Here's what I learned from completing this lab:
- Don't trust data just because it's part of a request.
- Fix the real problem, not just the symptoms.
- Small mistakes can lead to big problems.
- Explaining what I found matters just as much as finding it.
## How This Exercise Improved My Understanding of Web Application Exploitation & Penetration Testing
 
Before this lab, I thought of penetration testing mostly as using tools to scan for known issues. This exercise showed me it's actually more about asking questions, noticing how an app behaves, testing assumptions, and following the evidence step by step.
 
Here's what I learned from it:
- Vulnerabilities often come from broken trust, not just broken code. I now look for places where a system trusts something it shouldn't, not just obvious bugs.
- Testing step by step matters — observing, forming a hypothesis, then testing it.
- Small details can lead to big risks. One unchecked header led to a serious account takeover.
- Explaining a finding is just as important as finding it. Writing this report helped me realize that clear communication is a real skill in security work.
 
