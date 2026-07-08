# Atlas Product Requirements (MVP)

Version: 0.1

Status: Draft

---

# Purpose

The goal of the MVP is to allow a user to describe where they want to go using natural language.

Atlas should collect any missing information through conversational follow-up questions and then present a Ride Board comparing available transportation options.

The MVP is intended to validate the product experience, not build a complete transportation marketplace.

---

# Primary User Story

As a traveler,

I want to describe where I need to go,

So Atlas can recommend the best transportation options for me.

---

# Secondary User Story

As a driver,

I want to offer a ride,

So nearby travelers can request a seat.

---

# MVP Scope

Included

✅ Conversational trip planning

✅ Ride Board

✅ Mock transportation providers

✅ Mock community drivers

✅ Responsive desktop/mobile UI

Not Included

❌ Authentication

❌ Payments

❌ Real Uber integration

❌ Real Waymo integration

❌ Messaging

❌ Driver verification

❌ Maps

---

# Functional Requirements

## FR-001

User can type a natural language request.

Examples:

"I need to get to SFO."

"I need a ride downtown."

"I need to get to work tomorrow morning."

---

## FR-002

Atlas identifies missing information.

Example:

Destination missing

↓

Ask:

"Where are you going?"

---

## FR-003

Atlas asks follow-up questions one at a time.

Possible questions:

Origin

Destination

Departure time

Arrival time

Passengers

Luggage

Transportation preference

Budget

Accessibility needs

---

## FR-004

Atlas generates a Ride Board.

Ride Board contains:

Uber / Lyft

Waymo

Public Transit

Community Drivers

---

## FR-005

Each transportation option contains:

Provider

Estimated cost

Estimated travel time

Timeline

Best For

Action button

---

## FR-006

Community Drivers section

Display:

Top three drivers

Name

Vehicle

Price

Rating

Completed rides

Pickup time

Button:

Show More

---

## FR-007

Ride Board recommendations

Atlas should recommend one option.

Example:

Recommended

Fastest

Cheapest

Best Value

---

# Non-Functional Requirements

Fast

Responsive

Mobile-first

Accessible

Modern UI

Simple interaction

---

# Error Handling

If origin is unknown

↓

Ask for it.

If destination is unknown

↓

Ask for it.

If Atlas cannot determine enough information

↓

Continue asking questions.

Never display an empty Ride Board.

---

# Acceptance Criteria

Given

"I need a ride."

Atlas should ask where the user is leaving from.

Then ask where they are going.

Then produce a Ride Board.

The Ride Board should always contain four transportation categories using mock data.

---

# Future Features

User accounts

Driver profiles

Payments

Messaging

Live pricing

Maps

AI trip optimization

Ride history

Recurring commutes

Scheduled rides

Events

Groups

Corporate commuting

Trust & Safety

Insurance

Verification

# MVP Success Criteria

The MVP is successful if:

- A first-time user can request a ride in under 30 seconds.
- Users understand the Ride Board without explanation.
- Users can identify the recommended option immediately.
- At least 80% of test users say they would use Atlas again.
- Users express interest in community ride options.