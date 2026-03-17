# Implementation Plan: Cloud Cost Optimizer

## Overview

This implementation plan breaks down the Cloud Cost Optimization Platform into discrete coding tasks. The system will be built using TypeScript for all services (API, Workers, Frontend) with PostgreSQL, TimescaleDB, and Redis as data stores. The implementation follows an incremental approach where each task builds on previous work, with property-based tests integrated throughout to catch errors early.

The architecture consists of:
- API Service (NestJS/Express with TypeScript)
- Worker Services (BullMQ with TypeScript)
- Terraform Analyzer (TypeScript with HCL parser)
- Frontend (Next.js with TypeScript)
- Databases (PostgreSQL, TimescaleDB, Redis)

## Tasks

- [ ] 1. Project Setup and Infrastructure
  - Initialize monorepo structure with TypeScript, pnpm workspaces
  - Set up Docker Compose for PostgreSQL, TimescaleDB, Redis
  - Configure TypeScript with strict mode and path aliases
  - Set up ESLint, Prettier, and Husky for code quality
  - Create shared types package for cross-service type definitions
  - _Requirements: All (foundational)_

- [ ] 2. Database Schema and Migrations
  - [ ] 2.1 Create PostgreSQL schema with all tables
    - Implement tenants, users, cloud_credentials tables
    - Implement resources, recommendations, remediations tables
    - Implement audit_logs with append-only rules
    - Implement maintenance_windows, blast_radius_config tables
    - Implement resource_correlations, terraform_analyses tables
    - Implement anomaly_alerts table
    - Add all indexes and constraints
    - _Requirements: 1.3, 3.6, 5.1, 6.1, 8.1, 10.1, 11.1, 16.2, 18.1, 19.1_
  
  - [ ]* 2.2 Write property test for database schema
    - **Property 13: Audit Log Immutability**
    - **Validates: Requirements 6.3**
  
  - [ ] 2.3 Create TimescaleDB schema for cost data
    - Implement cost_data hypertable with hourly chunks
    - Implement cost_forecasts hypertable
    - Create continuous aggregates for daily and monthly costs
    - Add compression policy for data older than 90 days
    - _Requirements: 1.4, 10.1, 14.3_
  
  - [ ] 2.4 Set up database migration system
    - Use node-pg-migrate or TypeORM migrations
    - Create initial migration files
    - Add migration scripts to package.json
    - _Requirements: All (foundational)_

- [ ] 3. Shared Types and Utilities
  - [ ] 3.1 Define core TypeScript types and interfaces
    - Create types for Tenant, User, Resource, Recommendation
    - Create types for Remediation, AuditLog, CostData
    - Create types for API requests and responses
    - Create enums for status values, roles, cloud providers
    - _Requirements: All (foundational)_
  
  - [ ] 3.2 Implement tenant isolation utilities
    - Create tenant context middleware
    - Create tenant-scoped database query helpers
    - _Requirements: 8.1, 8.2, 8.3, 8.4_
  
  - [ ]* 3.3 Write property test for tenant isolation
    - **Property 15: Tenant Isolation**
    - **Validates: Requirements 8.2, 8.3, 8.4**
  
  - [ ] 3.4 Implement audit logging utility
    - Create audit log writer function
    - Ensure all required fields are captured
    - _Requirements: 6.1, 6.2_
  
  - [ ]* 3.5 Write property test for audit logging
    - **Property 12: Audit Log Completeness**
    - **Validates: Requirements 6.1, 6.2**


- [ ] 4. Authentication and Authorization
  - [ ] 4.1 Implement JWT authentication
    - Create JWT token generation and validation
    - Implement login endpoint with password hashing (bcrypt)
    - Implement token refresh endpoint
    - _Requirements: 20.1_
  
  - [ ]* 4.2 Write property test for authentication
    - **Property 31: API Authentication Enforcement**
    - **Validates: Requirements 20.1**
  
  - [ ] 4.3 Implement RBAC middleware
    - Create role-based permission checks
    - Implement middleware for protecting routes by role
    - _Requirements: 9.1, 9.2, 9.3, 9.4, 9.5_
  
  - [ ]* 4.4 Write property test for RBAC
    - **Property 16: Unauthorized Action Rejection**
    - **Validates: Requirements 9.5**
  
  - [ ] 4.5 Implement user management endpoints
    - Create user CRUD operations
    - Implement user deactivation with immediate access revocation
    - _Requirements: 9.1, 9.6, 9.7_

