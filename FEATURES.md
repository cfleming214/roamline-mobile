# Features

**Roamline** — A real local number in any country, live in 60 seconds.

Roamline turns your existing phone into a multi-country communications hub. Spin up a genuine local number in dozens of supported countries, receive SMS and voice calls in a clean chat-style inbox, and keep, rent, or retire each Line on your own schedule — no SIM swap, no carrier contract, no shipping delays.

---

## Core Features

### Instant Line Provisioning

Browse available numbers by country, city, and area code, then activate one in under 60 seconds with a single tap and no paperwork for most regions. Each provisioned number is a genuine carrier-grade local number — not a VoIP range that gets rejected by verification systems — and begins receiving traffic the moment activation completes.

**User benefit:** You can sign up for a Japanese delivery app or verify a German bank account tonight instead of waiting two weeks for a SIM to arrive at a hotel you may have already left.

**Tech note:** Numbers are drawn from a pooled inventory held across multiple upstream carrier partners per country, with a pre-warmed reserve for high-demand regions. Provisioning runs as an idempotent async job with optimistic UI; the client holds a soft reservation lock (90s TTL) while the carrier API confirms assignment, and rolls back cleanly on failure so the user is never charged for a number they didn't get.

### Unified Chat-Style Inbox

All SMS, MMS, and voice activity across every Line flows into one threaded inbox that reads like a familiar messaging app, with per-Line color coding and country flags so you always know which identity a message belongs to. Threads merge SMS, missed calls, and voicemail transcripts into a single chronological conversation per contact.

**User benefit:** Your Tokyo verification codes, Berlin bank alerts, and London landlord callbacks live side by side with zero confusion about which number to reply from — and never get lost in a separate app you forget to open.

**Tech note:** Messages persist to a per-user encrypted store with a local SQLite mirror for offline read access and optimistic send. Inbound traffic arrives via carrier webhooks into a normalization layer that collapses provider-specific payloads (Twilio, Vonage, regional aggregators) into a single canonical message schema, then fans out over WebSocket to active clients with APNs/FCM fallback.

### One-Tap Verification Code Capture

Roamline detects one-time passcodes in inbound SMS, surfaces them in the notification itself, and copies them to your clipboard with a single tap — no opening the app, no squinting at a 6-digit string. Codes are flagged with their likely sender service and auto-expire from the clipboard after use.

**User benefit:** Signing up for a foreign service becomes a two-second interaction instead of an app-switching scramble while a 60-second verification timer runs down.

**Tech note:** A lightweight on-device classifier combines regex patterns across 40+ locales with sender-ID heuristics and message structure scoring; extraction runs client-side so code contents are never parsed server-side beyond encrypted-at-rest storage. Notification payloads use mutable content with a notification service extension to render the code inline on iOS and an inline-reply action on Android.

### Live Voice Calling In and Out

Answer incoming calls to any Line directly in Roamline, or place outbound calls that display your local number as the caller ID — so a UK landlord sees a UK number ringing them back. Calls route over data (Wi-Fi or cellular) with automatic codec adaptation for weak connections.

**User benefit:** You get a callback number people will actually answer, and you can return that call yourself without burning international minutes or revealing your home number.

**Tech note:** WebRTC with Opus, DTX, and adaptive jitter buffering; native CallKit (iOS) and ConnectionService (Android) integration so calls appear in the system call UI and survive backgrounding. SIP trunking terminates at regional media servers (EU, NA, APAC, LATAM) selected by latency probe to keep one-way audio delay under 150ms for most routes.

### Flexible Line Lifecycle: Keep, Rent, Retire

Every Line can be held on a rolling monthly subscription, rented as a short-term Trip Line with a fixed end date, or retired instantly when you're done with it. Trip Lines show a countdown and a one-tap extend option, and any Line can be converted between modes without losing message history.

**User benefit:** You pay for a Tokyo number for the eleven days you're actually in Tokyo, while keeping your long-term German number running quietly in the background for years.

