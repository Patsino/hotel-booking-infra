# Stakeholders & Users

## Target Audience

| Persona | Description | Key Needs |
|---------|-------------|-----------|
| Traveler (User) | Individuals seeking hotel accommodations for business or leisure travel | Easy search and booking, secure payments, flexible cancellation options |
| Hotel Owner | Property owners or managers wanting to list and manage their hotels | Hotel registration, room management, booking visibility, approval workflow |
| Platform Administrator | System operators responsible for platform oversight | User management, hotel approval, cancellation request handling, system monitoring |

## User Personas

### Persona 1: Pavlo the Traveler

| Attribute | Details |
|-----------|---------|
| **Role** | Business Traveler & Pet Owner |
| **Age** | 20-55 |
| **Tech Savviness** | Low |
| **Goals** | Quickly find and book suitable accommodation without complex multi-step processes, find pet care when traveling |
| **Frustrations** | Overly complex booking flows with too many steps, pop-ups and upsells, unclear cancellation policies, difficulty finding pet-friendly options |
| **Scenario** | Pavlo needs to book a hotel for a business trip to Klaipėda, but he also has a cat that needs care while he's away. He uses the platform to quickly book his hotel in Klaipėda—searching, selecting a room, and paying in just a few clicks. Then he searches for a pet hotel in Vilnius, filters by "Pet Hotel", finds a suitable facility, and books accommodation for his cat. The entire process takes minutes, not the usual frustrating ordeal of navigating complex booking platforms. Later, his meeting is rescheduled so he needs to cancel—the system automatically processes his refund since he's within the free cancellation window. |

### Persona 2: Maria the Hotel Owner

| Attribute | Details |
|-----------|---------|
| **Role** | Small Hotel Owner |
| **Age** | 35-55 |
| **Tech Savviness** | Low |
| **Goals** | List his hotel on the platform, manage room inventory and pricing, receive bookings and payments |
| **Frustrations** | Complicated onboarding processes, lack of control over cancellation policies, difficulty tracking reservations |
| **Scenario** | Maria registers as a Hotel Owner, submits his hotel for approval with details like location, amenities, and cancellation policy (free cancellation 7 days before check-in). After admin approval, she adds room listings with capacity, price per night, and accommodation type. She can view incoming reservations and manage his property details. |

### Persona 3: Dmitry the Administrator

| Attribute | Details |
|-----------|---------|
| **Role** | Platform Administrator |
| **Age** | 25-55 |
| **Tech Savviness** | High |
| **Goals** | Ensure platform quality by reviewing hotel submissions, handle cancellation disputes, manage user accounts |
| **Frustrations** | Fraudulent hotel listings, complex cancellation disputes requiring manual intervention |
| **Scenario** | Dmitry reviews pending hotel submissions daily, approving legitimate listings and rejecting those that don't meet quality standards. When a user requests cancellation outside the free period, he reviews the request and decides whether to approve a refund based on the circumstances provided. |

## Stakeholder Map

### High Influence / High Interest

- **Platform Administrators**: Responsible for overall platform quality, user management, and dispute resolution. Direct stake in system reliability and usability.

### High Influence / Low Interest

- **Payment Processor (Stripe)**: Critical external dependency for payment processing. Defines API constraints and security requirements but not involved in day-to-day operations.

### Low Influence / High Interest

- **Travelers (Users)**: Primary end users who benefit from the platform. Their satisfaction drives platform success but individual users have limited influence on platform decisions.
- **Hotel Owners**: Key content providers for the platform. Dependent on approval workflow and interested in booking volume.

### Low Influence / Low Interest

- **External Service Providers**: Azure infrastructure, database services. Essential for operations but not directly interested in business outcomes.
