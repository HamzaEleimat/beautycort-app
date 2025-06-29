# BeautyCort MVP - Business Logic Specification

**Document Version:** 1.0  
**Last Updated:** June 29, 2025  
**References:** functional_requirements.md (FR-IDs)  
**Scope:** Customer & Provider App Logic Flows

---

## 🔍 Business Rules & Technical Specifications

The following business rules and technical specifications are now defined:

### Business Rules (Updated from Requirements)
- **Maximum booking lead time:** No maximum limit for advance bookings
- **Minimum booking lead time:** 60 minutes notice required
- **Currency rounding:** JOD amounts rounded UP to closest whole number
- **Service duration:** Providers choose min/max appointment durations freely
- **Provider working hours:** Set during registration, editable via dashboard. Auto-blocked on national holidays (provider can override)
- **Cancellation windows:** Free cancellation up to 3 hours before appointment
- **Cancellation penalties:** 
  - Providers: -0.1 rating per cancellation within 3 hours
  - Customers: 1hr ban (1st offense), 24hr ban (2nd offense), indefinite ban (3rd+ offense)
- **Multi-service booking:** Unlimited services per appointment within provider's available calendar slots
- **Age restrictions:** Minimum age 18+ for all services

### Technical Specifications (Updated from Requirements)
- **Offline functionality:** Customers view upcoming bookings, providers view calendar (no editing offline)
- **File upload limits:** Profile photos 2MB, license documents 10MB per file, chat/in-app images 5MB per image
- **Search results:** Infinite scroll implementation
- **Real-time sync frequency:** Available slots 5-10 seconds while viewing timeslots, booking status every 15 seconds
- **Language support:** Arabic and English language support
- **Data retention:** Booking history permanent, chat messages 1 year, user data until account deletion

---

## A. Authentication & User Management Flows

### A.1 Customer Registration Sequence
**Referenced FR-IDs:** FR-AUTH-001, FR-AUTH-002, FR-AUTH-003, FR-AUTH-007

```
Customer Registration Flow:
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ Start           │───▶│ Choose Method   │───▶│ Email/Google    │
│ Registration    │    │ Email/Google    │    │ Authentication  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
                       ┌─────────────────┐            ▼
                       │ Account Created │◀───┌─────────────────┐
                       │ Session Started │    │ OTP Verification│
                       └─────────────────┘    │ (Email only)    │
                                             └─────────────────┘
                                                       │
                                             ┌─────────────────┐
                                             │ Terms & Privacy │
                                             │ Acceptance      │
                                             └─────────────────┘
```

**Derived Rules:**
1. **Email Registration (FR-AUTH-001):**
   - Generate 6-digit OTP valid for 10 minutes
   - Max 3 OTP requests per email per hour
   - OTP expires after successful verification

2. **Google OAuth (FR-AUTH-002):**
   - Direct account creation on successful OAuth
   - Email verification skipped for Google accounts
   - Profile data pre-populated from Google

3. **Password Validation (FR-AUTH-003):**
   ```
   Password Rules:
   - Minimum 8 characters
   - At least 1 uppercase letter
   - At least 1 lowercase letter  
   - At least 1 number
   - Special characters allowed but not required
   ```

4. **Session Management (FR-AUTH-006):**
   - JWT tokens with 30-day expiry
   - Refresh token rotation every 7 days
   - "Remember me" extends to 90 days

**Validation & Error Handling:**
- **Invalid email format:** Show inline error, prevent submission
- **OTP timeout:** Allow resend after 60 seconds, max 3 attempts
- **Weak password:** Real-time strength indicator, block weak passwords
- **OAuth failure:** Fallback to email registration with error message
- **Terms rejection:** Block account creation, show explanation

### A.2 Provider Verification Workflow (Aesthetic Clinics Only)
**Referenced FR-IDs:** FR-PROV-001, FR-PROV-002, FR-PROV-003, FR-PROV-004, FR-HEALTH-001

