# BeautyCort MVP - Functional Requirements

**Document Version:** 1.0  
**Last Updated:** June 29, 2025  
**Target Platform:** Flutter (iOS/Android) + AWS Backend  
**Geographic Scope:** Jordan (Amman Primary)

---

## 📋 Gap Analysis - Missing Domain Facts

The following domain-specific requirements need clarification for complete MVP specification:

### Business Rules & Constraints
- [ ] **Maximum booking lead time** - How far in advance can customers book? (e.g., 30 days, 60 days)
- [ ] **Minimum booking lead time** - What's the shortest notice for booking? (e.g., 2 hours, same-day)
- [ ] **Currency rounding rules** - How should JOD amounts be rounded? (nearest 0.05, 0.1, etc.)
- [ ] **Service duration constraints** - Min/max appointment durations, overlap policies
- [ ] **Provider working hours** - Standard business hours, break times, holiday handling
- [ ] **Cancellation windows** - Free cancellation periods, late cancellation fees
- [ ] **Multi-service booking limits** - Max services per appointment, cross-provider bookings
- [ ] **Age restrictions** - Minimum age for booking certain services (aesthetic treatments)

### Technical Specifications  
- [ ] **Offline functionality scope** - What features work without internet connection?
- [ ] **Data retention policies** - How long to store booking history, chat messages, user data
- [ ] **File upload limits** - Max size for profile photos, license documents, chat images
- [ ] **Search result pagination** - Results per page, infinite scroll vs pagination
- [ ] **Real-time sync frequency** - How often to refresh availability, booking status
- [ ] **Language localization scope** - Arabic only, or Arabic + English support?

### Integration Requirements
- [ ] **Tap Payments fee structure** - Exact transaction fees, settlement terms
- [ ] **MoH license verification API** - Available endpoints, response format, verification time
- [ ] **SMS/Push notification costs** - Rate limits, delivery guarantees, fallback methods
- [ ] **Map integration scope** - Google Maps, Apple Maps, or custom solution for location

---

## 🎯 Core Functional Requirements

### 1. User Authentication & Profile Management

#### 1.1 Customer Registration & Authentication
- **FR-AUTH-001**: System shall support email registration with OTP verification
- **FR-AUTH-002**: System shall integrate Google OAuth for social login
- **FR-AUTH-003**: System shall enforce password strength requirements (8+ chars, mixed case, numbers)
- **FR-AUTH-004**: System shall provide password reset via email link
- **FR-AUTH-005**: System shall support optional phone number verification
- **FR-AUTH-006**: System shall maintain user sessions for 30 days with remember-me option
- **FR-AUTH-007**: System shall require terms of service and privacy policy acceptance

#### 1.2 Provider Registration & Verification
- **FR-PROV-001**: System shall require MoH license upload during provider registration
- **FR-PROV-002**: System shall implement OCR verification for license documents
- **FR-PROV-003**: System shall assign verified practitioner badges upon successful verification
- **FR-PROV-004**: System shall support provider profile creation with business details
- **FR-PROV-005**: System shall allow multiple service categories per provider
- **FR-PROV-006**: System shall require business location verification

#### 1.3 Profile Management
- **FR-PROF-001**: Users shall update personal information (name, phone, preferences)
- **FR-PROF-002**: Providers shall manage business profiles (description, photos, services)
- **FR-PROF-003**: System shall support profile photo upload (max 5MB, JPG/PNG)
- **FR-PROF-004**: System shall maintain booking history for both user types
- **FR-PROF-005**: Users shall configure notification preferences (push, SMS, email)

### 2. Search & Discovery Engine

#### 2.1 Location-Based Search
- **FR-SEARCH-001**: System shall default to Jordan with city selection (Amman primary)
- **FR-SEARCH-002**: System shall support location-based radius search (1km, 5km, 10km, 20km)
- **FR-SEARCH-003**: System shall integrate with maps for provider location display
- **FR-SEARCH-004**: System shall support GPS-based "near me" functionality
- **FR-SEARCH-005**: System shall show distance from user's location to provider

