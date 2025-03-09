# Microservices Gopher

This project is a Go-based microservices architecture for an e-commerce system, consisting of four independent services: User, Order, Payment, and Notification. The User Service handles user creation, while the Order Service manages order creation and updates. The Payment Service integrates with Stripe to process transactions, and the Notification Service sends email updates using SMTP.

## Design

<img width="1318" alt="Screenshot 2025-02-22 at 12 06 53 PM" src="https://github.com/user-attachments/assets/7547d795-f682-4e8e-9ccf-25b84ec8e87c" />

## Architecture

The system follows a microservices architecture built with Go, where each service is independent and communicates via HTTP APIs. Below is the flow and the API endpoints for each service:

#### Service Flow
1. **User Service**: Handles user creation and retrieval. It serves as the entry point for user-related operations.
2. **Order Service**: Manages order creation and updates, interacting with the User Service for user validation and Payment Service for transaction processing.
3. **Payment Service**: Processes payments by integrating with Stripe. It receives payment requests from the Order Service.
4. **Notification Service**: Sends email updates via SMTP after receiving notifications from other services (e.g., order confirmation or payment success).

#### API Endpoints

- **User Service (`user-svc`)**
  - `POST /api/users`: Creates a new user.
    - Handler: `userHandler(db)`
  - `GET /api/users/{id}`: Retrieves a user by ID.
    - Handler: `getUserHandler(db)`

- **Order Service (`order-svc`)**
  - `POST /api/orders`: Creates or updates an order.
    - Handler: `orderHandler(db)`

- **Payment Service (`payment-svc`)**
  - `POST /api/payments`: Processes a payment via Stripe.
    - Example client call: `client.Post("http://payment-svc:3003/api/payments")`

- **Notification Service (`notification-svc`)**
  - `POST /api/notifications`: Sends an email notification.
    - Endpoint: `http://notification-svc:3004/api/notifications`

Each service runs in its own Docker container and communicates via these endpoints using HTTP requests. The services rely on a shared database (`db`) where applicable, and environment variables configure service-specific settings (e.g., Stripe API keys, SMTP credentials).

## Keploy Test

Below are the commands to build Docker images and run tests for each service using Keploy. These assume each service has its own Dockerfile in the respective service directory and is part of a `keploy-network` for testing.

- **User Service (`user-svc`)**
  - **Build Command**: 
    ```bash
    docker build -t user-service-contract ./user-svc

  - **Test Command**: 
    ```bash
    keploy record -c "docker run -p 3001:3001 --name user-svc --network keploy-network user-service-contract" --container-name "user-svc" --buildDelay 60 
  
- **Order Service (`order-svc`)**
  - **Build Command**: 
    ```bash
    docker build -t order-service-contract ./order-svc

  - **Test Command**: 
    ```bash
    keploy record -c "docker run -p 3002:3002 --name order-svc --network keploy-network order-service-contract" --container-name "order-svc" --buildDelay 60 
  
- **Payment Service (`payment-svc`)**
  - **Build Command**: 
    ```bash
    docker build -t payment-service-contract ./user-svc

  - **Test Command**: 
    ```bash
    keploy record -c "docker run -p 3003:3003 --name payment-svc --network keploy-network payment-service-contract" --container-name "payment-svc" --buildDelay 60
  
- **Notification Service (`notification-svc`)**
  - **Build Command**: 
    ```bash
    docker build -t notification-service-contract ./user-svc

  - **Test Command**: 
    ```bash
    keploy record -c "docker run -p 3004:3004 --name notification-svc --network keploy-network notification-service-contract" --container-name "notification-svc" --buildDelay 60
  
## Contract Testing for each service

1. Generating OpenAPI schemas for each service  

```bash 
 keploy contract generate 
```

2. Run the  command to run consumer driven contract testing

```bash 
 keploy contract test --driven "consumer" 
```