```
Provider Verification State Machine:
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ PENDING         │───▶│ UNDER_REVIEW    │───▶│ VERIFIED        │
│ (Registration)  │    │ (OCR + Manual)  │    │ (Badge Assigned)│
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         │              ┌─────────────────┐              │
         └─────────────▶│ REJECTED        │              │
                        │ (Failed Verify) │              │
                        └─────────────────┘              │
                                 │                       │
                        ┌─────────────────┐              │
                        │ EXPIRED         │◀─────────────┘
                        │ (License Ended) │
                        └─────────────────┘
```

**Derived Rules:**
1. **MoH License Upload (FR-PROV-001, FR-HEALTH-001) - Aesthetic Clinics Only:**
   - **Scope:** Required only for aesthetic clinics performing health-related procedures
   - **Salon Exception:** Salons providing beauty services (hair, nails, etc.) do not require MoH licenses
   - Accepted formats: PDF, JPG, PNG (max 10MB per file - updated requirement)
   - OCR extraction of license number, expiry date, provider name
   - Cross-verification with MoH database (simulated API call)

2. **Verification Timeline:**
   - OCR processing: 5-15 minutes
   - Manual review: 24-48 hours for edge cases
   - Auto-approval for 95% of clear documents

3. **Badge Assignment (FR-PROV-003):**
   - Verified badge appears on all provider listings
   - Badge includes verification date and license number (masked)
   - Re-verification required 30 days before license expiry

**Validation & Error Handling:**
- **Invalid document:** OCR confidence <80% triggers manual review
- **Expired license:** Auto-rejection with renewal instructions
- **Name mismatch:** Manual review required
- **Duplicate license:** Block registration, show error
- **OCR failure:** Fallback to manual review queue
- **File size exceeded:** Reject files >10MB per updated limits
- **Age verification required:** All providers must be 18+ for license validity

---

## B. Search & Discovery Engine Logic

### B.1 Location-Based Search Algorithm
**Referenced FR-IDs:** FR-SEARCH-001, FR-SEARCH-002, FR-SEARCH-004, FR-SEARCH-005

```
Search Flow with Filtering:
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ User Location   │───▶│ Apply Filters   │───▶│ Calculate       │
│ (GPS/Manual)    │    │ (Radius/Cat)    │    │ Distances       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
┌─────────────────┐    ┌─────────────────┐            ▼
│ Return Ranked   │◀───│ Apply Sorting   │◀───┌─────────────────┐
│ Results         │    │ & Pagination    │    │ Filter Available│
└─────────────────┘    └─────────────────┘    │ Providers       │
                                             └─────────────────┘
```

**Derived Rules:**
1. **Location Priority (FR-SEARCH-001):**
   ```sql
   -- Default search order
   1. GPS location (if permitted)
   2. Last searched location
   3. Profile city (Amman default)
   4. Country default (Jordan)
   ```

2. **Radius Search (FR-SEARCH-002):**
   ```
   Distance Calculation: Haversine formula
   - 1km: Walking distance
   - 5km: Short drive
   - 10km: Medium drive  
   - 20km: Extended area
   - Custom: User input (max 50km)
   ```

3. **Availability Integration (FR-FILTER-005) - Updated for 60min minimum:**
   - "Today": Available slots remaining today after current time + 60 minutes
   - "Tomorrow": Any available slots for next calendar day
   - "This week": Any availability in next 7 days

**Search Ranking Algorithm with Infinite Scroll:**
```
// Infinite scroll implementation
function loadSearchResults(query, offset = 0, limit = 20) {
    const results = performSearch(query, offset, limit);
    
    // Calculate scores for ranking
    const rankedResults = results.map(provider => ({
        ...provider,
        score: calculateProviderScore(provider, query.userLocation)
    })).sort((a, b) => b.score - a.score);
    
    return {
        results: rankedResults,
        hasMore: results.length === limit,
        nextOffset: offset + limit
    };
}

Score = (Distance_Score × 0.3) + (Rating_Score × 0.25) + 
        (Availability_Score × 0.25) + (Verification_Score × 0.2)

Where:
- Distance_Score: Inverse distance (closer = higher score)
- Rating_Score: Normalized rating (1-5 stars)
- Availability_Score: Slot availability in next 7 days
- Verification_Score: Verified badge = 1.0, unverified = 0.7
```

