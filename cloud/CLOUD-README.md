# Azure Deployment Guide - Hotel Booking Users Service

## Overview

This document describes the Azure cloud deployment configuration for the Hotel Booking microservices solution. All four microservices (Users, Hotels, Reservations, Payments) are deployed to Azure App Service with centralized secret management via Azure Key Vault.

## Infrastructure Components

### Resource Group
- **Name**: `hotel_booking_solution2`
- **Location**: `North Europe`

### Azure SQL Database
- **Server**: SQL Server instance in `hotel_booking_solution2` resource group
- **Database**:
  - sql-hotel-booking 

### Azure Key Vault
- **Name**: `kv-hotel-booking-2`
- **Purpose**: Centralized secret management for all microservices
- **Managed Identity**: Each Web App has system-assigned managed identity with Key Vault access

### Azure App Services (Web Apps)

| Service | App Service Name | Default Domain |
|---------|-----------------|----------------|
| Users | `hotel-booking-users-api` | `https://hotel-booking-users-api-csbghtd2f9cph7g5.northeurope-01.azurewebsites.net/swagger/index.html` |
| Hotels | `hotel-booking-hotels-api` | `https://hotel-booking-hotels-api-evhhefafhhbrgrbs.northeurope-01.azurewebsites.net/swagger/index.html` |
| Reservations | `hotel-booking-reservations-api` | `https://hotel-booking-reservations-api-dwfzh9bydth3fke0.northeurope-01.azurewebsites.net/swagger/index.html` |
| Payments | `hotel-booking-payments-api` | `https://hotel-booking-payments-api-gch8e7fyeqenfje8.northeurope-01.azurewebsites.net/swagger/index.html` |

## Key Vault Secrets

All secrets are stored in Azure Key Vault `kv-hotel-booking-2`:

### JWT Configuration
- `Jwt-SecretKey` - JWT signing key
- `Jwt-Issuer` - JWT issuer claim
- `Jwt-Audience` - JWT audience claim
- `Jwt-ExpirationMinutes` - Token expiration time

### Encryption
- `Encryption-Key` - AES encryption key
- `Encryption-IV` - AES initialization vector

### API Keys (Inter-Service Authentication)
- `ApiKeys-Services-UsersService`
- `ApiKeys-Services-HotelsService`
- `ApiKeys-Services-ReservationsService`
- `ApiKeys-Services-PaymentsService`

### Database Connection Strings
- `SqlConnectionString-Users`
- `SqlConnectionString-Hotels`
- `SqlConnectionString-Reservations`
- `SqlConnectionString-Payments`

### Service URLs
- `ServiceUrls-Users`
- `ServiceUrls-Hotels`
- `ServiceUrls-Reservations`
- `ServiceUrls-Payments`

### Seeding
- `Seeding-DefaultPassword` - Default password for seeded users

## Environment Variables Configuration

### Users Service (`hotel-booking-users-api`)

Environment variables configured in Azure App Service:

