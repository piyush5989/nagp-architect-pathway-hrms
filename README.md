# NewFinTech HRMS Documentation

**Version:** 1.0  
**Date:** November 24, 2025  
**Status:** Final  
**Prepared for:** NewFinTech  

---
## Table of Contents
1. [Executive Summary](#executive-summary)
2. [Non-Functional Requirements (NFRs)](#non-functional-requirements-nfrs)
3. [Assumptions](#assumptions)
4. [Scope Definition](#scope-definition)
5. [Architecture Diagram](#architecture-diagram)
6. [Deployment Diagram](#deployment-diagram)
7. [CI/CD Architecture](#cicd-architecture)
8. [Technology Stack with DARs](#technology-stack-with-dars)
9. [Effort Estimation](#effort-estimation)
10. [Conclusion and Next Steps](#conclusion-and-next-steps)

---


## Executive Summary

This document presents a comprehensive architecture design for NewFinTech's multi-tenant HRMS (Human Resource Management System) solution. The system is designed to serve 25 tenants initially, with each tenant supporting up to 10,000 employees and 250 concurrent users, with provisions for 10% annual growth over the next 5 years.

### Key Highlights

**System Capabilities:**
- Multi-tenant SaaS architecture with strong data isolation
- Web application (React) and native mobile apps (Flutter) for iOS and Android
- Three core modules: Employee Management, Payroll Management, and Leave Management
- Offline capability for mobile applications with intelligent sync
- RESTful APIs for third-party integrations
- Flexible licensing model with feature-based tiers

**Technology Decisions:**
- **Backend:** Node.js with TypeScript for rapid development and excellent async handling
- **Frontend:** React for web, Flutter for mobile (95%+ code reuse)
- **Database:** PostgreSQL with schema-based multi-tenancy for strong data isolation
- **Cloud:** AWS for comprehensive managed services and global infrastructure
- **Architecture:** Microservices with event-driven patterns for scalability
---

## Non-Functional Requirements (NFRs)

### 1. Performance Requirements

| Metric | Requirement | Rationale |
|--------|-------------|-----------|
| API Response Time | < 200ms (p95) for read operations | Ensure smooth user experience for data retrieval |
| API Response Time | < 500ms (p95) for write operations | Acceptable latency for data modifications |
| Page Load Time | < 2 seconds (initial load) | Industry standard for web applications |
| Database Query Time | < 100ms (p95) | Prevent bottlenecks at data layer |
| Concurrent Users per Tenant | 250 users | As specified in requirements |
| Throughput | 1000 requests/second per tenant (peak) | Handle 250 concurrent users with 4 req/sec average |

### 2. Scalability Requirements

| Aspect | Current | Year 5 Projection | Design Consideration |
|--------|---------|-------------------|---------------------|
| Number of Tenants | 25 | 41 (10% annual growth) | Horizontal scaling of services |
| Users per Tenant | 250 concurrent | 403 concurrent | Auto-scaling based on load |
| Employees per Tenant | 10,000 | 16,105 | Database partitioning strategy |
| Total System Load | 6,250 concurrent users | 16,523 concurrent users | Cloud-native architecture |
| Storage per Tenant | ~50 GB (estimated) | ~80 GB | Scalable storage solution |

**Scalability Strategy:**
- Horizontal scaling for stateless services
- Database read replicas for read-heavy operations
- Caching layer to reduce database load
- CDN for static assets
- Auto-scaling groups based on CPU/memory metrics

### 3. Security Requirements

| Requirement | Implementation | Priority |
|-------------|----------------|----------|
| Multi-tenant Data Isolation | Tenant ID in all queries, separate schemas/databases | Critical |
| Authentication | Multi-factor authentication (MFA) support | High |
| Authorization | Role-Based Access Control (RBAC) | Critical |
| Data Encryption at Rest | AES-256 encryption for databases and file storage | Critical |
| Data Encryption in Transit | TLS 1.3 for all communications | Critical |
| Password Policy | Min 12 characters, complexity requirements, rotation every 90 days | High |
| Session Management | JWT with 1-hour expiry, refresh token mechanism | High |
| API Security | Rate limiting, API key management, OAuth2 | High |
| Audit Logging | All data modifications logged with user/timestamp | High |
| Compliance | GDPR, SOC2, ISO 27001 ready | High |
| Penetration Testing | Quarterly security audits | Medium |
| Vulnerability Scanning | Automated scanning in CI/CD pipeline | High |

### 4. Availability Requirements

| Metric | Target | Implementation |
|--------|--------|----------------|
| System Uptime | 99.9% (8.76 hours downtime/year) | Multi-AZ deployment, redundancy |
| Planned Maintenance Window | 4 hours/month (off-peak hours) | Blue-green deployment strategy |
| Recovery Time Objective (RTO) | < 1 hour | Automated failover mechanisms |
| Recovery Point Objective (RPO) | < 15 minutes | Continuous database replication |
| Backup Frequency | Daily full backup, hourly incremental | Automated backup solution |
| Backup Retention | 30 days online, 7 years archived | Compliance with data retention laws |
| Disaster Recovery | Active-passive DR site in different region | Cross-region replication |

### 5. Maintainability Requirements

| Aspect | Requirement | Implementation |
|--------|-------------|----------------|
| Code Quality | Minimum 80% test coverage | Automated testing in CI/CD |
| Code Standards | Consistent coding standards across teams | ESLint, Prettier, SonarQube |
| Documentation | API documentation, architecture docs, runbooks | Swagger/OpenAPI, Confluence |
| Monitoring | Real-time monitoring and alerting | Prometheus, Grafana, CloudWatch |
| Logging | Centralized logging with correlation IDs | ELK Stack or CloudWatch Logs |
| Deployment | Zero-downtime deployments | Rolling updates, canary deployments |

### 6. Compatibility Requirements

| Platform | Requirement | Details |
|----------|-------------|---------|
| Web Browsers | Chrome 90+, Firefox 88+, Safari 14+, Edge 90+ | Last 2 major versions |
| Android | Android 8.0 (API 26) and above | 95%+ market coverage |
| iOS | iOS 13.0 and above | 98%+ market coverage |
| Screen Resolutions | Responsive design (320px to 4K) | Mobile-first approach |

### 7. Offline Capability (Mobile Apps)

| Feature | Offline Support | Sync Strategy |
|---------|----------------|---------------|
| View Employee Profile | Full offline access | Sync on app open |
| View Salary Slips | Last 12 months cached | Background sync |
| View Leave Balance | Cached data | Sync every 4 hours |
| Apply for Leave | Queue requests offline | Sync when online |
| View Organization Holidays | Full offline access | Sync weekly |
| Notifications | Local notifications for cached data | Push when online |

### 8. API Integration Requirements

| Aspect | Requirement | Details |
|--------|-------------|---------|
| API Style | RESTful APIs | Standard HTTP methods |
| API Versioning | URL-based versioning (v1, v2) | Backward compatibility for 2 versions |
| Rate Limiting | 1000 requests/hour per API key | Prevent abuse |
| Documentation | OpenAPI 3.0 specification | Interactive documentation |
| Authentication | OAuth2 with client credentials | Secure third-party access |
| Webhooks | Event-driven notifications | For real-time integrations |
| Data Format | JSON (primary), CSV for bulk exports | Standard formats |


---

## Assumptions

### Infrastructure Assumptions

1. **Cloud Provider**: AWS will be the primary cloud provider
   - Rationale: Mature services, global presence, strong enterprise support
   - Alternative: Azure or GCP can be considered based on client preference

2. **Deployment Regions**: 
   - Primary: ap-south-1 (Mumbai)
   - DR: ap-southeast-1 (Singapore)
   - Future expansion to other regions based on tenant location

3. **Network Connectivity**: 
   - Minimum 10 Mbps internet connectivity for web users
   - 3G or better for mobile users
   - VPN not required for standard users (HTTPS sufficient)

4. **Infrastructure as Code**: Terraform will be used for infrastructure provisioning

### Data Assumptions

5. **Data Retention**: 
   - Active employee data retained indefinitely
   - Terminated employee data retained for 7 years (compliance)
   - Audit logs retained for 7 years
   - Backup retention: 30 days online, 7 years archived

6. **Data Volume per Employee**:
   - Employee profile: ~50 KB
   - Salary records: ~10 KB per month
   - Leave records: ~5 KB per year
   - Documents: ~10 MB per employee (average)
   - Total per employee: ~10.5 MB

7. **Data Growth**: 10% annual growth in both tenants and employees per tenant

### User Behavior Assumptions

8. **Peak Usage Hours**: 
   - 9 AM - 11 AM and 2 PM - 4 PM local time
   - Peak load: 3x average load
   - Month-end spike for payroll processing

9. **Concurrent User Ratio**: 
   - 2.5% of total employees are concurrent users during peak hours
   - Average session duration: 15 minutes
   - Average requests per user: 4 requests/second during active use

10. **Mobile vs Web Usage**: 
    - 60% web users
    - 40% mobile users
    - Mobile users primarily view data, web users for data entry

### Feature Usage Assumptions

11. **Employee Management**: 
    - 5% employee churn annually (add/remove operations)
    - 10% employee updates monthly (profile changes)

12. **Payroll Management**: 
    - Salary slips generated once per month for all employees
    - Annual salary adjustments for 30% of employees
    - Compensation structure changes: 2-3 times per year

13. **Leave Management**: 
    - Average 15 leave days per employee per year
    - 70% leave approval rate (30% rejected/cancelled)
    - Peak leave periods: Summer (June-August), Year-end (December)

### Technology Assumptions

14. **Development Team**: 
    - Team has experience with modern web technologies
    - Team size: 15-20 developers (full stack, mobile, DevOps)
    - Agile/Scrum methodology

15. **Development Timeline**: 
    - 10-12 months for full implementation
    - Phased rollout approach

16. **Third-Party Services**: 
    - Email service (SendGrid/AWS SES) available
    - SMS service (Twilio/AWS SNS) for notifications
    - PDF generation service available
    - Payment gateway for subscription billing

### Security Assumptions

17. **Authentication**: 
    - Corporate email addresses used for user identification
    - SSO integration not required in Phase 1 (future enhancement)
    - MFA optional but recommended

18. **Compliance**: 
    - Client responsible for country-specific compliance (tax laws, labor laws)
    - System provides audit trails and data export capabilities

### Mobile App Assumptions

19. **App Store Policies**: 
    - Apps will be published under client's developer accounts
    - App review process: 1-2 weeks per platform
    - Updates released monthly (bug fixes), quarterly (features)

20. **Offline Mode**: 
    - Limited to read-only operations and leave application
    - Sync required within 7 days to prevent data staleness
    - Conflict resolution: server data takes precedence with user notification

### Support and Maintenance Assumptions

23. **Support Model**: 
    - 24/7 support for critical issues (P1)
    - Business hours support for non-critical issues
    - SLA: 1 hour response for P1, 4 hours for P2, 24 hours for P3

24. **Maintenance Windows**: 
    - Weekly maintenance: Sundays 2 AM - 4 AM (tenant local time)
    - Major updates: Quarterly with 2-week advance notice

---

## Scope Definition

### In Scope

#### 1. Employee Management Module
- **Employee CRUD Operations**
  - Add new employees with complete profile information
  - Update employee details (personal, professional, contact)
  - Soft delete employees (retain data for compliance)
  - Bulk import/export of employee data (CSV)

- **Employee Profile Management**
  - Personal details (name, DOB, gender, contact information)
  - Professional details (designation, department, joining date, reporting manager)
  - Project assignments and history
  - Document management (ID proofs, certificates, contracts)
  - Emergency contact information
  - Bank account details for salary processing

- **Role-Based Access**
  - Admin: Full access to all employee data
  - Manager: Access to team members' data
  - Employee: Access to own profile (read-only for most fields)

#### 2. Payroll/Salary Management Module
- **Compensation Management**
  - Define salary structure with components:
    - Basic salary
    - House Rent Allowance (HRA)
    - Dearness Allowance (DA)
    - Special Allowance
    - Performance Bonus
    - Deductions (PF, Tax, Insurance)
  - Adjust compensation for individual employees
  - Bulk salary adjustments (percentage-based)
  - Salary revision history

- **Salary Slip Generation**
  - Monthly salary slip generation (PDF format)
  - Annual salary statement (Form 16 ready)
  - Email delivery of salary slips
  - Employee self-service portal to download slips
  - Salary slip template customization per tenant

- **Payroll Processing**
  - Monthly payroll run with approval workflow
  - Payroll reports for finance team
  - Bank transfer file generation (NEFT/RTGS format)

#### 3. Leave Management Module
- **Holiday Management**
  - Define organization-wide holidays for the year
  - Region-specific holidays (if multi-location)
  - Holiday calendar view
  - Import/export holiday list

- **Leave Types and Balance**
  - Casual Leave (CL)
  - Paid Leave (PL) / Earned Leave (EL)
  - Sick Leave (SL)
  - Leave balance tracking per employee
  - Leave accrual rules (monthly/annual)
  - Leave carry-forward policies

- **Leave Application and Approval**
  - Employee applies for leave with date range and reason
  - Manager approval workflow
  - Multi-level approval (if required)
  - Leave cancellation by employee
  - Leave rejection with reason
  - Email notifications for all leave actions
  - Leave calendar view (team and organization)

- **Leave Reports**
  - Employee leave history
  - Team leave summary for managers
  - Organization-wide leave reports

#### 4. Multi-Tenant Architecture
- **Tenant Isolation**
  - Complete data isolation between tenants
  - Tenant-specific configurations
  - Tenant-specific branding (logo, colors)
  - Tenant-specific email templates

- **Tenant Management**
  - Tenant onboarding process
  - Tenant configuration portal
  - Tenant usage analytics
  - Tenant billing and subscription management

#### 5. User Management and Security
- **Authentication**
  - Email/password-based login
  - Multi-factor authentication (MFA) support
  - Password reset via email
  - Session management with timeout

- **Authorization**
  - Role-Based Access Control (RBAC)
  - Predefined roles: Super Admin, Tenant Admin, Manager, Employee
  - Custom role creation with granular permissions
  - Resource-level access control

- **Audit and Compliance**
  - Audit logs for all data modifications
  - User activity tracking
  - GDPR compliance (data export, right to be forgotten)
  - Data encryption at rest and in transit

#### 6. Web Application
- **Responsive Web Interface**
  - Desktop and tablet optimized
  - Mobile-responsive design
  - Modern, intuitive UI/UX
  - Dashboard with key metrics and pending actions

- **Features**
  - All employee, payroll, and leave management features
  - Admin portal for tenant configuration
  - Reports and analytics
  - Bulk operations support

#### 7. Mobile Applications (Android & iOS)
- **Core Features**
  - View employee profile
  - View salary slips (download PDF)
  - View leave balance
  - Apply for leave
  - View leave history
  - View organization holidays
  - Push notifications for leave approvals/rejections

- **Offline Mode (Limited)**
  - View cached employee profile
  - View cached salary slips (last 12 months)
  - View cached leave balance
  - Apply for leave (queued for sync)
  - View cached organization holidays

- **Native Features**
  - Biometric authentication (fingerprint/face ID)
  - Push notifications
  - Offline data sync
  - Camera integration for document upload

#### 8. API Layer
- **RESTful APIs**
  - Complete API coverage for all features
  - API documentation (Swagger/OpenAPI)
  - API versioning (v1, v2)
  - Rate limiting and throttling
  - OAuth2 authentication for third-party access

- **Webhooks**
  - Event notifications for integrations
  - Employee created/updated/deleted events
  - Leave approved/rejected events
  - Payroll processed events

#### 9. Licensing System
- **License Management**
  - Tenant license validation
  - User limit enforcement (soft limits)
  - Feature tier management (Basic, Professional, Enterprise)
  - License expiry notifications
  - Self-service license upgrade/downgrade
  - Grace period handling

- **Billing Integration**
  - Subscription management
  - Usage tracking
  - Invoice generation
  - Payment gateway integration

#### 10. Reporting and Analytics
- **Standard Reports**
  - Employee headcount reports
  - Payroll summary reports
  - Leave utilization reports
  - Department-wise analytics
  - Export to PDF/Excel

#### 11. Notifications
- **Email Notifications**
  - Leave application/approval/rejection
  - Salary slip availability
  - Password reset
  - Account activation
  - System announcements

- **Push Notifications (Mobile)**
  - Leave status updates
  - Salary slip availability
  - Important announcements

#### 12. DevOps and Infrastructure
- **CI/CD Pipeline**
  - Automated build and deployment
  - Automated testing (unit, integration, E2E)
  - Code quality checks
  - Security scanning

- **Monitoring and Logging**
  - Application performance monitoring
  - Error tracking and alerting
  - Centralized logging
  - Infrastructure monitoring

- **Backup and Disaster Recovery**
  - Automated daily backups
  - Cross-region replication
  - Disaster recovery procedures

---

### Out of Scope

#### 1. Recruitment and Hiring
- Job posting and applicant tracking
- Interview scheduling and feedback
- Offer letter generation
- Onboarding workflows (beyond adding employee)

#### 2. Performance Management
- Goal setting and tracking (OKRs, KPIs)
- Performance review cycles
- Performance improvement plans (PIPs)
- Rating and ranking systems

#### 3. Training and Development
- Training course catalog
- Training enrollment and tracking
- Certification management
- Learning Management System (LMS) integration
- Skill matrix and competency mapping

#### 4. Time and Attendance
- Clock in/clock out functionality
- Biometric device integration
- Shift scheduling
- Overtime tracking
- Timesheet management
- Attendance reports

#### 5. Benefits Administration
- Health insurance enrollment
- Retirement plan management
- Stock options/ESOP management
- Reimbursement claims (travel, medical)
- Expense management

#### 6. Tax Filing and Compliance
- Automatic tax calculation (country-specific)
- Tax form generation (W2, 1099, etc.)
- Tax filing with government portals
- Compliance reporting (EEO, OSHA)
- Statutory compliance automation

#### 7. Asset Management
- IT asset allocation and tracking
- Asset maintenance and depreciation
- Asset return workflow

#### 8. Project Management
- Project creation and tracking
- Task assignment and management
- Project timelines and milestones
- Resource allocation across projects

#### 9. Employee Self-Service Portal (Advanced)
- Internal job postings and applications
- Referral management
- Employee surveys and polls
- Employee directory with search

#### 10. Leave Management (Advanced)
- Compensatory off (comp-off) management
- Leave encashment
- Sabbatical leave
- Unpaid leave with policy enforcement
- Leave forecasting and planning

---

## Architecture Diagram

### High-Level System Architecture

The following diagram illustrates the complete system architecture for the multi-tenant HRMS solution, showcasing the client layer, API gateway, microservices, data layer, and supporting infrastructure.

[Click here to view the diagram](./images/system-architecture-diagram.png)

### Multi-Tenant Data Isolation Strategy
lick here to view the diagram](./images/tenant-isolation-approach.png)


### Data Flow Diagrams

#### Employee Management Flow

[Click here to view the diagram](./images/employee-management-workflow.png)

#### Payroll Processing Flow

[Click here to view the diagram](./images/payroll-management.png)

#### Leave Approval Flow

[Click here to view the diagram](./images/leave-management.png)

### Offline Sync Architecture (Mobile)

[Click here to view the diagram](./images/offline-sync-architecture.png)


## Deployment Diagram

### AWS Cloud Infrastructure Deployment

The following diagram illustrates the complete infrastructure deployment on AWS, including networking, compute, storage, and supporting services across multiple availability zones.

[Click here to view the diagram](./images/deployment-diagram.png)


### Disaster Recovery Architecture

[Click here to view the diagram](./images/disaster-recovery-diagram.png)


## CI/CD Architecture

### Complete CI/CD Pipeline

The following diagram illustrates the end-to-end CI/CD pipeline for backend services, web application, and mobile applications.

[Click here to view the diagram](./images/CI-CD-diagram.png)


### Deployment Strategies

#### Blue-Green Deployment

[Click here to view the diagram](./images/deployment-strategy.png)

## Final Technology Stack Summary

### Complete Technology Stack Overview

Based on the weighted Decision Architecture Records (DARs), here is the complete technology stack for the NewFinTech HRMS system:

[Click here to view the diagram](./images/technology-stack-diagram.png)

#### 8. External Services

| Component | Service | Purpose |
|-----------|---------|---------|
| **Email** | AWS SES | Transactional emails |
| **SMS** | AWS SNS or Twilio | SMS notifications |
| **Push Notifications** | Firebase Cloud Messaging (FCM) | Mobile push notifications |
| **Apple Push** | Apple Push Notification Service (APNs) | iOS notifications |
| **Payment Gateway** | Stripe or Razorpay | Subscription billing |

### Technology Stack Comparison Matrix

| Aspect | Choice | Alternative Considered | Why Chosen |
|--------|--------|----------------------|------------|
| **Backend** | Node.js/Express | Spring Boot, .NET Core | Fastest development, best team fit |
| **Frontend** | React | Angular, Vue.js | Largest ecosystem, best community |
| **Mobile** | Flutter | React Native, Native | Best performance/code reuse balance |
| **Database** | PostgreSQL | MySQL, MongoDB | Best multi-tenancy support |
| **Cloud** | AWS | Azure, GCP | Most comprehensive services |
| **Cache** | Redis | Memcached | Rich features, persistence |
| **Message Queue** | RabbitMQ | Kafka, SQS | Best fit for job queue use case |
| **API Gateway** | Kong | AWS API Gateway | Most flexible, cost-effective |
| **Auth** | JWT | OAuth2 only, SAML | Simplest, most scalable |
| **Containers** | ECS Fargate | Kubernetes, Swarm | Easiest to manage, serverless |

### Conclusion

This architecture document provides a solid foundation for building a scalable, secure, and maintainable multi-tenant HRMS system. The design decisions are backed by comprehensive analysis and weighted scoring, ensuring the best technology choices for NewFinTech's requirements.

---


