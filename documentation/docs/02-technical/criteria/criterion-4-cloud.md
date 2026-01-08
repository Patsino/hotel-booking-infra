# Criterion 4: Cloud Deployment

## Architecture Decision Record

### Status

**Status:** Accepted

**Date:** 2025-12-11

### Context

The project requires cloud deployment to demonstrate production-readiness. The deployment must support:
- All four microservices running independently
- Secure secret management
- Database hosting
- HTTPS enforcement
- Cost-effective solution (student budget)

### Decision

Deploy to Microsoft Azure using:
- **Azure App Service** (Free F1 tier) for all four microservices
- **Azure SQL Database** (Free tier, General Purpose Serverless) for data storage
- **Azure Key Vault** (Standard tier) for centralized secrets management
- **System-assigned Managed Identity** for secure Key Vault access

### Alternatives Considered

| Alternative | Pros | Cons | Why Not Chosen |
|-------------|------|------|----------------|
| AWS | Mature platform, wide adoption | Less integrated with .NET, more complex setup | Azure has better .NET integration |

### Consequences

**Positive:**
- Excellent .NET and Visual Studio integration
- Managed Identity eliminates credential management
- Free tier sufficient for diploma demonstration
- Built-in SSL certificates
- Easy deployment via Visual Studio

**Negative:**
- Vendor lock-in to Azure
- Free tier has limitations (cold starts, resource limits)
- Single region deployment

**Neutral:**
- Azure portal learning curve

## Implementation Details

### Azure Resource Group

```
Resource Group: hotel_booking_solution2
Location: North Europe

Resources:
├── App Services
│   ├── hotel-booking-users-api
│   ├── hotel-booking-hotels-api
│   ├── hotel-booking-reservations-api
│   └── hotel-booking-payments-api
├── Azure SQL Server
│   └── sql-hotel-booking (database)
└── Key Vault
    └── kv-hotel-booking-2
```

### App Service Configuration

Each App Service is configured with:

| Setting | Value |
|---------|-------|
| Runtime | .NET 9 |
| OS | Linux |
| Plan | Free F1 |
| HTTPS Only | Enabled |
| Managed Identity | System-assigned |

### Key Vault Secrets

All secrets stored in Azure Key Vault `kv-hotel-booking-2`:

| Secret | Purpose |
|--------|---------|
| `Jwt-SecretKey` | JWT signing key |
| `Jwt-Issuer` | JWT issuer claim |
| `Jwt-Audience` | JWT audience claim |
| `Jwt-ExpirationMinutes` | Token expiration |
| `Encryption-Key` | AES encryption key |
| `Encryption-IV` | AES initialization vector |
| `ApiKeys-Services-UsersService` | Users service API key |
| `ApiKeys-Services-HotelsService` | Hotels service API key |
| `ApiKeys-Services-ReservationsService` | Reservations service API key |
| `ApiKeys-Services-PaymentsService` | Payments service API key |
| `SqlConnectionString-Users` | Database connection |
| `ServiceUrls-*` | Inter-service URLs |

### Environment Variable Configuration

App Service environment variables reference Key Vault secrets:

```
ASPNETCORE_ENVIRONMENT=Production
AZURE_KEYVAULT_RESOURCEENDPOINT=https://kv-hotel-booking-2.vault.azure.net/

# Key Vault References
Jwt:SecretKey=@Microsoft.KeyVault(SecretUri=https://kv-hotel-booking-2.vault.azure.net/secrets/Jwt-SecretKey/)
Jwt:Issuer=@Microsoft.KeyVault(SecretUri=https://kv-hotel-booking-2.vault.azure.net/secrets/Jwt-Issuer/)
ApiKeys:Services:UsersService=@Microsoft.KeyVault(SecretUri=https://kv-hotel-booking-2.vault.azure.net/secrets/ApiKeys-Services-UsersService/)
```

### Production URLs

