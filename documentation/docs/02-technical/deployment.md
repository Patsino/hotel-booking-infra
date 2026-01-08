# Deployment & DevOps

## Infrastructure

### Deployment Architecture

```
┌────────────────────────────────────────────────────────────────────────────────────┐
│                              Azure Cloud (North Europe)                             │
├────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐            │
│  │    Users     │  │    Hotels    │  │ Reservations │  │   Payments   │            │
│  │  App Service │  │  App Service │  │  App Service │  │  App Service │            │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘            │
│         │                 │                 │                 │                    │
│         └─────────────────┴─────────────────┴─────────────────┘                    │
│                                     │                                              │
│                       ┌─────────────┴─────────────┐                                │
│                       │    Azure SQL Database     │                                │
│                       │  (Schemas: users, hotels, │                                │
│                       │   reservations, payments) │                                │
│                       └─────────────┬─────────────┘                                │
│                                     │                                              │
│                       ┌─────────────┴─────────────┐                                │
│                       │     Azure Key Vault       │                                │
│                       │    (Secrets, Config)      │                                │
│                       └───────────────────────────┘                                │
│                                                                                     │
└────────────────────────────────────────────────────────────────────────────────────┘
                                      │
                        ┌─────────────┴─────────────┐
                        │        Stripe API         │
                        │    (Payment Processing)   │
                        └───────────────────────────┘
```

### Environments

| Environment | URL | Purpose |
|-------------|-----|---------|
| **Development** | `localhost:8081-8084` | Local Docker Compose development |
| **Production** | Azure App Services | Live deployment with Azure infrastructure |

### Production URLs

| Service | Swagger URL |
|---------|-------------|
| Users | `https://hotel-booking-users-api-csbghtd2f9cph7g5.northeurope-01.azurewebsites.net/swagger/index.html` |
| Hotels | `https://hotel-booking-hotels-api-evhhefafhhbrgrbs.northeurope-01.azurewebsites.net/swagger/index.html` |
| Reservations | `https://hotel-booking-reservations-api-dwfzh9bydth3fke0.northeurope-01.azurewebsites.net/swagger/index.html` |
| Payments | `https://hotel-booking-payments-api-gch8e7fyeqenfje8.northeurope-01.azurewebsites.net/swagger/index.html` |


## Environment Variables

| Variable | Description | Required | Example |
|----------|-------------|----------|---------|
| `ASPNETCORE_ENVIRONMENT` | Runtime environment | Yes | `Development` / `Production` |
| `AZURE_KEYVAULT_RESOURCEENDPOINT` | Key Vault URL | Yes (Production) | `https://kv-hotel-booking-2.vault.azure.net/` |
| `ConnectionStrings__DefaultConnection` | Database connection | Yes | `Server=...;Database=...` |
| `Jwt__SecretKey` | JWT signing key | Yes | `***` (Key Vault) |
| `Jwt__Issuer` | JWT issuer claim | Yes | `HotelBooking` |
| `Jwt__Audience` | JWT audience claim | Yes | `HotelBookingUsers` |
| `Jwt__ExpirationMinutes` | Access token TTL | Yes | `60` |
| `Jwt__RefreshTokenExpirationDays` | Refresh token TTL | Yes | `7` |
| `ApiKeys__Services__*` | Service API keys | Yes | `***` (Key Vault) |
| `ServiceUrls__*` | Inter-service URLs | Yes | `https://...azurewebsites.net` |
| `Stripe__SecretKey` | Stripe API key | Yes (Payments) | `sk_test_***` |
| `Stripe__WebhookSecret` | Webhook signature key | Yes (Payments) | `whsec_***` |

**Secrets Management:** Azure Key Vault with Managed Identity access. Local development uses environment variables or `appsettings.Development.json`.

## How to Run Locally

### Prerequisites