### B.2 Multi-Criteria Filtering Logic
**Referenced FR-IDs:** FR-FILTER-001 to FR-FILTER-007

**Filter Application Order (Infinite Scroll Compatible):**
1. **Category Filter (FR-FILTER-001):** Primary filter, reduces result set by ~70%
2. **Location/Distance (FR-SEARCH-002):** Geographic constraint
3. **Price Range (FR-FILTER-003):** Min/max JOD filtering (rounded up to whole numbers)
4. **Rating Filter (FR-FILTER-004):** Star rating threshold
5. **Availability (FR-FILTER-005):** Real-time slot checking (60min minimum notice)
6. **Service Type (FR-FILTER-002):** Specific service matching
7. **Gender Preference (FR-FILTER-006):** Provider gender/mixed services/"Ladies only"
8. **Age Verification (18+):** Applied to all service bookings

**Derived Rules:**
```javascript
// Price filtering logic - Updated for round up to whole JOD
if (servicePrice >= filterMinPrice && servicePrice <= filterMaxPrice) {
    // Round UP to whole JOD for display
    displayPrice = Math.ceil(servicePrice);
    includeInResults = true;
}

// Rating calculation with provider cancellation penalty tracking
providerRating = (totalRatingPoints - cancellationPenalties * 0.1) / totalReviews;
if (providerRating >= filterMinRating && totalReviews >= 3) {
    includeInResults = true;
}

// Age verification for all bookings
function validateBookingAge(customerId) {
    const customer = getCustomer(customerId);
    const age = calculateAge(customer.dateOfBirth);
    if (age < 18) {
        throw new ValidationError('Minimum age requirement: 18+ for all services');
    }
    return true;
}
```

---

## C. Booking State Machine & Transaction Flow

### C.1 Real-Time Booking Engine
**Referenced FR-IDs:** FR-BOOK-001 to FR-BOOK-006, FR-PAY-001 to FR-PAY-006

```
Booking State Machine:
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ SLOT_SELECTED   │───▶│ PAYMENT_PENDING │───▶│ CONFIRMED       │
│ (Slot Reserved) │    │ (Tap Processing)│    │ (Booking Active)│
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ EXPIRED         │    │ PAYMENT_FAILED  │    │ COMPLETED       │
│ (15min timeout) │    │ (Retry Option)  │    │ (Service Done)  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │                       │
                       ┌─────────────────┐              │
                       │ CANCELLED       │◀─────────────┘
                       │ (User/Provider) │
                       └─────────────────┘
                                │
                       ┌─────────────────┐
                       │ NO_SHOW         │
                       │ (Penalty Applied)│
                       └─────────────────┘
```

**Real-Time Booking Logic (FR-BOOK-001, FR-BOOK-002):**
```javascript
// Atomic booking transaction
async function processBooking(slotId, customerId, serviceId) {
    const transaction = await db.beginTransaction();
    try {
        // 1. Check slot availability (optimistic locking)
        const slot = await db.slots.findById(slotId, { lock: true });
        if (slot.status !== 'AVAILABLE') {
            throw new Error('Slot no longer available');
        }
        
        // 2. Reserve slot with 15-minute hold
        await db.slots.update(slotId, {
            status: 'RESERVED',
            customerId: customerId,
            reservedUntil: new Date(Date.now() + 15 * 60 * 1000)
        });
        
        // 3. Create booking record
        const booking = await db.bookings.create({
            slotId, customerId, serviceId,
            status: 'SLOT_SELECTED',
            createdAt: new Date()
        });
        
        await transaction.commit();
        return { bookingId: booking.id, timeout: 900 }; // 15 minutes
    } catch (error) {
        await transaction.rollback();
        throw error;
    }
}
```

