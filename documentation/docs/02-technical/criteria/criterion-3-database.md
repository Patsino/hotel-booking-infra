# Criterion 3: Database

## Architecture Decision Record

### Status

**Status:** Accepted

**Date:** 2025-12-04

### Context

The project requires persistent data storage for users, hotels, rooms, reservations, and payments. The database solution must support:
- Relational data with proper constraints
- Code-first migrations for schema evolution
- Efficient querying for search operations
- Support for both local development and cloud deployment

### Decision

Use SQL Server with Entity Framework Core, implementing:
- Single database with schema separation per microservice
- Code-first migrations with automatic application on startup
- Repository pattern for data access abstraction
- EF Core In-Memory provider for testing

### Alternatives Considered

| Alternative | Pros | Cons | Why Not Chosen |
|-------------|------|------|----------------|
| Separate databases per service | True isolation, independent scaling | Azure free tier allows only one database | Cost constraint |
| PostgreSQL | Open source, rich features | Less integrated with Azure ecosystem | SQL Server has better Azure integration |
| Dapper | Performance, control | Manual mapping, no migrations | EF Core productivity benefits outweigh |

### Consequences

**Positive:**
- Familiar SQL Server ecosystem
- Automatic migrations simplify deployment
- Strong Azure SQL integration
- EF Core provides type-safe queries

**Negative:**
- Schema separation is not true database isolation
- Potential for accidental cross-schema access
- Single database is potential bottleneck

**Neutral:**
- Need to coordinate schema changes across services

## Implementation Details

### Database Architecture

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                         HotelBooking (SQL Database)                          │
├──────────────────────────────────────────────────────────────────────────────┤
│  Schema: users                                                               │
│  ├── Users                                                                   │
│  ├── RefreshTokens                                                           │
│  └── DataExportRequests                                                      │
├──────────────────────────────────────────────────────────────────────────────┤
│  Schema: hotels                                                              │
│  ├── Hotels                                                                  │
│  └── Rooms                                                                   │
├──────────────────────────────────────────────────────────────────────────────┤
│  Schema: reservations                                                        │
│  └── Reservations                                                            │
├──────────────────────────────────────────────────────────────────────────────┤
│  Schema: payments                                                            │
│  └── Payments                                                                │
└──────────────────────────────────────────────────────────────────────────────┘
```

### Entity Relationship Diagram

```
┌─────────────────┐       ┌─────────────────┐
│     Users       │       │  RefreshTokens  │
├─────────────────┤       ├─────────────────┤
│ Id (PK)         │───┐   │ Id (PK)         │
│ Email           │   │   │ UserId (FK)     │◄──┐
│ PasswordHash    │   └──▶│ Token           │   │
│ Role            │       │ ExpiresAt       │   │
│ IsDeleted       │       │ IsRevoked       │   │
│ DeletedAt       │       └─────────────────┘   │
│ CreatedAt       │                             │
└─────────────────┘       ┌─────────────────┐   │
                          │DataExportRequests│   │
                          ├─────────────────┤   │
                          │ Id (PK)         │   │
                          │ UserId (FK)     │◄──┘
                          │ Status          │
                          │ ExportFilePath  │
                          │ RequestedAt     │
                          └─────────────────┘

┌─────────────────┐       ┌─────────────────┐
│     Hotels      │       │      Rooms      │
├─────────────────┤       ├─────────────────┤
│ Id (PK)         │───┐   │ Id (PK)         │
│ OwnerId         │   │   │ HotelId (FK)    │◄──┐
│ Name            │   └──▶│ RoomNumber      │   │
│ Description     │       │ Description     │   │
│ Country         │       │ Capacity        │   │
│ City            │       │ Bedrooms        │   │
│ District        │       │ PricePerNight   │   │
│ AddressLine     │       │ Visible         │   │
│ PetsAllowed     │       │ PetsAllowed     │   │
│ IsPetHotel      │       │ Accommodation   │   │
│ CancelFreeDays  │       │ CreatedAt       │   │
│ Approval        │       └─────────────────┘   │
│ SubmittedAt     │                             │
│ ReviewedAt      │                             │
└─────────────────┘                             │
                                                │
┌─────────────────┐       ┌─────────────────┐   │
│  Reservations   │       │    Payments     │   │
├─────────────────┤       ├─────────────────┤   │
│ Id (PK)         │◄──┐   │ Id (PK)         │   │
│ UserId          │   │   │ ReservationId   │◄──┤
│ RoomId          │◄──┼───│ Amount          │   │
│ StartDate       │   │   │ Currency        │   │
│ EndDate         │   │   │ Provider        │   │
│ GuestsCount     │   │   │ Status          │   │
│ GuestsNames     │   │   │ PaymentIntentId │   │
│ Status          │   │   │ AmountRefunded  │   │
│ CancellationStat│   │   │ PaidAt          │   │
│ CreatedAt       │   │   └─────────────────┘   │
└─────────────────┘   │                         │
                      └─────────────────────────┘
