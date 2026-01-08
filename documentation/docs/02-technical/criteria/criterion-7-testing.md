# Criterion 7: Automated Tests (≥70% Coverage)

## Architecture Decision Record

### Status

**Status:** Accepted

**Date:** 2025-12-18

### Context

The project requires comprehensive automated testing with a minimum of 70% code coverage for both line and branch coverage. Tests must:
- Verify business logic correctness
- Enable safe refactoring
- Cover all critical paths

### Decision

Implement a multi-layered testing strategy:
- **Unit Tests**: Test individual components in isolation with mocked dependencies
- **Integration Tests**: Test component interactions using WebApplicationFactory

Testing tools:
- **xUnit**: Test framework
- **FluentAssertions**: Expressive assertions
- **Moq**: Mocking framework
- **Coverlet**: Code coverage collection
- **ReportGenerator**: HTML coverage reports

### Alternatives Considered

| Alternative | Pros | Cons | Why Not Chosen |
|-------------|------|------|----------------|
| NUnit | Mature, feature-rich | Less ASP.NET Core integration | xUnit preferred for .NET Core |
| MSTest | Microsoft official | Less community adoption | xUnit more popular |
| SpecFlow (BDD) | Business-readable | Overhead for small project | Not needed for scope |

### Consequences

**Positive:**
- High confidence in code correctness
- Safe refactoring enabled
- Documentation through tests
- Quick feedback loop

**Negative:**
- Test maintenance overhead
- Some edge cases hard to test
- In-memory database limitations

**Neutral:**
- Coverage exclusions need justification

## Implementation Details

### Test Project Structure

Each service has a corresponding test project:

```
Tests/
├── Unit/
│   ├── Controllers/
│   │   └── *ControllerTests.cs
│   ├── Handlers/
│   │   └── *HandlerTests.cs
│   ├── Services/
│   │   └── *ServiceTests.cs
│   └── Repositories/
│       └── *RepositoryTests.cs
├── Integration/
│   ├── *IntegrationTests.cs
│   └── TestWebApplicationFactory.cs
└── Helpers/
    └── TestHelpers.cs
```

### Testing Patterns

#### Unit Test Example (Handler)
```csharp
public class RegisterUserHandlerTests
{
    private readonly Mock<IUsersRepository> _repositoryMock;
    private readonly Mock<IPasswordHasher> _passwordHasherMock;
    private readonly RegisterUserHandler _handler;

    public RegisterUserHandlerTests()
    {
        _repositoryMock = new Mock<IUsersRepository>();
        _passwordHasherMock = new Mock<IPasswordHasher>();
        _handler = new RegisterUserHandler(
            _repositoryMock.Object,
            _passwordHasherMock.Object);
    }

    [Fact]
    public async Task HandleAsync_WithValidCommand_ShouldCreateUser()
    {
        // Arrange
        var command = new RegisterUserCommand
        {
            Email = "test@example.com",
            Password = "Password123!",
            FirstName = "John",
            LastName = "Doe",
            Role = UserRole.User
        };

        _repositoryMock.Setup(r => r.GetByEmailAsync(command.Email))
            .ReturnsAsync((User?)null);
        _passwordHasherMock.Setup(p => p.HashPassword(command.Password))
            .Returns("hashed_password");

        // Act
        var result = await _handler.HandleAsync(command);

        // Assert
        result.Should().NotBeNull();
        result.Email.Should().Be(command.Email);
        _repositoryMock.Verify(r => r.AddAsync(It.IsAny<User>()), Times.Once);
    }

    [Fact]
    public async Task HandleAsync_WithExistingEmail_ShouldThrowException()
    {
        // Arrange
        var command = new RegisterUserCommand { Email = "existing@example.com" };
        _repositoryMock.Setup(r => r.GetByEmailAsync(command.Email))
            .ReturnsAsync(new User());

        // Act & Assert
        await _handler.Invoking(h => h.HandleAsync(command))
            .Should().ThrowAsync<InvalidOperationException>()
            .WithMessage("*already registered*");
    }
}
```

#### Integration Test Example
```csharp
public class AuthControllerIntegrationTests : IClassFixture<TestWebApplicationFactory>
{
    private readonly HttpClient _client;
    private readonly TestWebApplicationFactory _factory;

    public AuthControllerIntegrationTests(TestWebApplicationFactory factory)
    {
        _factory = factory;
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task Register_WithValidData_ShouldReturn201()
    {
        // Arrange
        var command = new
        {
            Email = $"test-{Guid.NewGuid()}@example.com",
            Password = "Password123!",
            FirstName = "Test",
            LastName = "User",
            Role = "User"
        };

        // Act
        var response = await _client.PostAsJsonAsync("/api/auth/register", command);

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.Created);
        var result = await response.Content.ReadFromJsonAsync<RegisterUserResult>();
        result.Should().NotBeNull();
        result!.Email.Should().Be(command.Email);
    }

    [Fact]
    public async Task Login_WithInvalidCredentials_ShouldReturn401()
    {
        // Arrange
        var command = new { Email = "nonexistent@example.com", Password = "wrong" };

        // Act
        var response = await _client.PostAsJsonAsync("/api/auth/login", command);

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.Unauthorized);
    }
}
```

#### Test Web Application Factory
```csharp
public class TestWebApplicationFactory : WebApplicationFactory<Program>
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.UseEnvironment("Testing");

        builder.ConfigureServices(services =>
        {
            // Replace SQL Server with In-Memory database
            var descriptor = services.SingleOrDefault(
                d => d.ServiceType == typeof(DbContextOptions<UsersDbContext>));
            if (descriptor != null)
                services.Remove(descriptor);

            services.AddDbContext<UsersDbContext>(options =>
            {
                options.UseInMemoryDatabase($"TestDb-{Guid.NewGuid()}");
            });

            // Configure test authentication
            services.AddAuthentication("Test")
                .AddScheme<AuthenticationSchemeOptions, TestAuthHandler>("Test", null);
        });
    }
}
```

