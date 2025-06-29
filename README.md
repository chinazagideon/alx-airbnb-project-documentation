
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
      * `email`: Required, valid email format, must be unique across all users. Max length 255.
      * `password`: Required, min 8 characters, at least one uppercase letter, one lowercase letter, one number, one special character. Max length 255 (plaintext, will be hashed).
      * `first_name`: Required, min 2 chars, max 50 chars.
      * `last_name`: Required, min 2 chars, max 50 chars.
      * `role`: Required, must be "guest" or "host".
  * **Output (Response Body - `application/json`):**
      * **Success (HTTP 201 Created):**
        ```json
        {
            "user_id": "uuid-of-new-user",
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
            "user_id": "uuid-of-existing-user",
            "token": "jwt.web.token", // JWT for subsequent authenticated requests
            "role": "guest" // or "host"
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
        "phone_number": "+2348012345678",
        "profile_picture_url": "https://cdn.example.com/user/profile_pic.jpg"
    }
    ```
  * **Validation Rules (for `PUT /users/me`):**
      * `first_name`, `last_name`: Optional, min 2 chars, max 50 chars.
      * `phone_number`: Optional, valid phone number format (e.g., E.164).
      * `profile_picture_url`: Optional, valid URL format.
      * `email`, `password`, `role`: Cannot be changed via this endpoint.
  * **Output (Response Body - `application/json`):**
      * **Success (HTTP 200 OK):**
        ```json
        {
            "user_id": "uuid-of-user",
            "email": "user@example.com",
            "first_name": "Jonathan",
            "last_name": "Doe",
            "phone_number": "+2348012345678",
            "profile_picture_url": "https://cdn.example.com/user/profile_pic.jpg",
            "role": "guest"
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

  * **Endpoint:** `POST /listings`
  * **Description:** Allows a host to create a new property listing.
  * **Authorization:** Requires valid JWT in `Authorization: Bearer <token>` header, and user must have `host` role.
  * **Input (Request Body - `application/json`):**
    ```json
    {
        "title": "Cozy Apartment in City Center",
        "description": "A charming and modern apartment perfect for your stay.",
        "address": "123 Main St",
        "city": "Abuja",
        "state": "FCT",
        "zip_code": "900001",
        "country": "Nigeria",
        "latitude": 9.0765,
        "longitude": 7.3986,
        "price_per_night": 75.50,
        "num_bedrooms": 1,
        "num_bathrooms": 1,
        "max_guests": 2,
        "amenities": ["Wifi", "Kitchen", "Air Conditioning"],
        "image_urls": ["url_from_file_service_1", "url_from_file_service_2"]
    }
    ```
  * **Validation Rules:**
      * All fields are `required` except `amenities` and `image_urls` (though at least one image is recommended).
      * `title`: Min 10 chars, max 100 chars.
      * `description`: Min 50 chars, max 1000 chars.
      * `address`, `city`, `state`, `zip_code`, `country`: Standard string validation, max 100 chars.
      * `latitude`, `longitude`: Valid geographical coordinates (e.g., -90 to 90 for latitude, -180 to 180 for longitude).
      * `price_per_night`: Numeric, positive value.
      * `num_bedrooms`, `num_bathrooms`, `max_guests`: Integer, positive value.
      * `amenities`: Array of strings, each string max 50 chars.
      * `image_urls`: Array of valid URLs, max 10 URLs. (Assumes images are uploaded via a separate file service first).
  * **Output (Response Body - `application/json`):**
      * **Success (HTTP 201 Created):**
        ```json
        {
            "listing_id": "uuid-of-new-listing",
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
      * `GET /listings/{id}`: Retrieve details of a single listing.
      * `GET /listings`: Retrieve a list of listings with optional filters.
  * **Description:** Allows users to view property details or browse/search listings.
  * **Authorization:** No authentication required for public listings (can be public or authenticated for more details).
  * **Input (for `GET /listings` - Query Parameters):**
      * `city`: String, filter by city.
      * `min_price`, `max_price`: Numeric, price range.
      * `num_guests`: Integer, filter by max guests.
      * `amenities`: Comma-separated strings, filter by amenities.
      * `check_in_date`, `check_out_date`: Date (YYYY-MM-DD), filter by availability (requires interaction with Booking Service).
      * `page`: Integer, default 1.
      * `limit`: Integer, default 10, max 100.
      * `sort_by`: String (e.g., `price_asc`, `price_desc`, `popularity`).
  * **Output (Response Body - `application/json`):**
      * **Success (HTTP 200 OK for `GET /listings/{id}`):**
        ```json
        {
            "listing_id": "uuid-of-listing",
            "host_id": "uuid-of-host",
            "title": "Cozy Apartment...",
            "description": "...",
            "address": "...",
            "city": "...",
            "state": "...",
            "zip_code": "...",
            "country": "...",
            "latitude": 9.0765,
            "longitude": 7.3986,
            "price_per_night": 75.50,
            "num_bedrooms": 1,
            "num_bathrooms": 1,
            "max_guests": 2,
            "amenities": ["Wifi", "Kitchen", "Air Conditioning"],
            "image_urls": ["url_1", "url_2"],
            "average_rating": 4.5, // From Review Service
            "total_reviews": 10,    // From Review Service
            "status": "available",  // e.g., published, unpublished
            "created_at": "2024-01-01T10:00:00Z",
            "updated_at": "2024-01-01T10:00:00Z"
        }
        ```
      * **Success (HTTP 200 OK for `GET /listings`):** An array of listing objects, potentially paginated.
        ```json
        {
            "listings": [
                { "listing_id": "uuid-1", "title": "...", "price_per_night": 100, ... },
                { "listing_id": "uuid-2", "title": "...", "price_per_night": 120, ... }
            ],
            "total_count": 150,
            "page": 1,
            "limit": 10
        }
        ```
      * **Error (HTTP 404 Not Found):** For `GET /listings/{id}` if ID does not exist.
      * **Error (HTTP 400 Bad Request):** For invalid query parameters.
  * **Performance Criteria:**
      * **Latency:** Average \< 100ms for single get; \< 150ms for list/search (for basic filtering, AI search will have its own criteria).
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
        "listing_id": "uuid-of-listing",
        "check_in_date": "2025-07-10",
        "check_out_date": "2025-07-15",
        "num_guests": 2
    }
    ```
  * **Validation Rules:**
      * `listing_id`: Required, must be a valid, existing, and available listing.
      * `check_in_date`, `check_out_date`: Required, valid date format (YYYY-MM-DD), `check_in_date` must be before `check_out_date`, both dates must be in the future.
      * `num_guests`: Required, positive integer, must be less than or equal to the listing's `max_guests`.
      * **Availability Check (Critical):** The system must verify that the property is available for the entire requested date range, considering existing bookings. This involves querying `BookingDB` for conflicts.
      * **Price Calculation:** The total price for the stay (price per night \* number of nights + fees) is calculated internally by the service.
  * **Output (Response Body - `application/json`):**
      * **Success (HTTP 201 Created):**
        ```json
        {
            "booking_id": "uuid-of-new-booking",
            "guest_id": "uuid-of-guest",
            "listing_id": "uuid-of-listing",
            "check_in_date": "2025-07-10",
            "check_out_date": "2025-07-15",
            "num_guests": 2,
            "total_price": 377.50, // Calculated by the system
            "status": "pending_payment", // Initial status
            "message": "Booking initiated. Proceed to payment."
        }
        ```
      * **Error (HTTP 400 Bad Request):** For validation or availability failures.
      * **Error (HTTP 403 Forbidden):** If user is not a guest.
      * **Error (HTTP 404 Not Found):** If `listing_id` does not exist.
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
      * `user_id`: String (optional, filter by guest).
      * `host_id`: String (optional, filter by host).
      * `status`: String (e.g., `pending`, `confirmed`, `cancelled`, `completed`).
      * `page`, `limit`: Pagination parameters.
  * **Output (Response Body - `application/json`):**
      * **Success (HTTP 200 OK for `GET /bookings/{id}`):**
        ```json
        {
            "booking_id": "uuid-of-booking",
            "guest_id": "uuid-of-guest",
            "host_id": "uuid-of-host",
            "listing_id": "uuid-of-listing",
            "check_in_date": "2025-07-10",
            "check_out_date": "2025-07-15",
            "num_guests": 2,
            "total_price": 377.50,
            "status": "confirmed", // e.g., pending_payment, confirmed, cancelled, completed
            "payment_status": "paid", // From Payment Service
            "created_at": "2025-06-29T10:00:00Z",
            "updated_at": "2025-06-29T10:05:00Z"
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
      * **Status Check:** Only bookings with certain statuses (e.g., `pending_payment`, `confirmed`) can be cancelled.
      * **Cancellation Policy Enforcement:** System applies relevant cancellation policy rules (e.g., full refund within 24h, 50% refund after 24h). This might trigger a refund process with the Payment Service.
  * **Output (Response Body - `application/json`):**
      * **Success (HTTP 200 OK):**
        ```json
        {
            "booking_id": "uuid-of-cancelled-booking",
            "status": "cancelled",
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

<!-- REQUIREMENTS_END -->
