# Test Automation Framework

[![CI](https://github.com/demiremirhan/test-automation-framework/actions/workflows/ci.yml/badge.svg)](https://github.com/demiremirhan/test-automation-framework/actions/workflows/ci.yml)
[![Allure Report](https://img.shields.io/badge/Allure-Report-orange?logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIyNCIgaGVpZ2h0PSIyNCIgdmlld0JveD0iMCAwIDI0IDI0Ij48cGF0aCBmaWxsPSIjZmZmIiBkPSJNMTIgMkw0IDdWMTdMMTIgMjJMMjAgMTdWN0wxMiAyWiIvPjwvc3ZnPg==)](https://demiremirhan.github.io/test-automation-framework)

Multi-module Maven project implementing production-grade test automation for API and UI testing. Built with Java 21, REST Assured, Selenium WebDriver, and JUnit 5.

## Tech Stack

| Layer | Tools |
|-------|-------|
| Language | Java 21 |
| API Testing | REST Assured, JUnit 5 |
| UI Testing | Selenium WebDriver 4.27, Page Object Model |
| DB Testing     | Testcontainers 1.20, PostgreSQL 16, JDBC |
| BDD | Cucumber / Gherkin (planned) |
| Reporting | Allure 2.29 → GitHub Pages |
| CI/CD | GitHub Actions (parallel jobs) |
| Infrastructure | Docker Compose, Selenium Grid 4.27 (Hub + Chrome Node) |
| Config | OWNER library (type-safe, environment-aware) |

## Project Structure

```
test-automation-framework/
├── common/                          # Shared config module
│   └── src/main/java/.../config/
│       ├── FrameworkConfig.java      # Type-safe config interface (OWNER)
│       └── ConfigProvider.java       # Singleton config access
├── api-tests/                       # API test module
│   └── src/test/java/.../api/
│       ├── client/                   # Service objects (ProductClient, SpecFactory)
│       ├── model/                    # POJOs (Product, ProductList)
│       └── tests/                    # Test classes (ProductTests)
├── ui-tests/                        # UI test module
│   └── src/test/java/.../ui/
│       ├── driver/                   # DriverFactory (ThreadLocal, RemoteWebDriver)
│       ├── pages/                    # Page Objects (Login, Products, Cart, Checkout)
│       └── tests/                    # Test classes (LoginTests, CartTests, CheckoutTests)
├── db-tests/                        # Database test module
│   ├── src/test/java/.../db/
│   │   ├── dao/                      # Data access objects (ProductDao, OrderDao)
│   │   ├── model/                    # Java records (Product, Customer, Order)
│   │   └── tests/                    # Test classes
│   └── src/test/resources/
│       ├── init-schema.sql           # Tables, FK, CHECK, INDEX
│       └── seed-data.sql             # Test fixtures
├── .github/workflows/ci.yml         # CI pipeline (API + UI parallel, Allure deploy)
├── docker-compose.yml               # Selenium Grid + test runners
├── Dockerfile                       # API test container
└── Dockerfile-ui                    # UI test container
```

## Test Coverage

**API Tests** — DummyJSON API (`https://dummyjson.com`)
- Product CRUD operations
- Response validation with POJOs
- Request/response spec reuse via SpecFactory

**UI Tests** — SauceDemo (`https://www.saucedemo.com`)
- Login: standard user, locked-out user, invalid credentials
- Cart: add single/multiple items, remove items, verify contents
- Checkout: single item flow, multi-item flow, order completion

**DB Tests** — PostgreSQL 16 via Testcontainers

- CRUD operations through DAO layer (`ProductDao`, `OrderDao`)
- Aggregation queries (SUM, AVG, GROUP BY) and multi-table JOINs
- Constraint enforcement: foreign key, CHECK, and NOT NULL violations
- Schema and fixtures loaded per container from `init-schema.sql` / `seed-data.sql`
- Container lifecycle managed by JUnit 5, no local database required

## Getting Started

**Prerequisites:** Java 21, Maven 3.9+, Docker

### Run with Docker (Selenium Grid)

```bash
# Start Selenium Grid
docker compose --profile ui up -d selenium-hub chrome-node

# Run UI tests against Grid
mvn -pl ui-tests -am clean test -Dselenium.grid.url=http://localhost:4444/wd/hub

# Run API tests
mvn -pl api-tests -am clean test

# Run everything
mvn clean test
```

### Run in Docker containers

```bash
# API tests only
docker compose up api-tests

# Full UI stack (Grid + tests)
docker compose --profile ui up
```

### Configuration

Defined in `common/src/main/resources/config.properties`, overridable via system properties:

| Property | Default | Description |
|----------|---------|-------------|
| `api.base.uri` | `https://dummyjson.com` | API base URL |
| `api.timeout.ms` | `10000` | API request timeout |
| `ui.base.url` | `https://www.saucedemo.com` | UI test target |
| `selenium.grid.url` | `http://localhost:4444/wd/hub` | Selenium Grid endpoint |

## CI/CD Pipeline

GitHub Actions runs on every push:

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  API Tests   │  │  UI Tests    │  │  DB Tests    │
│  (JDK 21)    │  │  Selenium    │  │  Test-       │
│  mvn test    │  │  Grid        │  │  containers  │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                 │
       └────────┬────────┴─────────────────┘
                ▼
       ┌─────────────────────┐
       │  Allure Report      │
       │  → GitHub Pages     │
       └─────────────────────┘
```

API and UI tests run in parallel. Results are merged into a single Allure report and deployed to GitHub Pages on `master`.

📊 **[Live Allure Report](https://demiremirhan.github.io/test-automation-framework)**

## Design Decisions

- **ThreadLocal WebDriver** — each test gets its own browser session, safe for parallel execution
- **Page Object Model** — UI interactions encapsulated per page, tests read like business flows
- **Service Object Pattern** — API calls abstracted behind client classes with reusable specs
- **OWNER Config** — type-safe configuration with environment-based property file resolution
- **Allure @Step annotations** — every page action appears in the report as a readable step
