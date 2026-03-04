# Fake Store API Test Suite

A Postman collection for functional API testing of the [Fake Store API](https://fakestoreapi.com) — a free, open REST API for e-commerce prototyping and testing. The collection covers full CRUD operations across all four core API resources, with both valid and invalid request scenarios, environment-driven variables, and pre-request schema validation.

---

## Table of Contents

- [What This Project Tests](#what-this-project-tests)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [Collection Structure](#collection-structure)
- [Request Coverage](#request-coverage)
- [Test Techniques Used](#test-techniques-used)
- [How to Run](#how-to-run)
- [Test Report](#test-report)
- [API Reference](#api-reference)

---

## What This Project Tests

This collection validates the Fake Store API across four resource types — Authentication, Users, Products, and Carts — testing both the happy path and error scenarios for each endpoint. Each request folder contains a valid and an invalid variant to verify that the API handles both correct inputs and bad requests appropriately.

---

## Tech Stack

| Tool | Purpose |
|---|---|
| Postman | API test authoring and manual test execution |
| Postman Environments | Variable management across requests |
| JavaScript (pm.*) | Pre-request scripts and test assertions |
| Newman (optional) | Command-line collection runner |
| Chai Assertion Library | Assertion syntax used within Postman test scripts |

---

## Prerequisites

### 1. Postman
Download and install from [postman.com](https://www.postman.com/downloads/)

### 2. Node.js + Newman (optional — for command-line execution)
Download Node.js from [nodejs.org](https://nodejs.org/), then install Newman:
```bash
npm install -g newman
npm install -g newman-reporter-htmlextra
```

---

## Getting Started

### 1. Clone the repository
```bash
git clone https://github.com/YOUR_USERNAME/fakestore-api-tests.git
cd fakestore-api-tests
```

### 2. Import into Postman

**Import the collection:**
- Open Postman
- Click **Import** (top left)
- Select `Fake Store API Test Suite.postman_collection.json`

**Import the environment:**
- Click **Import** again
- Select `Fake Store API ENV.postman_environment.json`
- In the top-right environment dropdown, select **Fake Store API ENV**

### 3. Run the collection
- Right-click the collection in the sidebar
- Select **Run collection**
- Click **Run Fake Store API Test Suite**

---

## Environment Variables

The collection uses the **Fake Store API ENV** environment. Variables are either pre-set or populated dynamically during test execution.

| Variable | Type | Set By | Description |
|---|---|---|---|
| `base_url` | default | Pre-set | `https://fakestoreapi.com` |
| `auth_token` | dynamic | Valid Login test | JWT token saved after successful login |
| `user_id` | dynamic | Add a New User test | ID of newly created user |
| `newProductId` | dynamic | Add A New Product test | ID of newly created product |
| `productTitle` | dynamic | Pre-request script | Product title used in request body |
| `productPrice` | dynamic | Pre-request script | Product price used in request body |
| `productDescription` | dynamic | Pre-request script | Product description used in request body |
| `productCategory` | dynamic | Pre-request script | Product category used in request body |
| `productImage` | dynamic | Pre-request script | Product image URL used in request body |

> **Note:** Dynamic variables are set automatically during test execution. No manual setup is required beyond importing the environment file.

---

## Collection Structure

```
Fake Store API Test Suite
│
├── Authentication
│   ├── Missing Username         POST /auth/login  (empty username)
│   ├── Invalid Details          POST /auth/login  (wrong credentials)
│   └── Valid Login              POST /auth/login  (saves auth_token)
│
├── Users
│   ├── Get All Users
│   │   ├── Get All Users - Valid    GET /users
│   │   └── Get All Users - Invalid  GET /users   (same endpoint, tests resilience)
│   ├── Add a New User
│   │   ├── Add a New User - Valid    POST /users  (saves user_id)
│   │   └── Add a New User - Invalid  POST /userss (typo endpoint → 404)
│   └── {id} required
│       ├── Get a Single User        GET /users/1
│       ├── Update a User            PUT /users/{{user_id}}
│       └── Delete a User            DELETE /users/{{user_id}}
│
├── Products
│   ├── Get All Products
│   │   ├── Get All Products - Valid    GET /products
│   │   └── Get All Products - Invalid  GET /productss (typo endpoint → 404)
│   ├── Add A New Product
│   │   ├── Add A New Product - Valid    POST /products  (saves newProductId)
│   │   └── Add A New Product - Invalid  POST /productss (typo endpoint → 404)
│   ├── Get A Single Product
│   │   ├── Get A Single Product - Valid    GET /products/{{newProductId}}
│   │   └── Get A Single Product - Invalid  GET /productss/{{newProductId}} → 404
│   ├── Update A Product
│   │   ├── Update A Product - Valid    PUT /products/{{newProductId}}
│   │   └── Update A Product - Invalid  PUT /productss/{{newProductId}} → 404
│   └── Delete A Product             DELETE /products/{{newProductId}}
│
└── Carts
    ├── Get All Carts
    │   ├── Get All Carts - Valid    GET /carts
    │   └── Get All Carts - Invalid  GET /cartss → 404
    ├── Add A New Cart
    │   ├── Add A New Cart - Valid    POST /carts
    │   └── Add A New Cart - Invalid  POST /cartss → 404
    └── Get A Single Cart
        ├── Get A Single Cart - Valid    GET /carts/1
        └── Get A Single Cart - Invalid  GET /cartss → 404
```

---

## Request Coverage

### Authentication — `POST /auth/login`

| Request | Scenario | Key Assertions |
|---|---|---|
| Missing Username | Empty username field | Status 400/401, error message contains "not provided", no token returned |
| Invalid Details | Wrong username and password | Status 400/401, error message contains "incorrect", no token returned |
| Valid Login | Correct credentials | Status 200/201, response contains JWT token as string, token saved to environment |

**Notable:** Valid Login uses a **pre-request script** to validate the request body schema before sending — checking it is valid JSON, contains required fields (`username`, `password`), and that both are strings.

---

### Users — `/users`

| Request | Method | Scenario | Key Assertions |
|---|---|---|---|
| Get All Users - Valid | GET | Retrieve full user list | Status 200, array response, each user has `id`, `username`, `email`, `password`, correct data types, response time under 2000ms |
| Get All Users - Invalid | GET | Same endpoint, resilience check | Status 200, array not empty, data types correct |
| Add a New User - Valid | POST | Create new user | Status 201, valid JSON, response contains only `id`, no sensitive data leaked (`password`, `email`, `username` absent), ID is a number, `user_id` saved |
| Add a New User - Invalid | POST | Invalid endpoint `/userss` | Status 404 |
| Get a Single User | GET | Retrieve user by ID | Status 200, valid JSON, response has `id`, `username`, `email`, `password` |
| Update a User | PUT | Update user by `{{user_id}}` | No assertions (request only) |
| Delete a User | DELETE | Delete user by `{{user_id}}` | Status 200 |

---

### Products — `/products`

| Request | Method | Scenario | Key Assertions |
|---|---|---|---|
| Get All Products - Valid | GET | Retrieve all products | Status 200, array response, each product has `id`, `title`, `price`, `description`, correct data types, response time under 2000ms |
| Get All Products - Invalid | GET | Invalid endpoint `/productss` | Status 404 |
| Add A New Product - Valid | POST | Create product with env variables | Status 200/201, mandatory fields present (`id`, `title`, `price`, `description`, `category`, `image`), correct data types, image matches URL format, response matches sent payload, `newProductId` saved |
| Add A New Product - Invalid | POST | Invalid endpoint `/productss` | Status 404 |
| Get A Single Product - Valid | GET | Retrieve product by `{{newProductId}}` | Status 200 |
| Get A Single Product - Invalid | GET | Invalid endpoint `/productss/{{newProductId}}` | Status 404 |
| Update A Product - Valid | PUT | Update product with new env variables | Status 200/201, mandatory fields present, data types correct, response matches updated payload |
| Update A Product - Invalid | PUT | Invalid endpoint `/productss/{{newProductId}}` | Status 404 |
| Delete A Product | DELETE | Delete product by `{{newProductId}}` | Status 200 |

**Notable:** Add and Update Product requests use **pre-request scripts** to dynamically set environment variables (`productTitle`, `productPrice`, etc.) before sending, enabling data-driven requests without hardcoded values.

---

### Carts — `/carts`

| Request | Method | Scenario | Key Assertions |
|---|---|---|---|
| Get All Carts - Valid | GET | Retrieve all carts | Status 200, array response, each cart has `id`, `userId`, `date`, `products`, correct data types, response time under 2000ms |
| Get All Carts - Invalid | GET | Invalid endpoint `/cartss` | Status 404 |
| Add A New Cart - Valid | POST | Create new cart | Status 200/201, response contains `id`, `userId`, `products`, correct data types (`id` and `userId` are numbers, `products` is array) |
| Add A New Cart - Invalid | POST | Invalid endpoint `/cartss` | Status 404 |
| Get A Single Cart - Valid | GET | Retrieve cart by ID | Status 200 |
| Get A Single Cart - Invalid | GET | Invalid endpoint `/cartss` | Status 404 |

---

## Test Techniques Used

**Status code validation** — every request asserts the expected HTTP status code, covering both success codes (200, 201) and error codes (400, 401, 404).

**Response body assertions** — tests verify the structure, presence, and values of response fields using Chai-style assertions within Postman's `pm.test()` framework.

**Schema/data type validation** — tests confirm that each field in the response is the correct type (number, string, array) to catch type mismatches that status codes alone would not reveal.

**Response time checks** — collection-level and request-level tests assert that responses are returned within 2000ms, establishing a basic performance baseline.

**Pre-request schema validation** — the Valid Login request validates the outgoing request body before sending, checking it is well-formed JSON and contains required fields with correct types.

**Data-driven pre-request scripts** — product requests dynamically set environment variables in pre-request scripts so request payloads are constructed from environment data rather than hardcoded values.

**Environment variable chaining** — IDs generated by create operations (`user_id`, `newProductId`, `auth_token`) are saved to the environment and automatically consumed by subsequent read, update, and delete requests. This enables dependent request flows without manual copy-pasting of IDs.

**Security assertion** — the Add a New User test verifies that no sensitive fields (`password`, `email`, `username`) are returned in the creation response, and that only the generated `id` is present.

---

## How to Run

### In Postman (manual)

1. Open the collection in Postman
2. Ensure the **Fake Store API ENV** environment is selected in the top-right dropdown
3. Right-click the collection → **Run collection**
4. Click **Run Fake Store API Test Suite**

### Via Newman (command line)

Run with console output:
```bash
newman run "Fake Store API Test Suite.postman_collection.json" \
  -e "Fake Store API ENV.postman_environment.json"
```

Run with HTML report:
```bash
newman run "Fake Store API Test Suite.postman_collection.json" \
  -e "Fake Store API ENV.postman_environment.json" \
  -r htmlextra \
  --reporter-htmlextra-export report.html
```

---

## Test Report

A pre-generated HTML test report (`report.html`) is included in this repository. It shows the full collection run results including pass/fail status for each request and assertion.

To view it, open `report.html` in any browser.

To generate a fresh report after making changes, run the Newman command above with the `htmlextra` reporter.

---

## API Reference

Full API documentation is available at [fakestoreapi.com](https://fakestoreapi.com).

| Resource | Base Endpoint |
|---|---|
| Authentication | `POST /auth/login` |
| Users | `/users` |
| Products | `/products` |
| Carts | `/carts` |