| Service | URL |
|---------|-----|
| Users | https://hotel-booking-users-api-csbghtd2f9cph7g5.northeurope-01.azurewebsites.net |
| Hotels | https://hotel-booking-hotels-api-evhhefafhhbrgrbs.northeurope-01.azurewebsites.net |
| Reservations | https://hotel-booking-reservations-api-dwfzh9bydth3fke0.northeurope-01.azurewebsites.net |
| Payments | https://hotel-booking-payments-api-gch8e7fyeqenfje8.northeurope-01.azurewebsites.net |

### Deployment Process

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│ Visual Studio   │────▶│  Azure App Svc  │────▶│   Key Vault     │
│ Publish         │     │  Deployment     │     │   (Secrets)     │
└─────────────────┘     └────────┬────────┘     └─────────────────┘
                                 │
                                 ▼
                        ┌─────────────────┐
                        │   App Starts    │
                        │   + Migrations  │
                        │   + Seeding     │
                        └────────┬────────┘
                                 │
                                 ▼
                        ┌─────────────────┐
                        │  Azure SQL DB   │
                        └─────────────────┘
```

1. Right-click project → Publish
2. Select Azure App Service profile
3. Click Publish
4. Azure automatically:
   - Stops application
   - Deploys new files
   - Restarts application
   - Runs migrations on startup
   - Loads secrets from Key Vault

### Security Configuration

| Feature | Implementation |
|---------|----------------|
| **Managed Identity** | System-assigned identity for each App Service |
| **Key Vault Access** | Access policies grant identity secret read permissions |
| **HTTPS** | Enforced on all App Services |
| **TLS** | TLS 1.2 minimum |
| **Connection Strings** | Encrypted in Key Vault |

### Cost Analysis

| Resource | Tier | Monthly Cost |
|----------|------|--------------|
| App Service Plan (x4) | Free F1 | $0 |
| Azure SQL Database | Free tier | $0 |
| Key Vault | Standard | $0 (free tier usage) |
| **Total** | | **$0** (student subscription) |

## Requirements Checklist

### Minimum Requirements (Grade 5)

| # | Requirement | Status | Evidence |
|---|-------------|--------|----------|
| 1 | Deploy to cloud (AWS/Azure/GCP/etc.) | ✅ | Azure App Service deployment |
| 2 | At least one component accessible via public URL | ✅ | Four services with public Swagger URLs |
| 3 | Clear deployment documentation | ✅ | docs/02-technical/deployment.md |
| 4 | At least one managed cloud service | ✅ | Azure SQL Database + Azure Key Vault |
| 5 | Secrets stored securely (not in repo) | ✅ | Azure Key Vault with Managed Identity |
| 6 | Git repository with commit history | ✅ | GitHub repositories for all services |
| 7 | Application consistently reachable | ✅ | Services available (cold start delays expected) |
| 8 | No unnecessary paid resources running | ✅ | Using free tier only |

### Maximum Requirements (Grade 10)

| # | Requirement | Status | Notes |
|---|-------------|--------|-------|
| 1 | Containers/orchestration + CI/CD | ⚠️ | Docker for local dev; CI/CD not selected as criterion |
| 2 | At least 2 managed cloud services | ✅ | App Service + SQL Database + Key Vault |
| 3 | Infrastructure as Code | ❌ | Manual Azure Portal setup |
| 4 | Monitoring/logging + auto-scaling | ❌ | Basic health checks only; no auto-scaling on free tier |
| 5 | IAM roles and least privilege | ✅ | Managed Identity with minimal permissions |
| 6 | Screenshots with redacted secrets | ✅ | Can be provided for defense |
| 7 | Reproducible environment scripts | ⚠️ | Guide on enviroment reproducibility provided |

## Known Limitations

| Limitation | Impact | Potential Solution |
|------------|--------|-------------------|
| Free tier cold starts | First request slow after idle | Upgrade to paid tier or implement keep-alive |
| Single region | No geographic redundancy | Multi-region deployment in production |
| Shared App Service Plan | Limited resources | Dedicated plans for production |
| No custom domain | Using azurewebsites.net URLs | Configure custom domain in production |

## References

- [Azure App Service Documentation](https://docs.microsoft.com/azure/app-service/)
- [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/)
- [Azure SQL Database](https://docs.microsoft.com/azure/azure-sql/)
- Deployment profiles: `.pubxml` files in each service