**Payment Processing Flow (FR-PAY-001 to FR-PAY-005):**
```
Payment Sequence:
Customer Pays ─▶ Tap Payments ─▶ Webhook ─▶ Booking Confirmed
     │               │               │              │
     │               │               ▼              │
     │               │    Update Booking Status     │
     │               │               │              │
     │               ▼               ▼              ▼
     │         Payment Token   Provider Notified  Customer Notified
     │               │                            
     ▼               ▼                            
Retry Logic    Settlement (T+2)                 
```

### C.2 Booking Lifecycle Rules

**Time Constraints:**
```javascript
// Booking window validation
const now = new Date();
const appointmentTime = new Date(slot.dateTime);
const hoursDifference = (appointmentTime - now) / (1000 * 60 * 60);

// Standard booking rules - Updated per requirements
if (hoursDifference < 1) {
    throw new Error('Minimum 60-minute booking notice required');
}
// No maximum booking lead time limit

// Hot Slots exception (still applies for last-minute deals)
if (slot.isHotSlot && hoursDifference >= 0.5) { // 30 minutes
    // Allow Hot Slot booking with shorter notice
    return true;
}
```

**Cancellation Logic (FR-MANAGE-002):**
```javascript
function processCancellation(booking, cancelledBy) {
    const now = new Date();
    const appointmentTime = new Date(booking.slot.dateTime);
    const hoursUntilAppointment = (appointmentTime - now) / (1000 * 60 * 60);
    
    if (hoursUntilAppointment >= 3) {
        // Free cancellation for both customers and providers
        return { fee: 0, penalty: null };
    } else {
        // Within 3-hour window - penalties apply
        if (cancelledBy === 'PROVIDER') {
            return {
                fee: 0,
                penalty: {
                    type: 'RATING_DEDUCTION',
                    amount: 0.1,
                    reason: 'Late cancellation within 3 hours'
                }
            };
        } else if (cancelledBy === 'CUSTOMER') {
            const customerHistory = getCancellationHistory(booking.customerId, 30); // 30 days
            const violationCount = customerHistory.filter(c => c.wasWithin3Hours).length;
            
            let banDuration;
            if (violationCount === 0) banDuration = 1; // 1 hour
            else if (violationCount === 1) banDuration = 24; // 24 hours  
            else banDuration = -1; // Indefinite ban
            
            return {
                fee: 0,
                penalty: {
                    type: 'BOOKING_BAN',
                    duration: banDuration,
                    reason: `Late cancellation violation #${violationCount + 1}`
                }
            };
        }
    }
}
```

---

## D. Commission & Tier Calculation Rules

### D.1 Provider Tier System
**Referenced FR-IDs:** FR-FEE-003, FR-ANALYTICS-005

```
Tier Calculation (Monthly Basis):
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ Silver Tier     │───▶│ Gold Tier       │───▶│ Diamond Tier    │
│ 0-999 bookings  │    │ 1000-1500       │    │ 1500+ bookings  │
│ Keep 10% of fee │    │ Keep 12% of fee │    │ Keep 15% of fee │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

**Tier Calculation Logic:**
```javascript
function calculateProviderTier(providerId, month, year) {
    const startDate = new Date(year, month - 1, 1);
    const endDate = new Date(year, month, 0, 23, 59, 59);
    
    const bookingCount = await db.bookings.count({
        providerId: providerId,
        status: 'COMPLETED',
        completedAt: { $gte: startDate, $lte: endDate }
    });
    
    if (bookingCount >= 1500) return 'DIAMOND';
    if (bookingCount >= 1000) return 'GOLD';
    return 'SILVER';
}

const tierBenefits = {
    SILVER: { feeRetention: 0.10, priority: 1 },
    GOLD: { feeRetention: 0.12, priority: 2 },
    DIAMOND: { feeRetention: 0.15, priority: 3 }
};
```

### D.2 Commission Calculation
**Referenced FR-IDs:** FR-FEE-001, FR-FEE-002, FR-FEE-005

