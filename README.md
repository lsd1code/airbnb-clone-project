# Airbnb Clone Project

The backend for the Airbnb Clone project is designed to provide a robust and scalable foundation for managing user interactions, property listings, bookings, and payments. This backend will support various functionalities required to mimic the core features of Airbnb, ensuring a smooth experience for users and hosts.

## Team Roles 👥

- `Backend Developer`: Responsible for implementing API endpoints, database schemas, and business logic
- `Database Administrator`: Manages database design, indexing and optimization
- `DevOps Engineer`: Handles deployment, monitoring, and scaling of backend services
- `QA Engineer`: Ensures backend functionalities are thoroughly tested and meet quality standards

## Technology Stack ⚙️

- `Django`: A high-level Python web framework used for building the RESTful API 
- `Django REST Framework`: Provides tools for creating and managing RESTful APIs
- `PostgreSQL`: A powerful, open-source object-relational database management system (ORDBMS) that is known for its reliability, robustness, and performance
- `GraphQL`: A query language for APIs and a runtime for executing those queries using your existing data. Developed by Facebook in 2012 and open-sourced in 2015, it provides a more efficient, flexible, and powerful alternative to REST APIs. (Allows for flexible and efficient data querying)
- `Celery`: An open-source, distributed task queue system written in Python. It enables asynchronous processing of tasks in the background, improving application performance by offloading time-consuming operations 
- `Redis`: An open-source, in-memory data structure store that is know fro its blazingly fast performance and versatile data structures (Caching and session management)
- `Docker`: An open-source containerization platform that allows you to package applications and their dependencies into lightweight, portable containers. These containers run consistently across any environment
- `CI\CD Pipelines`: Automated pipelines for testing and deploying code changes

## Database Design 🛠️

1. Users

Represents all platform users (guests and hosts)

##### Key fields

+ `user_id` [UUID, primary key]
+ `user_email` [String, Unique]
+ `password_hash` [Hash_String]
+ `role` [Enum: ('host', 'guest', 'admin')]
+ `verified` [Boolean]

We are using the password_hash field because they application is self-contained, however for enhanced security we might consider an additional layer of security by using a dedicated authentication service(Auth0 or Firebase) 

Relationships:
  + One user can host multiple properties
  + One user can make multiple bookings
  + One user can write multiple reviews (as `guest`)
  + One user can write multiple reviews (as `host`)

2. Properties

Represents rental listings on the platform

##### Key fields:

+ `user_id` [UUID, primary key]
+ `host_id` [UUID, foreign key -> Users]
+ `title` [String]
+ `price_per_night` [Numeric]
+ `amenities` [JSONB array]

Relationships:
  + A property belongs to one user
  + A property can have multiple bookings
  + A property can receive multiple reviews
  + A property has a `one-to-one` relationship with `location`

3. Bookings 

Represents reservation transactions

##### Key fields:

+ `booking_id` [UUID, primary key]
+ `property_id` [UUID, Foreign Key -> Properties]
+ `guest_id` [UUID, Foreign Key -> Users]
+ `check_in_date` [date]
+ `check_out_date` [date]
+ `total_price` [Numeric]

Relationships:
  + Belongs to one property
  + Belongs to one user
  + Generates one 
  + Generate two reviews (property review + guest review)

4. Reviews

Represents ratings and feedback 

##### Key fields:

+ `review_id` [UUID, primary key]
+ `booking_id` [UUID, Foreign Key -> Bookings]
+ `reviewer_id` [UUID, Foreign Key -> Users]
+ `rating` [Integer]
+ `review_type` [Enum: ('property', 'guest')]

Relationships:
  + Associated with one booking
  + Written by one user
  + When `review_type = property`: About one property
  + When `review_type = guest`: About one guest  


5. Payments

Represents financial transactions 

##### Key fields:

+ `payment_id` [UUID, primary key]
+ `booking_id` [UUID, Foreign Key -> Bookings]
+ `amount` [Numeric]
+ `status` [Enum: ('pending', 'completed', 'refunded')]
+ `payment_method` [String]

Relationships:
  + Associated with one booking
  + Processed by one payment gateway (external service)
  
6. Locations

Geospatial data for properties

##### Key fields:

+ `property_id` [UUID, Foreign Key -> Properties]
+ `coordinates` [GEOGRAPHY(point)]
+ `address` [String]
+ `city` [String]
+ `country` [String]

Relationships:
  + One-to-one relationship with Properties 

