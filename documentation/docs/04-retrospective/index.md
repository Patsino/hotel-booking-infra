# 4. Retrospective

This section reflects on the project development process, lessons learned, and future improvements.

## What Went Well ✅

### Technical Successes

- **Clean Architecture implementation** worked excellently, providing clear separation of concerns and testability across all four microservices
- **Entity Framework Core migrations** streamlined database schema management and made deployments predictable
- **JWT authentication with refresh tokens** provided secure, stateless authentication that scaled well
- **Stripe integration** proved straightforward with excellent documentation and test mode support
- **Docker Compose** made local development simple with consistent environment setup
- **Azure Free Tier deployment** successfully demonstrated cloud-native capabilities without cost

### Process Successes

- **Microservices-first approach** allowed independent development and deployment of each service
- **API-first design** using Swagger/OpenAPI provided clear contracts between services
- **Test-driven development** caught issues early and provided confidence during refactoring
- **Consistent project structure** across services reduced cognitive load when switching contexts

### Personal Achievements

- Gained practical experience with microservices architecture patterns
- Learned Azure cloud deployment and configuration
- Developed skills in payment gateway integration (Stripe)
- Improved understanding of JWT security and token lifecycle

## What Didn't Go As Planned ⚠️

| Planned | Actual Outcome | Cause | Impact |
|---------|---------------|-------|--------|
| Backend-only project | Created React frontend for demos | Swagger UI difficult to demonstrate to non-technical evaluators | High |
| CI/CD pipeline | Manual deployment via Visual Studio | CI/CD not selected as evaluation criterion | Low |
| Inter-service messaging | Direct HTTP communication | Complexity vs. timeline trade-off | Low |
| Real-time notifications | Not implemented | Would require additional infrastructure | Low |

### Challenges Encountered

1. **Demonstrating Backend to Non-Technical Stakeholders**
   - Problem: Swagger UI works well for developers but is confusing for non-technical evaluators
   - Impact: Difficult to show complete booking flow during demonstrations
   - Resolution: Built a simple React frontend specifically for demo purposes

2. **No CI/CD Pipeline**
   - Problem: Manual deployment process via Visual Studio Publish
   - Impact: More effort for deployments, no automated testing on push
   - Reason: Implementing a CI/CD pipeline would require considerable additional time and configuration effort and was therefore deprioritized, as CI/CD was not part of the evaluation criteria; deployment done manually to Azure

3. **Azure Free Tier Limitations**
   - Problem: Cold start delays and limited resources on F1 tier
   - Impact: First requests after idle period are slow (10-30 seconds)
   - Resolution: Documented as expected behavior, acceptable for diploma demonstration

## Technical Debt & Known Issues

| ID | Issue | Severity | Description | Potential Fix |
|----|-------|----------|-------------|---------------|
| TD-001 | No CI/CD | Medium | Manual deployment via Visual Studio | GitHub Actions or Azure DevOps pipeline |
| TD-002 | No message queue | Medium | Direct HTTP calls between services | Azure Service Bus for async communication |
| TD-003 | No caching layer | Low | No Redis for frequent queries | Add distributed cache |



## Future Improvements (Backlog)

If there was more time, these features/improvements would be prioritized:

### High Priority

1. **Message Queue Integration**
   - Description: Implement Azure Service Bus for async communication
   - Value: Improves reliability and decoupling between services
   - Effort: Medium - requires infrastructure changes and handler refactoring

2. **API Gateway**
   - Description: Add centralized gateway for routing, authentication, and rate limiting
   - Value: Single entry point, improved security, better monitoring
   - Effort: Medium - Azure API Management or Kong setup

### Medium Priority

3. **Full Frontend Application**
   - Description: Complete React/TypeScript frontend with proper UI/UX
   - Value: Better user experience for actual production use
   - Effort: High - significant frontend development work

4. **Caching Layer**
   - Description: Add Redis for caching hotel searches and user sessions
   - Value: Improved performance, reduced database load
   - Effort: Medium - infrastructure and code changes

### Nice to Have

	- Email notifications for booking confirmations and reminders
	- Multi-language support
	- Hotel owner analytics dashboard

## Lessons Learned

### Technical Lessons

| Lesson | Context | Application |
|--------|---------|-------------|
| Clean Architecture pays off | Easy testing and modification of business logic | Apply to all future .NET projects |
| Start with authentication | User service must be solid before other services | Build auth infrastructure first |
| Azure free tier is capable | Ran full microservices system at zero cost | Use for MVPs and learning projects |
| Stripe sandbox is essential | Tested all payment flows without real money | Always develop against sandbox first |

### Process Lessons

| Lesson | Context | Application |
|--------|---------|-------------|
| Document as you build | API documentation emerged naturally from code | Use Swagger annotations throughout |
| Test early, test often | Caught integration issues between services early | Maintain high test coverage |
| Keep services focused | Each service doing one thing well | Resist adding unrelated features |

### What Would Be Done Differently

| Area | Current Approach | What Would Change | Why |
|------|-----------------|-------------------|-----|
| CI/CD | Manual deployment | Add GitHub Actions pipeline | Automated testing and deployment |
| Frontend | Added late for demos | Consider from start or skip entirely | Better planning for demo requirements |
| Messaging | HTTP only | Add message broker if async needed | More resilient communication |
| Monitoring | Basic health checks | Add Application Insights | Better debugging in production |

## Personal Growth

### Skills Developed

| Skill | Before Project | After Project |
|-------|---------------|---------------|
| Microservices Architecture | Intermediate | Advanced |
| Azure Cloud Services | Intermediate | Advanced |
| Docker & Containerization | Intermediate | Advanced |
| .NET 9 / ASP.NET Core | Intermediate | Advanced |
| Payment Integration | Beginner | Intermediate |
| Clean Architecture | Intermediate | Advanced |

### Key Takeaways

1. **Microservices require careful planning** - service boundaries and communication patterns must be well-defined upfront
2. **Cloud deployment is accessible** - Azure free tier makes it possible to deploy production-grade systems without cost
3. **Testing is non-negotiable** - high test coverage enables confident refactoring and deployments
4. **Documentation must serve its audience** - Swagger is great for developers, but demos for non-technical stakeholders need visual interfaces

---

*Retrospective completed: 2026-01-05*