```javascript
function calculateCommission(booking) {
    const serviceAmount = booking.service.price;
    
    // Base BeautyCort fee
    let beautyCortFee;
    if (serviceAmount < 25.00) {
        beautyCortFee = 2.00; // Fixed 2 JOD
    } else {
        beautyCortFee = 5.00; // Fixed 5 JOD
    }
    
    // Provider tier retention
    const providerTier = booking.provider.currentTier;
    const retentionRate = tierBenefits[providerTier].feeRetention;
    const providerRetention = beautyCortFee * retentionRate;
    
    // Final amounts (rounded UP to nearest whole JOD)
    const netBeautyCortFee = roundUpToWhole(beautyCortFee - providerRetention);
    const providerPayout = roundUpToWhole(serviceAmount - netBeautyCortFee);
    
    return {
        serviceAmount: serviceAmount,
        beautyCortFee: beautyCortFee,
        providerRetention: providerRetention,
        netBeautyCortFee: netBeautyCortFee,
        providerPayout: providerPayout
    };
}

function roundUpToWhole(amount) {
    return Math.ceil(amount); // Always round UP to closest whole number
}
```

**Settlement Rules (FR-FEE-005):**
```
T+2 Settlement Schedule:
- Monday bookings → Wednesday payout
- Tuesday bookings → Thursday payout
- Friday bookings → Monday payout (next week)
- Weekend bookings → Tuesday payout

Minimum payout threshold: 10 JOD
Hold period for new providers: 7 days
```

---

## E. Communication & Notification Orchestration

### E.1 In-App Messaging Flow
**Referenced FR-IDs:** FR-CHAT-001 to FR-CHAT-006

```
Chat Message Processing:
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ Message Sent    │───▶│ Content Filter  │───▶│ Store & Route   │
│ (Text/Media)    │    │ (Spam/Safety)   │    │ to Recipient    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
┌─────────────────┐    ┌─────────────────┐            ▼
│ Push Notification│◀───│ Delivery Status │◀───┌─────────────────┐
│ (If Offline)     │    │ Update          │    │ Real-time Sync  │
└─────────────────┘    └─────────────────┘    │ (If Online)     │
                                             └─────────────────┘
```

**Message Types & Rules:**
```javascript
const messageTypes = {
    TEXT: { maxLength: 1000, filterRequired: true },
    IMAGE: { maxSize: 5 * 1024 * 1024, formats: ['jpg', 'png', 'gif'] }, // 5MB per requirements
    VOICE: { maxDuration: 60, format: 'mp3', maxSize: 2 * 1024 * 1024 },
    SYSTEM: { automated: true, noFilter: true }
};

// Message retention policy - Updated per requirements
const retentionRules = {
    allMessages: '1 year from creation', // Standard retention period
    userDataDeletion: 'Delete when user account deleted',
    bookingHistory: 'permanent', // Booking history retained permanently
    automaticCleanup: true // Auto-delete expired messages
};
```

### E.2 Automated Notification System
**Referenced FR-IDs:** FR-NOTIFY-001 to FR-NOTIFY-006, FR-NOSHOW-001

```
Notification Trigger Engine:
Event Occurred ─▶ Check User Preferences ─▶ Route to Channels ─▶ Track Delivery
     │                     │                        │                    │
     ▼                     ▼                        ▼                    ▼
Booking Created    Push Enabled?           Push + SMS + Email      Success/Retry
Reminder Due       SMS Enabled?            (Based on priority)     Rate Limiting
Payment Failed     Email Enabled?                                  Do-Not-Disturb
```

**Notification Schedule:**
```javascript
const notificationSchedule = {
    bookingConfirmed: { delay: 0, channels: ['push', 'email'] },
    reminder24h: { 
        delay: 'appointment - 24 hours', 
        channels: ['push', 'sms'],
        condition: 'booking.status === CONFIRMED' 
    },
    reminder2h: { 
        delay: 'appointment - 2 hours', 
        channels: ['push'],
        condition: 'booking.status === CONFIRMED'
    },
    paymentFailed: { delay: 0, channels: ['push', 'email'], retry: 3 },
    hotSlotAvailable: { 
        delay: 0, 
        channels: ['push'], 
        targeting: 'location + preferences'
    }
};

// Do-not-disturb rules
const quietHours = {
    start: '22:00',
    end: '08:00',
    timezone: 'Asia/Amman',
    exceptions: ['urgent', 'payment_failed']
};
```