- [ ] 5. API Service Core
  - [ ] 5.1 Set up NestJS application structure
    - Initialize NestJS project with modules for auth, resources, recommendations, costs
    - Configure dependency injection and module imports
    - Set up global exception filters and interceptors
    - _Requirements: 20.1, 20.2, 20.3_
  
  - [ ] 5.2 Implement error handling and validation
    - Create global exception filter with structured error responses
    - Implement request validation using class-validator
    - Add error code constants and error response builder
    - _Requirements: 20.3, 20.6_
  
  - [ ]* 5.3 Write property test for error handling
    - **Property 32: Input Validation Error Codes**
    - **Validates: Requirements 20.3, 20.6**
  
  - [ ] 5.4 Implement Redis job queue integration
    - Set up BullMQ for job queuing
    - Create job queue service with queue creation and job submission
    - Implement job status tracking in Redis
    - _Requirements: 12.1, 12.4, 12.5_
  
  - [ ]* 5.5 Write property test for job queuing
    - **Property 20: Asynchronous Job Queuing**
    - **Validates: Requirements 12.1**
  
  - [ ]* 5.6 Write property test for job status
    - **Property 22: Job Status Validity**
    - **Validates: Requirements 12.4**

- [ ] 6. Resource Inventory API
  - [ ] 6.1 Implement resource endpoints
    - Create GET /api/v1/resources with pagination, filtering, search
    - Create GET /api/v1/resources/:id endpoint
    - Enforce tenant-scoped queries
    - _Requirements: 1.3, 8.2_
  
  - [ ] 6.2 Implement resource query builders
    - Create query builder for filtering by cloud provider, type, tags
    - Implement full-text search on resource names
    - _Requirements: 1.3, 13.1_

- [ ] 7. Discovery Worker
  - [ ] 7.1 Set up worker infrastructure
    - Create BullMQ worker process
    - Implement job retry logic with exponential backoff
    - Set up worker health monitoring
    - _Requirements: 12.2, 12.3, 12.6_
  
  - [ ]* 7.2 Write property test for job retry
    - **Property 21: Job Retry with Exponential Backoff**
    - **Validates: Requirements 12.3**
  
  - [ ] 7.3 Implement GCP resource discovery
    - Set up Workload Identity Federation authentication
    - Implement discovery for Compute Engine, Cloud Storage, BigQuery, Cloud SQL
    - Store discovered resources in PostgreSQL with tenant tagging
    - Store cost data in TimescaleDB with hourly granularity
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 1.7_
  
  - [ ]* 7.4 Write property test for resource storage
    - **Property 1: Resource Discovery Storage Completeness**
    - **Validates: Requirements 1.3, 1.4, 1.7**
  
  - [ ] 7.5 Implement AWS resource discovery (optional for MVP)
    - Set up IAM role authentication with temporary credentials
    - Implement discovery for EC2, S3, RDS, Lambda
    - _Requirements: 1.5_
  
  - [ ] 7.6 Implement Azure resource discovery (optional for MVP)
    - Set up Managed Identity authentication
    - Implement discovery for VMs, Storage, SQL Database
    - _Requirements: 1.6_

- [ ] 8. Checkpoint - Ensure discovery worker tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 9. Cost Data Management
  - [ ] 9.1 Implement cost data storage
    - Create service for writing cost data to TimescaleDB
    - Ensure hourly granularity and required fields (tags, provider, category)
    - _Requirements: 10.1, 10.3_
  
  - [ ]* 9.2 Write property test for cost data storage
    - **Property 17: Cost Data Field Completeness**
    - **Validates: Requirements 10.1, 10.3**
  
  - [ ] 9.3 Implement cost query API
    - Create GET /api/v1/costs/timeseries endpoint
    - Support date range filters and aggregation by day/week/month
    - Implement grouping by tags, provider, service category
    - _Requirements: 10.2, 10.3_
  
  - [ ] 9.4 Implement cost anomaly detection
    - Create service for comparing costs against historical baselines
    - Generate anomaly alerts when thresholds are exceeded
    - Identify specific resources causing spikes
    - _Requirements: 10.5, 16.1, 16.2, 16.3_
  
  - [ ]* 9.5 Write property test for anomaly detection
    - **Property 26: Anomaly Alert Generation**
    - **Validates: Requirements 16.2**
  
  - [ ]* 9.6 Write property test for anomaly resource identification
    - **Property 27: Anomaly Alert Resource Identification**
    - **Validates: Requirements 16.3**

