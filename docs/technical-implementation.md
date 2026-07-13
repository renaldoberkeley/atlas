# Atlas Technical Implementation

## Document Status

Version: 0.1  
Status: Draft  
Scope: MVP responsive web prototype

---

# 1. Purpose

This document describes the proposed technical implementation for the Atlas MVP.

The MVP will provide a responsive web experience where a user can describe a transportation need, answer conversational follow-up questions, and view a Ride Board comparing multiple transportation options.

The initial implementation will use mocked transportation data. It will not include real bookings, payments, authentication, live provider integrations, or community-driver transactions.

---

# 2. MVP Technical Goals

The initial implementation should:

- Provide a polished responsive web interface.
- Support a conversational trip-planning flow.
- Represent trip details using structured application state.
- Generate a Ride Board from mocked data.
- Display commercial, public-transit, autonomous, and community options.
- Make future backend and API integration possible without requiring it now.
- Be easy to test with consumers on desktop and mobile browsers.

---

# 3. Non-Goals

The first implementation will not include:

- Native iOS or Android applications.
- Real Uber, Lyft, Waymo, or transit-provider APIs.
- Real-time pricing.
- Real-time vehicle availability.
- User authentication.
- Persistent user accounts.
- Payments.
- Messaging.
- Community-driver verification.
- Maps or turn-by-turn navigation.
- Production AI model integration.
- A production marketplace backend.

These capabilities may be added after the product experience has been validated.

---

# 4. Technology Stack

## Frontend

- Next.js
- TypeScript
- React
- App Router
- Tailwind CSS

## Initial Data Layer

- Local TypeScript mock data
- No database for the first prototype
- No external transportation APIs

## Testing

- Vitest or Jest for unit tests
- React Testing Library for component behavior
- Playwright for critical end-to-end flows

## Code Quality

- ESLint
- Prettier
- Strict TypeScript configuration

---

# 5. Application Architecture

The MVP should use a simple layered structure:

```text
User Interface
    ↓
Trip Conversation State
    ↓
Trip Planning Rules
    ↓
Mock Transportation Provider Layer
    ↓
Ride Board View Model
```

## Responsibilities

### User Interface
Responsible for:
- User input
- Follow-up questions
- Ride Board rendering
- Loading, empty, and error states
- Responsive behavior

### Trip Conversation State
Responsible for tracking:
- Origin
- Destination
- Departure time
- Arrival time
- Passenger count
- Luggage
- User priority
- Accessibility needs
- Current conversation step

### Trip Planning Rules
Responsible for:
- Determining which required information is missing
- Selecting the next follow-up question
- Determining when the trip request is complete
- Producing a normalized trip request

### Mock Transportation Provider Layer
Responsible for returning mocked options for:
- Rideshare
- Autonomous ride
- Public transit
- Community drivers

### Ride Board View Model
Responsible for transforming provider data into consistent cards that the UI can render.

---

# 6. Proposed Directory Structure

```text
src/
  app/
    layout.tsx
    page.tsx
    globals.css

  components/
    trip-planner/
      TripPlanner.tsx
      ConversationMessage.tsx
      TripInput.tsx
      SuggestedPrompt.tsx

    ride-board/
      RideBoard.tsx
      RideOptionCard.tsx
      RideTimeline.tsx
      RecommendationBadge.tsx
      CommunityDriversCard.tsx
      CommunityDriverRow.tsx

    shared/
      Button.tsx
      Card.tsx
      Badge.tsx
      LoadingState.tsx
      EmptyState.tsx

  data/
    mockRideOptions.ts
    mockCommunityDrivers.ts

  lib/
    trip-planning/
      getNextQuestion.ts
      isTripRequestComplete.ts
      normalizeTripRequest.ts
      buildRideBoard.ts

  types/
    trip.ts
    ride-option.ts
    community-driver.ts

  tests/
    unit/
    integration/
    e2e/
```

The exact structure may evolve, but business logic should remain separate from presentation components.

---

# 7. Core Domain Models

## Trip Request