**Tech note:** Lifecycle is modeled as a state machine (reserved → active → grace → released) with scheduled transition jobs and a configurable grace window — 7 days by default — during which a retired number can be reclaimed before returning to the carrier pool. Billing proration is computed against the subscription ledger, and message history is retained independently of number assignment so records survive release.

### Country Eligibility & Compliance Engine

Before you commit, Roamline shows exactly what each country requires — some need nothing, others need a passport scan or proof of local address — plus what the number can and can't do, such as whether it accepts bank OTPs or short-code traffic. Required documents are collected in-app and tracked to approval with live status.

**User benefit:** You never pay for a number only to discover it can't receive the one message you needed it for, and regulated-market paperwork takes minutes instead of a week of email tag.

**Tech note:** A rules engine holds per-country regulatory profiles (KYC tier, address proof requirements, emergency-service obligations, short-code reachability matrices) versioned in a policy store and refreshed from partner feeds. Identity documents route to a third-party KYC provider over a tokenized flow; Roamline stores only the verification result and a reference token, never the raw document.

---

## Secondary Features

### Line Profiles & Smart Labels

Name each Line, attach a purpose and an emoji, and set a custom notification sound and vibration pattern so you know what's ringing without looking. Profiles can be set to Quiet Hours on local time for that number's country.

**User benefit:** A 3am Berlin bank alert doesn't wake you in Lisbon, and you instantly recognize which life a notification belongs to.

**Tech note:** Per-Line notification channels on Android and thread-identifier grouping on iOS; quiet-hours scheduling resolves against the Line's IANA timezone rather than device time, with DST-aware next-fire computation.

### Forwarding & Auto-Reply Rules

Route calls or SMS from any Line to your real number, another Line, email, or a webhook, with conditional rules based on sender, time of day, or keyword. Auto-replies support per-Line templates in the local language.

**User benefit:** You can keep a Trip Line reachable after you've gone home, or auto-reply "please use email" to a landlord without being on call.

**Tech note:** Rules compile to an ordered predicate chain evaluated at the routing layer before delivery, with a per-Line execution log for debugging. Webhook delivery uses signed payloads (HMAC-SHA256) with exponential-backoff retry and a dead-letter queue.

### Shared Lines & Team Access

Invite a partner, family member, or colleague to view and respond on a specific Line without sharing your account. Permissions are per-Line and granular: read-only, reply, or full management including retirement.

**User benefit:** A travel companion can handle the apartment check-in texts, or a co-founder can field calls on the market-entry number, without anyone handing over a password.

**Tech note:** Role-based access control scoped at the Line resource level, with per-member message visibility cursors so new members don't retroactively read prior history unless explicitly granted. Shared threads use presence indicators and optimistic-concurrency send tokens to avoid duplicate replies.

### Usage Dashboard & Spend Controls

See per-Line message counts, call minutes, and accrued cost in real time, with a projected month-end total and alerts at configurable thresholds. Hard caps can pause outbound spend on a Line automatically.

**User benefit:** No surprise bill after a long call to a foreign hotline, and clear visibility into which Lines are actually worth keeping.

**Tech note:** Usage events stream into a rated-ledger pipeline where each event is priced at ingest against the active tariff table, giving sub-minute dashboard freshness. Threshold evaluation runs on a rolling window with idempotent alerting to prevent duplicate notifications on retry.

---

## v2 Roadmap

- **Data eSIM bundles paired to a Line** — purchase local mobile data alongside a number in the same flow, so a single activation gets you online and reachable in a new country without a separate provider or QR-code hunt.
- **Business Lines with IVR and team routing** — menu trees, business-hours routing, ring groups, and shared voicemail, turning a Roamline number into a lightweight local presence for small companies entering a new market.
- **Number portability in and out** — port an existing foreign number into Roamline to consolidate it into the unified inbox, and port a Roamline number out to a traditional carrier if your needs change, with guided per-country port workflows and status tracking.