Note: Cross-schema references (UserId, RoomId, ReservationId) 
are logical - no physical foreign keys across schemas
```

### DbContext Configuration

**Users Service DbContext:**
```csharp
public class UsersDbContext : DbContext
{
    public DbSet<User> Users => Set<User>();
    public DbSet<RefreshToken> RefreshTokens => Set<RefreshToken>();
    public DbSet<DataExportRequest> DataExportRequests => Set<DataExportRequest>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.HasDefaultSchema("users");
        
        modelBuilder.Entity<User>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.HasIndex(e => e.Email).IsUnique();
            entity.Property(e => e.Email).HasMaxLength(256).IsRequired();
            entity.Property(e => e.PasswordHash).IsRequired();
            entity.Property(e => e.Role).HasConversion<string>();
        });
        
        modelBuilder.Entity<RefreshToken>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.HasIndex(e => e.Token).IsUnique();
            entity.HasOne<User>().WithMany().HasForeignKey(e => e.UserId);
        });
    }
}
```

### Migration Strategy

Migrations are applied automatically on application startup:

```csharp
// Program.cs
if (app.Environment.EnvironmentName != "Testing")
{
    using var scope = app.Services.CreateScope();
    var dbContext = scope.ServiceProvider.GetRequiredService<UsersDbContext>();
    await dbContext.Database.MigrateAsync();
    
    // Seed initial data if needed
    var seeder = scope.ServiceProvider.GetRequiredService<IDataSeeder>();
    await seeder.SeedAsync();
}
```

### Key Tables

#### Users Schema

| Table | Purpose | Key Fields |
|-------|---------|------------|
| Users | User accounts | Id, Email, PasswordHash, Role, IsDeleted |
| RefreshTokens | JWT refresh tokens | Token, UserId, ExpiresAt, IsRevoked |
| DataExportRequests | GDPR export tracking | UserId, Status, ExportFilePath |

#### Hotels Schema

| Table | Purpose | Key Fields |
|-------|---------|------------|
| Hotels | Hotel listings | Id, OwnerId, Name, Country, City, Approval |
| Rooms | Room inventory | Id, HotelId, RoomNumber, Capacity, PricePerNight |

#### Reservations Schema

| Table | Purpose | Key Fields |
|-------|---------|------------|
| Reservations | Bookings | Id, UserId, RoomId, StartDate, EndDate, Status |

#### Payments Schema

| Table | Purpose | Key Fields |
|-------|---------|------------|
| Payments | Payment records | Id, ReservationId, Amount, Status, PaymentIntentId |

## Requirements Checklist

### Minimum Requirements (Grade 5)

| # | Requirement | Status | Evidence |
|---|-------------|--------|----------|
| 1 | Data dictionary (tables, columns, types) | ✅ | docs/appendices/db-schema.md |
| 2 | Data integrity and transactions | ✅ | EF Core transactions, constraints |
| 3 | ER diagram or logical schema | ✅ | Diagrams in docs/appendices/db-schema.md |
| 4 | DDL via migrations | ✅ | EF Core code-first migrations |
| 5 | Modern RDBMS | ✅ | SQL Server / Azure SQL |
| 6 | 3NF normalization | ✅ | Normalized schema design |
| 7 | Primary/foreign keys, constraints | ✅ | PK/FK in model configuration |
| 8 | Migrations in version control | ✅ | Infrastructure/Migrations/ in Git |
| 9 | Test data seeding | ✅ | DataSeeder classes, scripts |
| 10 | Database roles | ✅ Implemented | Each microservice has dedicated database role with minimal required permissions |
| 11 | Encrypted passwords | ✅ | BCrypt hashing (not stored in plain text) |

### Maximum Requirements (Grade 10)

| # | Requirement | Status | Notes |
|---|-------------|--------|-------|
| 1 | Multiple DBMS types if justified | ❌ | Single SQL Server - sufficient for domain |
| 2 | Data layers (raw/staging/mart) | ❌ | Not applicable - OLTP system |
| 3 | Schema versioning documented | ✅ | EF migrations track changes |
| 4 | Indexes documented and justified | ⚠️ | Indexes exist but not fully documented |
| 5 | Triggers/stored procedures | ❌ | Business logic in application layer |
| 6 | Views for complex queries | ❌ | Not needed for current use cases |
| 7 | PII masking | ❌ | Not implemented |

## Known Limitations

| Limitation | Impact | Potential Solution |
|------------|--------|-------------------|
| Shared database | Not true microservices isolation | Separate databases in production (paid tier) |
| No cross-schema FKs | Referential integrity in code only | Accept trade-off for microservices pattern |
| String enum storage | Larger storage, slower queries | Use integer enum conversion |

## References

- [EF Core Documentation](https://docs.microsoft.com/ef/core/)
- [Azure SQL Database](https://docs.microsoft.com/azure/azure-sql/)
- DbContext: `Infrastructure/Persistence/*DbContext.cs` in each service
- Migrations: `Infrastructure/Migrations/` in each service