#### 2.2 Category & Service Filtering
- **FR-FILTER-001**: System shall support three main categories (salon, spa, aesthetic-medical)
- **FR-FILTER-002**: System shall allow multiple service type selection within categories
- **FR-FILTER-003**: System shall implement price range filtering (min/max JOD)
- **FR-FILTER-004**: System shall support rating filter (4+ stars, 3+ stars, etc.)
- **FR-FILTER-005**: System shall filter by availability (today, tomorrow, this week)
- **FR-FILTER-006**: System shall support gender-specific service filtering
- **FR-FILTER-007**: System shall allow sorting by (distance, price, rating, availability)

#### 2.3 Search Results & Display
- **FR-RESULTS-001**: System shall display provider cards with key information (name, rating, distance, price range)
- **FR-RESULTS-002**: System shall show real-time availability status indicators
- **FR-RESULTS-003**: System shall display verified practitioner badges prominently
- **FR-RESULTS-004**: System shall support search result pagination or infinite scroll
- **FR-RESULTS-005**: System shall maintain search history for quick re-access

### 3. Provider Profiles & Services

#### 3.1 Provider Profile Display
- **FR-PROFILE-001**: System shall display provider business information (name, description, contact)
- **FR-PROFILE-002**: System shall show provider photo gallery (max 20 photos)
- **FR-PROFILE-003**: System shall display MoH license verification status and badge
- **FR-PROFILE-004**: System shall show provider ratings and review summary
- **FR-PROFILE-005**: System shall display operating hours and contact information
- **FR-PROFILE-006**: System shall show provider location on map with directions link

#### 3.2 Service Catalog Management
- **FR-SERVICE-001**: Providers shall create service listings with descriptions and prices
- **FR-SERVICE-002**: System shall support service categories and subcategories
- **FR-SERVICE-003**: System shall allow service duration specification (15min to 4hr)
- **FR-SERVICE-004**: System shall support service photo upload (before/after galleries)
- **FR-SERVICE-005**: System shall enable service-specific pricing (fixed, range, on-request)
- **FR-SERVICE-006**: System shall support service add-ons and package deals

#### 3.3 Availability & Calendar Management
- **FR-CALENDAR-001**: Providers shall set weekly recurring availability schedules
- **FR-CALENDAR-002**: System shall support break times and blocked periods
- **FR-CALENDAR-003**: System shall handle holiday and exception date scheduling
- **FR-CALENDAR-004**: System shall show real-time slot availability to customers
- **FR-CALENDAR-005**: System shall prevent double-booking of time slots
- **FR-CALENDAR-006**: System shall support buffer times between appointments

### 4. Booking System

#### 4.1 Real-Time Booking Engine
- **FR-BOOK-001**: System shall provide real-time slot availability checking
- **FR-BOOK-002**: System shall complete booking confirmation in <10 seconds
- **FR-BOOK-003**: System shall prevent booking conflicts through atomic transactions
- **FR-BOOK-004**: System shall support same-day and advance booking
- **FR-BOOK-005**: System shall handle concurrent booking attempts gracefully
- **FR-BOOK-006**: System shall generate unique booking reference numbers

#### 4.2 Booking Flow & User Experience
- **FR-BOOKING-001**: Customers shall select provider → service → time slot → confirm
- **FR-BOOKING-002**: System shall display total price before payment confirmation
- **FR-BOOKING-003**: System shall support multiple service booking in single session
- **FR-BOOKING-004**: System shall show booking summary with all details
- **FR-BOOKING-005**: System shall send immediate booking confirmation
- **FR-BOOKING-006**: System shall support booking notes/special requests

#### 4.3 Hot Slots Flash Deals
- **FR-HOTSLOT-001**: System shall identify last-minute slots (<4 hours from appointment)
- **FR-HOTSLOT-002**: Providers shall offer discounted rates for Hot Slots
- **FR-HOTSLOT-003**: System shall highlight Hot Slots with special badges/colors
- **FR-HOTSLOT-004**: System shall send push notifications for relevant Hot Slots
- **FR-HOTSLOT-005**: System shall handle Hot Slot booking with priority processing

### 5. Payment Processing

