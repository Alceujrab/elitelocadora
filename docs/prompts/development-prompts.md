# Development Prompts for Elite Locadora

## 🚀 Backend Development Prompts

### Laravel API Development

#### Vehicle Management API
```
You are developing the vehicle management API for Elite Locadora, a premium vehicle rental system using Laravel.

Requirements:
- Create RESTful endpoints for vehicle CRUD operations
- Implement vehicle search with filters (location, dates, category, price range)
- Handle real-time availability checking
- Include dynamic pricing based on demand and season
- Add vehicle image upload and management
- Implement proper validation and error handling

Specific Endpoints Needed:
1. GET /api/vehicles - List vehicles with pagination and filters
2. GET /api/vehicles/{id} - Get vehicle details
3. POST /api/vehicles - Create new vehicle (admin only)
4. PUT /api/vehicles/{id} - Update vehicle (admin only)
5. DELETE /api/vehicles/{id} - Soft delete vehicle (admin only)
6. GET /api/vehicles/availability - Check availability for date range
7. POST /api/vehicles/{id}/images - Upload vehicle images

Include:
- Eloquent models with relationships
- Form request validation classes
- Resource transformers for API responses
- Authorization policies
- Comprehensive test cases
- OpenAPI documentation

Follow Laravel best practices and implement proper error handling.
```

#### Booking System API
```
Create a comprehensive booking system API for Elite Locadora using Laravel.

Business Requirements:
- Multi-step booking process with validation at each step
- Real-time price calculation including taxes and fees
- Booking modification and cancellation handling
- Payment integration with Stripe
- Email notifications for booking lifecycle
- Booking conflict prevention
- Corporate booking support with different pricing

API Endpoints:
1. POST /api/bookings/quote - Get booking quote with pricing
2. POST /api/bookings - Create new booking
3. GET /api/bookings - List user bookings with filters
4. GET /api/bookings/{id} - Get booking details
5. PUT /api/bookings/{id} - Modify booking (if allowed)
6. DELETE /api/bookings/{id} - Cancel booking
7. POST /api/bookings/{id}/payment - Process payment
8. GET /api/bookings/{id}/invoice - Generate invoice

Implementation Requirements:
- Use database transactions for booking creation
- Implement booking status state machine
- Create queue jobs for email notifications
- Add proper logging for audit trail
- Include rate limiting for API endpoints
- Implement idempotency for payment operations

Include comprehensive validation, error handling, and test coverage.
```

### Frontend Development Prompts

#### React Component Development
```
Create a comprehensive vehicle search and booking interface for Elite Locadora using React and TypeScript.

Component Requirements:

1. VehicleSearchForm Component:
- Location selection with autocomplete
- Date/time pickers with availability validation
- Vehicle category filters
- Price range slider
- Real-time search results update
- Mobile-responsive design

2. VehicleCard Component:
- Vehicle image carousel
- Key specifications display
- Pricing information with taxes
- Availability indicator
- "Book Now" call-to-action
- Favorite/compare functionality

3. BookingFlow Component:
- Multi-step form with progress indicator
- Vehicle selection confirmation
- Customer information collection
- Add-ons and extras selection
- Payment information form
- Booking confirmation display

Technical Requirements:
- Use TypeScript for type safety
- Implement proper state management (React Query + Zustand)
- Create reusable UI components with Tailwind CSS
- Add form validation with react-hook-form
- Implement error boundaries and loading states
- Include accessibility features (ARIA labels, keyboard navigation)
- Add responsive design for mobile and desktop
- Implement progressive enhancement

Include unit tests with React Testing Library and Storybook stories for components.
```