- [ ] 10. PSO Worker
  - [ ] 10.1 Implement PSO algorithm core
    - Create particle swarm optimization algorithm using mathjs or similar
    - Implement fitness function for cost optimization
    - Configure particle count, iterations, and convergence criteria
    - _Requirements: 2.1_
  
  - [ ] 10.2 Implement recommendation generation
    - Convert PSO results into recommendations
    - Calculate confidence scores based on PSO convergence and historical feedback
    - Calculate estimated monthly savings
    - Generate detailed explanations and Terraform snippets
    - _Requirements: 2.2, 2.3, 3.1, 3.2, 3.3_
  
  - [ ]* 10.3 Write property test for recommendation fields
    - **Property 2: Recommendation Field Completeness**
    - **Validates: Requirements 2.2, 2.3, 3.1**
  
  - [ ] 10.4 Implement feedback incorporation
    - Create service for updating PSO parameters based on approval/rejection feedback
    - Adjust confidence scoring based on historical accuracy
    - _Requirements: 2.5_
  
  - [ ] 10.5 Implement multi-cloud resource correlation
    - Create correlation algorithm for identifying relationships across cloud providers
    - Use tags, naming patterns, and network topology for correlation
    - Store correlations with confidence scores
    - _Requirements: 2.6, 17.1, 17.2, 17.6_
  
  - [ ]* 10.6 Write property test for resource correlation
    - **Property 28: Resource Correlation Validity**
    - **Validates: Requirements 17.1**
  
  - [ ] 10.7 Implement scheduled PSO jobs
    - Create cron job for nightly PSO execution
    - Implement job scheduling using BullMQ repeatable jobs
    - _Requirements: 2.4_

- [ ] 11. Recommendation API
  - [ ] 11.1 Implement recommendation endpoints
    - Create GET /api/v1/recommendations with filtering and sorting
    - Create POST /api/v1/recommendations/:id/approve endpoint
    - Create POST /api/v1/recommendations/:id/reject endpoint
    - _Requirements: 3.4, 3.5, 3.6, 3.7_
  
  - [ ]* 11.2 Write property test for approval records
    - **Property 3: Approval Record Creation**
    - **Validates: Requirements 3.6**
  
  - [ ]* 11.3 Write property test for rejection reasons
    - **Property 4: Rejection Reason Storage**
    - **Validates: Requirements 3.7**
  
  - [ ] 11.4 Implement approval workflow
    - Create approval notification service (email, Slack)
    - Implement priority-based approval queue
    - _Requirements: 4.2, 4.6, 4.7_
  
  - [ ]* 11.5 Write property test for approval requirement
    - **Property 5: Approval Requirement for Remediation**
    - **Validates: Requirements 4.1**
  
  - [ ]* 11.6 Write property test for remediation queuing
    - **Property 6: Approved Remediation Queuing**
    - **Validates: Requirements 4.4**

