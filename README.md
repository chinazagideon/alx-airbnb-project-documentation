
# Airbnb Clone Backend (Microservices & AI-Powered)

-----

##  Project Overview

This project is an ambitious, enterprise-scalable backend for an Airbnb-like rental marketplace. Developed as a key component of a professional portfolio, it showcases a robust **microservices architecture** seamlessly integrated with **Big Data** capabilities and **AI-powered features** such as personalized search and ad listings.

The core objective is to simulate a real-world, high-traffic application, demonstrating proficiency in scalable system design, secure API development, asynchronous communication, and intelligent data processing.


## Requirements
<!-- REQUIREMENTS_START -->

# Backend Feature Specifications: Airbnb Clone

This document outlines detailed requirements for key backend features, including API endpoints, input/output specifications, validation rules, and performance criteria.

-----

## 1\. User Authentication & Profile Management

**Service:** `Identity & User Service`

This service handles all user-related operations, including registration, login, and profile management.

### **1.1. User Registration**

  * **Endpoint:** `POST /auth/register`
  * **Description:** Allows a new user to create an account.
  * **Input (Request Body - `application/json`):**
    ```json
    {
        "email": "user@example.com",
        "password": "StrongPassword123!",
        "first_name": "John",
        "last_name": "Doe",
        "role": "guest" // or "host"
    }
    ```
  * **Validation Rules:**
      * `email`: Required, valid email format, must be unique across all users. Max length 100.
      * `password`: Required, min 8 characters, at least one uppercase letter, one lowercase letter, one number, one special character. Max length 255 (plaintext, will be hashed).
      * `first_name`: Required, min 2 chars, max 50 chars.
      * `last_name`: Required, min 2 chars, max 50 chars.
      * `role`: Required, must be "guest", "host", or "admin". Defaults to "guest".
  * **Output (Response Body - `application/json`):**
      * **Success (HTTP 201 Created):**
        ```json
        {
            "user_id": 123,
            "message": "User registered successfully.",
            "token": "jwt.web.token" // JWT for immediate authentication
        }
        ```
      * **Error (HTTP 400 Bad Request):** If validation fails or email already exists.
        ```json
        {
            "error": "Validation failed",
            "details": {
                "email": "Email already registered."
            }
        }
        ```
  * **Performance Criteria:**
      * **Latency:** Average \< 100ms.
      * **Throughput:** Capable of handling \> 500 registrations/second.

### **1.2. User Login**

  * **Endpoint:** `POST /auth/login`
  * **Description:** Authenticates an existing user.
  * **Input (Request Body - `application/json`):**
    ```json
    {
        "email": "user@example.com",
        "password": "StrongPassword123!"
    }
    ```
  * **Validation Rules:**
      * `email`: Required, valid email format.
      * `password`: Required.
  * **Output (Response Body - `application/json`):**
      * **Success (HTTP 200 OK):**
        ```json
        {
            "user_id": 123,
            "token": "jwt.web.token", // JWT for subsequent authenticated requests
            "role": "guest" // or "host" or "admin"
        }
        ```
      * **Error (HTTP 401 Unauthorized):** If credentials are invalid.
        ```json
        {
            "error": "Invalid email or password."
        }
        ```
  * **Performance Criteria:**
      * **Latency:** Average \< 80ms.
      * **Throughput:** Capable of handling \> 1000 logins/second.

### **1.3. User Profile Management**

  * **Endpoints:**
      * `GET /users/me`: Retrieve authenticated user's profile.
      * `PUT /users/me`: Update authenticated user's profile.
  * **Description:** Allows users to view and modify their personal information.
  * **Authorization:** Requires valid JWT in `Authorization: Bearer <token>` header.
  * **Input (for `PUT /users/me` - `application/json`):**
    ```json
    {
        "first_name": "Jonathan",
        "last_name": "Smith",
        "phone_number": "+2348012345678"
    }
    ```
  * **Validation Rules (for `PUT /users/me`):**
      * `first_name`, `last_name`: Optional, min 2 chars, max 50 chars.
      * `phone_number`: Optional, valid phone number format, max 20 chars.
      * `email`, `password`, `role`: Cannot be changed via this endpoint.
  * **Output (Response Body - `application/json`):**
      * **Success (HTTP 200 OK):**
        ```json
        {
            "user_id": 123,
            "email": "user@example.com",
            "first_name": "Jonathan",
            "last_name": "Smith",
            "phone_number": "+2348012345678",
            "role": "guest",
            "created_at": "2024-01-01T10:00:00Z"
        }
        ```
      * **Error (HTTP 401 Unauthorized):** If token is missing/invalid.
      * **Error (HTTP 400 Bad Request):** If validation fails for input fields.
      * **Error (HTTP 404 Not Found):** If user does not exist (unlikely for `/me`).
  * **Performance Criteria:**
      * **Latency:** Average \< 50ms.
      * **Throughput:** Capable of handling \> 1000 requests/second.