This schema supports core platform operations including property search, booking flow, payment processing, and review systems while maintaining data constancy and performance   

## Feature BreakDown 🧰

1. API Documentation

+ `OpenAPI Standard`: The backend APIs are documented using OpenAPI standard to ensure clarity and ease of integration 
+ `Django REST Framework`: Provides a comprehensive RESTful API for handling CRUD operations on user and property data 

2. User Authentication

+ Able to register new users, authenticate, and manage user profiles
+ Includes role-based access control for guests, hosts, and administrators

3. Property Management

+ Allows hosts to create, update, and manage property listings with photos, descriptions, pricing, and availability calendars. Supports amenity configuration and dynamic pricing rules
+ Enables hosts to effectively market their properties while providing guests with comprehensive information for booking decisions

4. Booking System

+ Manages the end-to-end reservation process including availability checks, date selection, pricing calculations, and instant booking confirmations
+ Make, update, and manage bookings, including check-in and check-out details

5. Payment Processing

+ Securely handles transactions with multiple payment methods, processes refunds, and manages payout schedules to hosts
+ Handle payment transactions related to bookings

6. Review System

+ Enables two-way reviews where guests rate properties and hosts rate guests, and includes moderation tools and displays aggregate scores on property listings
+ Builds trust through transparent feedback from previous stays. The dual-review system maintains accountability for both guests and hosts, improving overall platform quality

7. Database Optimizations

+ `Indexing`: Implement indexes for fast retrieval of frequently accessed data
+ `Caching`: Use caching strategies to reduce database load and improve performance


## API Security 🔐

1. Authentication and Authorization

+ JWT-based authentication with OAuth 2.0 flows. Role-based access control (RBAC) with granular permissions
+ Prevents unauthorized access to user accounts and sensitive operations, and ensure hosts can only manage their own properties and guests can only access their own booking history

2. Input validation and Sanitization

+ Strict schema validation for all API payloads using JSON Schema, and use parameterized queries to prevent SQL injection
+ Blocks injection attacks and malformed data that could compromise database integrity, it is essential for payment processing where malformed inputs could trigger financial errors

3. Rate limiting and Throttling

+ Redis-backed rate limiting (maybe 100 req/minute per IP), and utilize progressive delays for abusive clients
+ Prevents brute force attacks on authentication endpoints and protects against DDoS attempts that could take down booking availability services during peak times

4. Encryption and Data protection

+ HTTPS with HSTS enforcement. AES-256 encryption for sensitive data at rest. TLS 1.3 for all communications
+ Protects payment details and PII during transmission. Meets PCI-DSS requirements for financial transactions and GDPR requirements for user data privacy

5. Payment processing and Booking system

+ Direct financial impact. Insecure payments expose credit card details, enable fraudulent transactions, and violate PCI compliance - risking massive fines and loss of payment processing privileges
+ Manipulation of booking data could enable reservation fraud, double-booking scams, or service denials, we ensure booking integrity and accurate financial settlements

## CI/CD Pipelines 🔄

CI/CD stands for Continuous Integration and Continuous Deployment/Delivery, an automated process that enables development teams to build, test, and deploy applications rapidly and reliably. CI focuses on automatically merging code changes and running tests, while CD automates the release of validated code to production environments

#### Why it is important

1. `Accelerated development cycles`: Enables daily deployment of new features like enhanced search filters or payment options while maintaining system stability during peak booking seasons

2. `Quality Assurance`: Automated testing catches most of the bugs before deployment, crucial for preventing financial errors in bookings and payment processing

3. `Risk Mitigation`: Rollback mechanisms and canary deployments prevent system-wide outages during high-traffic events like holiday booking rushes

4. `Security Compliance`: Automated security scans maintain PCI-DSS(Payment Card Industry Data Security Standard) compliance for payment handling and GDPR compliance for user data protection

#### Core Tools

1. `Github Actions`: Orchestrates our CI pipeline with automated workflows on every code push

2. `Docker`: Containerizes our application for consistent environments from development to production

3. `Kubernetes + Argo CD`: Manages production deployments with progressive rollout strategies

#### Key Benefits

+ `Booking System Reliability`: Zero-downtime deployments ensure 24/7 availability

+ `Payment Security`: Automated PCI scans before each release

+ `Rapid Experimentation`: Test new features like dynamic pricing with controlled rollouts

+ `Incident Response`: Automatic rollback if error rates exceed thresholds

+ `Team Efficiency`: Reduces release process from days to minutes