- [ ] 12. Checkpoint - Ensure PSO and recommendation tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 13. Remediation Worker
  - [ ] 13.1 Implement blast radius controls
    - Create service for tracking changes per hour
    - Implement blast radius limit enforcement
    - Add pause/resume logic when limit is reached
    - _Requirements: 5.4, 19.1, 19.2_
  
  - [ ]* 13.2 Write property test for blast radius
    - **Property 10: Blast Radius Enforcement**
    - **Validates: Requirements 5.4, 19.1, 19.2**
  
  - [ ] 13.3 Implement protection tag filtering
    - Create service for checking resource protection tags
    - Implement production opt-in tag checking
    - Skip resources with protection tags
    - _Requirements: 5.5, 5.6, 19.3, 19.4_
  
  - [ ]* 13.4 Write property test for protection tags
    - **Property 30: Protection Tag Exclusion**
    - **Validates: Requirements 19.3**
  
  - [ ] 13.5 Implement snapshot and rollback
    - Create service for capturing resource state snapshots
    - Implement rollback logic using snapshots
    - Store snapshots in remediation records
    - _Requirements: 5.1, 5.2_
  
  - [ ]* 13.6 Write property test for snapshot creation
    - **Property 7: Snapshot Creation Before Changes**
    - **Validates: Requirements 5.1**
  
  - [ ]* 13.7 Write property test for rollback
    - **Property 8: Automatic Rollback on Failure**
    - **Validates: Requirements 5.2**
  
  - [ ] 13.8 Implement remediation execution
    - Create service for applying infrastructure changes via cloud provider APIs
    - Implement idempotent change application
    - Add verification step after changes
    - _Requirements: 5.3, 5.7_
  
  - [ ]* 13.9 Write property test for idempotence
    - **Property 9: Remediation Idempotence**
    - **Validates: Requirements 5.3**
  
  - [ ]* 13.10 Write property test for verification
    - **Property 11: Remediation Verification**
    - **Validates: Requirements 5.7**
  
  - [ ] 13.11 Implement maintenance window checking
    - Create service for checking if current time is within maintenance window
    - Delay remediation execution until window opens
    - _Requirements: 4.5, 18.1, 18.2, 18.3, 18.4_
  
  - [ ]* 13.12 Write property test for maintenance windows
    - **Property 29: Maintenance Window Enforcement**
    - **Validates: Requirements 18.3**
  
  - [ ] 13.13 Implement multi-cloud remediation coordination
    - Create service for coordinating changes across cloud providers
    - Ensure consistency when remediating correlated resources
    - _Requirements: 17.4_

- [ ] 14. Terraform Analyzer
  - [ ] 14.1 Set up Terraform parser
    - Integrate HCL parser library (hcl2-parser or similar)
    - Create service for parsing Terraform files to AST
    - _Requirements: 11.1_
  
  - [ ]* 14.2 Write property test for Terraform parsing
    - **Property 18: Terraform Parsing Round-Trip**
    - **Validates: Requirements 11.1**
  
  - [ ] 14.3 Implement cost optimization analysis
    - Create rules for identifying cost optimization opportunities
    - Check for oversized instances, unused resources, inefficient configurations
    - _Requirements: 11.2_
  
  - [ ] 14.4 Implement security analysis
    - Create rules for identifying security misconfigurations
    - Check for public access, missing encryption, weak IAM policies
    - _Requirements: 11.3_
  
  - [ ] 14.5 Implement module analysis
    - Create service for recursively analyzing Terraform modules
    - Resolve module dependencies and analyze them
    - _Requirements: 11.5_
  
  - [ ] 14.6 Implement state comparison
    - Create service for comparing Terraform plan against current state
    - Identify drift and planned changes
    - _Requirements: 11.6_
  
  - [ ] 14.7 Implement Terraform API endpoints
    - Create POST /api/v1/terraform/analyze endpoint
    - Create GET /api/v1/terraform/results/:jobId endpoint
    - _Requirements: 11.4, 11.7_
  
  - [ ]* 14.8 Write property test for analysis severity levels
    - **Property 19: Terraform Analysis Severity Levels**
    - **Validates: Requirements 11.4**

- [ ] 15. Cost Forecasting
  - [ ] 15.1 Implement time-series forecasting
    - Integrate forecasting library (e.g., Prophet.js or custom ARIMA)
    - Create service for generating 90-day forecasts
    - Include confidence intervals (80% and 95%)
    - _Requirements: 14.1, 14.2, 14.3_
  
  - [ ]* 15.2 Write property test for forecast confidence intervals
    - **Property 24: Forecast Confidence Intervals**
    - **Validates: Requirements 14.3**
  
  - [ ] 15.3 Implement forecast API
    - Create GET /api/v1/costs/forecast endpoint
    - Support configurable forecast periods and confidence levels
    - _Requirements: 14.4_
  
  - [ ] 15.4 Implement forecast variance detection
    - Create service for comparing actual costs against forecasts
    - Flag significant variances for investigation
    - _Requirements: 14.5_
  
  - [ ] 15.5 Implement seasonality detection
    - Add seasonality components to forecasting model
    - Detect and account for weekly/monthly patterns
    - _Requirements: 14.6_