-----

## 2\. Property Listing Management

**Service:** `Listing Service`

This service manages the lifecycle of property listings created by hosts.

### **2.1. Create Property Listing**

  * **Endpoint:** `POST /properties`
  * **Description:** Allows a host to create a new property listing.
  * **Authorization:** Requires valid JWT in `Authorization: Bearer <token>` header, and user must have `host` role.
  * **Input (Request Body - `application/json`):**
    ```json
    {
        "name": "Cozy Apartment in City Center",
        "description": "A charming and modern apartment perfect for your stay.",
        "location": "123 Main St, Abuja, FCT, Nigeria",
        "price_per_night": 75.50
    }
    ```
  * **Validation Rules:**
      * All fields are `required`.
      * `name`: Min 5 chars, max 100 chars.
      * `description`: Min 20 chars, max 65535 chars (TEXT field).
      * `location`: Min 10 chars, max 255 chars.
      * `price_per_night`: Numeric, positive value, max 10 digits with 2 decimal places.
  * **Output (Response Body - `application/json`):**
      * **Success (HTTP 201 Created):**
        ```json
        {
            "property_id": 456,
            "message": "Property listing created successfully."
        }
        ```
      * **Error (HTTP 400 Bad Request):** For validation failures.
      * **Error (HTTP 403 Forbidden):** If user is not a host.
  * **Performance Criteria:**
      * **Latency:** Average \< 150ms.
      * **Throughput:** Capable of handling \> 300 listings/second.

### **2.2. Get Property Listings (Single & List)**

  * **Endpoints:**
      * `GET /properties/{id}`: Retrieve details of a single listing.
      * `GET /properties`: Retrieve a list of listings with optional filters.
  * **Description:** Allows users to view property details or browse/search listings.
  * **Authorization:** No authentication required for public listings (can be public or authenticated for more details).
  * **Input (for `GET /properties` - Query Parameters):**
      * `location`: String, filter by location (partial match).
      * `min_price`, `max_price`: Numeric, price range.
      * `host_id`: Integer, filter by host.
      * `page`: Integer, default 1.
      * `limit`: Integer, default 10, max 100.
      * `sort_by`: String (e.g., `price_asc`, `price_desc`, `created_at_desc`).
  * **Output (Response Body - `application/json`):**
      * **Success (HTTP 200 OK for `GET /properties/{id}`):**
        ```json
        {
            "property_id": 456,
            "host_id": 123,
            "name": "Cozy Apartment in City Center",
            "description": "A charming and modern apartment perfect for your stay.",
            "location": "123 Main St, Abuja, FCT, Nigeria",
            "price_per_night": 75.50,
            "status": "available",  // e.g., published, unpublished
            "created_at": "2024-01-01T10:00:00Z",
            "updated_at": "2024-01-01T10:00:00Z"
        }
        ```
      * **Success (HTTP 200 OK for `GET /properties`):** An array of property objects, potentially paginated.
        ```json
        {
            "properties": [
                { "property_id": 456, "name": "...", "price_per_night": 75.50, ... },
                { "property_id": 457, "name": "...", "price_per_night": 120.00, ... }
            ],
            "total_count": 150,
            "page": 1,
            "limit": 10
        }
        ```
      * **Error (HTTP 404 Not Found):** For `GET /properties/{id}` if ID does not exist.
      * **Error (HTTP 400 Bad Request):** For invalid query parameters.
  * **Performance Criteria:**
      * **Latency:** Average \< 100ms for single get; \< 150ms for list/search.
      * **Throughput:** Capable of handling \> 500 requests/second.

-----

## 3\. Booking System

**Service:** `Booking Service`

This service manages the creation, modification, and cancellation of property bookings.

### **3.1. Create Booking**

  * **Endpoint:** `POST /bookings`
  * **Description:** Allows a guest to initiate a booking for a property.
  * **Authorization:** Requires valid JWT in `Authorization: Bearer <token>` header, and user must have `guest` role.
  * **Input (Request Body - `application/json`):**
    ```json
    {
        "property_id": 456,
        "start_date": "2025-07-10",
        "end_date": "2025-07-15"
    }
    ```
  * **Validation Rules:**
      * `property_id`: Required, must be a valid, existing, and available property.
      * `start_date`, `end_date`: Required, valid date format (YYYY-MM-DD), `start_date` must be before `end_date`, both dates must be in the future.
      * **Availability Check (Critical):** The system must verify that the property is available for the entire requested date range, considering existing bookings. This involves querying `BookingDB` for conflicts.
      * **Price Calculation:** The `locked_price_per_night` is set to the current property price at the time of booking.
  * **Output (Response Body - `application/json`):**
      * **Success (HTTP 201 Created):**
        ```json
        {
            "booking_id": 789,
            "user_id": 123,
            "property_id": 456,
            "start_date": "2025-07-10",
            "end_date": "2025-07-15",
            "locked_price_per_night": 75.50,
            "status": "pending", // Initial status
            "message": "Booking initiated. Proceed to payment."
        }
        ```
      * **Error (HTTP 400 Bad Request):** For validation or availability failures.
      * **Error (HTTP 403 Forbidden):** If user is not a guest.
      * **Error (HTTP 404 Not Found):** If `property_id` does not exist.
  * **Performance Criteria:**
      * **Latency:** Average \< 200ms (due to availability checks and price calculation).
      * **Throughput:** Capable of handling \> 200 bookings/second.