---

## F. Provider Dashboard & Analytics Logic

### F.1 Calendar Management Rules
**Referenced FR-IDs:** FR-DASH-001 to FR-DASH-005

```
Calendar Operations:
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ Provider Action │───▶│ Conflict Check  │───▶│ Update Schedule │
│ (Add/Edit Slot) │    │ & Validation    │    │ & Notify System │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                       ┌─────────────────┐
                       │ Handle Conflicts│
                       │ (Cancel/Move)   │
                       └─────────────────┘
```

**Availability Rules:**
```javascript
function validateSlotCreation(providerId, startTime, endTime, serviceId) {
    const service = getService(serviceId);
    const duration = service.duration; // in minutes - provider defined freely
    const bufferTime = service.bufferTime || 0; // Provider-defined buffer time
    
    // Check for overlapping bookings
    const conflicts = checkBookingConflicts(providerId, startTime, endTime + bufferTime);
    if (conflicts.length > 0) {
        throw new ConflictError('Overlapping bookings detected');
    }
    
    // Validate business hours (provider-configurable, editable via dashboard)
    if (!isWithinBusinessHours(providerId, startTime, endTime)) {
        throw new ValidationError('Outside business hours');
    }
    
    // Check for national holidays (auto-blocked unless provider overrides)
    if (isNationalHoliday(startTime) && !provider.holidayOverride) {
        throw new ValidationError('Slot blocked for national holiday');
    }
    
    // Check service duration alignment (unlimited services per appointment allowed)
    const slotDuration = (endTime - startTime) / (1000 * 60);
    if (slotDuration !== duration) {
        throw new ValidationError('Slot duration must match service duration');
    }
    
    // Validate 60-minute minimum booking notice
    const now = new Date();
    if ((startTime - now) < (60 * 60 * 1000)) {
        throw new ValidationError('Minimum 60-minute booking notice required');
    }
    
    return true;
}
```

### F.2 Business Analytics Calculations
**Referenced FR-IDs:** FR-ANALYTICS-001 to FR-ANALYTICS-005

**Revenue Metrics:**
```javascript
function calculateProviderAnalytics(providerId, period) {
    const bookings = getCompletedBookings(providerId, period);
    
    return {
        totalRevenue: bookings.reduce((sum, b) => sum + b.providerPayout, 0),
        totalBookings: bookings.length,
        averageBookingValue: totalRevenue / totalBookings,
        conversionRate: calculateConversionRate(providerId, period),
        customerRetention: calculateRetentionRate(providerId, period),
        noShowRate: calculateNoShowRate(providerId, period),
        utilizationRate: calculateSlotUtilization(providerId, period)
    };
}

function calculateConversionRate(providerId, period) {
    const profileViews = getProfileViews(providerId, period);
    const bookings = getBookings(providerId, period);
    return (bookings.length / profileViews) * 100;
}
```

---

## G. Premium Features & Monetization Logic

### G.1 Spotlight Positioning Algorithm
**Referenced FR-IDs:** FR-SPOTLIGHT-001 to FR-SPOTLIGHT-005

```
Spotlight Rotation Logic:
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ Load Spotlight  │───▶│ Apply Fairness  │───▶│ Geographic      │
│ Subscribers     │    │ Algorithm       │    │ Targeting       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                       ┌─────────────────┐
                       │ Position Ranking│
                       │ & Display       │
                       └─────────────────┘
```

