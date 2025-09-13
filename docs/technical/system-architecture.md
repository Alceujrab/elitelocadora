# System Architecture & Development Prompts

## 🏗️ High-Level Architecture

### Architecture Pattern: Microservices with API Gateway

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Web Frontend  │    │  Mobile App      │    │  Admin Panel    │
│   (React)       │    │  (PWA)          │    │  (React)        │
└─────────┬───────┘    └────────┬─────────┘    └─────────┬───────┘
          │                     │                        │
          └─────────────────────┼────────────────────────┘
                                │
                    ┌───────────┴──────────┐
                    │   API Gateway        │
                    │   (Kong/AWS API GW)  │
                    └───────────┬──────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
┌───────┴────────┐    ┌────────┴────────┐    ┌────────┴────────┐
│  User Service  │    │ Vehicle Service │    │ Booking Service │
│  (Auth/Users)  │    │ (Fleet Mgmt)   │    │ (Reservations) │
└────────────────┘    └─────────────────┘    └─────────────────┘
        │                       │                       │
        └───────────────────────┼───────────────────────┘
                                │
                    ┌───────────┴──────────┐
                    │   Shared Database    │
                    │   (MySQL/PostgreSQL) │
                    └──────────────────────┘
```

## 🛠️ Technology Stack Prompts

### Backend Development Prompt
```
Create a Laravel-based vehicle rental system with the following requirements:

Core Models:
- User (customers, admins, staff)
- Vehicle (cars, availability, pricing)
- Booking (reservations, status, payments)
- Location (pickup/dropoff points)
- Category (vehicle types and classifications)

Key Features to Implement:
1. Authentication system with role-based permissions
2. Vehicle availability calculation with real-time updates
3. Dynamic pricing based on demand, season, and duration
4. Booking workflow with status management
5. Payment integration (Stripe/PayPal)
6. Email notifications for booking lifecycle
7. Admin dashboard with reporting capabilities
8. RESTful API with proper documentation

Technical Requirements:
- Follow Laravel best practices and conventions
- Implement proper validation and error handling
- Use Laravel's built-in features (Eloquent, Queue, Cache)
- Create comprehensive test suite
- Include database seeding for development
- Implement proper logging and monitoring
```

### Frontend Development Prompt
```
Create a React-based vehicle rental booking interface with:

Core Components:
- VehicleSearch: Filter and search vehicles
- VehicleCard: Display vehicle information and pricing
- BookingForm: Multi-step booking process
- UserDashboard: Customer booking management
- AdminPanel: Fleet and booking management

Features to Implement:
1. Responsive design for mobile and desktop
2. Real-time availability checking
3. Interactive date/time pickers
4. Vehicle image galleries with zoom
5. Step-by-step booking flow with validation
6. Payment processing integration
7. User authentication and profile management
8. Admin tools for fleet and booking management

Technical Requirements:
- Use TypeScript for type safety
- Implement proper state management
- Create reusable component library
- Include comprehensive error handling
- Optimize for performance and SEO
- Follow accessibility best practices
- Implement progressive web app features
```

## 🗄️ Database Architecture

### Core Database Schema Prompt
```
Design a comprehensive database schema for a vehicle rental system:

Essential Tables:
1. users (id, name, email, phone, role, created_at, updated_at)
2. vehicles (id, make, model, year, category_id, location_id, daily_rate, status)
3. bookings (id, user_id, vehicle_id, start_date, end_date, total_amount, status)
4. categories (id, name, description, features)
5. locations (id, name, address, coordinates, operating_hours)
6. payments (id, booking_id, amount, payment_method, transaction_id, status)

Relationships:
- Users have many bookings
- Vehicles belong to categories and locations
- Bookings belong to users and vehicles
- Payments belong to bookings

Additional Considerations:
- Vehicle availability tracking
- Pricing tiers and seasonal rates
- Maintenance schedules
- Insurance information
- Customer reviews and ratings
- Discount codes and promotions

Include proper indexes, constraints, and data types for optimal performance.
```

### Migration Prompt for Laravel
```
Create Laravel migrations for the vehicle rental system with:

