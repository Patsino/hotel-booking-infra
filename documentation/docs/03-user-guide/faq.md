# FAQ & Troubleshooting

## Frequently Asked Questions

### General

**Q: What is the Hotel Booking Platform?**

A: The Hotel Booking Platform is a microservices-based system for hotel reservations. It consists of four services: Users (authentication), Hotels (property management), Reservations (bookings), and Payments (Stripe integration).

---

**Q: How do I access the platform?**

A: The platform is accessed via Swagger UI at each service's URL. There is no traditional web frontend - API testing is done directly through Swagger's interactive interface.

---

**Q: What user roles are available?**

A: Three roles exist:
- **User**: Can search hotels, make reservations, and manage own bookings
- **HotelOwner**: Can submit hotels for approval and manage rooms
- **Admin**: Full system access including approvals and cancellation management

---

### Account & Access

**Q: How do I create an account?**

A: Use the `POST /api/auth/register` endpoint in the Users Service with your email, password, name, and desired role.

---

**Q: My access token expired. What do I do?**

A: Use the `POST /api/auth/refresh` endpoint with your refresh token to get a new access token. Access tokens expire after 60 minutes, refresh tokens after 7 days.

---

**Q: Can I change my role after registration?**

A: Yes, regular Users can upgrade to HotelOwner role using the `POST /api/auth/become-hotel-owner` endpoint. This returns new JWT tokens with the updated role. 

---

**Q: How do I logout?**

A: Use the `POST /api/auth/logout` endpoint. This revokes your refresh token, though the access token remains valid until it expires.

---

### Booking & Reservations

**Q: Why is my reservation still "Pending"?**

A: Reservations remain Pending until payment is completed. Create a payment intent and confirm payment through Stripe to change status to Confirmed.

---

**Q: Can I cancel a reservation?**

A: Yes. Use `POST /api/reservations/{id}/cancel`. If within the hotel's free cancellation period, it's auto-canceled with refund. Otherwise, it requires admin approval.

---

**Q: How is the cancellation policy determined?**

A: Each hotel sets a `cancelFreeDaysBefore` value. If you cancel more than this many days before check-in, you get automatic cancellation and refund. Otherwise, admin review is required.

---

### Payments

**Q: What payment methods are supported?**

A: The system uses Stripe for payments. In test mode, use Stripe test cards. Credit/debit cards are supported.

---

**Q: How do I test payments without real money?**

A: Use Stripe test card `4242 4242 4242 4242` with any future expiration date and any 3-digit CVC.

---

**Q: My payment failed. What happened?**

A: Check the payment status via `GET /api/payments/{id}`. The `errorMessage` field will indicate the reason (insufficient funds, card declined, etc.).

---

## Troubleshooting

### Common Issues

| Problem | Possible Cause | Solution |
|---------|---------------|----------|
| 401 Unauthorized | Token expired or missing | Refresh your token or re-login |
| 403 Forbidden | Insufficient permissions | Check your role has access to this endpoint |
| 404 Not Found | Resource doesn't exist | Verify the ID is correct |
| 400 Bad Request | Invalid input data | Check request body format and required fields |
| Room not available | Dates overlap with existing reservation | Choose different dates or room |
| Hotel not visible | Hotel not approved yet | Wait for admin approval |

### Error Messages

| Error Code/Message | Meaning | How to Fix |
|-------------------|---------|------------|
| "Email already registered" | Duplicate email address | Use a different email or login to existing account |
| "Invalid credentials" | Wrong email or password | Verify credentials and try again |
| "Token has expired" | Access token expired | Use refresh token to get new access token |
| "Room is not available" | Booking conflict | Select different dates or another room |
| "Reservation not found" | Invalid reservation ID | Verify the reservation ID exists |
| "Insufficient funds" | Payment card declined | Use different payment method |
| "Hotel not approved" | Hotel pending approval | Wait for admin to approve the hotel |

### Authentication Issues

| Issue | Solution |
|-------|----------|
| "Bearer" not included | Format token as: `Bearer <your-token>` |
| Token not working | Ensure no extra spaces or characters |
| Swagger not authorized | Click "Authorize" button and enter token |
| Refresh token invalid | Re-login to get new refresh token |

### API-Specific Issues

| Service | Issue | Solution |
|---------|-------|----------|
| Users | Registration fails | Check email format, password length (8+ chars) |
| Hotels | Search returns empty | Verify location exists, dates are valid |
| Reservations | Cannot create | Ensure room exists, dates available, guest count valid |
| Payments | Intent fails | Verify reservation ID correct, amount positive |

## Getting Help

### Self-Service Resources

- This documentation
- Swagger UI endpoint descriptions
- API response error messages

### Service Health Checks

Check if services are running:
- Users: `https://hotel-booking-users-api-*.azurewebsites.net/health`
- Hotels: `https://hotel-booking-hotels-api-*.azurewebsites.net/health`
- Reservations: `https://hotel-booking-reservations-api-*.azurewebsites.net/health`
- Payments: `https://hotel-booking-payments-api-*.azurewebsites.net/health`

### Reporting Issues

When reporting a problem, please include:

1. **Service and endpoint** - Which API and endpoint were you calling?
2. **Request body** - What data did you send?
3. **Response** - What status code and message did you receive?
4. **Token info** - Was the request authenticated? What role?
5. **Steps to reproduce** - What actions led to the issue?

### GitHub Repositories

- Users: https://github.com/Patsino/hotel-booking-users-service
- Hotels: https://github.com/Patsino/hotel-booking-hotels-service
- Reservations: https://github.com/Patsino/hotel-booking-reservations-service
- Payments: https://github.com/Patsino/hotel-booking-payments-service
