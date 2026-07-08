# Codex Instructions for Atlas

## Project Codename
Atlas

## Future Public Product Name
rideboard.ai

Do not use the public product name throughout the app yet. Use "Atlas" internally.

## Product Vision
Atlas is an AI-powered transportation marketplace that helps people discover, compare, and eventually book the smartest way to get from one place to another.

The app should compare:
- Rideshare options
- Public transit
- Autonomous rides
- Community ride offers from regular people
- Future transportation providers

## Core User Experience
The user should be able to type a natural language request such as:

"I need a ride from Potrero Hill to SFO."

The app should ask useful follow-up questions if needed:
- Where are you leaving from?
- Where are you going?
- When do you need to leave or arrive?
- How many passengers?
- Do you have luggage?
- Do you prefer fastest, cheapest, or most convenient?

Then the app should show a Ride Board with timeline cards.

## MVP Ride Board
For now, use mocked data.

Show 4 main option types:

1. Uber / Lyft
2. Waymo
3. Public Transit
4. Community Drivers

Each option card should include:
- Option name
- Estimated time
- Estimated cost
- Best-for label
- Step-by-step timeline
- CTA button

Community Drivers should show:
- Top 3 drivers
- Name
- Rating
- Ride count
- Price
- Vehicle
- “Show more” button

## Product Principles
- Build for consumers, not just a demo.
- Prioritize clarity over technical complexity.
- The UI should feel trustworthy, modern, and mobile-friendly.
- Do not overbuild maps, payments, or authentication in the first version.
- Start with a polished fake-data prototype.
- Avoid airport-specific naming because Atlas is for rides anywhere.

## Initial Tech Direction
Use:
- Next.js
- TypeScript
- Tailwind CSS
- App Router
- Component-based structure

Suggested structure:

src/
  app/
    page.tsx
  components/
    RideChat.tsx
    RideResultsBoard.tsx
    RideOptionCard.tsx
    CommunityDriversCard.tsx
  data/
    mockRideOptions.ts
  types/
    ride.ts

## First Build Goal
Create a beautiful responsive prototype with:
- Landing/chat interface
- Example prompt input
- Ride Board results
- Timeline cards
- Community driver section

Do not add:
- Real APIs
- Database
- Authentication
- Payments
- Native mobile apps

Those come later.

## Long-Term Marketplace Concept
Atlas should eventually allow regular people to offer rides.

Example:
A driver says:

"I'm driving from Walnut Creek to San Francisco tomorrow morning at 8:30. I have 2 open seats for $15 each."

Riders can discover and request those rides.

Important future features:
- Driver verification
- Ratings
- Ride history
- Messaging
- Payment escrow
- Safety reporting
- Insurance/legal review

## Validation Strategy
Atlas needs testers early.

Add docs later for:
- UI testing
- user interviews
- founding drivers
- city/community beta
- trust and safety validation

## Design Tone
The product should feel like:
- A smart travel concierge
- A clean transportation dashboard
- A trustworthy marketplace

Avoid making it feel like:
- A generic chatbot
- A clone of Uber
- A map-only app

## Engineering Principles

Before implementing any feature:

1. Read the relevant documentation in `/docs`.
2. Reuse existing components whenever possible.
3. Keep code modular and clearly organized and documented where useful.
4. Prefer maintainability over cleverness.
5. If requirements are ambiguous, explain the tradeoffs before implementing.
6. Do not introduce unnecessary dependencies.
7. Build desktop and mobile responsive layouts from the beginning.
8. Write production-quality code rather than prototype-quality code unless explicitly instructed otherwise.
9. Keep commits focused and small.