---
name: instrumentation-walkthrough
description: This skill should be used when the user asks to "build an interactive demo", "create an instrumentation walkthrough", "build a demo for [customer]", "prep a walkthrough for [customer]", "generate a tailored demo", or mentions "instrumentation walkthrough" for a specific customer. Generates a customer-specific Amplitude instrumentation demo HTML file by looking up the customer in Salesforce and pre-configuring the correct products and CDP path.
version: 1.0.0
---

# Amplitude Instrumentation Walkthrough Generator

Generates a customer-tailored Amplitude instrumentation walkthrough HTML file before a customer call.

## When This Skill Applies

Activate when the user says anything like:
- "build me an interactive demo for [customer]"
- "instrumentation walkthrough for [customer]"
- "create a demo for [customer]"
- "prep a walkthrough for my call with [customer]"
- "generate a tailored demo for [customer]"

Extract the customer name from the phrase and proceed through the steps below.

---

## Step 1 — Look up the customer in Salesforce

Use the Salesforce MCP tools to find the account. Try `salesforce_run_soql_live` with:

```sql
SELECT Id, Name, Amplitude_Products__c, Tech_Stack__c, Type, Description, Website
FROM Account
WHERE Name LIKE '%[customer name]%'
LIMIT 5
```

If those field names return no data, use `salesforce_describe_object_live` on the `Account` object to find the right field names for products and tech stack, then re-query.

Also check the most recent opportunity for contracted products:

```sql
SELECT Id, Name, StageName, CloseDate,
  (SELECT PricebookEntry.Product2.Name FROM OpportunityLineItems)
FROM Opportunity
WHERE AccountId = '[account id]'
ORDER BY CloseDate DESC
LIMIT 3
```

From this, determine:
- **products** — which Amplitude products they have contracted: Analytics, Experiment, Guides & Surveys, Session Replay
- **cdp** — their analytics/CDP tool: Amplitude Browser SDK, Segment, mParticle, RudderStack, or other

---

## Step 2 — Confirm details with the user

Present a summary of what was found in Salesforce and ask the user to confirm or correct before proceeding. Format it clearly, for example:

```
Here's what I found for FanDuel in Salesforce — does this look right for your call?

  Products:  Analytics, Experiment, Guides & Surveys, Session Replay
  CDP:       Amplitude native (Amplitude Enterprise CDP on contract)
  API key:   not found — paste live during the call

Reply with any corrections, or say "looks good" to generate the file.
```

Wait for the user's confirmation before proceeding to Step 3.

---

## Step 3 — Ask follow-up questions for any gaps

After confirmation, if any of the following are still unclear, ask the user. Ask all open questions at once, not one by one.

1. **Products in scope** — "Which products should the demo cover? (Analytics, Experiment, Guides & Surveys, Session Replay)"
2. **Analytics stack** — "Is the customer using Amplitude Browser SDK directly, or a third-party CDP like Segment or mParticle?"
3. **API key** — "Do you have their Amplitude API key? (optional — they can paste it live during the call)"

Skip any question that Salesforce already answered clearly.

---

## Step 4 — Build the CUSTOMER_CONFIG

Map what you found to this exact config shape:

```js
const CUSTOMER_CONFIG = {
  customer: "[Customer Name]",
  products: [...],   // subset of: "analytics", "experiment", "guides", "session_replay"
  cdp: "...",        // "amplitude" | "segment" | "rudderstack" | "mparticle"
  apiKey: "",        // leave empty string if unknown
};
```

**Product mapping:**
- Always include `"analytics"` unless explicitly told it's out of scope
- Guides & Surveys → `"guides"`
- Session Replay → `"session_replay"`

**CDP mapping:**
- Amplitude Browser SDK → `"amplitude"` (plugin paths + `initializeWithAmplitudeAnalytics`)
- Segment → `"segment"` (standalone paths with Segment snippets)
- mParticle, RudderStack, or any other CDP → treat as `"segment"` (standalone paths apply)

---

## Step 5 — Fetch the HTML template from GitHub

Use `WebFetch` to get the latest template:

```
https://raw.githubusercontent.com/amp-anuj/Amplitude-Instrumentation-Walkthrough/main/amplitude-sdk-demo.html
```

In the fetched HTML, find and replace the `CUSTOMER_CONFIG` block near the top of the `<script>` tag. It looks like this:

```js
const CUSTOMER_CONFIG = {
  customer: "",          // e.g. "Acme Corp" — shown in header
  products: ["analytics", "experiment", "guides", "session_replay"],
  cdp: "amplitude",     // "amplitude" | "segment" | "rudderstack" | "mparticle"
  apiKey: "",           // pre-filled into all SDK init inputs
};
```

Replace it with the customer-specific values from Step 3. Keep all other HTML untouched.

---

## Step 6 — Save the file

Save the output to the current working directory as:

```
[customer-slug]-demo.html
```

where `[customer-slug]` is the customer name lowercased with spaces replaced by hyphens.
Example: "Acme Corp" → `acme-corp-demo.html`

---

## Step 7 — Report back to the user

Tell the user:
- The filename and that it was saved to the current directory
- Which tabs are shown and which are hidden
- Which path was pre-selected (Amplitude native vs Segment/standalone)
- Whether the API key was pre-filled or needs to be pasted live
- What came from Salesforce vs what was assumed
- A reminder: **open in Chrome** and have the Amplitude Chrome Extension ready for live event debugging

### Example summary

```
Generated: acme-corp-demo.html

Config applied:
  Customer:  Acme Corp
  Products:  Analytics, Experiment, Session Replay  (Guides & Surveys hidden — not on contract)
  CDP:       Segment → standalone paths pre-selected on Experiment, G&S, and SR tabs
  API key:   not pre-filled (paste live during call)

Salesforce: Experiment + SR confirmed on contract. Guides not found.
            Tech stack: Segment device-mode per opportunity notes.

Tip: Open in Chrome with the Amplitude Chrome Extension for live event debugging.
```