#### Admin Dashboard Development
```
Develop a comprehensive admin dashboard for Elite Locadora fleet management using React.

Dashboard Features:

1. Fleet Overview:
- Real-time vehicle status display
- Availability calendar view
- Maintenance schedule tracking
- Revenue analytics charts
- Quick action buttons

2. Booking Management:
- Booking list with advanced filters
- Booking details modal
- Status update functionality
- Payment tracking
- Customer communication tools

3. Vehicle Management:
- Vehicle inventory grid
- Add/edit vehicle forms
- Image upload with preview
- Maintenance log tracking
- Vehicle performance metrics

4. Customer Management:
- Customer list with search
- Customer profile views
- Booking history
- Communication history
- Loyalty program management

5. Reporting & Analytics:
- Revenue dashboards
- Fleet utilization reports
- Customer behavior analytics
- Performance KPI tracking
- Export functionality

Technical Implementation:
- Use React with TypeScript
- Implement role-based access control
- Create responsive dashboard layout
- Add real-time updates with WebSockets
- Include data visualization with Chart.js or D3
- Implement advanced filtering and sorting
- Add export functionality (PDF, Excel)
- Include audit logging for admin actions

Ensure proper error handling, loading states, and user feedback throughout the interface.
```

## 🗄️ Database Development Prompts

### Schema Design
```
Design a comprehensive database schema for Elite Locadora vehicle rental system using MySQL.

Core Requirements:
- Support multiple vehicle types and categories
- Handle complex booking scenarios and modifications
- Track pricing tiers and seasonal rates
- Manage multiple pickup/dropoff locations
- Support corporate accounts and individual customers
- Include maintenance scheduling and tracking
- Handle payment processing and financial reporting

Tables to Design:

1. Users & Authentication:
- users (customers, staff, admins)
- roles and permissions
- user_profiles with extended information
- password_resets and email_verifications

2. Vehicle Management:
- vehicles with detailed specifications
- vehicle_categories with features and pricing
- vehicle_images and documents
- vehicle_maintenance_records
- vehicle_availability_calendar

3. Location Management:
- locations with operating hours
- location_vehicle_assignments
- pickup_dropoff_rules

4. Booking System:
- bookings with status tracking
- booking_modifications history
- booking_extras and add-ons
- booking_payments and refunds

5. Pricing & Inventory:
- pricing_tiers for different customer types
- seasonal_rates and promotions
- discount_codes and usage tracking

Technical Requirements:
- Optimize for read-heavy booking searches
- Include proper indexing strategy
- Implement soft deletes where appropriate
- Add audit logging for critical operations
- Support for multiple currencies
- Handle timezone considerations
- Include data integrity constraints

Provide Laravel migration files with proper relationships, indexes, and seed data.
```

### Data Access Layer
```
Create a comprehensive data access layer for Elite Locadora using Laravel Eloquent.

Model Requirements:

1. Vehicle Model:
- Relationships: category, location, bookings, images, maintenance
- Scopes: available, by_category, by_location, price_range
- Accessors: formatted_price, availability_status, rating_average
- Methods: checkAvailability(), calculatePrice(), getMaintenanceSchedule()

2. Booking Model:
- Relationships: user, vehicle, payments, modifications
- Scopes: active, upcoming, past, by_status
- State management: pending, confirmed, in_progress, completed, cancelled
- Methods: calculateTotal(), canModify(), canCancel(), generateInvoice()

3. User Model:
- Relationships: bookings, payments, reviews
- Scopes: customers, staff, admins
- Methods: getTotalSpent(), getBookingHistory(), getLoyaltyLevel()

Repository Pattern Implementation:
- Create repository interfaces for testability
- Implement caching layer for frequently accessed data
- Add query optimization for complex searches
- Include proper error handling and validation

Advanced Features:
- Implement full-text search for vehicles
- Add geospatial queries for location-based searches
- Create booking conflict detection
- Implement dynamic pricing algorithms
- Add audit logging for all model changes

Include comprehensive test coverage for all models and repositories.
```

## 🔌 Integration Development Prompts

