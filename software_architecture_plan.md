# BeautyCort Software Architecture Plan

**Document Version:** 1.0  
**Created:** July 1, 2025  
**References:** functional_requirements.md, business_logic.md  
**Target Architecture:** Flutter Mobile + AWS Cloud Backend

---

## 🏗️ Architecture Overview

BeautyCort is architected as a **mobile-first, cloud-native marketplace** with real-time booking capabilities. The system follows a **microservices architecture** with event-driven communication and supports high availability for instant booking confirmations (<10 seconds).

### Core Design Principles
- **Mobile-First**: Flutter cross-platform development (iOS/Android)
- **Real-Time**: Sub-10 second booking confirmations
- **Scalable**: AWS cloud infrastructure with auto-scaling
- **Secure**: End-to-end encryption with multi-factor authentication
- **Resilient**: Graceful degradation and offline capabilities
- **Localized**: Arabic/English bilingual support

---

## 🎯 System Architecture Components

### 1. Frontend Architecture (Mobile Applications)

#### 1.1 Flutter Mobile Apps
```
BeautyCort Mobile Apps
├── Customer App (iOS/Android)
│   ├── Authentication Module
│   ├── Search & Discovery Engine
│   ├── Booking Engine
│   ├── Chat/Communication System
│   ├── Payment Integration (Tap Payments)
│   └── Profile Management
│
└── Provider App (iOS/Android)
    ├── Provider Dashboard
    ├── Calendar Management
    ├── Service Catalog
    ├── Customer Management
    ├── Analytics & Reporting
    └── Business Profile Management
```

**Technology Stack:**
- **Framework**: Flutter 3.x with Dart
- **State Management**: Provider/Riverpod pattern
- **Local Storage**: SQLite (via sqflite)
- **HTTP Client**: Dio with interceptors
- **Real-time**: WebSocket integration
- **Push Notifications**: Firebase Cloud Messaging
- **Maps**: Google Maps/Apple Maps integration
- **Camera/Media**: Image picker and compression

#### 1.2 Key Mobile Features
- **Offline Capability**: View bookings/calendar when offline
- **Real-time Updates**: Live booking status and availability
- **Biometric Authentication**: Touch/Face ID support
- **Deep Linking**: Direct booking/profile access
- **Arabic/English**: RTL/LTR layout switching

### 2. Backend Architecture (AWS Cloud)

#### 2.1 Microservices Architecture
```
AWS Cloud Backend
├── API Gateway (Load Balancer + Rate Limiting)
├── Authentication Service (AWS Cognito + Custom)
├── User Management Service
├── Provider Service (with MoH verification)
├── Booking Engine Service
├── Payment Service (Tap Integration)
├── Communication Service (Chat/Notifications)
├── Search Service (ElasticSearch)
├── Analytics Service
├── File Storage Service (S3)
└── Background Jobs Service (SQS/Lambda)
```

**Core Services Detail:**

##### 2.1.1 Authentication Service
- **Technology**: AWS Cognito + Custom JWT
- **Features**: 
  - Email/OTP verification
  - Google OAuth integration
  - Session management (30-day expiry)
  - Password reset flows
- **Security**: Multi-factor authentication, rate limiting

##### 2.1.2 User Management Service
- **Technology**: Node.js/Express with PostgreSQL
- **Responsibilities**:
  - Customer profile management
  - Provider profile management
  - Role-based access control
  - Age verification (18+ requirement)

##### 2.1.3 Provider Service
- **Technology**: Node.js with OCR integration
- **Key Features**:
  - MoH license verification (aesthetic clinics only)
  - Provider tier calculation (Silver/Gold/Diamond)
  - Service catalog management
  - Business profile management
- **Integrations**: OCR API for license verification

##### 2.1.4 Booking Engine Service
- **Technology**: Node.js with Redis for caching
- **Core Functions**:
  - Real-time slot availability
  - Atomic booking transactions
  - Conflict prevention (optimistic locking)
  - Hot Slots management (<4 hours)
  - Cancellation logic with penalties
- **Performance**: <10 second booking confirmation target