**Fairness Algorithm:**
```javascript
function calculateSpotlightPositions(searchLocation, searchCategory) {
    const eligibleProviders = getSpotlightSubscribers(searchLocation, searchCategory);
    
    // Weighted rotation based on:
    // 1. Subscription tier (premium vs standard)
    // 2. Recent exposure (fairness factor)
    // 3. Performance metrics (CTR, conversion)
    
    const weightedProviders = eligibleProviders.map(provider => ({
        ...provider,
        score: calculateSpotlightScore(provider)
    }));
    
    return weightedProviders
        .sort((a, b) => b.score - a.score)
        .slice(0, 3); // Max 3 spotlight positions
}

function calculateSpotlightScore(provider) {
    const tierWeight = provider.subscriptionTier === 'PREMIUM' ? 1.5 : 1.0;
    const fairnessWeight = 1 / (provider.recentExposure + 1);
    const performanceWeight = (provider.clickThroughRate * 0.7) + (provider.conversionRate * 0.3);
    
    return tierWeight * fairnessWeight * performanceWeight;
}
```

### G.2 Hot Slots Identification
**Referenced FR-IDs:** FR-HOTSLOT-001 to FR-HOTSLOT-005

```javascript
function identifyHotSlots() {
    const now = new Date();
    const cutoffTime = new Date(now.getTime() + (4 * 60 * 60 * 1000)); // 4 hours
    
    const availableSlots = db.slots.find({
        dateTime: { $lte: cutoffTime, $gte: now },
        status: 'AVAILABLE',
        provider: { isActive: true }
    });
    
    return availableSlots.map(slot => ({
        ...slot,
        isHotSlot: true,
        originalPrice: slot.service.price,
        discountedPrice: calculateHotSlotDiscount(slot.service.price),
        urgencyLevel: calculateUrgencyLevel(slot.dateTime, now)
    }));
}

function calculateHotSlotDiscount(originalPrice) {
    // Progressive discount based on urgency
    const baseDiscount = 0.15; // 15% minimum discount
    const maxDiscount = 0.30;  // 30% maximum discount
    
    const randomDiscount = baseDiscount + (Math.random() * (maxDiscount - baseDiscount));
    const discountedPrice = originalPrice * (1 - randomDiscount);
    
    return roundUpToWhole(discountedPrice); // Round UP to whole number
}
```

---

## H. Compliance & Security Workflows

### H.1 Data Protection Procedures
**Referenced FR-IDs:** FR-SECURITY-001 to FR-SECURITY-006

**PDPL 2023 Compliance Workflow:**
```
Data Collection ─▶ Purpose Limitation ─▶ Consent Management ─▶ Processing Log
      │                    │                      │                    │
      ▼                    ▼                      ▼                    ▼
Encryption      Retention Policies    User Rights     Audit Trail
At Rest/Transit     Auto-Deletion      (Export/Delete)   (All Access)
```

**User Rights Implementation:**
```javascript
class DataProtectionService {
    async exportUserData(userId, userType) {
        const userData = await this.gatherUserData(userId, userType);
        const encryptedExport = await this.encryptForExport(userData);
        
        // Log data export request
        await this.auditLog.create({
            action: 'DATA_EXPORT',
            userId: userId,
            timestamp: new Date(),
            ipAddress: request.ip
        });
        
        return encryptedExport;
    }
    
    async deleteUserData(userId, userType) {
        // Soft delete with anonymization
        await this.anonymizeUserData(userId);
        await this.scheduleHardDelete(userId, 30); // 30-day grace period
        
        await this.auditLog.create({
            action: 'DATA_DELETION_REQUESTED',
            userId: userId,
            timestamp: new Date()
        });
    }
}
```

### H.2 Healthcare Compliance & Age Verification (Aesthetic Clinics)
**Referenced FR-IDs:** FR-HEALTH-001 to FR-HEALTH-005