#### 5.1 Tap Payments Integration
- **FR-PAY-001**: System shall integrate Tap Payments for secure transactions
- **FR-PAY-002**: System shall support Apple Pay for iOS users
- **FR-PAY-003**: System shall support JoPACC wallet payments
- **FR-PAY-004**: System shall process card payments (Visa, MasterCard, local cards)
- **FR-PAY-005**: System shall handle payment confirmation within 30 seconds
- **FR-PAY-006**: System shall support payment retry for failed transactions

#### 5.2 Fee Structure & Commission
- **FR-FEE-001**: System shall charge 2 JOD fee for bookings <25 JOD
- **FR-FEE-002**: System shall charge 5 JOD fee for bookings ≥25 JOD
- **FR-FEE-003**: System shall implement tiered provider fee retention system:
  - Silver (0-999 bookings/month): Keep 10% of BeautyCort fee
  - Gold (1000-1500 bookings/month): Keep 12% of BeautyCort fee  
  - Diamond (1500+ bookings/month): Keep 15% of BeautyCort fee
- **FR-FEE-004**: System shall calculate and display all fees transparently
- **FR-FEE-005**: System shall handle T+2 settlement to providers

#### 5.3 Financial Management
- **FR-FINANCE-001**: System shall generate booking receipts for customers
- **FR-FINANCE-002**: System shall provide provider payout summaries
- **FR-FINANCE-003**: System shall track transaction history for all parties
- **FR-FINANCE-004**: System shall handle refund processing for cancellations
- **FR-FINANCE-005**: System shall support tax calculation and reporting

### 6. Communication System

#### 6.1 In-App Messaging
- **FR-CHAT-001**: System shall provide WhatsApp-style chat interface
- **FR-CHAT-002**: System shall support text messaging between customers and providers
- **FR-CHAT-003**: System shall enable image sharing in chat (max 10MB per image)
- **FR-CHAT-004**: System shall support voice note recording and playback
- **FR-CHAT-005**: System shall show message delivery and read status
- **FR-CHAT-006**: System shall maintain chat history for booking duration + 30 days

#### 6.2 Automated Notifications
- **FR-NOTIFY-001**: System shall send booking confirmation notifications immediately
- **FR-NOTIFY-002**: System shall send reminder notifications 24 hours before appointment
- **FR-NOTIFY-003**: System shall send reminder notifications 2 hours before appointment
- **FR-NOTIFY-004**: System shall notify providers of new bookings instantly
- **FR-NOTIFY-005**: System shall send cancellation notifications to both parties
- **FR-NOTIFY-006**: System shall support push notifications, SMS, and email channels

#### 6.3 Communication Preferences
- **FR-COMM-001**: Users shall configure notification preferences by type
- **FR-COMM-002**: System shall respect do-not-disturb hours (10 PM - 8 AM)
- **FR-COMM-003**: System shall support Arabic and English notification content
- **FR-COMM-004**: System shall provide opt-out mechanisms for marketing messages

### 7. Booking Management

#### 7.1 Customer Booking Management
- **FR-MANAGE-001**: Customers shall view all upcoming and past bookings
- **FR-MANAGE-002**: Customers shall cancel bookings with appropriate notice
- **FR-MANAGE-003**: Customers shall reschedule bookings subject to availability
- **FR-MANAGE-004**: System shall show cancellation policies before booking
- **FR-MANAGE-005**: System shall support booking modification requests
- **FR-MANAGE-006**: Customers shall rate and review completed services

#### 7.2 Provider Booking Management  
- **FR-PROVIDER-001**: Providers shall view daily, weekly, and monthly booking calendars
- **FR-PROVIDER-002**: Providers shall accept or decline booking modification requests
- **FR-PROVIDER-003**: Providers shall mark appointments as completed or no-show
- **FR-PROVIDER-004**: System shall track provider response times to customer messages
- **FR-PROVIDER-005**: Providers shall set automatic responses for common inquiries