##### 2.1.5 Payment Service
- **Technology**: Node.js with Tap Payments SDK
- **Features**:
  - Flat fee calculation (2 JOD <25 JOD, 5 JOD ≥25 JOD)
  - Per-service fee breakdown
  - T+2 settlement processing
  - Refund handling
  - Provider tier-based retention

##### 2.1.6 Communication Service
- **Technology**: Node.js with Socket.IO
- **Capabilities**:
  - WhatsApp-style in-app chat
  - Voice note support
  - Image sharing (5MB limit)
  - Push notifications
  - SMS/Email notifications
  - Real-time message delivery

##### 2.1.7 Search Service
- **Technology**: ElasticSearch with geospatial indexing
- **Features**:
  - Location-based search (1km-20km radius)
  - Multi-criteria filtering
  - Infinite scroll pagination
  - Real-time availability integration
  - Ranking algorithm (distance + rating + availability)

#### 2.2 Database Architecture

##### 2.2.1 Primary Database (PostgreSQL)
```sql
-- Core Tables Structure
Users (customers, providers)
├── Authentication data
├── Profile information
├── Location data
└── Preferences

Providers
├── Business details
├── Verification status
├── MoH license data (aesthetic clinics)
├── Tier information
└── Service catalog

Bookings
├── Booking states
├── Time slots
├── Service details
├── Payment information
└── Cancellation data

Services
├── Service catalog
├── Pricing information
├── Duration specifications
└── Category classifications

Reviews
├── Rating data
├── Review content
├── Photo attachments
└── Response threads
```

##### 2.2.2 Caching Layer (Redis)
- **Real-time Data**: Slot availability, booking states
- **Session Management**: User sessions and tokens
- **Rate Limiting**: API request throttling
- **Search Cache**: Frequently accessed search results

##### 2.2.3 File Storage (AWS S3)
- **Profile Photos**: Max 2MB per image
- **Service Galleries**: Before/after photos
- **MoH Licenses**: Max 10MB per document
- **Chat Media**: Images up to 5MB
- **Voice Notes**: Compressed audio files

#### 2.3 Event-Driven Architecture

##### 2.3.1 Message Queues (AWS SQS)
```
Event Flow:
Booking Created → Payment Processing → Provider Notification → Customer Confirmation
      ↓               ↓                    ↓                     ↓
   Audit Log    Settlement Queue    Push Notification     Email/SMS
```

##### 2.3.2 Background Jobs (AWS Lambda)
- **Tier Calculation**: Monthly provider tier updates
- **License Monitoring**: Expiry notifications (aesthetic clinics)
- **Reminder System**: Booking notifications (24h, 2h before)
- **Analytics Processing**: Daily/weekly/monthly reports
- **Data Cleanup**: Expired bookings, old chat messages

### 3. Integration Architecture

#### 3.1 Third-Party Integrations

##### 3.1.1 Payment Gateway (Tap Payments)
```javascript
// Payment Flow Architecture
Customer Payment → Tap Payment Gateway → Webhook → Booking Confirmation
                      ↓
                  Apple Pay/JoPACC/Cards → Settlement (T+2) → Provider Payout
```

##### 3.1.2 OCR Service (MoH License Verification)
```
License Upload → OCR Processing → Data Extraction → MoH Verification → Badge Assignment
```

##### 3.1.3 Maps & Location Services
- **Google Maps API**: Location search and directions
- **Geolocation Services**: GPS-based "near me" functionality
- **Distance Calculation**: Haversine formula for accurate distances

##### 3.1.4 Notification Services
- **Firebase Cloud Messaging**: Push notifications
- **AWS SNS**: SMS notifications
- **SendGrid/AWS SES**: Email notifications

#### 3.2 API Architecture

##### 3.2.1 RESTful APIs
```
Base URL: https://api.beautycort.com/v1/

Authentication:
POST /auth/register
POST /auth/login
POST /auth/google-oauth
POST /auth/otp/verify

Search & Discovery:
GET /search/providers
GET /providers/{id}
GET /services/categories

Booking Management:
POST /bookings
GET /bookings/{id}
PUT /bookings/{id}/cancel
PUT /bookings/{id}/reschedule

Payment:
POST /payments/process
GET /payments/{id}/status
POST /payments/refund

Communication:
GET /chats/{booking_id}
POST /chats/{booking_id}/messages
POST /notifications/send
```

