WorklogPRO Stored XSS – Proof of Concept

Vulnerability Overview

A Stored XSS vulnerability exists in WorklogPRO when users create Jira issues.
Any user with permission to create issues (including low-privilege users) can inject malicious JavaScript into the issue Summary field.

The payload becomes permanently stored and is automatically executed:

When any user (including administrators) views the issue inside WorklogPRO

When loading the Worklog Calendar view

This results in a reliable multi-trigger persistent XSS impact.

Bug Location
```
http://localhost:9090/secure/WorklogPROWorklogCalendar!default.jspa
```

Steps to Reproduce
1. Inject the Payload

Authenticate to Jira using a low-privilege account (e.g., test2).

Enable an HTTP interception proxy (Burp Suite, OWASP ZAP).

Create a new issue and intercept the request.

Modify the summary parameter to include a JavaScript payload:
```
POST /secure/QuickCreateIssue.jspa?decorator=none HTTP/1.1
Host: localhost:9090
Content-Type: application/x-www-form-urlencoded
Cookie: [Your Session Cookies Here]

pid=10105&issuetype=10001&summary=%3Cscript%3Ealert('CreateIssueSummary')%3C%2Fscript%3E&description=&priority=3&assignee=-1
```

A successful HTTP 200 confirms the issue has been created with the stored XSS payload.

2. Trigger the Stored XSS (Log Work)

Log in as any user with access to the project (including administrators).

Navigate to:
WorklogPRO → Worklog Calendar → Log Work

Select All Projects, then choose the project containing the malicious issue.

Select the issue and click Log Work.

The JavaScript payload executes immediately.

3. Bonus Trigger – Worklog Calendar View

While logging work on the malicious issue, set Time Spent = 3h and submit.

Navigate to:
WorklogPRO → Worklog Calendar

The stored XSS payload executes on page load, without further interaction.

Affected Versions

Confirmed in versions prior to 4.23.7

Fixed Version

The vulnerability was fixed in:

WorklogPRO 4.23.7 (30/07/25)
https://thestarware.atlassian.net/wiki/spaces/WLP/pages/824016944/Release+Notes+4.x

Disclosure

The vulnerability was responsibly reported through the vendor's program.
CVE Request ID: 1907945 (pending MITRE processing)