- [ ] 16. Checkpoint - Ensure remediation and forecasting tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 17. Natural Language Query Interface
  - [ ] 17.1 Integrate LLM for query parsing
    - Set up OpenAI API or similar LLM integration
    - Create service for parsing natural language queries
    - _Requirements: 15.1_
  
  - [ ] 17.2 Implement query translation
    - Create service for translating parsed intent into database queries
    - Support queries for costs, resources, recommendations
    - _Requirements: 15.2_
  
  - [ ]* 17.3 Write property test for query translation
    - **Property 25: Natural Language Query Translation**
    - **Validates: Requirements 15.2**
  
  - [ ] 17.4 Implement response formatting
    - Create service for formatting query results as natural language
    - Generate visualizations based on query type
    - _Requirements: 15.3, 15.6_
  
  - [ ] 17.5 Implement disambiguation and clarification
    - Add logic for handling ambiguous queries
    - Generate clarifying questions when needed
    - _Requirements: 15.4, 15.5_
  
  - [ ] 17.6 Implement NL query API
    - Create POST /api/v1/query/nl endpoint
    - Return formatted responses with optional visualizations
    - _Requirements: 15.7_

- [ ] 18. Audit Log API
  - [ ] 18.1 Implement audit log endpoints
    - Create GET /api/v1/audit endpoint with filtering and pagination
    - Enforce tenant-scoped queries with row-level security
    - Support filtering by date range, actor, action type
    - _Requirements: 6.4, 13.4_
  
  - [ ] 18.2 Implement audit log export
    - Create export functionality for compliance reporting
    - Support CSV and JSON formats
    - _Requirements: 6.7_

- [ ] 19. Frontend - Project Setup
  - [ ] 19.1 Initialize Next.js project
    - Set up Next.js 14+ with App Router
    - Configure TypeScript, TailwindCSS, and ESLint
    - Set up API client with axios or fetch
    - _Requirements: 13.1_
  
  - [ ] 19.2 Implement authentication flow
    - Create login page with JWT token storage
    - Implement token refresh logic
    - Create protected route wrapper
    - _Requirements: 20.1_
  
  - [ ] 19.3 Implement layout and navigation
    - Create main layout with sidebar navigation
    - Implement role-based menu item visibility
    - _Requirements: 13.7_

- [ ] 20. Frontend - Resource Inventory
  - [ ] 20.1 Create resource inventory page
    - Implement resource list with pagination
    - Add search and filter controls (provider, type, tags)
    - Display resource metadata in cards or table
    - _Requirements: 13.1_
  
  - [ ] 20.2 Create resource detail page
    - Display full resource metadata
    - Show associated cost data
    - Show related recommendations
    - _Requirements: 13.1_

- [ ] 21. Frontend - Recommendations
  - [ ] 21.1 Create recommendations page
    - Display recommendations sorted by savings
    - Show confidence scores, savings estimates, explanations
    - Implement filtering by status and minimum savings
    - _Requirements: 13.2, 3.4_
  
  - [ ]* 21.2 Write property test for recommendation display
    - **Property 23: Recommendation Display Completeness**
    - **Validates: Requirements 13.2**
  
  - [ ] 21.3 Implement approval workflow UI
    - Create one-click approve/reject buttons
    - Add modal for approval notes and rejection reasons
    - Show Terraform snippets and CLI commands
    - _Requirements: 13.3, 3.5_
  
  - [ ] 21.4 Implement real-time updates
    - Set up WebSocket connection or polling for status updates
    - Update recommendation status in real-time
    - _Requirements: 13.6_