### **3.2. Get Booking Details (Single & List)**

  * **Endpoints:**
      * `GET /bookings/{id}`: Retrieve details of a single booking.
      * `GET /bookings`: Retrieve a list of bookings filtered by user/host.
  * **Description:** Allows guests to view their own bookings, and hosts to view bookings for their properties. Admins can view any booking.
  * **Authorization:** Requires valid JWT. Only the associated guest, host, or an admin can access booking details.
  * **Input (for `GET /bookings` - Query Parameters):**
      * `user_id`: Integer (optional, filter by guest).
      * `property_id`: Integer (optional, filter by property).
      * `status`: String (e.g., `pending`, `confirmed`, `canceled`).
      * `page`, `limit`: Pagination parameters.
  * **Output (Response Body - `application/json`):**
      * **Success (HTTP 200 OK for `GET /bookings/{id}`):**
        ```json
        {
            "booking_id": 789,
            "user_id": 123,
            "property_id": 456,
            "start_date": "2025-07-10",
            "end_date": "2025-07-15",
            "locked_price_per_night": 75.50,
            "status": "confirmed", // e.g., pending, confirmed, canceled
            "created_at": "2025-06-29T10:00:00Z"
        }
        ```
      * **Success (HTTP 200 OK for `GET /bookings`):** An array of booking objects.
      * **Error (HTTP 401 Unauthorized):** If token is missing/invalid.
      * **Error (HTTP 403 Forbidden):** If user is not authorized to view the booking.
      * **Error (HTTP 404 Not Found):** If `booking_id` does not exist.
  * **Performance Criteria:**
      * **Latency:** Average \< 80ms.
      * **Throughput:** Capable of handling \> 400 requests/second.

### **3.3. Cancel Booking**

  * **Endpoint:** `PUT /bookings/{id}/cancel`
  * **Description:** Allows a guest or host (under certain conditions) to cancel a booking. Admins can cancel any booking.
  * **Authorization:** Requires valid JWT. Only the associated guest, host, or an admin can cancel.
  * **Input (Request Body - `application/json`):**
    ```json
    {
        "reason": "Change of plans" // Optional cancellation reason
    }
    ```
  * **Validation Rules:**
      * `booking_id`: Required, must exist.
      * **Status Check:** Only bookings with certain statuses (e.g., `pending`, `confirmed`) can be cancelled.
      * **Cancellation Policy Enforcement:** System applies relevant cancellation policy rules (e.g., full refund within 24h, 50% refund after 24h). This might trigger a refund process with the Payment Service.
  * **Output (Response Body - `application/json`):**
      * **Success (HTTP 200 OK):**
        ```json
        {
            "booking_id": 789,
            "status": "canceled",
            "refund_amount": 188.75, // Amount refunded based on policy
            "message": "Booking cancelled successfully. Refund initiated."
        }
        ```
      * **Error (HTTP 401 Unauthorized):** If token is missing/invalid.
      * **Error (HTTP 403 Forbidden):** If user is not authorized to cancel this booking.
      * **Error (HTTP 404 Not Found):** If `booking_id` does not exist.
      * **Error (HTTP 400 Bad Request):** If booking cannot be cancelled in its current state (e.g., already completed, too late for cancellation).
  * **Performance Criteria:**
      * **Latency:** Average \< 120ms (due to policy checks and potential refund initiation).
      * **Throughput:** Capable of handling \> 200 requests/second.

-----

## 4\. Payment System

**Service:** `Payment Service`

This service manages payment processing for bookings.

