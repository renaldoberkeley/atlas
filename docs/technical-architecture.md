# Atlas Technical Architecture

## Document Status

Version: 1.0-draft
Status: Draft
Scope: MVP to scalable production platform
Primary audience: Engineering, product, and technical leadership

---

## 1. Purpose

### Goals

- Define Atlas as a Next.js frontend with a FastAPI backend.
- Establish clear architectural boundaries for planning, recommendation, and marketplace capabilities.
- Provide an implementation path from mocked MVP to real provider integrations and community marketplace operations.
- Create a shared engineering language for long-term system evolution.

### Scope

This document covers:
- Product and engineering boundaries
- Architecture principles
- Frontend and backend architecture
- Domain models
- API contracts
- AI interaction boundaries
- Recommendation strategy
- Marketplace architecture
- Security, deployment, analytics, and testing
- Near-term implementation sequence and future architecture direction

### Non-goals

This document does not define:
- Final UI copy and visual design system details
- Final commercial terms with transportation partners
- Vendor-specific lock-in decisions for AI, payments, or cloud
- Irreversible implementation constraints for the MVP

---

## 2. Engineering Principles

Engineers should build Atlas with the following operating principles:

- Build for clarity first, then optimize for scale.
- Keep deterministic business rules separate from probabilistic AI behavior.
- Make every major component independently testable.
- Prefer explicit interfaces and typed contracts over implicit coupling.
- Keep frontend responsibilities presentation-focused.
- Keep backend responsibilities domain-focused.
- Use mocks that behave like realistic providers during the MVP.
- Design every integration boundary to be replaceable.

### Architectural Principles

- Atlas should never depend on a single transportation provider.
- Business logic belongs in Python.
- The frontend should remain presentation-focused.
- Every provider should implement the same interface.
- AI should interpret language, not make business decisions.
- Every major component should be independently testable.
- Every API should be replaceable.
- Mock providers should behave like real providers.
- No component should directly depend on an LLM vendor.
- Every feature should support future marketplace expansion.

---

## 3. High-Level System Architecture

```text
                Browser
                   |
             Next.js Frontend
                   |
          FastAPI Backend (Python)
        +----------+-----------+
        |          |           |
    Trip Planner  Provider Hub  AI Parser
        |          |
   Recommendation Engine
        |
Transportation Providers
        |
Community Marketplace
        |
      PostgreSQL
```

### Responsibility split

- Frontend: Collect input, orchestrate UX state, and present Ride Board results.
- Backend: Parse requests, aggregate providers, rank recommendations, and enforce domain logic.
- Data layer: Persist users, trips, marketplace entities, and analytics events as the system matures.

---

## 4. Repository Structure

Recommended target structure:

```text
atlas/
  apps/
    web/
    api/

  packages/
    shared-types/
    ui/

  docs/

  infrastructure/

  docker/

  scripts/
```

### Notes

- Keep apps deployable independently.
- Keep shared contracts in packages/shared-types.
- Keep design system primitives in packages/ui.
- Keep IaC and environment setup in infrastructure.

---

## 5. Frontend Architecture

### Technology direction

- Next.js with App Router
- TypeScript in strict mode
- React Server Components where useful
- Client Components for interactive conversation and dynamic Ride Board interactions
- Tailwind CSS for rapid, consistent UI development

### Frontend responsibilities

- Trip conversation UI
- Form and message input
- Follow-up prompts and clarification questions
- Ride Board rendering and revision workflow
- Loading, empty, and error states
- Responsive behavior across mobile, tablet, and desktop
- Accessibility-first interaction patterns

### State model

- Conversation state: messages, current question, completion state
- Trip draft state: origin, destination, time, party size, preferences
- Result state: ride options, recommendation labels, community drivers

### Application states

- Initial state
- Gathering information
- Loading results
- Results
- Error and retry

---

## 6. Backend Architecture

### Core stack

- FastAPI
- Pydantic models
- Async I/O for provider calls
- PostgreSQL (introduced when persistence is required)
- Optional Redis/queue for background workflows when needed

### Backend service layers

- API layer: HTTP endpoints, validation, auth hooks
- Parser layer: natural language to structured TripRequest
- Trip planner layer: completion checks and normalization
- Provider hub layer: provider fan-out and response normalization
- Recommendation layer: deterministic scoring and labels
- Marketplace layer: community ride offer and request lifecycle

### Provider abstraction

All providers implement a shared interface so integrations are swappable:

```python
class TransportationProvider(ABC):
    @abstractmethod
    async def get_options(self, request: TripRequest) -> list[RideOption]:
        raise NotImplementedError
```

### DI and modularity

- Use dependency injection for parser, providers, and scoring strategies.
- Keep transport/provider adapters isolated from core domain logic.
- Ensure each layer can be unit-tested without live external dependencies.

---

## 7. Domain Models

Core domain language for Atlas:

- Trip
- Passenger
- Driver
- RideRequest
- RideOffer
- Ride
- RideBoard
- TransportationOption
- Timeline
- Recommendation

### MVP TripRequest model

```python
class TripRequest(BaseModel):
    origin: str | None = None
    destination: str | None = None
    departure_time: datetime | None = None
    arrival_time: datetime | None = None
    passenger_count: int | None = Field(default=None, ge=1, le=8)
    luggage_count: int | None = Field(default=None, ge=0)
    priority: TripPriority | None = None
    accessibility_needs: list[str] = Field(default_factory=list)
```

### MVP RideOption model intent

Each option should include:
- Provider identity
- Duration and cost estimates
- Recommendation label
- Timeline segments
- User action metadata