### Naming Conventions

**Test Methods**: `{MethodName}_{Scenario}_{ExpectedBehavior}`

Examples:
- `GetByIdAsync_WithExistingId_ShouldReturnReservation`
- `Create_ShouldReturnForbidden_WhenCreatingForDifferentOwner`
- `Login_WithValidCredentials_ShouldReturnToken`
- `Update_ShouldAllowNullOptionalValues`

### Coverage Exclusions

Code excluded from coverage metrics (justified):

```xml
<!-- Test .csproj -->
<Exclude>
  [*]*.Migrations.*,
  [*]*ModelSnapshot,
  [*]*DbContextFactory,
  [*]*DataSeeder*,
  [*]*.Configurations.*
</Exclude>

<ExcludeByAttribute>
  Obsolete,
  GeneratedCode,
  CompilerGenerated,
  ExcludeFromCodeCoverage
</ExcludeByAttribute>
```

**Rationale:**
- **Migrations**: Auto-generated EF Core code
- **DbContextFactory**: Design-time tooling only
- **DataSeeder**: Development/test utilities
- **Middleware**: Framework infrastructure with no business logic
- **Swagger Filters**: Documentation generation

### Running Tests

```powershell
# Run all tests for a service
cd hotel-booking-users-service
dotnet test

# Run tests with coverage
.\run-tests-with-coverage.ps1
```

### Coverage Report Generation Script

```powershell
# run-tests-with-coverage.ps1

# Clean previous coverage
Remove-Item -Recurse -Force ./CoverageReport -ErrorAction SilentlyContinue

# Run tests with Coverlet
dotnet test `
    --collect:"XPlat Code Coverage" `
    --results-directory ./CoverageReport `
    -- DataCollectionRunSettings.DataCollectors.DataCollector.Configuration.Format=cobertura

# Install ReportGenerator if needed
dotnet tool install -g dotnet-reportgenerator-globaltool

# Generate HTML report
reportgenerator `
    -reports:./CoverageReport/**/coverage.cobertura.xml `
    -targetdir:./CoverageReport/Report `
    -reporttypes:Html

# Open report
Start-Process ./CoverageReport/Report/index.html
```

### Coverage Results

| Service | Tests | Line Coverage | Branch Coverage |
|---------|-------|---------------|-----------------|
| Users | ~200 | >70% | >70% |
| Hotels | ~200 | >70% | >70% |
| Reservations | ~200 | >70% | >70% |
| Payments | ~190 | >70% | >70% |
| **Total** | **~790** | **>70%** | **>70%** |

### FIRST Principles Compliance

| Principle | Implementation |
|-----------|----------------|
| **Fast** | Unit tests < 1s, integration tests 2-5s per service |
| **Independent** | Each test uses unique database identifier |
| **Repeatable** | No shared state, deterministic data |
| **Self-Validating** | Clear pass/fail with FluentAssertions |
| **Timely** | Tests written alongside implementation |

## Requirements Checklist

### Minimum Requirements (Grade 5)

| # | Requirement | Status | Evidence |
|---|-------------|--------|----------|
| 1 | Structured automated testing setup | ✅ | xUnit with FluentAssertions |
| 2 | At least 70% code coverage | ✅ | 75-85% per service via Coverlet |
| 3 | Unit tests with FIRST principles | ✅ | Fast, Independent, Repeatable tests |
| 4 | Edge cases covered | ✅ | Validation, error scenarios tested |
| 5 | Integration tests | ✅ | WebApplicationFactory for API tests |
| 6 | Frontend tests (if applicable) | N/A | Skipped - Frontend criterion not selected |
| 7 | Tests run in CI | N/A | Skipped - CI/CD criterion not selected |
| 8 | Testing strategy documented | ✅ | This document |
| 9 | Clear test naming/structure | ✅ | Tests/Unit/, Tests/Integration/ |

### Maximum Requirements (Grade 10)

| # | Requirement | Status | Notes |
|---|-------------|--------|-------|
| 1 | Multi-layer testing (unit, integration, E2E) | ⚠️ Partial | no E2E |
| 2 | Mocking, fixtures, test doubles | ✅ | Moq for all dependencies |
| 3 | Complex integration scenarios | ⚠️ Partial | Single-service focus |
| 4 | Property-based/mutation/contract testing | ⚠️ Partial  | Сontract testing implemented |
| 5 | CI/CD with quality gates | N/A | CI/CD not selected as criterion |
| 6 | Test architecture rationale | ✅ | Documented in this file |
| 7 | Performance/load testing | ❌ | Not implemented |

## Known Limitations

| Limitation | Impact | Potential Solution |
|------------|--------|-------------------|
| In-memory DB limitations | Some EF features not supported | Use SQL Server TestContainers |
| No E2E tests | Full flow not tested | Add Playwright or Selenium tests |
| No load tests | Performance unknown | Add k6 or similar |
| Limited webhook testing | Stripe webhooks hard to test | WireMock for webhook simulation |

## References

- [xUnit Documentation](https://xunit.net/)
- [FluentAssertions](https://fluentassertions.com/)
- [Coverlet](https://github.com/coverlet-coverage/coverlet)
- [ASP.NET Core Integration Testing](https://docs.microsoft.com/aspnet/core/test/integration-tests)
- Test projects: `Tests/` folder in each service
- Coverage scripts: `run-tests-with-coverage.ps1` in each service
