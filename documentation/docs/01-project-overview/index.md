# 1. Project Overview

This section covers the business context, goals, and requirements for the project.

## Contents

- [Problem Statement & Goals](problem-and-goals.md)
- [Stakeholders & Users](stakeholders.md)
- [Scope](scope.md)
- [Features](features.md)

## Executive Summary

The Hotel Booking Platform solves a common frustration: **overly complex booking processes** that discourage users before they even complete a reservation. Modern platforms burden users with endless steps, pop-ups, and forms. Our solution provides a **lightweight, minimal-effort booking experience**—search, select, pay, done. Additionally, the platform serves an underserved market: **pet owners** seeking pet-friendly hotels or dedicated pet care facilities when traveling. With clear pet filters and a pet hotel category, finding accommodation for pets is straightforward. The system is built as a microservices architecture with four independent services (Users, Hotels, Reservations, Payments), deployed on Azure, with a React frontend for a clean user experience.

## Key Highlights

| Aspect | Description |
|--------|-------------|
| **Problem** | Modern booking platforms are overly complex, leading to high abandonment. Pet owners lack visibility into pet-friendly options. |
| **Solution** | Lightweight booking interface with minimal steps + pet hotel support |
| **Target Users** | Travelers wanting fast booking, Pet owners, Hotel Owners (including pet hotels), Administrators |
| **Key Features** | Fast 3-step booking, Pet-friendly filters, Stripe payments, Smart cancellation policies, GDPR compliance |
| **Tech Stack** | .NET 9, ASP.NET Core, React, SQL Server, Azure, Docker, Stripe API |