**License Monitoring System (Aesthetic Clinics Only):**
```javascript
// Automated license expiration checking for aesthetic clinics only
async function checkLicenseExpiration() {
    const expiringLicenses = await db.providers.find({
        'license.expiryDate': {
            $lte: new Date(Date.now() + (30 * 24 * 60 * 60 * 1000)) // 30 days
        },
        status: 'VERIFIED',
        providerType: 'AESTHETIC_CLINIC' // Only check aesthetic clinics
    });
    
    for (const provider of expiringLicenses) {
        await notificationService.send({
            recipientId: provider.id,
            type: 'LICENSE_EXPIRING',
            urgency: 'HIGH',
            data: { expiryDate: provider.license.expiryDate }
        });
        
        // Auto-suspend if expired
        if (provider.license.expiryDate < new Date()) {
            await this.suspendProvider(provider.id, 'LICENSE_EXPIRED');
        }
    }
}

// Age verification for all services (18+ requirement)
function validateCustomerAge(customerId, serviceId) {
    const customer = getCustomer(customerId);
    const birthDate = new Date(customer.dateOfBirth);
    const today = new Date();
    
    // Accurate age calculation accounting for birth month/day
    let age = today.getFullYear() - birthDate.getFullYear();
    const monthDiff = today.getMonth() - birthDate.getMonth();
    if (monthDiff < 0 || (monthDiff === 0 && today.getDate() < birthDate.getDate())) {
        age--;
    }
    
    if (age < 18) {
        throw new ValidationError('Minimum age requirement: 18+ for all services');
    }
    
    return true;
}
```

---

## 🎯 Validation & Error Handling Summary

### Global Error Categories

**Category 1: Business Logic Errors**
- Booking conflicts, payment failures, invalid state transitions
- **Response:** User-friendly error message + retry option + support contact

**Category 2: Validation Errors**  
- Input validation, constraint violations, format errors
- **Response:** Inline field validation + correction guidance

**Category 3: System Errors**
- Database timeouts, external API failures, server errors  
- **Response:** Generic error message + automatic retry + escalation

**Category 4: Security Violations**
- Authentication failures, unauthorized access, suspicious activity
- **Response:** Block action + security log + account review

### Error Handling Patterns

```javascript
// Centralized error handling
class ErrorHandler {
    static handle(error, context) {
        const errorCategory = this.categorizeError(error);
        const userMessage = this.generateUserMessage(error, context);
        const logData = this.prepareLogData(error, context);
        
        // Log for analysis
        logger.error(logData);
        
        // Notify monitoring
        if (errorCategory === 'SYSTEM') {
            monitoring.alert(error, context);
        }
        
        // User response
        return {
            success: false,
            error: {
                code: error.code,
                message: userMessage,
                retryable: this.isRetryable(error),
                supportContact: errorCategory === 'BUSINESS' ? true : false
            }
        };
    }
}
```

---

## 📋 Implementation Priority Map

### Phase 1 (MVP Core): FR-IDs Priority
1. **Authentication & Age Verification:** FR-AUTH-001 to FR-AUTH-007 + 18+ validation (Critical path)
2. **Basic Booking with 60min Notice:** FR-BOOK-001 to FR-BOOK-003 (Revenue generation)
3. **Payment Processing with JOD Rounding:** FR-PAY-001 to FR-PAY-003 (Business critical)
4. **Provider Verification & Working Hours:** FR-PROV-001 to FR-PROV-003 (Trust factor)

### Phase 2 (User Experience): FR-IDs Priority  
1. **Search with Infinite Scroll:** FR-SEARCH-001 to FR-FILTER-007 (Discovery)
2. **Communication with File Limits:** FR-CHAT-001 to FR-CHAT-004 (Engagement)
3. **Booking Management with 3hr Cancellation:** FR-MANAGE-001 to FR-MANAGE-003 (Retention)
4. **Offline Functionality:** Limited view-only access (User convenience)

### Phase 3 (Advanced Features): FR-IDs Priority
1. **Analytics with Penalty Tracking:** FR-ANALYTICS-001 to FR-ANALYTICS-005 (Provider value)
2. **Premium Features with Pricing:** FR-SPOTLIGHT-001 to FR-HOTSLOT-005 (Monetization)
3. **Compliance with Data Retention:** FR-SECURITY-001 to FR-HEALTH-005 (Legal requirements)
4. **Multi-language Support:** Arabic/English throughout platform (Accessibility)

---

**Document Status:** Complete v1.0 - Ready for technical implementation  
**Cross-References:** All FR-IDs from functional_requirements.md mapped  
**Next Steps:** Technical architecture design and API specification