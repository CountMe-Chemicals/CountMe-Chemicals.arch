# CountMe Chemicals — Technical Architecture Document

![CountMe Chemicals](https://countmechemicals.com/countme_logo.png)

---

## 1. Executive Summary

CountMe Chemicals is an Ai powered B2B chemicals commerce platform designed to digitise the procurement and supply lifecycle for chemical products. The platform connects buyers, sellers, and administrators across three coordinated client surfaces — a consumer web application, an administrative web application, and a mobile application — all of which are governed by a single centralised backend service.

The system facilitates the full commercial journey: product discovery, request-for-quote (RFQ) origination and negotiation, order creation, payment settlement, logistics coordination, and post-transaction operations. It incorporates artificial intelligence, real-time communication, geolocation, identity verification, and multi-channel notification delivery as first-class capabilities. The platform is containerised for consistent and reproducible deployment.

---

## 2. System Architecture Overview

```
+-----------------------------------------------------------------------------------+
|                             CLIENT CHANNELS                                       |
|                                                                                   |
|   +----------------------+   +----------------------+   +---------------------+  |
|   | Consumer Web App     |   | Admin Web App        |   | Mobile App          |  |
|   | React · TypeScript   |   | React · TypeScript   |   | Flutter · Dart      |  |
|   | Buyer & Seller       |   | Operations &         |   | Buyer & Seller      |  |
|   | Commerce Journeys    |   | Governance Workflows |   | Commerce Journeys   |  |
|   +---------+------------+   +---------+------------+   +----------+----------+  |
+-------------|-------------------------|---------------------------|---------------+
              |   HTTPS / WebSocket     |   HTTPS / WebSocket       |   HTTPS / WebSocket
              +-------------------------+---------------------------+
                                        |
                                        v
+-----------------------------------------------------------------------------------+
|                         BACKEND CONTROL PLANE                                     |
|                    Node.js · Express · TypeScript · MongoDB                       |
|                                                                                   |
|   Identity & Access  |  ChemNet RFQ Engine  |  Quote & Negotiation               |
|   Order Lifecycle    |  Payment Orchestration|  Logistics Coordination            |
|   Realtime Dispatch  |  AI Intelligence     |  KYC / KYB Verification            |
|   Referral & Rewards |  Analytics Engine    |  Content & Media Management        |
+----------+-----------+----------+-----------+-----------+----------+--------------+
           |                      |                       |          |
           v                      v                       v          v
+------------------+   +-------------------+   +--------------+  +----------------+
| Data Persistence |   | Realtime Fabric   |   | AI Layer     |  | Scheduled Jobs |
| MongoDB          |   | Socket.IO / WSS   |   | Google Gemini|  | Agenda / Cron  |
+------------------+   +-------------------+   +--------------+  +----------------+
           |                      |
           +----------+-----------+
                      |
                      v
+-----------------------------------------------------------------------------------+
|                         EXTERNAL INTEGRATION ECOSYSTEM                            |
|                                                                                   |
|  Paystack (Payments)    |  Fez Delivery (Logistics)  |  Google Maps (Geolocation)|
|  Vove (KYC/KYB)         |  Cloudinary (Media)        |  Firebase FCM (Push)      |
|  Resend (Email)         |  ExchangeRate API (FX)     |  Slack (Ops Alerts)       |
+-----------------------------------------------------------------------------------+

+-----------------------------------------------------------------------------------+
|                         INFRASTRUCTURE & DEPLOYMENT                               |
|                         Docker · Containerised Deployment                         |
+-----------------------------------------------------------------------------------+
```

---

## 3. Platform Channels

### 3.1 Consumer Web Application

The consumer web application serves both buyers and sellers on the platform. It is built with React and TypeScript and provides a rich, interactive commerce experience.

**Buyer-facing capabilities:**
- Chemical product discovery, browsing by category, alphabet index, and supplier
- Origination of ChemNet requests (the platform's RFQ mechanism)
- Quote review, counter-negotiation, and acceptance
- Cart management and checkout
- Order tracking and delivery status monitoring
- AI-powered market insights and price intelligence
- Real-time chat with suppliers
- Referral programme management
- Subscription plan management
- Document downloads (invoices, receipts)

**Seller-facing capabilities:**
- Chemical product listing and catalogue management
- Quote response, negotiation, and approval workflows
- Order fulfilment oversight
- Payout and withdrawal management
- Public supplier profile and ratings

### 3.2 Admin Web Application

The admin application provides platform operators with governance, moderation, and operational oversight capabilities. It is a distinct, role-gated surface with no buyer/seller commerce flows.

**Capabilities include:**
- User management and KYC/KYB status oversight
- Order supervision and exception handling
- Delivery queue management and logistics oversight
- Chemical catalogue governance (approvals, categorisation)
- Dispute resolution workflows
- Referral and bonus code management
- Subscription plan configuration
- Blog and podcast content management
- Platform analytics and performance dashboards
- Operational alerting and audit-trail visibility

### 3.3 Mobile Application

The mobile application (Flutter/Dart) provides buyers and sellers with a native mobile experience that mirrors the core commerce and negotiation capabilities of the consumer web application. It operates under the same backend control plane and enforces the same business rules. Role-aware rendering adapts the interface for buyers versus sellers on the same device installation.

---

## 4. Backend Control Plane

All three client channels communicate exclusively with a single, authoritative backend service. The backend is responsible for all business rule enforcement, state management, and integration orchestration. No client channel can mutate platform state without passing through this central control plane.

### 4.1 Domain Capability Modules

The backend is organised into discrete functional domains:

| Domain | Responsibility |
|---|---|
| **Identity & Access** | User registration, authentication (email and Google OAuth), session management, role and permission enforcement across buyer, seller, and admin contexts |
| **KYC / KYB Verification** | Identity and business verification workflows for individuals and corporate entities, including multi-country verification flows |
| **Chemical Catalogue** | Product ingestion, categorisation, naming normalisation, visibility controls, and search indexing |
| **ChemNet RFQ Engine** | Platform's core request-for-quote mechanism — buyers submit chemical procurement requests, the engine routes them to relevant sellers, and facilitates structured negotiation |
| **Quote & Negotiation** | End-to-end lifecycle management for quote origination, counter-offer exchange, approval, expiry, and dismissal with full audit trail |
| **Order Management** | Order creation from accepted quotes, lifecycle state progression, invoice generation, and cross-surface status synchronisation |
| **Payment Orchestration** | Secure collection and settlement of buyer payments, seller payout coordination, and transaction record management |
| **Logistics Coordination** | Delivery request creation, carrier dispatch through Fez Delivery, shipment tracking, and ETA management |
| **Messaging & Chat** | Real-time buyer-seller direct messaging within transaction contexts, powered by persistent conversation threads |
| **Dispute Resolution** | Structured dispute filing, evidence management, and resolution workflows for contested transactions |
| **Referral & Rewards** | Referral link generation, reward ledger tracking, and bonus code issuance and redemption |
| **Subscription Management** | Plan configuration, subscription lifecycle management, and feature entitlement enforcement |
| **Notifications** | Multi-channel notification dispatch — push (Firebase FCM), email (Resend), and in-app realtime (Socket.IO) |
| **Analytics & Dashboard** | Aggregated platform metrics, trend visualisation, and business performance reporting |
| **AI Intelligence** | Market trend analysis, price recommendation, and AI-assisted workflow support via Google Gemini |
| **Content Management** | Blog article authoring and publishing, podcast content management, and editorial workflow |
| **Media Management** | Image and document upload, transformation, storage, and CDN delivery via Cloudinary |
| **Geolocation Services** | Address resolution, map-based location display, and logistics routing support via Google Maps |
| **Currency Exchange** | Live foreign exchange rate resolution for multi-currency pricing across international quotes |

### 4.2 Realtime Communication Layer

The platform employs WebSocket connections (Socket.IO) to maintain persistent, authenticated channels between clients and the backend. This enables:

- Live quote and negotiation status updates delivered to all relevant parties simultaneously
- Real-time buyer-seller chat within active transaction threads
- Instant delivery status updates as shipments progress through carrier milestones
- Cross-surface synchronisation — an admin action is immediately visible to the affected buyer or seller without a page refresh

### 4.3 Background Processing & Scheduled Jobs

The backend runs a scheduled job layer (Agenda.js and node-cron) that handles time-sensitive operations autonomously:

- Automatic expiry of stale or abandoned negotiation sessions
- Periodic synchronisation with delivery carrier systems for shipment status updates
- Scheduled notification dispatch for time-based business events
- Routine data maintenance and cleanup operations

---

## 5. Data Architecture

The platform's data is persisted in MongoDB, a document-oriented database well-suited to the variable-schema nature of chemical product attributes, negotiation histories, and multi-party transaction records.

**Primary data domains:**

| Domain | Key Entities |
|---|---|
| **Users & Identity** | User accounts, KYC/KYB records, session tokens, activity logs |
| **Catalogue** | Chemical records, categories, naming indices, visibility states |
| **Commerce** | Quotes, ChemNet requests, negotiation threads, orders, invoices |
| **Payments** | Transaction records, payout ledgers, withdrawal requests |
| **Logistics** | Delivery requests, shipment tracking records |
| **Social & Engagement** | Reviews, ratings, conversations, messages |
| **Platform Operations** | Referral ledgers, bonus codes, subscriptions, disputes |
| **Content** | Blog articles, podcast entries, media attachments |
| **Analytics** | User activity records, dashboard aggregates |

---

## 6. External Integration Ecosystem

CountMe Chemicals integrates with a curated set of specialised third-party services. Each integration handles a distinct, bounded concern:

### 6.1 Paystack — Payment Processing
Paystack handles all payment collection and fund transfer operations. Buyers initiate payments through Paystack-secured flows; the platform reconciles settlement confirmations before advancing order status. Seller payouts are also disbursed through Paystack's transfer infrastructure.

### 6.2 Fez Delivery — Last-Mile Logistics
Fez Delivery is the primary logistics partner for physical shipment of chemical orders. The platform creates delivery requests via Fez, receives real-time shipment status updates through Fez webhooks, and surfaces carrier milestones (dispatch, in-transit, delivered) directly to buyers and sellers within their order tracking views.

### 6.3 Google Maps — Geolocation & Address Services
Google Maps provides address resolution and geolocation capabilities used during delivery setup and supplier location display. Buyers can view supplier locations, and the logistics module uses geolocation data to support accurate delivery routing and address validation.

### 6.4 Vove — Identity & Business Verification (KYC/KYB)
Vove powers the platform's identity verification layer. Individual users undergo Know Your Customer (KYC) verification, while business entities complete Know Your Business (KYB) verification. Multi-country verification flows are supported, covering different compliance requirements for different operating regions. Vove communicates verification outcomes to the platform via secured webhooks.

### 6.5 Google Gemini — AI Intelligence
Google Gemini provides the generative AI backbone for the platform's intelligence features. This includes market trend analysis presented to buyers and sellers, price benchmarking and recommendation within negotiation contexts, and the AI-assisted Quotana feature within ChemNet that helps match buyers with suitable suppliers.

### 6.6 Firebase Cloud Messaging (FCM) — Push Notifications
Firebase Cloud Messaging delivers push notifications to web browsers and mobile devices. Platform events such as new quote responses, order status changes, delivery updates, and message arrivals are dispatched through FCM to ensure users receive timely alerts regardless of whether they are actively using the application.

### 6.7 Resend — Transactional Email
Resend handles all transactional email delivery. This covers account registration and verification, order confirmations, invoice delivery, payment receipts, dispute updates, and other workflow-triggered communications. Email templates are managed server-side to ensure consistency and auditability.

### 6.8 Cloudinary — Media Storage & Delivery
All user-uploaded images and documents (chemical product images, KYC documents, profile photos, blog media) are stored in and delivered from Cloudinary. Cloudinary's CDN ensures fast, globally distributed media delivery, and its transformation layer is used for image resizing and optimisation.

### 6.9 ExchangeRate API — Foreign Exchange
The ExchangeRate API provides live currency conversion rates used when buyers and sellers operate in different currencies. Quote pricing and order totals are resolved against up-to-date exchange rates to ensure transactional accuracy across international procurement.

### 6.10 Slack — Operational Alerting
Slack receives real-time operational alerts from the backend for monitoring and incident response purposes. Key events such as KYC submission notifications, payment anomalies, and system exceptions are routed to dedicated Slack channels for the operations team, complementing the structured logging infrastructure.

---

## 7. Security Architecture

Security is enforced at the server boundary and cannot be bypassed by client-side state manipulation.

### 7.1 Identity & Session Security
- All authentication flows are validated server-side, including standard credential authentication and Google OAuth flows.
- Session tokens are issued as signed, time-limited JWTs and validated on every request.
- Two-factor authentication (TOTP) is available for accounts requiring elevated security.
- Passwords are stored exclusively as cryptographic hashes; plaintext credentials are never persisted.

### 7.2 Role-Based Access Control
- Every request is evaluated against the requestor's assigned role: buyer, seller, admin, or super-admin.
- Resource-level ownership checks ensure buyers and sellers can only act on records they own or are a party to.
- Admin-only operations are categorically inaccessible to consumer role holders, regardless of client-side state.

### 7.3 Transaction & Lifecycle State Security
- Order and negotiation lifecycle state transitions are strictly validated server-side. A client cannot skip states or reactivate expired records by presenting a manipulated request.
- Pricing and payable totals are always resolved server-side at the point of checkout; client-submitted totals are ignored.
- Payment confirmation is verified against the payment provider's authoritative record before any order is advanced.

### 7.4 Integration Security
- Inbound webhooks from third-party services (Fez, Vove, Paystack) are validated using provider-issued signatures before any state is mutated.
- HTTP security headers, rate limiting, and request validation are applied at the server boundary.
- All service-to-service communication uses provider-specific credentials stored in secured environment configuration, never embedded in client-accessible code.

---

## 8. Infrastructure & Deployment

### 8.1 Containerisation with Docker
The entire platform is containerised using Docker with a multi-stage build process:

- **Stage 1** — The frontend React application is compiled into a production-optimised static asset bundle.
- **Stage 2** — The backend TypeScript service is compiled into a production Node.js runtime.
- **Stage 3** — A final, minimal production image is assembled that runs the backend server and serves the compiled frontend assets from the same process, reducing infrastructure overhead.

This containerised approach ensures the application runs identically across development, staging, and production environments, eliminating environmental drift and simplifying deployment to any container-capable hosting infrastructure.

### 8.2 Observability & Logging
- Structured application logs are generated throughout the backend using a dedicated logging layer (Winston), capturing operational events, warnings, and exceptions with contextual metadata.
- HTTP request logs provide a complete audit trail of all inbound traffic.
- Frontend user behaviour is tracked through PostHog analytics integration.
- Critical operational events are surfaced to the operations team in real time via Slack.

---

## 9. AI & Intelligence Layer

The platform's AI capabilities are powered by Google Gemini and are integrated in a controlled, policy-governed manner:

- **Quotana (ChemNet AI Assistant):** Embedded within the ChemNet RFQ workflow, Quotana assists buyers in structuring procurement requests and helps surface relevant supplier matches. AI outputs are channelled into defined platform workflows (browse, request, negotiate) and cannot directly mutate transaction state.
- **Market Intelligence:** Buyers and sellers receive AI-generated market trend analysis and price benchmarking insights drawn from catalogue and transaction patterns. These are advisory in nature and surfaced through dedicated analytics views.
- **Negotiation Context:** During active negotiation, AI-assisted price guidance is available to inform counter-offer decisions.

All AI interactions are bounded by the user's role and current workflow context, and the full surrounding transaction history provides a human-auditable record of any AI-influenced decisions.

---

## 10. End-to-End Workflow Summary

The following describes the primary commerce flow from user intent to fulfilment:

```
1. DISCOVERY
   Buyer browses chemical catalogue (by category, name index, or supplier profile)
   OR submits a ChemNet RFQ with Quotana AI assistance

2. MATCHING & NEGOTIATION
   Sellers receive and respond to ChemNet requests or buyer quote enquiries
   Both parties exchange counter-offers through the structured negotiation engine
   Real-time updates keep both parties synchronised across web and mobile

3. ORDER CREATION
   Accepted quote transitions to a confirmed order
   Invoice is generated server-side
   Buyer receives email confirmation and in-app notification

4. PAYMENT
   Buyer completes payment through Paystack
   Payment confirmation is verified server-side before order is advanced
   Seller is notified of confirmed payment via push, email, and in-app

5. LOGISTICS & FULFILMENT
   Order is dispatched through Fez Delivery
   Shipment tracking updates are received via Fez webhooks
   Real-time delivery status is surfaced to buyer and seller
   Google Maps supports address resolution and location display

6. COMPLETION & POST-TRANSACTION
   Delivery confirmation closes the order lifecycle
   Buyer can submit a review and rating for the supplier
   Seller payout is initiated through Paystack
   Full transaction history remains auditable in the admin dashboard
```

---

## 11. Compliance & Governance Posture

- The platform enforces a clean separation between consumer-facing commerce surfaces and the administrative governance surface.
- Every significant state transition across the negotiation, payment, and order lifecycle is recorded with timestamps and actor identity, providing a complete, auditable trail.
- KYC and KYB verification through Vove ensures that users and businesses operating on the platform have undergone regulated identity validation appropriate to their jurisdiction.
- The admin layer provides operators with full visibility into disputes, fulfilment exceptions, and user verification statuses, enabling timely intervention and oversight.
- The architecture is designed to accommodate evolving compliance requirements, with verification flows configurable per-country without structural changes to the platform.

---

## 12. Technology Summary

| Layer | Technology |
|---|---|
| Consumer & Admin Web | React 18, TypeScript, Vite, TailwindCSS |
| Mobile | Flutter, Dart |
| Backend Service | Node.js, Express, TypeScript |
| Database | MongoDB |
| Realtime Communication | Socket.IO (WebSocket) |
| Containerisation | Docker (multi-stage build) |
| AI Intelligence | Google Gemini |
| Payments | Paystack |
| Logistics | Fez Delivery |
| Geolocation | Google Maps |
| Identity Verification | Vove (KYC/KYB) |
| Push Notifications | Firebase Cloud Messaging |
| Transactional Email | Resend |
| Media Storage & CDN | Cloudinary |
| Foreign Exchange | ExchangeRate API |
| Background Jobs | Agenda.js, node-cron |
| Operational Alerting | Slack Webhooks |
| Frontend Analytics | PostHog |