```typescript
type TripPriority =
  | "fastest"
  | "cheapest"
  | "best-value"
  | "fewest-transfers"
  | "most-comfortable";

interface TripRequest {
  origin?: string;
  destination?: string;
  departureTime?: string;
  arrivalTime?: string;
  passengerCount?: number;
  luggageCount?: number;
  priority?: TripPriority;
  accessibilityNeeds?: string[];
}
```

## Ride Option

```typescript
type RideOptionType =
  | "rideshare"
  | "autonomous"
  | "public-transit"
  | "community";

interface RideOption {
  id: string;
  type: RideOptionType;
  providerName: string;
  title: string;
  estimatedDurationMinutes: number;
  estimatedCostMin: number;
  estimatedCostMax: number;
  currency: "USD";
  bestFor: string;
  recommendationLabel?: string;
  timeline: RideTimelineStep[];
  actionLabel: string;
}
```

## Timeline Step

```typescript
interface RideTimelineStep {
  id: string;
  label: string;
  detail?: string;
  estimatedMinutes?: number;
}
```

## Community Driver

```typescript
interface CommunityDriver {
  id: string;
  displayName: string;
  rating: number;
  completedRideCount: number;
  vehicle: string;
  pickupTime: string;
  price: number;
  verified: boolean;
}
```

---

# 8. Trip Conversation Flow

The application should not simulate an unnecessarily long chatbot conversation.
The conversation exists to collect enough information to create a useful Ride Board.

## Required Information
For the initial MVP:
- Origin
- Destination
- Travel time

The user may provide several fields in one message.  
**Example:**  
*I need to go from Potrero Hill to SFO tomorrow at 8 AM.*  
Atlas should recognize that no additional required questions are necessary.

## Follow-Up Order
When information is missing, ask in this order:
1. Origin
2. Destination
3. Departure or arrival time
4. Passenger count, when relevant
5. Priority, when useful

Only ask one question at a time.

## Completion Rule
Once the minimum required fields are present, the application should generate the Ride Board.
Optional questions should not block the user from viewing results.

---

# 9. Natural-Language Handling for the Prototype

The first prototype does not need a production AI integration.
Use a deterministic parser that supports common demo phrases and falls back to follow-up questions.

**Examples:**
- "I need to get to SFO."
- "I need a ride from Potrero Hill to SFO."
- "Take me downtown tomorrow morning."
- "I need to get to Levi's Stadium by 6:30 PM."

The parser should be isolated behind a function or interface so it can later be replaced with an AI service.

**Example:**
```typescript
interface TripRequestParser {
  parse(input: string): Partial<TripRequest>;
}
```

The UI should not depend directly on a specific AI provider.

---

# 10. Ride Board Generation

The Ride Board should be generated from a normalized `TripRequest`.
For the MVP, the builder will:

- Read the trip request.
- Select mocked options appropriate for the scenario.
- Adjust basic values such as duration and cost from fixed mock rules.
- Assign recommendation labels.
- Return a consistent list of ride options.

**Example recommendation labels:**
- Recommended
- Fastest
- Cheapest
- Best Value

At least one option should be clearly recommended.

---

# 11. Community Driver Presentation

Community drivers should appear as a distinct transportation option within the Ride Board.
The card should initially display the top three drivers ranked by:
- Rating
- Completed rides
- Price

The interface should include a **Show more** action.
For the MVP, this may expand a mocked list on the same page. It does not need a separate marketplace backend.
Community rides must visually communicate that they are separate from Uber, Lyft, or Waymo.

---

# 12. Responsive Design

The MVP must be usable on:
- Mobile phones
- Tablets
- Desktop browsers

## Mobile
- Single-column layout
- Large touch targets
- Sticky or easily accessible trip input
- Cards stacked vertically
- Timeline details optimized for narrow screens

## Desktop
- Centered application shell
- Wider Ride Board cards
- Optional split layout between conversation and results
- No desktop-only functionality

The web application should be mobile-first because transportation decisions will often be made on a phone.

---

# 13. Accessibility

The MVP should:
- Support keyboard navigation.
- Use semantic HTML.
- Include visible focus states.
- Use labels for all form controls.
- Avoid relying solely on color to communicate recommendations.
- Maintain reasonable contrast.
- Support reduced-motion preferences where animation is used.

