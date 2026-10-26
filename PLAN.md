# Roamline

## Overview

Roamline gives travellers, remote teams and cross-border sellers a **real local phone number in any country, live in 60 seconds**. Pick a country, pick a city prefix, tap once — and calls plus SMS start flowing into the app over data, with no SIM swap, no carrier contract and no roaming bill.

The app is built with Expo (React Native) for a single codebase across iOS and Android, using a managed workflow with config plugins for the native telephony SDK. Numbers are provisioned through a programmable telephony provider (Twilio/Telnyx) behind our own Node API, so inventory, compliance documents and per-country regulatory rules are handled server-side while the client stays thin, fast and auditable.

The core promise is speed: a user lands on the country picker, sees live inventory with real monthly prices, completes checkout via native in-app purchase or card, and receives an active number with a working inbound call within one minute. Everything after that — call history, SMS threads, voicemail transcripts, forwarding rules — lives in a familiar messaging-style UI.

## Tech Stack

- **Expo SDK 51 (React Native 0.74)** with expo-router for file-based navigation, EAS Build and EAS Update for over-the-air patching
- **TypeScript + Zustand + TanStack Query** for typed state, optimistic mutations and offline-tolerant caching of call/SMS history
- **react-native-callkeep + @config-plugins/react-native-webrtc** to render native incoming call UI (CallKit / ConnectionService) and carry audio over WebRTC
- **Supabase (Postgres, Realtime, Auth)** for user accounts, number ownership records, message threads and live inbound event push
- **Node/Fastify API on Fly.io** wrapping Twilio Programmable Voice & Messaging, Stripe and RevenueCat webhooks for provisioning and billing

## Features

1. **60-Second Provisioning** — Country and area-code picker backed by live carrier inventory; one tap reserves, pays for and activates a number, with a progress ring that finishes in under a minute.
2. **Native Incoming Calls** — Inbound calls ring the full-screen OS call UI via CallKeep, so Roamline behaves like the system dialler even when the app is backgrounded or the device is locked.
3. **SMS & MMS Threads** — Two-way messaging with a familiar chat layout, delivery receipts, image attachments and per-number thread separation.
4. **Multi-Number Wallet** — Hold several numbers across countries simultaneously; swipe the header to switch the active identity for outbound calls and texts.
5. **Smart Forwarding Rules** — Route a number to your real SIM, to another Roamline number or to voicemail based on schedule, caller allowlist or do-not-disturb hours.
6. **Voicemail Transcription** — Missed calls become searchable text via Whisper, with audio playback and one-tap callback from the transcript card.
7. **Compliance Wizard** — For regulated countries (DE, ES, AU, IN) the app collects proof of address and ID in-flow, tracks approval status and auto-activates once the carrier clears the document.
8. **Burner Mode** — Spin up a disposable 7-day number for marketplace listings or sign-ups, with auto-expiry, one-tap release and no data retained after deletion.

## Key Screens

- **Country Picker / Instant Buy** — searchable country list with flags, live pricing, capability badges (voice, SMS, MMS) and an inline area-code selector
- **Number Dashboard** — carousel of owned numbers showing status, renewal date, monthly usage meter and quick actions for forwarding or release
- **Unified Inbox** — merged call-log and SMS feed with filter chips per number, unread badges and swipe-to-call/archive gestures
- **Conversation View** — chat thread with composer, attachment picker, caller ID header and an in-thread call button
- **Settings & Billing** — subscription tier, payment method, compliance document status, call-quality diagnostics and data-usage preferences

## Core Components

- **`<CallSessionProvider>`** — context wrapping WebRTC peer connection, CallKeep bridge, audio-route switching and call-state machine (ringing → active → held → ended)
- **`<NumberCard>`** — reusable animated card rendering flag, formatted E.164 number, status pill, usage bar and contextual action sheet
- **`<ProvisionFlow>`** — multi-step wizard orchestrating inventory lock, payment, compliance upload and activation polling with rollback on failure
- **`<ThreadComposer>`** — message input handling attachments, character/segment counting, optimistic send and retry queue for offline messages

## Roadmap

### Week 1
- Scaffold Expo router app, Supabase auth, TypeScript config and EAS build profiles for dev/preview/prod
- Build country picker with mocked inventory API and the full purchase UI shell
- Stand up Fastify API with Twilio sandbox provisioning and number-ownership schema

### Week 2-3
- Integrate react-native-webrtc + CallKeep for inbound/outbound voice with native call UI on both platforms
- Ship SMS/MMS threads with Realtime subscriptions, optimistic send and push notifications via expo-notifications
- Implement Stripe + RevenueCat billing, renewal webhooks, forwarding rules engine and voicemail transcription pipeline

### Week 4+
- Add compliance document wizard, burner mode and multi-number switching with per-number analytics
- Harden call quality: ICE/TURN fallback, codec negotiation, network-change recovery and diagnostics screen
- Submit to App Store and Play Store, wire EAS Update channels and launch a referral/invite loop

## Monetisation

Roamline runs on a **number subscription model**: each number costs $4.99–$14.99/month depending on country tier, billed via native in-app purchase (RevenueCat) on mobile and Stripe on web. A free trial grants one 7-day burner number so users experience activation before paying.

Usage beyond the included bundle (300 minutes / 500 SMS) draws from prepaid **Roam Credits** sold in $5/$15/$40 packs with a margin of roughly 40% over wholesale carrier rates. Outbound international calls and premium short-code SMS are credit-metered.

**Roamline Pro** at $19.99/month bundles three numbers, unlimited domestic minutes, voicemail transcription, call recording and priority support. A **Teams** tier ($12/seat/month) adds shared inboxes, admin provisioning, SSO and consolidated invoicing for distributed companies — the highest-LTV segment and the primary expansion path after consumer launch.