# Phishing Attack Comprehensive Details

Created by: Vineeth M

---

## 1. Email Address Basics

An email address has two main parts: the local part (username) and the domain name. For phishing analysis, the domain is the most important part because it shows the real owner and mail server identity.

### 1.1 Username (Local Part)
- The username identifies the mailbox owner.
- Examples: `david`, `support`.
- This part is easily spoofed by attackers.

### 1.2 `@` Symbol
- The `@` symbol separates the username from the domain.
- It was introduced by Ray Tomlinson on ARPANET as part of the first email addressing format.

### 1.3 Domain Name
- The domain name is the most important part of the email address.
- It shows the real owner of the email domain.
- It also tells which server handles the email.

### Example
For `david@tryhackme.com`:
- Username → `david`
- Domain → `tryhackme.com`

**Rule:** Always trust the domain, not the username.

### Why It Matters in Phishing
Attackers often use a legitimate-looking username or display name to build trust. The real warning sign is usually the domain, especially when it does not match the organization it claims to represent.

---

## 2. How Email Actually Travels

### Basic Flow
Sender → SMTP Server → Internet → Recipient Mail Server → Inbox

### 2.1 Email is Sent
The sender composes an email and sends it using SMTP. SMTP is the protocol responsible for sending emails from one server to another.

### 2.2 DNS Lookup Happens
The sender’s mail server queries DNS to find where mail for the recipient’s domain should be delivered. DNS returns the domain’s MX record, which identifies the recipient mail server.

### 2.3 Email Travels Across the Internet
The message is routed through multiple mail servers before reaching its destination. Each server can add a `Received` header, which helps create a traceable delivery path.

### 2.4 Email Reaches the Recipient Mail Server
The recipient’s mail server receives the message and stores it. It may also run security and filtering checks such as SPF, DKIM, and DMARC to help detect spoofing and spam.

### 2.5 Email Is Stored in the Mailbox
After delivery, the email remains in the server mailbox until the user opens it.

### 2.6 Recipient Retrieves the Email
The user accesses the message using either POP3 or IMAP.

- **POP3** downloads email to one device and often removes it from the server.
- **IMAP** keeps email on the server and syncs it across multiple devices.

### Summary
In short, SMTP sends the email, DNS/MX records help find the right destination, mail servers deliver and filter the message, and POP3 or IMAP lets the recipient read it.

---

## 3. Key Email Protocols

Email communication depends on several protocols that control how messages are sent, received, and verified. Understanding these is critical in phishing analysis because attackers often exploit weaknesses in them.

### 3.1 SMTP (Simple Mail Transfer Protocol)
SMTP is used for sending emails. It works between the sender and the mail server, and then between the mail server and the recipient mail server.

**Key point:** SMTP does not verify sender identity by default, which is why email spoofing is possible.

**Example:** An attacker can send an email that appears to come from a trusted company.

### 3.2 POP3 (Post Office Protocol v3)
POP3 is used for receiving emails. It downloads emails from the server to the local device and usually removes them from the server after download.

**Key point:** POP3 has limited use in modern enterprise environments and is less relevant for phishing detection, but it is still part of email flow.

### 3.3 IMAP (Internet Message Access Protocol)
IMAP is also used for receiving emails. It keeps emails on the server and syncs them across devices.

**Key point:** IMAP is widely used today in services such as Gmail and Outlook, and it lets users access the same inbox from multiple devices.

### 3.4 Email Authentication Protocols
These protocols are designed to prevent spoofing and verify sender authenticity. SPF, DKIM, and DMARC are widely used together to authenticate mail and control what happens when checks fail.

#### SPF (Sender Policy Framework)
SPF checks whether the sender’s IP is authorized to send emails for that domain.

How it works:
- The domain owner defines allowed mail servers in DNS.
- The receiving server verifies the sender IP.

Result:
- Pass or fail.

#### DKIM (DomainKeys Identified Mail)
DKIM uses cryptographic signatures to verify email integrity.

How it works:
- The sender signs the email with a private key.
- The receiver verifies it using a public key stored in DNS.

This helps ensure the email was not modified and that the sender domain is valid.

#### DMARC (Domain-based Message Authentication, Reporting & Conformance)
DMARC builds on SPF and DKIM.

Purpose:
- It defines what to do if authentication fails.

Policies:
- None: monitor only.
- Quarantine: send to spam folder.
- Reject: block the email.

---

## 4. Types of Phishing Attacks

Phishing attacks come in several forms, but they all try to trick people into revealing information, clicking malicious links, or taking harmful actions.

### 4.1 Standard Phishing
Standard phishing is a mass email attack where attackers send the same generic message to a large number of users. The goal is to trick victims into revealing sensitive information or performing a malicious action.

### 4.2 Spear Phishing
Spear phishing is a targeted social engineering attack where the attacker crafts a personalized message for a specific individual. The goal is to convince the target to reveal sensitive information or execute a malicious action.

### 4.3 Whaling
Whaling is a type of phishing attack that specifically targets high-profile individuals in an organization, such as executives or senior managers.