##### 3.2.2 WebSocket APIs
```
Real-time Endpoints:
/ws/booking-updates
/ws/chat/{booking_id}
/ws/availability/{provider_id}
/ws/notifications/{user_id}
```

---

## 🔧 Technical Implementation Details

### 1. Real-Time Booking Engine

#### 1.1 Atomic Transaction Design
```javascript
// Booking Transaction Flow
async function processBooking(bookingRequest) {
    const transaction = await db.beginTransaction();
    try {
        // 1. Lock slot with optimistic locking
        const slot = await lockSlot(bookingRequest.slotId);
        
        // 2. Validate booking constraints
        await validateBookingRules(bookingRequest);
        
        // 3. Reserve slot (15-minute hold)
        await reserveSlot(slot, bookingRequest.customerId);
        
        // 4. Create booking record
        const booking = await createBooking(bookingRequest);
        
        // 5. Initialize payment
        const paymentToken = await initiatePayment(booking);
        
        await transaction.commit();
        return { booking, paymentToken, timeout: 900 };
    } catch (error) {
        await transaction.rollback();
        throw error;
    }
}
```

#### 1.2 Conflict Resolution Strategy
```javascript
// Concurrent booking handling
const BOOKING_CONFLICTS = {
    SLOT_TAKEN: 'redirect_to_similar_slots',
    PAYMENT_TIMEOUT: 'release_and_retry',
    PROVIDER_UNAVAILABLE: 'notify_and_suggest_alternatives'
};
```

### 2. Search Algorithm Implementation

#### 2.1 Geospatial Search with ElasticSearch
```json
{
  "query": {
    "bool": {
      "must": [
        {
          "geo_distance": {
            "distance": "10km",
            "location": { "lat": 31.9539, "lon": 35.9106 }
          }
        }
      ],
      "filter": [
        { "term": { "category": "salon" } },
        { "range": { "price": { "gte": 10, "lte": 100 } } },
        { "range": { "rating": { "gte": 4.0 } } }
      ]
    }
  },
  "sort": [
    { "_score": { "order": "desc" } },
    { "_geo_distance": { "location": "31.9539,35.9106", "order": "asc" } }
  ]
}
```

#### 2.2 Ranking Algorithm
```javascript
function calculateProviderScore(provider, userLocation) {
    const distanceScore = calculateDistanceScore(provider.location, userLocation);
    const ratingScore = provider.rating / 5.0;
    const availabilityScore = calculateAvailabilityScore(provider.availableSlots);
    const verificationScore = provider.isVerified ? 1.0 : 0.7;
    
    return (distanceScore * 0.3) + (ratingScore * 0.25) + 
           (availabilityScore * 0.25) + (verificationScore * 0.2);
}
```

### 3. Fee Calculation System

#### 3.1 Per-Service Flat Fee Logic
```javascript
function calculateServiceFees(services) {
    let totalFees = 0;
    const breakdown = services.map(service => {
        const fee = service.price < 25 ? 2.00 : 5.00;
        totalFees += fee;
        return {
            serviceName: service.name,
            servicePrice: service.price,
            fee: fee,
            display: `${service.price} JOD → ${fee} JOD fee`
        };
    });
    
    return { totalFees: Math.ceil(totalFees), breakdown };
}
```

#### 3.2 Provider Tier System
```javascript
const TIER_THRESHOLDS = {
    SILVER: { min: 0, max: 999, retention: 0.10 },
    GOLD: { min: 1000, max: 1499, retention: 0.12 },
    DIAMOND: { min: 1500, max: Infinity, retention: 0.15 }
};

function calculateProviderTier(monthlyBookings) {
    for (const [tier, config] of Object.entries(TIER_THRESHOLDS)) {
        if (monthlyBookings >= config.min && monthlyBookings <= config.max) {
            return { tier, retentionRate: config.retention };
        }
    }
}
```

### 4. Security Architecture