### Payment Integration
```
Implement comprehensive payment processing for Elite Locadora using Stripe and Laravel.

Payment Features Required:
- Credit card processing for bookings
- Authorization and capture for future charges
- Refund processing for cancellations
- Recurring billing for corporate accounts
- Multi-currency support
- Payment method storage for repeat customers
- 3D Secure authentication support
- Webhook handling for payment events

Implementation Requirements:

1. Payment Service Class:
- createPaymentIntent() for booking payments
- authorizePayment() for hold charges
- capturePayment() when rental begins
- refundPayment() for cancellations
- savePaymentMethod() for future use
- handleWebhook() for Stripe events

2. Payment Models:
- Payment model with status tracking
- PaymentMethod model for stored cards
- Refund model for cancellation tracking
- Transaction log for audit trail

3. API Endpoints:
- POST /api/payments/intent - Create payment intent
- POST /api/payments/confirm - Confirm payment
- POST /api/payments/refund - Process refund
- POST /api/webhooks/stripe - Handle Stripe webhooks
- GET /api/payments/methods - List saved payment methods

Security Requirements:
- PCI DSS compliance considerations
- Secure webhook signature validation
- Payment data encryption
- Fraud detection integration
- Rate limiting for payment endpoints

Error Handling:
- Network failure retry logic
- Payment failure notifications
- Partial refund handling
- Currency conversion errors
- Webhook replay protection

Include comprehensive logging, monitoring, and test coverage for all payment operations.
```

### Email Notification System
```
Create a comprehensive email notification system for Elite Locadora using Laravel.

Notification Types:

1. Booking Lifecycle:
- Booking confirmation with details
- Payment confirmation receipt
- Rental reminder (24 hours before)
- Pickup instructions and location
- Return reminder and instructions
- Booking completion thank you

2. Account Management:
- Welcome email for new customers
- Password reset instructions
- Email verification
- Account activity notifications
- Loyalty program updates

3. Administrative:
- New booking alerts for staff
- Payment failure notifications
- Maintenance reminders
- Low availability alerts
- Customer feedback requests

Implementation Requirements:

1. Mailable Classes:
- BookingConfirmationMail
- PaymentReceiptMail
- RentalReminderMail
- WelcomeMail
- PasswordResetMail

2. Notification Jobs:
- Queue-based email sending
- Retry logic for failed emails
- Email tracking and analytics
- Template personalization
- Multi-language support

3. Email Templates:
- Responsive HTML templates
- Plain text alternatives
- Brand-consistent design
- Dynamic content blocks
- Personalization tokens

Technical Features:
- Use Laravel's notification system
- Implement email preferences management
- Add unsubscribe functionality
- Include email tracking (opens, clicks)
- Support for multiple email providers
- Template versioning and A/B testing

Integration Requirements:
- SendGrid or Mailgun integration
- Email verification service
- Bounce and complaint handling
- Analytics and reporting
- GDPR compliance for email data

Include comprehensive testing for all email scenarios and templates.
```

## 🧪 Testing Prompts

### Comprehensive Test Suite
```
Create a comprehensive test suite for Elite Locadora using Laravel's testing framework.

Test Categories:

1. Unit Tests:
- Model relationships and methods
- Service class functionality
- Helper functions and utilities
- Validation rules and custom validators
- Business logic calculations

2. Feature Tests:
- API endpoint functionality
- Authentication and authorization
- Booking workflow end-to-end
- Payment processing flows
- Email notification delivery

3. Integration Tests:
- Database transactions
- External API integrations
- Queue job processing
- Cache functionality
- File upload and storage

Test Scenarios to Cover:

Booking System:
- Successful booking creation
- Booking conflicts prevention
- Pricing calculations accuracy
- Modification and cancellation flows
- Corporate vs individual booking differences

Payment Processing:
- Successful payment processing
- Payment failure handling
- Refund processing
- Webhook event handling
- Currency conversion accuracy

User Management:
- Registration and verification
- Login and logout functionality
- Password reset flows
- Role-based access control
- Profile management

Vehicle Management:
- Availability calculations
- Search and filtering accuracy
- Image upload and management
- Maintenance scheduling
- Fleet reporting

Testing Best Practices:
- Use database transactions for test isolation
- Create factory classes for test data
- Mock external services and APIs
- Test both success and failure scenarios
- Include edge cases and boundary conditions
- Maintain high test coverage (>90%)
- Use descriptive test names and comments

Performance Testing:
- API response time benchmarks
- Database query optimization
- Concurrent user scenarios
- Load testing for booking peaks
- Cache effectiveness testing

Include setup for continuous integration and automated test execution.
```