### 4.4 Clone Phishing
Clone phishing happens when an attacker copies a legitimate email that the victim has already received, replaces the original link or attachment with a malicious one, and sends it again.

### 4.5 Business Email Compromise (BEC)
Business Email Compromise (BEC) is a targeted phishing attack where an attacker impersonates a trusted business identity, such as a CEO, manager, or vendor, to trick someone into sending money or sensitive information.

---

## 5. Malicious Email Investigation

A malicious email investigation focuses on identifying spoofing, deception techniques, and hidden malicious intent by analyzing the sender, headers, and content.

### 5.1 Initial Sender Verification
Start with quick visible indicators before moving into deeper analysis.

#### Collected Details
- Email address (From field).
- Domain name.
- Sender IP address from the email headers.

#### Spoofed Email Address Detection
Attackers often fake trusted identities to build trust and bypass suspicion. In many cases, the display name looks legitimate while the actual domain is different.

**Goal:** Always verify the real domain, not just the display name.

**Example:**
- Display Name: `PayPal Support <service@paypal.com>`
- Actual Sender: `attacker@sultanbogor.com`

### 5.2 Email Header Analysis
Email headers reveal the true origin and route of the message. This is one of the most important parts of phishing investigation.

#### 5.2.1 Authentication Mechanisms
Check whether the sender is verified.

- **SPF:** Validates the sender IP.
- **DKIM:** Verifies message integrity.
- **DMARC:** Defines policy and enforcement.

#### Look for
- `Authentication-Results`
- `Received-SPF`
- `DKIM-Signature` (`b=` value)

#### Indicators
- `PASS` → More trustworthy.
- `FAIL` → Strong phishing indicator.

#### 5.2.2 Routing History
Analyze how the email traveled through servers.

- **Bottom-most `Received` header:** Shows the originating IP and the actual sending system.
- **Intermediate hops:** Look for unusual countries, suspicious servers, and unexpected routing paths.
- **Timestamps:** Large gaps may suggest malicious relays or server manipulation.

#### 5.2.3 Sender and Recipient Identity Checks
Verify consistency across message fields.

- **Return-Path (Envelope-From):** Where bounce messages go; should match the sender domain.
- **Reply-To:** Often manipulated in phishing and may redirect to an attacker inbox.
- **X-Original-To:** Shows the actual targeted recipient.

**Mismatch = strong phishing signal.**

#### 5.2.4 Technical Identifiers
These fields reveal how the email was generated.

- **Message-ID:** Should align with the sender domain; random or mismatched values are suspicious.
- **X-Mailer / User-Agent:** Shows the sending software.
  - Outlook → normal.
  - PHP mailer → suspicious.
  - Mass-mail-tool → high risk.

#### 5.2.5 Security Gateway Headers
These are added by email security systems and are useful in enterprise environments.

- `X-Spam-Status`
- `X-Spam-Level`
- `X-Forefront-Antispam-Report`
- `X-Microsoft-Antispam-Message-Info`

### 5.3 Email Body Analysis
Focus on what the user actually sees.

#### 5.3.1 URL and Link Analysis
Attackers often rely on malicious links. Check for:
- Shortened or obfuscated URLs.
- Typosquatting, such as `paypaI.com` instead of `paypal.com`.
- Suspicious symbols or redirects.

Perform:
- Reputation checks.
- DNS and infrastructure analysis.

Watch for:
- Multiple redirects.
- Newly registered domains.

#### 5.3.2 Attachment Analysis
Attachments are a major malware delivery method.

Steps:
- Identify file type: `.exe`, `.docm`, `.xlsx`, `.zip`.
- Check file name anomalies.
- Review metadata.
- Look for macros.
- Generate file hashes such as MD5 or SHA256.

Perform:
- Reputation checks.
- Static analysis.
- Dynamic analysis in a sandbox.

Indicators:
- Macro-enabled documents.
- Executables disguised as documents.

### 5.4 Tracking Pixel Analysis
Tracking pixels are used for reconnaissance and victim validation.

#### What is a Tracking Pixel?
A tiny invisible image embedded in the email.

#### What happens when it loads?
The attacker may receive:
- IP address.
- Approximate location.
- Email open time.
- Device or email client information.
- User activity in some cases.

#### Purpose
- Identify active targets.
- Refine spear phishing campaigns.
- Confirm that a mailbox is monitored.

### 5.5 Preventing Tracking Pixels
Effective defenses:
- Disable automatic image loading.
- Use privacy-focused email clients such as Apple Mail, Mozilla Thunderbird, and Proton Mail.
- Use browser extensions such as Ugly Email and PixelBlock.

---

## 6. Final Investigation Flow

### Quick Summary
1. Sender verification.
2. Header analysis.
3. Authentication checks.
4. Routing analysis.
5. Identity checks.
6. Technical indicators.
7. Body analysis.
8. Link analysis.
9. Attachment analysis.
10. Tracking pixel analysis.
11. Prevention techniques.

---

## 7. Investigator Notes

A strong phishing investigation does not rely on one indicator alone. The best results come from combining header evidence, sender-domain checks, content analysis, and attachment or link inspection.