---

## 8. API Architecture

### API design goals

- Stable and versioned endpoints
- Explicit schemas for request and response payloads
- Provider-independent response contracts
- Clear separation between parse, plan, and marketplace capabilities

### Initial endpoints

```http
POST /api/v1/trips/parse
POST /api/v1/ride-board
POST /api/v1/community-rides
GET  /api/v1/driver/{id}
GET  /api/v1/trip-history
GET  /api/v1/health
```

### Parse endpoint behavior

Input: free-form user message.
Output: structured TripRequest, missing fields, and optional next question.

### Ride Board endpoint behavior

Input: normalized trip request.
Output: ranked options with one or more recommendation labels.

---

## 9. AI Architecture

The AI system extracts intent. It does not own planning or policy decisions.

```text
Conversation
  -> Intent Parser
  -> TripRequest
  -> Trip Planner
  -> Provider Aggregation
  -> Recommendation Engine
  -> Ride Board
```

### Constraints

- LLM output must be validated against typed schemas.
- Deterministic logic must govern pricing rules, ranking policy, and eligibility.
- AI vendor adapters must be abstracted behind an internal parser interface.

---

## 10. Recommendation Engine

Recommendation quality becomes Atlas core differentiation.

### Inputs

- Price
- Travel time
- Transfer count
- Walking distance
- Accessibility constraints
- User preferences
- Past rides and driver affinity
- Ratings and trust signals
- Traffic and event context
- Safety heuristics

### Output

- Ranked transportation options
- Human-readable recommendation labels (Recommended, Fastest, Cheapest, Best Value)
- Explanation fields suitable for UI transparency

### Principles

- Deterministic by default for auditability
- Tunable weighting strategy
- Offline evaluation support for future model improvements

---

## 11. Marketplace Architecture

### Driver lifecycle

- Onboarding
- Verification
- Offer creation
- Availability updates
- Completion and rating feedback

### Passenger lifecycle

- Request creation
- Option selection
- Match confirmation
- Completion and review

### Matching scope

- Start with lightweight marketplace presentation in Ride Board
- Expand toward real supply-demand matching as trust, verification, and payments mature

---

## 12. Security and Privacy

### Security controls

- Authentication and authorization boundaries
- Rate limiting and abuse controls
- Fraud and spam safeguards
- Input validation at all API boundaries

### Privacy controls

- Minimize collection of PII in MVP
- Clear handling policies for location and trip data
- Data retention controls by environment

### MVP restrictions

- No payment collection
- No permanent storage of sensitive address data unless necessary
- No claims that mocked providers or mocked drivers are real

---

## 13. Deployment Architecture

### Environments

- Development
- Testing/Staging
- Production

### Platform expectations

- Containerized services with Docker
- CI/CD pipeline for lint, test, and deployment gates
- Infrastructure-as-code for reproducible environments
- Runtime monitoring and centralized logging

### Observability

- Request/response metrics
- Provider latency and error metrics
- Recommendation pipeline timing
- Marketplace funnel health indicators

---

## 14. Analytics Architecture

### Core events

- trip_request_started
- follow_up_question_answered
- ride_board_viewed
- ride_option_selected
- community_drivers_expanded
- trip_request_revised

### Product analytics goals

- Funnel conversion analysis
- Ride Board interaction quality
- Marketplace growth and completion signals
- Retention and repeat-request behavior

### Design rule

Use an analytics abstraction so product instrumentation is vendor-independent.

---

## 15. Testing Strategy

### Unit testing

- Trip parsing adapters
- Missing-field and follow-up logic
- Recommendation scoring rules
- Provider adapter transformations

### Integration testing

- API contracts and schema validation
- Provider hub aggregation behavior
- Ride Board generation from normalized TripRequest inputs

### End-to-end testing

- Incomplete request to follow-up to completed Ride Board
- Complete request path with no unnecessary follow-up
- Community drivers expansion flows
- Mobile and desktop critical paths

### Non-functional testing

- Accessibility checks
- Load and concurrency checks for provider fan-out
- Failure-path resilience (timeouts, degraded providers)

---

## 16. Future Architecture

Planned expansion areas:

- Native mobile apps
- Real provider integrations
- Real-time pricing updates
- Calendar and commute intelligence
- Event transportation planning
- Corporate commuting workflows
- AI agents for planning assistance
- Enhanced marketplace trust and verification systems

---

## 17. Implementation Sequence

### Milestone 1: Foundation

- Create web and API apps
- Establish shared contracts and base models
- Define parser, provider, and recommendation interfaces

### Milestone 2: MVP experience

- Build trip conversation and Ride Board UX
- Implement deterministic parser and trip completion logic
- Add mock provider hub and recommendation labeling

### Milestone 3: Integration readiness

- Stand up API contracts and health checks
- Connect frontend to backend endpoints
- Add analytics abstraction and baseline instrumentation

### Milestone 4: Validation and hardening

- Expand automated test coverage
- Add accessibility and responsive QA
- Run consumer testing scenarios and iterate

---

## 18. Definition of Done (MVP)

MVP architecture is successful when:

- A user can submit a natural-language trip request.
- System asks only required follow-up questions.
- A complete request returns a Ride Board with distinct transportation categories.
- Community driver options are visible and expandable.
- Mobile and desktop experiences are usable and test-validated.
- Core planner, parser boundary, and recommendation logic are tested.
- No real booking, payment, or provider claims are implied.
- Mock integrations can be replaced by real services without rewriting core domain logic.
