# Cloud-based hotel booking solution

## Project Information

| Field | Value |
|-------|-------|
| **Student** | Andrey Patsino |
| **Group** | 22HR C# |
| **Supervisor** | Pavlo Andriiash |
| **Date** | 2026-01-05 |

## Links


### Production: Azure App Services (Swagger)

| Service | App Service Name | Default Domain |
|---------|-----------------|----------------|
| Users | `hotel-booking-users-api` | `https://hotel-booking-users-api-csbghtd2f9cph7g5.northeurope-01.azurewebsites.net/swagger/index.html` |
| Hotels | `hotel-booking-hotels-api` | `https://hotel-booking-hotels-api-evhhefafhhbrgrbs.northeurope-01.azurewebsites.net/swagger/index.html` |
| Reservations | `hotel-booking-reservations-api` | `https://hotel-booking-reservations-api-dwfzh9bydth3fke0.northeurope-01.azurewebsites.net/swagger/index.html` |
| Payments | `hotel-booking-payments-api` | `https://hotel-booking-payments-api-gch8e7fyeqenfje8.northeurope-01.azurewebsites.net/swagger/index.html` |


### Repositories

| Repository | URL | Description |
|------------|-----|-------------|
| Infra | `https://github.com/Patsino/hotel-booking-infra` | Docker Compose, .env.example, shared docs |
| Users | `https://github.com/Patsino/hotel-booking-users-service` | User management, authentication, JWT |
| Hotels | `https://github.com/Patsino/hotel-booking-hotels-service` | Hotel/room listings, owner management |
| Reservations | `https://github.com/Patsino/hotel-booking-reservations-service` | Booking logic, availability |
| Payments | `https://github.com/Patsino/hotel-booking-payments-service` | Stripe integration, refunds |
| Frontend | https://github.com/Patsino/hotel-booking-frontend | React UI |


## Elevator Pitch

The Hotel Booking platform is a cloud-based system designed for travelers searching for hotel accommodations and for hotel owners who want to list and manage their properties. It addresses the complexity of modern hotel booking by providing a secure and reliable way to search hotels, make reservations, process payments, and handle cancellations. The platform is built as a set of independent services that each focus on a specific business area, allowing the system to scale and evolve efficiently. What makes it unique is the clear separation of responsibilities between services, which improves maintainability, reliability, and flexibility while supporting the complete booking lifecycle.

## Evaluation Criteria Checklist

| # | Criterion | Status | Documentation |
|---|-----------|--------|---------------|
| 1 | Back-end | ✅| [02-technical/criteria/criterion-1-backend.md](02-technical/criteria/criterion-1-backend.md) |
| 2 | API Documentation | ✅ | [02-technical/criteria/criterion-2-api-documentation.md](02-technical/criteria/criterion-2-api-documentation.md) |
| 3 | Database | ✅ | [02-technical/criteria/criterion-3-database.md](02-technical/criteria/criterion-3-database.md) |
| 4 | Cloud | ✅ | [02-technical/criteria/criterion-4-cloud.md](02-technical/criteria/criterion-4-cloud.md) |
| 5 | Microservices | ✅| [02-technical/criteria/criterion-5-microservices.md](02-technical/criteria/criterion-5-microservices.md) |
| 6 | Containerization | ✅ | [02-technical/criteria/criterion-6-containerization.md](02-technical/criteria/criterion-6-containerization.md) |
| 7 | Automated tests ≥ 70% coverage | ✅| [02-technical/criteria/criterion-7-testing.md](02-technical/criteria/criterion-7-testing.md) |

## Documentation Navigation

- [Project Overview](01-project-overview/index.md) - Business context, goals, and requirements
- [Technical Implementation](02-technical/index.md) - Architecture, tech stack, and criteria details
- [User Guide](03-user-guide/index.md) - How to use the application
- [Retrospective](04-retrospective/index.md) - Lessons learned and future improvements

---

*Document created: 2026-01-01*
*Last updated: 2026-01-05*