#### 7.3 No-Show Prevention & Management
- **FR-NOSHOW-001**: System shall implement automated reminder system
- **FR-NOSHOW-002**: System shall track customer no-show history
- **FR-NOSHOW-003**: System shall apply no-show penalties for repeat offenders
- **FR-NOSHOW-004**: Providers shall report no-shows within 1 hour of appointment time
- **FR-NOSHOW-005**: System shall offer rebooking assistance for genuine no-shows

### 8. Provider Dashboard

#### 8.1 Calendar & Slot Management
- **FR-DASH-001**: Providers shall manage availability calendar with drag-drop interface
- **FR-DASH-002**: System shall show booking density and utilization metrics
- **FR-DASH-003**: Providers shall set recurring availability patterns
- **FR-DASH-004**: System shall highlight conflicts and scheduling issues
- **FR-DASH-005**: Providers shall bulk update availability for multiple days

#### 8.2 Business Analytics
- **FR-ANALYTICS-001**: System shall show daily, weekly, monthly revenue reports
- **FR-ANALYTICS-002**: System shall display booking conversion rates and sources
- **FR-ANALYTICS-003**: System shall track customer retention and repeat booking rates
- **FR-ANALYTICS-004**: System shall show service popularity and pricing optimization
- **FR-ANALYTICS-005**: System shall display tier progression tracking (Silver/Gold/Diamond)

#### 8.3 Customer Management
- **FR-CUSTOMER-001**: Providers shall view customer profiles and booking history
- **FR-CUSTOMER-002**: System shall show customer preferences and notes
- **FR-CUSTOMER-003**: Providers shall add private notes about customers
- **FR-CUSTOMER-004**: System shall identify VIP customers and repeat clients
- **FR-CUSTOMER-005**: Providers shall export customer lists for marketing

### 9. Reviews & Rating System

#### 9.1 Customer Reviews
- **FR-REVIEW-001**: Customers shall rate services on 5-star scale
- **FR-REVIEW-002**: Customers shall leave written reviews up to 500 characters
- **FR-REVIEW-003**: System shall require completed booking for review eligibility
- **FR-REVIEW-004**: System shall support photo uploads with reviews
- **FR-REVIEW-005**: System shall prevent multiple reviews per booking

#### 9.2 Provider Responses & Management
- **FR-RESPOND-001**: Providers shall respond to customer reviews
- **FR-RESPOND-002**: System shall flag inappropriate reviews for moderation
- **FR-RESPOND-003**: System shall calculate and display average ratings
- **FR-RESPOND-004**: System shall show review trends and sentiment analysis
- **FR-RESPOND-005**: Providers shall request reviews from satisfied customers

### 10. Premium Features & Visibility

#### 10.1 Spotlight Carousel
- **FR-SPOTLIGHT-001**: System shall display premium provider carousel on search results
- **FR-SPOTLIGHT-002**: Providers shall purchase spotlight visibility packages
- **FR-SPOTLIGHT-003**: System shall rotate spotlight positions fairly among paying providers
- **FR-SPOTLIGHT-004**: System shall track spotlight performance metrics (views, clicks, bookings)
- **FR-SPOTLIGHT-005**: System shall support geographic targeting for spotlight ads

#### 10.2 Push-Blast Credits
- **FR-PUSHBLAST-001**: Providers shall purchase push notification credits
- **FR-PUSHBLAST-002**: System shall allow targeted push notifications to customer segments
- **FR-PUSHBLAST-003**: System shall respect customer notification preferences for marketing
- **FR-PUSHBLAST-004**: System shall track push notification performance and ROI
- **FR-PUSHBLAST-005**: System shall prevent spam through rate limiting and approval

---

## 🔒 Security & Compliance Requirements

### 11.1 Data Protection (PDPL 2023 Compliance)
- **FR-SECURITY-001**: System shall encrypt all personal data at rest and in transit
- **FR-SECURITY-002**: System shall implement data minimization principles
- **FR-SECURITY-003**: System shall provide user data export functionality
- **FR-SECURITY-004**: System shall support user data deletion requests
- **FR-SECURITY-005**: System shall maintain audit logs for data access and changes
- **FR-SECURITY-006**: System shall implement role-based access controls

