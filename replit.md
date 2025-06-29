# BeautyCort - Beauty Booking Platform

## Overview

BeautyCort is a mobile-first marketplace for booking beauty, spa, and aesthetic-medical appointments in Jordan, starting with Amman. The platform aims to solve the manual booking process that typically takes >48 hours by providing instant confirmations in <10 seconds. The core value proposition is "Book beauty in a blink" with features like real-time booking, verified provider badges, deposit systems to reduce no-shows, and integrated communication.

## System Architecture

### Frontend Architecture
- **Framework**: Flutter (mobile-first approach)
- **Target Platforms**: iOS and Android with mobile-first design
- **UI/UX Focus**: Simple, intuitive booking flow optimized for quick interactions

### Backend Architecture
- **Cloud Platform**: AWS
- **Database**: Not yet specified (likely to use Drizzle ORM with potential Postgres integration)
- **Authentication**: Multi-provider system supporting email/OTP and Google OAuth
- **Real-time Features**: Real-time slot availability engine for instant booking confirmations

### Key Integrations
- **Payment Processing**: Tap Payments integration supporting Apple Pay and JoPACC wallets
- **Communication**: WhatsApp-style in-app chat with text, images, and voice notes
- **Verification**: OCR-based MoH (Ministry of Health) license verification system
- **Notifications**: OTP verification system for user authentication

## Key Components

### User Management System
- Dual user types: Customers ("Beauty Sara" persona) and Providers ("Clinic Leen" persona)
- Email sign-up with OTP verification
- Google OAuth integration
- Phone number verification (optional)
- Profile management with terms acceptance

### Search & Discovery Engine
- Location-based search (Jordan focus, Amman primary)
- Multi-category filtering (salon, spa, aesthetic-medical)
- Price range and rating filters
- Distance and availability-based filtering
- Advanced search capabilities

### Booking Engine
- Real-time slot availability system
- <10 second confirmation target
- Hot Slots flash deals for last-minute appointments (<4 hours)
- Automated reminder system to reduce no-shows

### Trust & Verification System
- Mandatory MoH license upload with OCR verification
- Verified Practitioner Badges for licensed providers
- Review and rating system
- Provider certification display

### Provider Management
- Service catalog management
- Photo gallery and portfolio features
- Calendar and slot management
- Direct communication with customers
- Revenue optimization tools

## Data Flow

1. **User Registration**: Email/Google OAuth → OTP verification → Profile creation
2. **Search Process**: Location selection → Category filtering → Provider discovery
3. **Booking Flow**: Provider selection → Service choice → Time slot selection → Payment → Instant confirmation
4. **Communication**: In-app chat system for provider-customer interaction
5. **Verification**: Provider license upload → OCR processing → Badge assignment

## External Dependencies

- **Payment Gateway**: Tap Payments for Apple Pay and JoPACC wallet integration
- **Cloud Services**: AWS infrastructure for hosting and scaling
- **Authentication**: Google OAuth for social login
- **OCR Services**: For MoH license verification (specific provider TBD)
- **Communication**: WhatsApp-style messaging infrastructure

## Deployment Strategy

- **Current Phase**: Pre-MVP Development
- **Target Launch**: Q4 2025 (October 2025)
- **Geographic Rollout**: Jordan (Amman) first, then expansion within MENA region
- **Platform Strategy**: Mobile-first with Flutter for cross-platform deployment
- **Scaling Plan**: AWS-based infrastructure for handling real-time booking loads

## User Preferences

Preferred communication style: Simple, everyday language.

## Changelog

Changelog:
- June 29, 2025: Initial setup
- June 29, 2025: Removed deposit functionality per user request - no deposits will be collected from customers
- June 29, 2025: Created comprehensive functional requirements document with gap analysis identifying missing domain specifications
- June 29, 2025: Created comprehensive business logic specification mapping all FR-IDs to detailed workflows, state machines, and validation rules
- June 29, 2025: Updated functional requirements and business logic with specific business rules: 60min booking notice, 3hr free cancellation, JOD rounded up to whole numbers, 18+ age requirement, unlimited advance booking
- June 29, 2025: Refined business logic document to align with all functional requirements updates including file size limits, offline functionality, infinite scroll, and penalty systems
- June 29, 2025: Updated MoH license requirement to apply only to aesthetic clinics (not salons), added "ladies only" staff filter for cultural norms, modified fee structure to calculate per service rather than per booking