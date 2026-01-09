# Hotel Booking System - Docker compose setup for local development

Complete microservices architecture for hotel booking system with unified Docker orchestration.

##  Architecture

This system consists of 4 independent microservices sharing a common SQL Server database:

- **Users Service** (Port 8081) - Authentication, user management, GDPR
- **Hotels Service** (Port 8082) - Hotel and room management
- **Reservations Service** (Port 8083) - Booking and reservation management
- **Payments Service** (Port 8084) - Payment processing via Stripe

All services communicate via REST APIs with service-to-service authentication.

## Project Structure

```
Patsino/
├── docker-compose.yml              
├── .env               
├── hotel-booking-users-service/
│   ├── Dockerfile
│   └── ...
├── hotel-booking-hotels-service/
│   ├── Dockerfile
│   └── ...
├── hotel-booking-reservations-service/
│   ├── Dockerfile
│   └── ...
└── hotel-booking-payments-service/
    ├── Dockerfile
    └── ...
```

##  Start Guide

### Step 1: Clone All Repositories

Clone all 4 microservice repositories into the same parent folder:

# Clone all services
https://github.com/Patsino/hotel-booking-users-service

https://github.com/Patsino/hotel-booking-hotels-service

https://github.com/Patsino/hotel-booking-reservations-service

https://github.com/Patsino/hotel-booking-payments-service

### Step 2: Configure Environment Variables

```powershell
# Copy the template
cp .env.example .env

# Generate secrets (PowerShell - use Git Bash for openssl)
# JWT Secret (64 bytes)
openssl rand -base64 64

# Encryption Key (32 bytes)
openssl rand -base64 32

# Encryption IV (16 bytes)
openssl rand -base64 16

# API Keys (32 bytes each - generate 4 times)
openssl rand -base64 32
openssl rand -base64 32
openssl rand -base64 32
openssl rand -base64 32
```

Edit `.env` and fill in:
- All generated secrets
- Your Stripe test keys (see [Stripe Setup](#stripe-setup) below)
- SQL Server password
- Seed users password

### Step 3: Launch the System

```powershell
# Build and start all services
docker-compose --env-file .env.docker up -d --build
```

### Step 4: Verify Services

All services should be healthy and accessible:

- **Users Service**: http://localhost:8081/health
- **Hotels Service**: http://localhost:8082/health
- **Reservations Service**: http://localhost:8083/health
- **Payments Service**: http://localhost:8084/health
- **SQL Server**: localhost:1433

```powershell
# Check service status
docker-compose ps

# View logs
docker-compose logs -f

# View specific service logs
docker-compose logs -f users-service
```

## 💳 Stripe Setup

### Get Stripe Keys

1. Sign up at [Stripe](https://stripe.com)
2. Go to [Dashboard → API Keys](https://dashboard.stripe.com/test/apikeys)
3. Copy your **test** keys:
   - **Secret key** (starts with `sk_test_`)
   - **Publishable key** (starts with `pk_test_`)

### Configure Webhooks

Stripe sends webhook events for payment updates (successful payments, failures, refunds, etc.).

**For local development**, use Stripe CLI:

```powershell
# Install Stripe CLI (see https://stripe.com/docs/stripe-cli)
# Login to Stripe
stripe login

# Forward webhooks to local payments service
stripe listen --forward-to localhost:8084/api/webhook/stripe

# Copy the webhook signing secret (starts with whsec_)
# Add it to .env as STRIPE_WEBHOOK_SECRET
```

## 🛠️ Common Operations

### Stop All Services

```powershell
docker-compose down
```

### Stop and Remove All Data

```powershell
# WARNING: This deletes the database!
docker-compose down -v
```

### Rebuild Specific Service

```powershell
docker-compose up --build users-service
```

### View Database

```powershell
# Connect to SQL Server container
docker exec -it hotel-booking-sqlserver /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P YourPassword -C
```


## 🌐 API Endpoints

All services expose Swagger UI with full API documentation:
Users Service: http://localhost:8081/swagger

Hotels Service: http://localhost:8082/swagger

Reservations Service: http://localhost:8083/swagger

Payments Service: http://localhost:8084/swagger