- [ ] 22. Frontend - Cost Dashboard
  - [ ] 22.1 Create cost dashboard page
    - Implement time-series chart using Chart.js or Recharts
    - Add date range picker and granularity selector
    - Support zoom and pan on charts
    - _Requirements: 13.5, 10.4_
  
  - [ ] 22.2 Implement cost forecasting visualization
    - Overlay forecast data on historical charts
    - Display confidence intervals as shaded regions
    - _Requirements: 14.4_
  
  - [ ] 22.3 Implement anomaly alerts display
    - Show anomaly alerts with severity indicators
    - Allow acknowledgment and resolution of alerts
    - _Requirements: 16.4, 16.5, 16.6, 16.7_

- [ ] 23. Frontend - Audit Logs
  - [ ] 23.1 Create audit log page
    - Display filterable, paginated log entries
    - Implement search by actor, action type, date range
    - Show before/after state diffs
    - _Requirements: 13.4_
  
  - [ ] 23.2 Implement audit log export
    - Add export button for CSV/JSON download
    - _Requirements: 6.7_

- [ ] 24. Frontend - Terraform Analysis
  - [ ] 24.1 Create Terraform analysis page
    - Implement file upload component
    - Display analysis results with file/line references
    - Show severity levels and recommendations
    - _Requirements: 11.7_
  
  - [ ] 24.2 Implement code highlighting
    - Add syntax highlighting for Terraform code
    - Highlight lines with issues
    - _Requirements: 11.7_

- [ ] 25. Frontend - Natural Language Query
  - [ ] 25.1 Create query interface page
    - Implement text input for natural language queries
    - Display formatted responses
    - Show generated visualizations
    - _Requirements: 15.3, 15.6_
  
  - [ ] 25.2 Implement query history
    - Store and display previous queries
    - Allow re-running past queries
    - _Requirements: 15.3_

- [ ] 26. Checkpoint - Ensure frontend tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 27. Security Hardening
  - [ ] 27.1 Implement credential scanning
    - Create service for scanning database and config files for credential patterns
    - Run as part of CI/CD pipeline
    - _Requirements: 7.2_
  
  - [ ]* 27.2 Write property test for credential prohibition
    - **Property 14: Credential Storage Prohibition**
    - **Validates: Requirements 7.2**
  
  - [ ] 27.3 Implement rate limiting
    - Add rate limiting middleware to API endpoints
    - Use Redis for distributed rate limiting
    - _Requirements: 20.1_
  
  - [ ] 27.4 Implement request logging
    - Log all API requests with sanitized parameters
    - Include request ID for tracing
    - _Requirements: 20.7_

- [ ] 28. Integration and End-to-End Testing
  - [ ]* 28.1 Write integration tests for discovery flow
    - Test Discovery Worker → PostgreSQL → TimescaleDB
    - _Requirements: 1.3, 1.4_
  
  - [ ]* 28.2 Write integration tests for recommendation flow
    - Test PSO Worker → Recommendations → Approval → Remediation
    - _Requirements: 2.1, 3.6, 4.4_
  
  - [ ]* 28.3 Write integration tests for API → Worker flow
    - Test API → Redis → Worker → Result Storage
    - _Requirements: 12.1, 12.2_
  
  - [ ]* 28.4 Write end-to-end tests for frontend flows
    - Test login → view resources → approve recommendation
    - _Requirements: 13.1, 13.2, 13.3_

- [ ] 29. Documentation and Deployment
  - [ ] 29.1 Write API documentation
    - Generate OpenAPI/Swagger documentation
    - Document all endpoints with examples
    - _Requirements: All_
  
  - [ ] 29.2 Write deployment documentation
    - Document Docker Compose setup for local development
    - Document Kubernetes deployment for production
    - Document environment variables and configuration
    - _Requirements: All_
  
  - [ ] 29.3 Create deployment scripts
    - Create Docker images for all services
    - Create Kubernetes manifests or Helm charts
    - Set up CI/CD pipeline
    - _Requirements: All_

- [ ] 30. Final Checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation throughout development
- Property tests validate universal correctness properties using fast-check library
- Unit tests validate specific examples, edge cases, and integration points
- The implementation uses TypeScript for all services as specified by the user
- BullMQ is used for job queuing instead of Celery (Python equivalent)
- NestJS is used for the API service (TypeScript equivalent of FastAPI)