- [.NET 9 SDK](https://dotnet.microsoft.com/)
- [Docker](https://docker.com/) with Docker Compose
- [Stripe CLI](https://stripe.com/docs/stripe-cli) (for webhook testing)
- SQL Server or Docker

### Setup Steps

```powershell
# 1. Clone the infrastructure repo first (contains docker-compose.yml)
git clone https://github.com/Patsino/hotel-booking-infra
cd hotel-booking-infra

# 2. Clone all service repositories into the same folder
git clone https://github.com/Patsino/hotel-booking-users-service
git clone https://github.com/Patsino/hotel-booking-hotels-service
git clone https://github.com/Patsino/hotel-booking-reservations-service
git clone https://github.com/Patsino/hotel-booking-payments-service

# 3. Create .env file from template
cp .env.example .env
# Edit .env with your values (see environment variables above)

# 4. Generate secrets (PowerShell with Git Bash for openssl)
openssl rand -base64 64  # JWT Secret
openssl rand -base64 32  # Encryption Key
openssl rand -base64 16  # Encryption IV
openssl rand -base64 32  # API Keys (generate 4)

# 5. Start all services with Docker Compose
docker-compose --env-file .env.docker up -d --build

# 6. Verify services are running
docker-compose ps
```

### Docker Compose Services

```yaml
services:
  sqlserver:     # SQL Server database (port 1433)
  users-service: # Users API (port 8081)
  hotels-service: # Hotels API (port 8082)
  reservations-service: # Reservations API (port 8083)
  payments-service: # Payments API (port 8084)
```

### Frontend Setup

The React frontend provides a user-friendly interface for the booking platform.

**Repository:** https://github.com/Patsino/hotel-booking-frontend

```powershell
# 1. Clone the frontend repository
git clone https://github.com/Patsino/hotel-booking-frontend
cd hotel-booking-frontend

# 2. Install dependencies
npm install

# 3. Configure API endpoints (optional)
# Edit .env file for local development: use localhost URLs 
# Or use cloud: use Azure production URLs (default)

# 4. Start the development server
npm run dev

# 5. Open in browser
# http://localhost:5173/
```

### Verify Installation

After starting the services:

1. Open Swagger UI:
   - Users: http://localhost:8081/swagger
   - Hotels: http://localhost:8082/swagger
   - Reservations: http://localhost:8083/swagger
   - Payments: http://localhost:8084/swagger

2. Check health endpoints:
   - http://localhost:8081/health
   - http://localhost:8082/health
   - http://localhost:8083/health
   - http://localhost:8084/health

### Stripe Webhook Testing

```powershell
# Forward Stripe webhooks to local payments service
stripe listen --forward-to localhost:8084/api/webhooks/stripe

# Copy webhook signing secret to .env
```

## Azure Deployment Process

### Via Visual Studio

1. Right-click `Api` project → **Publish**
2. Select existing Azure App Service profile
3. Click **Publish**
4. Azure App Service automatically:
   - Stops the app
   - Deploys new files
   - Starts the app
   - Runs database migrations on startup
   - Loads configuration from Key Vault

### Database Migrations

Migrations run automatically on application startup:
1. App starts
2. `Program.cs` executes migration logic
3. Connects to Azure SQL using Key Vault connection string
4. Runs `dbContext.Database.MigrateAsync()` (idempotent)
5. Seeds initial data if database is empty

## Azure Resource Configuration

### Resource Group
- **Name**: `hotel_booking_solution2`
- **Location**: `North Europe`

### Costs (Student Subscription)
| Resource | Tier | Cost |
|----------|------|------|
| App Service Plan | Free F1 | Free |
| Azure SQL Database | Free tier, General Purpose Serverless | Free |
| Key Vault | Standard | Free tier usage |

### Security Configuration

| Feature | Implementation |
|---------|----------------|
| **Managed Identity** | System-assigned identity for Key Vault access |
| **HTTPS Enforcement** | All traffic redirected to HTTPS |
| **Key Vault Access** | Access policies per Web App identity |
| **Connection Strings** | Stored in Key Vault, referenced via `@Microsoft.KeyVault()` |