#### 4.1 Authentication & Authorization
```javascript
// JWT Token Structure
{
    "sub": "user_id",
    "role": "customer|provider|admin",
    "tier": "silver|gold|diamond", // providers only
    "verified": true, // providers only
    "exp": 1672531200,
    "iat": 1669939200
}

// Role-based access control
const PERMISSIONS = {
    customer: ['book_service', 'view_bookings', 'chat_provider'],
    provider: ['manage_calendar', 'view_analytics', 'chat_customer'],
    admin: ['manage_users', 'view_all_data', 'moderate_content']
};
```

#### 4.2 Data Encryption
- **In Transit**: TLS 1.3 for all API communications
- **At Rest**: AES-256 encryption for sensitive data
- **Personal Data**: Encrypted columns for PII (names, phones, emails)
- **Payment Data**: PCI DSS compliance via Tap Payments

### 5. Performance Optimization

#### 5.1 Caching Strategy
```javascript
// Multi-level caching
const CACHE_STRATEGY = {
    'provider_profiles': { ttl: 300, source: 'database' },
    'search_results': { ttl: 60, source: 'elasticsearch' },
    'availability_slots': { ttl: 10, source: 'realtime' },
    'user_sessions': { ttl: 1800, source: 'redis' }
};
```

#### 5.2 Database Optimization
```sql
-- Critical indexes for performance
CREATE INDEX idx_bookings_provider_date ON bookings(provider_id, booking_date);
CREATE INDEX idx_providers_location ON providers USING GIST(location);
CREATE INDEX idx_services_category_price ON services(category, price);
CREATE INDEX idx_availability_provider_datetime ON availability(provider_id, start_time);
```

---

## 📊 Scalability & Performance

### 1. Performance Targets
- **Booking Confirmation**: <10 seconds end-to-end
- **Search Response**: <2 seconds for results
- **Real-time Updates**: <5 seconds for availability
- **Payment Processing**: <30 seconds completion
- **Chat Messages**: <1 second delivery

### 2. Scalability Planning

#### 2.1 Auto-Scaling Configuration
```yaml
# AWS Auto Scaling Groups
booking_service:
  min_instances: 2
  max_instances: 20
  scale_out_threshold: 70% # CPU utilization
  scale_in_threshold: 30%

search_service:
  min_instances: 1
  max_instances: 10
  scale_out_threshold: 80%
  scale_in_threshold: 40%
```

#### 2.2 Database Scaling
- **Read Replicas**: 3 read replicas for query distribution
- **Connection Pooling**: 100 max connections per service
- **Query Optimization**: Regularly analyze and optimize slow queries
- **Partitioning**: Time-based partitioning for bookings table

### 3. Monitoring & Observability

#### 3.1 Application Monitoring
- **APM**: AWS X-Ray for distributed tracing
- **Metrics**: CloudWatch for system metrics
- **Logging**: Centralized logging with structured JSON
- **Alerting**: PagerDuty integration for critical alerts

#### 3.2 Business Metrics
- **Booking Success Rate**: Target >95%
- **Payment Success Rate**: Target >98%
- **Average Response Time**: Target <2 seconds
- **User Satisfaction**: Target >4.5 stars average

---

## 🔒 Security & Compliance

### 1. Data Protection
- **GDPR Compliance**: Right to deletion, data portability
- **Local Privacy Laws**: Jordan data protection compliance
- **Data Retention**: 
  - Booking history: Permanent
  - Chat messages: 1 year
  - User data: Until account deletion

### 2. Healthcare Compliance (Aesthetic Clinics)
- **MoH License Verification**: OCR + manual review
- **Medical Data Protection**: Enhanced encryption for health-related bookings
- **Audit Trails**: Complete audit logs for healthcare transactions

### 3. Financial Security
- **PCI DSS**: Compliance via Tap Payments
- **Fraud Detection**: Real-time transaction monitoring
- **Settlement Security**: Secure T+2 provider payouts

---

## 🚀 Deployment & DevOps

### 1. Infrastructure as Code
```yaml
# Terraform configuration for AWS infrastructure
provider "aws" {
  region = "eu-west-1" # Dublin (closest to Jordan)
}

# ECS Cluster for microservices
resource "aws_ecs_cluster" "beautycort" {
  name = "beautycort-cluster"
}

# RDS PostgreSQL instance
resource "aws_rds_instance" "main" {
  identifier = "beautycort-main"
  engine = "postgres"
  engine_version = "15.3"
  instance_class = "db.t3.medium"
  allocated_storage = 100
  multi_az = true
}
```