### 11.2 Payment Security
- **FR-PAYSEC-001**: System shall maintain PCI DSS compliance through Tap Payments
- **FR-PAYSEC-002**: System shall never store complete card details locally
- **FR-PAYSEC-003**: System shall implement fraud detection and prevention
- **FR-PAYSEC-004**: System shall provide secure tokenization for repeat payments
- **FR-PAYSEC-005**: System shall log all payment transactions for audit

### 11.3 Healthcare Compliance
- **FR-HEALTH-001**: System shall verify MoH licenses through official channels
- **FR-HEALTH-002**: System shall maintain provider credential databases
- **FR-HEALTH-003**: System shall flag expired or invalid licenses automatically
- **FR-HEALTH-004**: System shall support health data privacy for aesthetic treatments
- **FR-HEALTH-005**: System shall implement age verification for restricted services

---

## 📱 Mobile App Requirements

### 12.1 Cross-Platform Compatibility
- **FR-MOBILE-001**: App shall run natively on iOS 13+ and Android 8+
- **FR-MOBILE-002**: App shall maintain consistent UI/UX across platforms
- **FR-MOBILE-003**: App shall support device-specific features (Face ID, fingerprint)
- **FR-MOBILE-004**: App shall handle different screen sizes and orientations
- **FR-MOBILE-005**: App shall integrate with platform notification systems

### 12.2 Performance & Reliability
- **FR-PERF-001**: App shall start up in <3 seconds on standard devices
- **FR-PERF-002**: App shall maintain <5% crash rate across all users
- **FR-PERF-003**: App shall work with intermittent network connectivity
- **FR-PERF-004**: App shall cache essential data for offline viewing
- **FR-PERF-005**: App shall handle background app switching gracefully

### 12.3 Accessibility & Localization
- **FR-ACCESS-001**: App shall support Arabic RTL interface layout
- **FR-ACCESS-002**: App shall meet accessibility standards (screen readers, large text)
- **FR-ACCESS-003**: App shall support Arabic and English languages
- **FR-ACCESS-004**: App shall handle cultural considerations in design and content
- **FR-ACCESS-005**: App shall support local date/time formats and calendar

---

## 🎯 Success Metrics & KPIs

### 13.1 Technical Performance Metrics
- Booking confirmation time: <10 seconds (Target: <5 seconds)
- App crash rate: <5% (Target: <2%)
- Payment success rate: >95% (Target: >98%)
- Search response time: <2 seconds (Target: <1 second)
- Uptime availability: >99.5% (Target: >99.9%)

### 13.2 Business Performance Metrics
- Booking-to-install conversion: ≥15% within 30 days
- Attendance rate: ≥85% (Target: ≥90%)
- Customer retention: ≥60% month-over-month
- Provider satisfaction (NPS): >50 (Target: >70)
- Monthly completed bookings: 5,000+ by month 12

### 13.3 User Experience Metrics
- App store rating: >4.0 stars (Target: >4.5 stars)
- Average session duration: >5 minutes
- Feature adoption rate: >60% for core features
- Customer support tickets: <5% of total bookings
- Time to complete booking: <2 minutes

---

## 📋 Implementation Priorities

### Phase 1: Foundation (MVP Core) - July-August 2025
1. User authentication and basic profiles
2. Provider registration with MoH verification
3. Basic search and filtering
4. Simple booking flow with Tap Payments
5. Essential notifications and confirmations

### Phase 2: Communication & Management - August-September 2025
1. In-app chat system
2. Booking management (cancel/reschedule)
3. Provider dashboard basics
4. Review and rating system
5. Automated reminder system

### Phase 3: Advanced Features - September-October 2025
1. Hot Slots flash deals
2. Premium visibility features
3. Advanced analytics dashboard
4. Security and compliance hardening
5. Performance optimization

### Phase 4: Launch Preparation - October 2025
1. App store optimization
2. Beta testing with 25 providers
3. Customer support system
4. Final compliance verification
5. Launch marketing preparation

---

**Document Status:** Draft v1.0 - Requires domain expert review for gap resolution  
**Next Steps:** Address identified gaps and obtain stakeholder approval for implementation