### **4.1. Process Payment**

  * **Endpoint:** `POST /payments`
  * **Description:** Processes payment for a booking.
  * **Authorization:** Requires valid JWT in `Authorization: Bearer <token>` header.
  * **Input (Request Body - `application/json`):**
    ```json
    {
        "booking_id": 789,
        "amount": 377.50,
        "payment_method": "credit_card"
    }
    ```
  * **Validation Rules:**
      * `booking_id`: Required, must be a valid existing booking.
      * `amount`: Required, positive decimal value, max 10 digits with 2 decimal places.
      * `payment_method`: Required, must be "credit_card", "paypal", or "stripe".
      * `payment_date`: Automatically set to current date.
  * **Output (Response Body - `application/json`):**
      * **Success (HTTP 201 Created):**
        ```json
        {
            "payment_id": 101,
            "booking_id": 789,
            "amount": 377.50,
            "payment_date": "2025-06-29",
            "payment_method": "credit_card",
            "message": "Payment processed successfully."
        }
        ```
      * **Error (HTTP 400 Bad Request):** For validation failures.
      * **Error (HTTP 404 Not Found):** If `booking_id` does not exist.
  * **Performance Criteria:**
      * **Latency:** Average \< 300ms (due to payment processing).
      * **Throughput:** Capable of handling \> 100 payments/second.

-----

## 5\. Review System

**Service:** `Review Service`

This service manages property reviews and ratings.

### **5.1. Create Review**

  * **Endpoint:** `POST /reviews`
  * **Description:** Allows a guest to create a review for a property they've stayed at.
  * **Authorization:** Requires valid JWT in `Authorization: Bearer <token>` header.
  * **Input (Request Body - `application/json`):**
    ```json
    {
        "property_id": 456,
        "rating": 5,
        "comment": "Excellent stay! The apartment was clean and well-located."
    }
    ```
  * **Validation Rules:**
      * `property_id`: Required, must be a valid existing property.
      * `rating`: Required, integer between 1 and 5.
      * `comment`: Required, min 10 chars, max 65535 chars (TEXT field).
      * **Eligibility Check:** User must have completed a booking for this property.
  * **Output (Response Body - `application/json`):**
      * **Success (HTTP 201 Created):**
        ```json
        {
            "review_id": 202,
            "property_id": 456,
            "user_id": 123,
            "rating": 5,
            "comment": "Excellent stay! The apartment was clean and well-located.",
            "created_at": "2025-06-29T10:00:00Z",
            "message": "Review submitted successfully."
        }
        ```
      * **Error (HTTP 400 Bad Request):** For validation failures.
      * **Error (HTTP 403 Forbidden):** If user is not eligible to review this property.
  * **Performance Criteria:**
      * **Latency:** Average \< 100ms.
      * **Throughput:** Capable of handling \> 200 reviews/second.

-----

## 6\. Messaging System

**Service:** `Message Service`

This service manages communication between users.

### **6.1. Send Message**

  * **Endpoint:** `POST /messages`
  * **Description:** Allows users to send messages to each other.
  * **Authorization:** Requires valid JWT in `Authorization: Bearer <token>` header.
  * **Input (Request Body - `application/json`):**
    ```json
    {
        "recipient_id": 456,
        "message_body": "Hi! I'm interested in your property. Is it available for next weekend?"
    }
    ```
  * **Validation Rules:**
      * `recipient_id`: Required, must be a valid existing user ID, cannot be the same as sender.
      * `message_body`: Required, min 1 char, max 65535 chars (TEXT field).
      * `sent_at`: Automatically set to current timestamp.
  * **Output (Response Body - `application/json`):**
      * **Success (HTTP 201 Created):**
        ```json
        {
            "message_id": 303,
            "sender_id": 123,
            "recipient_id": 456,
            "message_body": "Hi! I'm interested in your property. Is it available for next weekend?",
            "sent_at": "2025-06-29T10:00:00Z",
            "message": "Message sent successfully."
        }
        ```
      * **Error (HTTP 400 Bad Request):** For validation failures.
      * **Error (HTTP 404 Not Found):** If `recipient_id` does not exist.
  * **Performance Criteria:**
      * **Latency:** Average \< 50ms.
      * **Throughput:** Capable of handling \> 500 messages/second.

### **6.2. Get Messages**

  * **Endpoints:**
      * `GET /messages`: Retrieve messages for the authenticated user.
  * **Description:** Allows users to view their message conversations.
  * **Authorization:** Requires valid JWT in `Authorization: Bearer <token>` header.
  * **Input (Query Parameters):**
      * `conversation_with`: Integer (optional, filter by specific user).
      * `page`, `limit`: Pagination parameters.
  * **Output (Response Body - `application/json`):**
      * **Success (HTTP 200 OK):**
        ```json
        {
            "messages": [
                {
                    "message_id": 303,
                    "sender_id": 123,
                    "recipient_id": 456,
                    "message_body": "Hi! I'm interested in your property.",
                    "sent_at": "2025-06-29T10:00:00Z"
                }
            ],
            "total_count": 50,
            "page": 1,
            "limit": 10
        }
        ```
      * **Error (HTTP 401 Unauthorized):** If token is missing/invalid.
  * **Performance Criteria:**
      * **Latency:** Average \< 80ms.
      * **Throughput:** Capable of handling \> 300 requests/second.

-----

<!-- REQUIREMENTS_END -->
