# CloudNative.NET-Commerce-System

CloudNative.NET-Commerce-System is a distributed e-commerce platform built with .NET 8 using a microservices architecture.  
The solution covers the full shopping journey: authentication, product browsing, cart management, checkout, order processing, and asynchronous post-order workflows.

## What This Project Includes

- API Gateway with Ocelot for unified API entry
- Independent microservices with database-per-service ownership
- JWT-based API security and role-based authorization
- Event-driven workflows through Azure Service Bus
- ASP.NET Core MVC frontend for end-to-end user flows
- Stripe integration for checkout and payment operations

## Solution Layout

The main solution is located at `Mango/Mango.sln`.

Projects included:

- `Mango.Web` - MVC frontend application
- `Mango.GatewaySolution` - Ocelot API gateway
- `Mango.MessageBus` - shared message publishing library
- `Mango.Services.AuthAPI` - identity, login, registration, JWT issuing
- `Mango.Services.ProductAPI` - product catalog and product image handling
- `Mango.Services.CouponAPI` - coupon management and coupon lookups
- `Mango.Services.ShoppingCartAPI` - cart operations and coupon application
- `Mango.Services.OrderAPI` - checkout, order lifecycle, payment validation
- `Mango.Services.EmailAPI` - background email event consumer
- `Mango.Services.RewardAPI` - background reward event consumer

## Architecture and Communication

### Synchronous flow

- Client requests are sent through `Mango.GatewaySolution`
- Gateway routes requests to downstream service APIs
- Services use REST APIs for request/response operations

### Asynchronous flow

- Services publish domain events to Azure Service Bus
- `EmailAPI` and `RewardAPI` consume events in background workers
- This keeps user-facing APIs responsive while handling side effects asynchronously

### Authentication model

- `AuthAPI` uses ASP.NET Core Identity and issues JWT tokens
- Gateway and protected APIs validate JWT bearer tokens
- `Mango.Web` uses cookie auth for UI session management and sends bearer tokens for API calls

## Data Ownership

Each service maintains its own SQL Server database, including its own EF Core migrations.  
Databases are isolated by service boundary (Auth, Product, Coupon, ShoppingCart, Order, Email, Reward).

## Key Technologies

- .NET 8 / ASP.NET Core
- C# / Entity Framework Core
- SQL Server
- Ocelot API Gateway
- Azure Service Bus
- ASP.NET Core Identity + JWT
- Stripe

## Running Locally

### Prerequisites

- .NET 8 SDK
- SQL Server
- Azure Service Bus namespace/connection string
- Stripe test keys (for order/checkout flow)

### 1) Clone repository

```bash
git clone <your-repository-url>
cd CloudNative.NET-Commerce-System
```

### 2) Restore and build

```bash
dotnet restore "Mango/Mango.sln"
dotnet build "Mango/Mango.sln"
```

### 3) Configure settings

For each service, configure values in `appsettings.json` / environment variables:

- `ConnectionStrings__DefaultConnection`
- `Jwt__Key`, `Jwt__Issuer`, `Jwt__Audience`
- `ServiceBusConnectionString` or `ServiceBus__ConnectionString` (varies by service config shape)
- Stripe configuration in Order/Coupon services

Never commit secrets to source control.

### 4) Run services

Start the gateway and required APIs from the solution (or run each project individually):

```bash
dotnet run --project "Mango/Mango.GatewaySolution/Mango.GatewaySolution.csproj"
dotnet run --project "Mango/Mango.Services.AuthAPI/Mango.Services.AuthAPI.csproj"
dotnet run --project "Mango/Mango.Services.ProductAPI/Mango.Services.ProductAPI.csproj"
dotnet run --project "Mango/Mango.Services.CouponAPI/Mango.Services.CouponAPI.csproj"
dotnet run --project "Mango/Mango.Services.ShoppingCartAPI/Mango.Services.ShoppingCartAPI.csproj"
dotnet run --project "Mango/Mango.Services.OrderAPI/Mango.Services.OrderAPI.csproj"
dotnet run --project "Mango/Mango.Services.EmailAPI/Mango.Services.EmailAPI.csproj"
dotnet run --project "Mango/Mango.Services.RewardAPI/Mango.Services.RewardAPI.csproj"
dotnet run --project "Mango/Mango.Web/Mango.Web.csproj"
```

## Notes

- Database migrations are applied by services on startup in this solution.
- If Service Bus is unavailable, asynchronous features (email/reward processing) will not function.
- Gateway routing is environment-specific (`ocelot.json`, `ocelotLocal.json`, `ocelot.Production.json`).