Key Features:
1. Proper foreign key relationships
2. Optimized indexes for search queries
3. Soft deletes where appropriate
4. JSON columns for flexible data storage
5. Timestamp tracking for all entities
6. Status enums for booking lifecycle
7. Price decimal handling with proper precision
8. Geographic data for location-based features

Migration Structure:
- Start with core entity tables (users, categories, locations)
- Add dependent tables (vehicles, bookings)
- Create junction tables for many-to-many relationships
- Add feature tables (reviews, promotions, maintenance)
- Include proper rollback functionality

Data Integrity:
- Implement cascading deletes where appropriate
- Add check constraints for business rules
- Create unique indexes for critical fields
- Handle timezone considerations for dates
```

## 🔐 Security Architecture

### Security Implementation Prompt
```
Implement comprehensive security measures for the vehicle rental system:

Authentication & Authorization:
1. JWT-based authentication with refresh tokens
2. Role-based access control (Customer, Staff, Admin, Super Admin)
3. Multi-factor authentication for admin accounts
4. Session management with secure logout
5. Password policies and encryption

Data Protection:
1. Encrypt sensitive customer data (PII, payment info)
2. Implement data anonymization for analytics
3. GDPR compliance with data export/deletion
4. Secure file upload with virus scanning
5. Database encryption at rest

API Security:
1. Rate limiting to prevent abuse
2. Input validation and sanitization
3. CORS configuration for frontend access
4. API versioning and deprecation handling
5. Comprehensive audit logging

Payment Security:
1. PCI DSS compliance implementation
2. Tokenization of payment data
3. Secure payment gateway integration
4. Fraud detection and prevention
5. Secure handling of refunds and chargebacks

Infrastructure Security:
1. TLS 1.3 implementation
2. Security headers (HSTS, CSP, etc.)
3. Regular security updates and patches
4. Monitoring and intrusion detection
5. Backup encryption and secure storage
```

## 🔄 Integration Architecture

### Third-Party Integration Prompt
```
Design and implement integrations for the vehicle rental system:

Payment Processors:
- Stripe for credit card processing
- PayPal for alternative payments
- Bank transfer options for corporate clients
- Cryptocurrency payments (future consideration)

Mapping & Location Services:
- Google Maps for location display and routing
- Geocoding for address validation
- Real-time traffic data for pickup/return estimates
- GPS tracking for fleet management

Communication Services:
- SendGrid for transactional emails
- Twilio for SMS notifications
- Push notification services for mobile apps
- WhatsApp Business API for customer support

Business Intelligence:
- Google Analytics for web traffic analysis
- Mixpanel for user behavior tracking
- Tableau or Power BI for business reporting
- Custom analytics dashboard

External Services:
- Vehicle identification number (VIN) decoding services
- Insurance API integrations
- Background check services for customer verification
- Weather API for dynamic pricing adjustments

Integration Patterns:
- Implement circuit breaker patterns for reliability
- Use message queues for asynchronous processing
- Create webhook handlers for real-time updates
- Build retry mechanisms with exponential backoff
- Monitor integration health and performance
```

## 📊 Performance Optimization

### Performance Enhancement Prompt
```
Optimize the vehicle rental system for high performance:

Database Optimization:
1. Implement proper indexing strategy for search queries
2. Use database connection pooling
3. Create read replicas for reporting queries
4. Implement query result caching
5. Optimize complex joins and aggregations

Application Performance:
1. Implement Redis caching for frequently accessed data
2. Use Laravel's query optimization features
3. Implement lazy loading for relationships
4. Create efficient search algorithms
5. Optimize image loading and compression

Frontend Performance:
1. Implement code splitting and lazy loading
2. Optimize bundle sizes and loading strategies
3. Use CDN for static assets
4. Implement progressive image loading
5. Create efficient state management

Real-time Features:
1. WebSocket connections for live availability updates
2. Server-sent events for booking notifications
3. Efficient polling strategies for status updates
4. Background job processing for heavy operations
5. Queue management for peak traffic handling

Monitoring & Analytics:
1. Application performance monitoring (APM)
2. Database query analysis and optimization
3. User experience monitoring
4. Error tracking and alerting
5. Capacity planning and scaling strategies
```