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
  
5. Locations

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