### 2. CI/CD Pipeline
```yaml
# GitHub Actions workflow
name: Deploy BeautyCort
on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Tests
        run: npm test
      
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to AWS
        run: |
          docker build -t beautycort:${{ github.sha }} .
          aws ecs update-service --cluster beautycort-cluster --service api
```

### 3. Mobile App Deployment
- **iOS**: App Store Connect with TestFlight beta
- **Android**: Google Play Console with internal testing
- **Code Push**: React Native CodePush for hot updates
- **Feature Flags**: LaunchDarkly for gradual rollouts

---

## 🗓️ Development Phases

### Phase 1: Foundation (Months 1-2)
- ✅ Authentication system
- ✅ Basic user management
- ✅ Provider registration
- ✅ Simple booking flow

### Phase 2: Core Features (Months 3-4)
- 🔄 Search & discovery engine
- 🔄 Real-time booking system
- 🔄 Payment integration
- 🔄 Basic chat system

### Phase 3: Advanced Features (Months 5-6)
- ⏳ Provider dashboard
- ⏳ Analytics system
- ⏳ Review system
- ⏳ Hot Slots feature

### Phase 4: Optimization (Months 7-8)
- ⏳ Performance optimization
- ⏳ Advanced security features
- ⏳ Comprehensive testing
- ⏳ Launch preparation

### Phase 5: Launch & Scale (Months 9-10)
- ⏳ Soft launch in Amman
- ⏳ User feedback integration
- ⏳ Performance monitoring
- ⏳ Scale preparation

---

## 📝 Architecture Decision Records

### ADR-001: Mobile-First Architecture
**Decision**: Flutter for cross-platform mobile development
**Rationale**: Single codebase, native performance, rapid development
**Status**: Accepted

### ADR-002: Microservices Architecture
**Decision**: Containerized microservices on AWS ECS
**Rationale**: Scalability, maintainability, team independence
**Status**: Accepted

### ADR-003: PostgreSQL as Primary Database
**Decision**: PostgreSQL with Redis caching
**Rationale**: ACID compliance, geospatial support, JSON capabilities
**Status**: Accepted

### ADR-004: Flat Fee Structure
**Decision**: Per-service flat fees (2 JOD <25 JOD, 5 JOD ≥25 JOD)
**Rationale**: Transparency, predictability, simplified calculations
**Status**: Accepted

### ADR-005: Real-Time Architecture
**Decision**: WebSocket + Redis for real-time features
**Rationale**: Low latency, scalable, reliable message delivery
**Status**: Accepted

---

## 🔧 Technical Stack Summary

### Frontend (Mobile)
- **Framework**: Flutter 3.x
- **Language**: Dart
- **State Management**: Provider/Riverpod
- **Local Storage**: SQLite
- **Maps**: Google Maps API
- **Push Notifications**: Firebase Cloud Messaging

### Backend (Cloud)
- **Runtime**: Node.js 18+
- **Framework**: Express.js
- **Database**: PostgreSQL 15
- **Cache**: Redis 7
- **Queue**: AWS SQS
- **Search**: ElasticSearch 8
- **File Storage**: AWS S3

### Infrastructure
- **Cloud Provider**: AWS
- **Container Orchestration**: ECS
- **Load Balancer**: Application Load Balancer
- **CDN**: CloudFront
- **Monitoring**: CloudWatch + X-Ray
- **Security**: WAF + Shield

### Third-Party Services
- **Payment**: Tap Payments
- **Authentication**: AWS Cognito
- **OCR**: AWS Textract or custom OCR service
- **SMS**: AWS SNS
- **Email**: SendGrid or AWS SES
- **Push**: Firebase Cloud Messaging

---

This architecture plan provides a comprehensive roadmap for building BeautyCort as a scalable, secure, and high-performance beauty booking platform. The mobile-first approach with cloud-native backend ensures rapid time-to-market while maintaining the flexibility to scale as the platform grows.