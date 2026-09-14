# System Overview

> **Instructions:** Replace this content with your project's description.
> This is the first page someone new reads. They must be able to understand the system in 5 minutes.
> Remove these instructions when the document is complete.

---

## What is FixGo?

FixGo is a mobile roadside-assistance platform that connects drivers experiencing an
unexpected vehicle breakdown with nearby available mechanics, so that help can be
requested, tracked, and confirmed without relying on phone calls or informal referrals
(SRS 1.1–1.2).

## Problem it solves

**Before the system:** Drivers with a vehicle breakdown had to search informally for a
mechanic — phone directories, word of mouth, or roadside luck — with no visibility into
availability, distance, or estimated arrival time. Independent mechanics had no digital
channel to be found by nearby drivers, losing potential work.

**With the system:** A driver registers the breakdown once, and FixGo automatically
matches it to a nearby available mechanic, shows the mechanic's live location and ETA,
and keeps a record of the service from request to completion. The mechanic gains a
visible, organized channel for dispatch requests instead of relying on informal contacts.

## Main users

| Role | Description | What they do in the system |
|---|---|---|
| **Driver** | Person requesting roadside assistance for a registered vehicle | Registers vehicles, creates service requests, tracks the assigned mechanic in real time, confirms completion |
| **Mechanic** | Verified technician or workshop offering on-site assistance | Receives and accepts dispatch requests, updates request status, registers the diagnostic on completion |
| **Administrator** | Platform operator | Verifies mechanic registrations, consults audit logs (SRS Module 7) |

## Technology stack

| Layer | Technology | Justification |
|---|---|---|
| Frontend | Flutter / Android | Cross-platform, native GPS access |
| Backend | Java 17 + Spring Boot | Team's existing expertise; strong support for structured business logic and REST APIs |
| Transactional database | MySQL | Driver, Vehicle, Mechanic, and ServiceRequest have strict relational integrity requirements (see `06-data/models.md`) |
| Real-time layer | Firebase Realtime Database | Live GPS position and request-status sync, per SRS RF4.2 |
| Notifications | Firebase Cloud Messaging (FCM) | Decided for this remediation: push notifications for status changes (SRS RF4.5). Kafka was considered in the team repository but is infrastructure overhead this project's scale does not need — FCM alone covers every notification requirement in the SRS |
| Infrastructure | Cloud hosting, AES-256 at rest | Encrypts stored PII per `04-requirements/non-functional.md` |

## Current status

- **Phase:** Documentation remediation (individual), in progress
- **Reference:** rewritten from `fixgo-docs` (team repository, branch `chore/initial-docs`)
  after the ADSO-3239188 governance evaluation of 11-sep-2026
- **Next milestone:** Complete sections 00–02 for Entrega 1 (14-sep-2026)

## Project contacts

| Role | Name |
|---|---|
| Tech Lead (team repo) | Johan Andrés Liñan Esquivel |
| This repository's author | Gabriel Tijaro Jimenez|
| Instructor | Javier Humberto Pinto | 