```
ASPNETCORE_ENVIRONMENT=Production
AZURE_KEYVAULT_RESOURCEENDPOINT=https://kv-hotel-booking-2.vault.azure.net/
AZURE_KEYVAULT_SCOPE=https://vault.azure.net/.default

# GDPR Configuration
Gdpr:ExportBasePath=/home/gdpr-exports

# JWT Configuration
Jwt:RefreshTokenExpirationDays=7

# References to Key Vault Secrets
ServiceUrls:Users=@Microsoft.KeyVault(SecretUri=https://kv-hotel-booking-2.vault.azure.net/secrets/ServiceUrls-Users/)
ServiceUrls:Hotels=@Microsoft.KeyVault(SecretUri=https://kv-hotel-booking-2.vault.azure.net/secrets/ServiceUrls-Hotels/)
ServiceUrls:Reservations=@Microsoft.KeyVault(SecretUri=https://kv-hotel-booking-2.vault.azure.net/secrets/ServiceUrls-Reservations/)
ServiceUrls:Payments=@Microsoft.KeyVault(SecretUri=https://kv-hotel-booking-2.vault.azure.net/secrets/ServiceUrls-Payments/)
ApiKeys:Services:UsersService=@Microsoft.KeyVault(SecretUri=https://kv-hotel-booking-2.vault.azure.net/secrets/ApiKeys-Services-UsersService/)
ApiKeys:Services:HotelsService=@Microsoft.KeyVault(SecretUri=https://kv-hotel-booking-2.vault.azure.net/secrets/ApiKeys-Services-HotelsService/)
ApiKeys:Services:ReservationsService=@Microsoft.KeyVault(SecretUri=https://kv-hotel-booking-2.vault.azure.net/secrets/ApiKeys-Services-ReservationsService/)
ApiKeys:Services:PaymentsService=@Microsoft.KeyVault(SecretUri=https://kv-hotel-booking-2.vault.azure.net/secrets/ApiKeys-Services-PaymentsService/)
Encryption:Key=@Microsoft.KeyVault(SecretUri=https://kv-hotel-booking-2.vault.azure.net/secrets/Encryption-Key/)
Encryption:IV=@Microsoft.KeyVault(SecretUri=https://kv-hotel-booking-2.vault.azure.net/secrets/Encryption-IV/)
Jwt:SecretKey=@Microsoft.KeyVault(SecretUri=https://kv-hotel-booking-2.vault.azure.net/secrets/Jwt-SecretKey/)
Jwt:Issuer=@Microsoft.KeyVault(SecretUri=https://kv-hotel-booking-2.vault.azure.net/secrets/Jwt-Issuer/)
Jwt:Audience=@Microsoft.KeyVault(SecretUri=https://kv-hotel-booking-2.vault.azure.net/secrets/Jwt-Audience/)
Jwt:ExpirationMinutes=@Microsoft.KeyVault(SecretUri=https://kv-hotel-booking-2.vault.azure.net/secrets/Jwt-ExpirationMinutes/)
Seeding:DefaultPassword=@Microsoft.KeyVault(SecretUri=https://kv-hotel-booking-2.vault.azure.net/secrets/Seeding-DefaultPassword/)
```

### Hotels, Reservations, Payments Services

Same configuration as Users service, with the following differences:
- **No** `Gdpr:ExportBasePath` (GDPR is Users-service specific)
- **No** `Jwt:RefreshTokenExpirationDays` (refresh tokens are Users-service specific)\
- **No** `Seeding:DefaultPassword` (users seeding is Users-service specific)
- **Stripe keys for Payment Service**


## Database Migrations

### Automatic Migration on Startup

The application automatically applies pending Entity Framework migrations on startup:

1. App starts
2. `Program.cs` executes migration logic
3. Checks if environment is "Testing" (skips for tests)
4. Connects to Azure SQL Database using connection string from Key Vault
5. Runs `dbContext.Database.MigrateAsync()` (idempotent - only applies pending migrations)
6. Seeds initial data if database is empty

### First Deployment

On first publish to Azure:
1. Database is empty
2. App starts and automatically creates schema via migrations
3. Seeds default users and roles


## Deployment Process

### Via Visual Studio

1. **Publish** `Api` project  
2. **Select** existing Azure App Service profile:
   - `hotel-booking-users-api - Web Deploy`
3. **Click** "Publish"
4. Visual Studio builds and deploys the application
5. Azure App Service automatically:
   - Stops the app
   - Deploys new files
   - Starts the app
   - Runs database migrations on startup
   - Loads configuration from environment variables and Key Vault

## Security Configuration

### Managed Identity
Each Web App uses **System-Assigned Managed Identity** to access Key Vault:
- No connection strings or secrets in code
- Azure automatically handles authentication
- Permissions managed via Key Vault Access Policies

### HTTPS Enforcement
All Web Apps enforce HTTPS:
- HTTP requests automatically redirect to HTTPS
- Default Azure domains include SSL certificates

## Inter-Service Communication

Services communicate via HTTPS using:
1. **Service URLs** from Key Vault
2. **API Key authentication** via `X-API-Key` header
3. Service-to-service calls use `AuthenticatedHttpClientHandler`

Example:
```csharp
// Hotels service calls Users service
// Uses URL from ServiceUrls:Users
// Includes X-API-Key from ApiKeys:Services:HotelsService
var response = await httpClient.GetAsync("api/internal-users/1");
```

## Costs
**Using student subscription free infrastructure**
- App Service plan: Free F1 tier shared infrastructure (student subscription)
- Azure SQL pricing tier: Free - General Purpose - Serverless: Standard-series (Gen5), 2 vCores
- Key Vault Sku (Pricing tier) : Standard

## Summary

The Azure deployment architecture provides:
- Secure secret management via Key Vault
- Automatic database migrations on deployment
- Environment-specific configuration
- Inter-service authentication via API keys
- HTTPS-enforced communication
- Scalable microservices architecture
- Centralized monitoring and logging

All configuration follows the same pattern across all four microservices, with differences only in service-specific settings.