---

# 14. Application States

The UI should explicitly support:

## Initial State
- Introductory message
- Example prompts
- Empty trip input

## Gathering Information
- Previous messages
- Current follow-up question
- Input enabled

## Loading Results
- Temporary Ride Board loading state
- No fake multi-second delay unless useful for testing

## Results
- Completed Ride Board
- Recommended option
- Ability to revise the trip

## Error State
- Friendly explanation
- Preserve the user's current trip information
- Allow retry

---

# 15. Testing Strategy

## Unit Tests
Test:
- Missing-field detection
- Follow-up question selection
- Trip completion rules
- Mock recommendation logic
- Cost and duration formatting

## Component Tests
Test:
- Trip input submission
- Follow-up rendering
- Ride option cards
- Community driver expansion
- Mobile interaction behavior

## End-to-End Tests
At minimum:

**Flow 1**
- User enters: "I need to get to SFO."
- Atlas asks for origin.
- User enters: "Potrero Hill."
- Atlas asks for time.
- User enters: "Tomorrow at 8 AM."
- Ride Board appears.

**Flow 2**
- User enters a complete request.
- Ride Board appears without unnecessary follow-up questions.

**Flow 3**
- User opens Community Drivers.
- Top three drivers appear.
- User selects Show more.
- Additional drivers appear.

---

# 16. Analytics and Product Validation

The initial prototype should support a simple analytics abstraction, even if no analytics provider is configured yet.
Important future events include:

- `trip_request_started`
- `follow_up_question_answered`
- `ride_board_viewed`
- `ride_option_selected`
- `community_drivers_expanded`
- `trip_request_revised`

Do not tightly couple application components to a specific analytics vendor.

**Example:**
```typescript
trackEvent("ride_board_viewed", {
  optionCount: 4,
  hasCommunityDrivers: true,
});
```

This matters because the MVP is intended for consumer testing.

---

# 17. Security and Privacy

The prototype should collect only the information required to demonstrate the experience.
It should not:
- Store precise home addresses permanently.
- Collect payment information.
- Collect driver's-license information.
- Claim that mock drivers are real.
- Present mocked prices as live pricing.

Mock data should be clearly identified during testing where confusion is possible.

---

# 18. Future Integration Boundaries

The implementation should allow future replacement of mocked layers with real services.
Potential future interfaces include:

```typescript
interface TransportationProvider {
  getOptions(request: TripRequest): Promise<RideOption[]>;
}
```

Possible implementations:
- Rideshare provider
- Transit provider
- Autonomous-vehicle provider
- Community marketplace provider

Future infrastructure may include:
- API routes or a dedicated backend
- PostgreSQL
- Authentication
- Geocoding
- Route estimation
- Payment processing
- Messaging
- Driver verification

These should not be implemented during the initial prototype.

---

# 19. Implementation Sequence

**Milestone 1: Project Setup**
- Initialize Next.js
- Configure TypeScript
- Configure Tailwind
- Configure linting and formatting
- Add test framework

**Milestone 2: Static Ride Board**
- Build reusable option cards
- Build timelines
- Build community-driver card
- Add mocked data
- Validate responsive layout

**Milestone 3: Trip Conversation**
- Add trip input
- Add conversation state
- Add deterministic parsing
- Add follow-up logic

**Milestone 4: Connect Conversation to Results**
- Normalize trip requests
- Generate mocked Ride Board
- Support revising a request

**Milestone 5: Test Readiness**
- Add analytics abstraction
- Add automated tests
- Add accessibility review
- Prepare representative testing scenarios

---

# 20. Definition of Done

The MVP technical implementation is complete when:
- A user can enter a natural-language transportation request.
- Atlas asks only for missing required information.
- A completed request produces a Ride Board.
- The Ride Board displays four transportation categories.
- Community Drivers displays three initial drivers and supports **Show more**.
- The experience works on mobile and desktop.
- Key conversation and Ride Board logic is tested.
- No real provider, booking, payment, or marketplace functionality is implied.
- The code is organized so mocked services can later be replaced.
