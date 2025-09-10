# CLAUDE.md - WhizoAI Development Guide

**Last Updated**: January 2025 - Complete Frontend Authentication System & Dashboard Implementation with Breadcrumb Navigation

This file provides comprehensive guidance for Claude Code when working with the WhizoAI codebase. It includes system architecture, APIs, database schema, authentication flows, development workflows, and specific operational procedures.

## 🚨 CRITICAL OPERATIONAL REQUIREMENTS

### **MANDATORY: Linear Issue Management**
- **ALWAYS create or update Linear issues** when working on any development task
- **EVERY MCP tool usage** must be tracked in Linear with appropriate status updates
- **Update issue status** as work progresses: `pending` → `in_progress` → `in_review` → `completed`
- **Link related issues** when working on dependent features
- **Tag issues appropriately** with labels like `bug`, `enhancement`, `documentation`, `api`

### **MANDATORY: Documentation Updates**
- **ALWAYS update Mintlify docs** when creating new API endpoints
- **Update whizoai-docs** repository for any new features, endpoints, or configuration changes
- **Maintain API documentation** in sync with actual implementation
- **Update CLAUDE.md** whenever system architecture changes

### **Development Workflow Requirements**
1. **Before starting any task**: Create or update Linear issue
2. **During development**: Update issue status and add comments on progress
3. **After API changes**: Update Mintlify documentation
4. **After completion**: Mark issue as completed and update this CLAUDE.md if needed

## 🚀 Quick Start Commands

### Backend (whizoai-backend)
```bash
# Development
npm run dev                    # Start development server with hot reload (port 8080)
npm run dev:legacy            # Start legacy v2 server for fallback testing
npm run build                 # Build TypeScript to JavaScript
npm run start                 # Start production server
npm run typecheck             # Run TypeScript type checking
npm run lint                  # Run ESLint for code quality

# Database Operations
npm run db:migrate            # Run database migrations
npm run db:seed               # Seed database with initial data
npm run db:setup              # Run both migrations and seeding

# Health & Monitoring
npm run health                # Check server health (curl localhost:8080/health)
npm run logs                  # View PM2 logs
npm run deploy               # Build and start production server

# Testing & Quality
npm test                     # Run test suite
npm run test:coverage        # Run tests with coverage report
```

### Frontend (whizoai-web)
```bash
# Development
npm run dev                  # Start Next.js development server with Turbo (port 3000)
npm run build               # Build Next.js application with Turbo
npm run start               # Start production Next.js server
npm run lint                # Run ESLint for code quality
npm run typecheck           # Run TypeScript type checking
```

### **Development Health Checks**
Always verify these before starting work:
- Backend: `curl http://localhost:8080/health`
- Frontend: `http://localhost:3000` accessible
- Database: Supabase connection working
- Environment variables properly configured

## 🏗️ System Architecture Overview

### **Technology Stack**
- **Backend**: Express.js + TypeScript + Node.js 18+
- **Frontend**: Next.js 15 + App Router + TypeScript + TailwindCSS + Radix UI
- **Database**: Supabase PostgreSQL with Row Level Security (RLS)
- **Authentication**: Dual system (Backend JWT + Supabase Auth)
- **Scraping**: Multi-engine system (Playwright, Puppeteer, Lightweight)
- **Caching**: Redis for performance and rate limiting
- **Storage**: AWS S3 for screenshots, PDFs, and exports
- **Real-time**: WebSocket for live job updates
- **Payments**: Stripe for billing and subscriptions
- **Documentation**: Mintlify for API docs
- **Project Management**: Linear for issue tracking

### **Core Design Principles**
1. **Security-First**: Input validation, rate limiting, comprehensive audit logging
2. **Performance-Optimized**: Redis caching, connection pooling, lazy loading
3. **Scalable Architecture**: Microservices patterns, horizontal scaling ready
4. **Developer Experience**: TypeScript strict mode, comprehensive error handling
5. **User-Centric**: Real-time feedback, intuitive UI, accessibility compliant

## 📁 Detailed Project Structure

### **Backend Structure (`whizoai-backend/`)**
```
src/
├── lib/                           # Core library modules
│   ├── database/                  # Supabase client with service/user separation
│   │   ├── index.ts              # Main database client with connection pooling
│   │   ├── types.ts              # Database type definitions
│   │   └── migrations/           # Database migration files
│   ├── scraping/                 # Multi-engine scraping system
│   │   ├── unified-engine.ts     # Main scraping orchestrator
│   │   ├── engines/              # Individual engine implementations
│   │   │   ├── playwright.ts     # Browser automation engine
│   │   │   ├── puppeteer.ts      # Alternative browser engine
│   │   │   └── lightweight.ts    # Fast HTTP-based scraping
│   │   ├── processors/           # Content processing modules
│   │   └── utils/               # Scraping utility functions
│   ├── integrations/             # External service integrations
│   │   ├── webshare-proxy.ts    # Proxy service for scraping reliability
│   │   ├── stripe.ts            # Payment processing integration
│   │   ├── aws-s3.ts            # File storage service
│   │   └── email-service.ts     # Email notification service
│   ├── redis/                   # Caching and session management
│   │   ├── client.ts            # Redis client configuration
│   │   ├── cache.ts             # Caching utilities
│   │   └── rate-limiter.ts      # Rate limiting implementation
│   ├── security/                # Security utilities and middleware
│   │   ├── validation.ts        # Input validation schemas (Zod)
│   │   ├── encryption.ts        # Data encryption utilities
│   │   ├── auth-helpers.ts      # JWT and password utilities
│   │   └── audit-logger.ts      # Security event logging
│   ├── storage/                 # File storage abstraction
│   │   ├── s3-client.ts         # AWS S3 operations
│   │   ├── file-processor.ts    # File processing utilities
│   │   └── upload-handler.ts    # Upload management
│   ├── utils/                   # Utility functions
│   │   ├── logger.ts            # Winston logging configuration
│   │   ├── helpers.ts           # General utility functions
│   │   ├── constants.ts         # Application constants
│   │   └── types.ts             # Shared type definitions
│   └── websocket/               # Real-time communication
│       ├── server.ts            # WebSocket server setup
│       ├── handlers.ts          # Event handlers
│       └── types.ts             # WebSocket type definitions
├── middleware/                  # Express middleware stack
│   ├── auth.ts                  # JWT authentication middleware
│   ├── security.ts              # Security headers, rate limiting
│   ├── validation.ts            # Request validation middleware
│   ├── error-handler.ts         # Global error handling
│   ├── cors.ts                  # CORS configuration
│   └── logging.ts               # Request logging middleware
├── routes/                      # API route handlers (versioned)
│   └── v1/                      # Version 1 API routes
│       ├── auth.ts              # Authentication endpoints
│       ├── users.ts             # User management endpoints
│       ├── scrape.ts            # Core scraping endpoints
│       ├── jobs.ts              # Job management and monitoring
│       ├── api-keys.ts          # API key management
│       ├── billing.ts           # Stripe billing integration
│       ├── analytics.ts         # Usage analytics and reporting
│       ├── files.ts             # File management endpoints
│       ├── admin/               # Admin-only endpoints
│       │   ├── users.ts         # User management
│       │   ├── system.ts        # System monitoring
│       │   ├── security.ts      # Security management
│       │   └── analytics.ts     # System-wide analytics
│       └── webhooks/            # Webhook endpoints
│           ├── stripe.ts        # Stripe webhook handlers
│           └── system.ts        # System webhook handlers
├── scripts/                     # Utility scripts
│   ├── migrate.ts              # Database migration runner
│   ├── seed.ts                 # Database seeding
│   ├── backup.ts               # Database backup utilities
│   └── cleanup.ts              # Cleanup and maintenance
├── tests/                      # Test suites
│   ├── unit/                   # Unit tests
│   ├── integration/            # Integration tests
│   └── e2e/                    # End-to-end tests
├── minimal-server.ts           # Main production server entry point
├── v2-server.ts               # Legacy server (backward compatibility)
└── config/                    # Configuration files
    ├── env.ts                 # Environment variable validation
    ├── database.ts            # Database configuration
    └── redis.ts               # Redis configuration
```

### **Frontend Structure (`whizoai-web/`)**
```
src/
├── app/                           # Next.js 15 App Router
│   ├── (auth)/                    # Auth route group
│   │   ├── login/                 # Login page
│   │   ├── signup/                # Registration page
│   │   ├── reset-password/        # Password reset flow
│   │   └── verify-email/          # Email verification
│   ├── app/                       # Main application (protected)
│   │   ├── (dashboard)/           # Dashboard route group
│   │   │   ├── page.tsx          # Main dashboard overview
│   │   │   ├── playground/        # API testing playground
│   │   │   ├── batch/            # Batch scraping interface
│   │   │   ├── jobs/             # Job monitoring and management
│   │   │   ├── analytics/        # Usage analytics dashboard
│   │   │   ├── api-keys/         # API key management
│   │   │   ├── billing/          # Billing and subscription
│   │   │   └── settings/         # Account settings
│   │   └── layout.tsx            # App layout with authentication
│   ├── api/                      # API routes (if needed for frontend)
│   ├── globals.css               # Global styles and CSS variables
│   ├── layout.tsx                # Root layout with providers
│   └── page.tsx                  # Landing page
├── components/                    # React components
│   ├── ui/                       # Reusable UI components
│   │   ├── whizo-card.tsx        # Custom WhizoAI branded cards
│   │   ├── button.tsx            # Button component variants
│   │   ├── input.tsx             # Input components
│   │   ├── badge.tsx             # Status badges and indicators
│   │   ├── avatar.tsx            # User avatar components
│   │   ├── dialog.tsx            # Modal and dialog components
│   │   ├── dropdown-menu.tsx     # Dropdown menu components
│   │   ├── sheet.tsx             # Side sheet components
│   │   ├── table.tsx             # Data table components
│   │   ├── toast.tsx             # Toast notification components
│   │   ├── breadcrumb.tsx        # Breadcrumb navigation components
│   │   └── ...                   # Other Radix UI components
│   ├── auth/                     # Authentication-specific components
│   │   ├── email-verification-banner.tsx
│   │   ├── login-form.tsx
│   │   └── signup-form.tsx
│   ├── dashboard/                # Dashboard-specific components
│   │   ├── stats-cards.tsx
│   │   ├── job-table.tsx
│   │   ├── analytics-charts.tsx
│   │   └── quick-actions.tsx
│   ├── scraping/                 # Scraping-related components
│   │   ├── url-input.tsx
│   │   ├── options-panel.tsx
│   │   ├── results-viewer.tsx
│   │   └── batch-upload.tsx
│   └── layout/                   # Layout components
│       ├── dashboard-layout.tsx  # Main dashboard navigation
│       ├── header.tsx           # Application header
│       ├── sidebar.tsx          # Navigation sidebar
│       └── footer.tsx           # Application footer
├── lib/                          # Core library modules
│   ├── auth/                     # Authentication utilities
│   │   ├── context.tsx          # Auth context provider
│   │   ├── hooks.ts             # Authentication hooks
│   │   └── types.ts             # Auth-related types
│   ├── api/                      # API client and services
│   │   ├── client.ts            # HTTP client with interceptors
│   │   ├── services.ts          # API service functions
│   │   ├── websocket.ts         # WebSocket client
│   │   └── types.ts             # API response types
│   ├── billing/                  # Billing context and utilities
│   │   ├── context.tsx          # Billing context provider
│   │   ├── stripe.ts            # Stripe client utilities
│   │   └── plans.ts             # Plan configurations
│   ├── supabase/                # Supabase client configuration
│   │   ├── client.ts            # Supabase client setup
│   │   ├── auth.ts              # Supabase auth helpers
│   │   └── types.ts             # Supabase type definitions
│   ├── config/                  # Configuration utilities
│   │   ├── env.ts               # Environment validation
│   │   └── constants.ts         # Application constants
│   ├── utils/                   # Utility functions
│   │   ├── toast.ts             # Enhanced toast utilities
│   │   ├── format.ts            # Data formatting utilities
│   │   ├── validation.ts        # Client-side validation
│   │   └── helpers.ts           # General helper functions
│   └── types/                   # TypeScript type definitions
│       ├── api.ts               # API response types
│       ├── billing.ts           # Billing-related types
│       └── global.ts            # Global type definitions
├── styles/                      # Additional styling
└── public/                      # Static assets
```

## 🗄️ Comprehensive Database Schema

### **User Management & Authentication**
```sql
-- Core user table with comprehensive profile management
users (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    email varchar UNIQUE NOT NULL,
    full_name varchar,
    phone varchar,
    avatar_url text,
    timezone varchar DEFAULT 'UTC',
    language varchar DEFAULT 'en',
    
    -- Status and verification
    status user_status DEFAULT 'active',
    email_verified boolean DEFAULT false,
    phone_verified boolean DEFAULT false,
    
    -- Permissions
    is_admin boolean DEFAULT false,
    is_owner boolean DEFAULT false,
    
    -- Subscription and credits
    plan user_plan DEFAULT 'free',
    monthly_credits integer DEFAULT 500,
    lifetime_credits integer DEFAULT 0,
    credits_used_this_month integer DEFAULT 0,
    monthly_credits_reset_date timestamptz DEFAULT date_trunc('month', now() + interval '1 month'),
    
    -- Stripe integration
    stripe_customer_id varchar,
    subscription_status varchar,
    subscription_id varchar,
    subscription_start_date timestamptz,
    subscription_end_date timestamptz,
    
    -- Security and tracking
    last_login_at timestamptz,
    last_login_ip varchar,
    login_count integer DEFAULT 0,
    failed_login_attempts integer DEFAULT 0,
    locked_until timestamptz,
    password_changed_at timestamptz DEFAULT now(),
    
    -- Preferences
    notifications_email boolean DEFAULT true,
    notifications_browser boolean DEFAULT true,
    theme varchar DEFAULT 'system',
    
    -- Marketing and referrals
    signup_source varchar,
    referrer_user_id uuid REFERENCES users(id),
    marketing_consent boolean DEFAULT false,
    terms_accepted_at timestamptz,
    privacy_accepted_at timestamptz,
    
    -- Timestamps
    created_at timestamptz DEFAULT now(),
    updated_at timestamptz DEFAULT now(),
    deleted_at timestamptz,
    
    -- Custom backend authentication
    password_hash varchar,
    reset_token varchar,
    reset_token_expires timestamptz,
    email_verification_token varchar,
    email_verification_expires timestamptz,
    
    -- Constraints
    CONSTRAINT email_format CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'),
    CONSTRAINT phone_format CHECK (phone IS NULL OR phone ~* '^\+?[1-9]\d{1,14}$')
);

-- User sessions for JWT token management
user_sessions (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id uuid REFERENCES users(id) ON DELETE CASCADE,
    token_hash varchar NOT NULL,
    refresh_token_hash varchar,
    
    -- Device and browser information
    device_type varchar,
    device_name varchar,
    browser varchar,
    os varchar,
    ip_address varchar,
    user_agent text,
    
    -- Location data
    location_country varchar,
    location_city varchar,
    
    -- Session management
    is_active boolean DEFAULT true,
    last_activity_at timestamptz DEFAULT now(),
    expires_at timestamptz,
    created_at timestamptz DEFAULT now()
);
```

### **API Management System**
```sql
-- Comprehensive API key management with permissions
api_keys (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id uuid REFERENCES users(id) ON DELETE CASCADE,
    name varchar NOT NULL,
    key_prefix varchar, -- First 8 characters for display
    key_hash varchar NOT NULL,
    
    -- Security and permissions
    permissions jsonb DEFAULT '{}',
    allowed_origins jsonb DEFAULT '[]',
    allowed_ips jsonb DEFAULT '[]',
    
    -- Rate limiting
    rate_limit_per_hour integer DEFAULT 100,
    rate_limit_per_day integer DEFAULT 1000,
    monthly_usage_limit integer,
    
    -- Status and usage
    is_active boolean DEFAULT true,
    last_used_at timestamptz,
    usage_count integer DEFAULT 0,
    
    -- Management
    created_by uuid REFERENCES users(id),
    revoked_at timestamptz,
    revoked_by uuid REFERENCES users(id),
    expires_at timestamptz,
    
    -- Timestamps
    created_at timestamptz DEFAULT now(),
    updated_at timestamptz DEFAULT now()
);
```

### **Advanced Job Management**
```sql
-- Comprehensive scraping jobs with detailed tracking
scrape_jobs (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id uuid REFERENCES users(id) ON DELETE CASCADE,
    api_key_id uuid REFERENCES api_keys(id),
    
    -- Job identification
    name varchar,
    description text,
    job_type job_type DEFAULT 'single',
    
    -- Input data
    url text,
    urls jsonb, -- For batch jobs
    crawl_config jsonb, -- For crawl jobs
    
    -- Scraping configuration
    engine scraping_engine DEFAULT 'lightweight',
    content_format content_format DEFAULT 'markdown',
    wait_time integer DEFAULT 0,
    screenshot boolean DEFAULT false,
    pdf boolean DEFAULT false,
    mobile boolean DEFAULT false,
    javascript boolean DEFAULT false,
    custom_headers jsonb DEFAULT '{}',
    cookies jsonb DEFAULT '{}',
    proxy_config jsonb DEFAULT '{}',
    
    -- Job execution
    status job_status DEFAULT 'pending',
    priority integer DEFAULT 0,
    total_pages integer DEFAULT 1,
    pages_completed integer DEFAULT 0,
    pages_failed integer DEFAULT 0,
    current_url text,
    progress_percentage numeric DEFAULT 0,
    
    -- Resource usage
    credits_used integer DEFAULT 0,
    processing_time_ms integer,
    data_extracted_kb integer,
    
    -- Results
    result_data jsonb,
    result_summary jsonb,
    output_files jsonb DEFAULT '[]',
    
    -- Error handling
    error_message text,
    error_code varchar,
    retry_count integer DEFAULT 0,
    max_retries integer DEFAULT 3,
    
    -- Scheduling (future feature)
    scheduled_at timestamptz,
    recurring_pattern varchar,
    cron_expression varchar,
    next_run_at timestamptz,
    
    -- Timestamps
    created_at timestamptz DEFAULT now(),
    started_at timestamptz,
    completed_at timestamptz,
    updated_at timestamptz DEFAULT now()
);

-- Job execution logs for debugging
job_logs (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    job_id uuid REFERENCES scrape_jobs(id) ON DELETE CASCADE,
    level varchar NOT NULL, -- debug, info, warn, error
    message text NOT NULL,
    component varchar, -- engine, processor, validator
    url text,
    page_number integer,
    metadata jsonb DEFAULT '{}',
    
    -- Performance metrics
    memory_usage_mb integer,
    cpu_usage_percent numeric,
    duration_ms integer,
    
    created_at timestamptz DEFAULT now()
);

-- Job files (screenshots, PDFs, exports)
job_files (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    job_id uuid REFERENCES scrape_jobs(id) ON DELETE CASCADE,
    file_type file_type NOT NULL,
    file_name varchar NOT NULL,
    file_size bigint,
    file_format varchar,
    
    -- Storage information
    storage_provider varchar DEFAULT 's3',
    storage_path text,
    storage_bucket varchar,
    is_public boolean DEFAULT false,
    access_token varchar,
    expires_at timestamptz,
    
    -- File metadata
    content_type varchar,
    compression varchar,
    checksum varchar,
    metadata jsonb DEFAULT '{}',
    
    created_at timestamptz DEFAULT now(),
    updated_at timestamptz DEFAULT now()
);
```

## 🔌 Comprehensive API Architecture

### **Authentication API (`/v1/auth/`)**

#### **Session Management**
```typescript
// Create session (login)
POST /v1/auth/sessions
Request: {
  email: string,
  password: string,
  rememberMe?: boolean,
  deviceInfo?: {
    name: string,
    type: 'desktop' | 'mobile' | 'tablet',
    browser: string,
    os: string
  }
}
Response: {
  success: true,
  data: {
    user: User,
    tokens: {
      accessToken: string,    // JWT token (1 hour expiry)
      refreshToken: string,   // Refresh token (30 days)
      expiresIn: number      // Seconds until expiry
    },
    session: {
      id: string,
      expiresAt: string
    }
  }
}

// Refresh token
POST /v1/auth/sessions/refresh
Request: { refreshToken: string }
Response: { success: true, data: { tokens: TokenSet } }

// Logout (destroy session)
DELETE /v1/auth/sessions
Headers: { Authorization: "Bearer <token>" }
Response: { success: true, message: "Logged out successfully" }
```

#### **User Registration & Management**
```typescript
// Register new user
POST /v1/auth/users
Request: {
  email: string,
  password: string,        // Min 8 chars, 1 upper, 1 lower, 1 number
  fullName: string,        // Min 2 characters
  phone?: string,
  acceptTerms: true,       // Must be true
  marketingConsent?: boolean
}
Response: {
  success: true,
  data: {
    user: User,
    tokens: TokenSet,
    verificationRequired: boolean
  }
}

// Verify email
POST /v1/auth/verify-email
Request: { token: string }
Response: { success: true, message: "Email verified successfully" }
```

### **Core Scraping API (`/v1/scrape/`)**

#### **Single Page Scraping**
```typescript
POST /v1/scrape
Headers: { 
  Authorization: "Bearer <token>",
  "X-API-Key": "<api-key>" // Alternative auth
}
Request: {
  url: string,
  options?: {
    format?: 'markdown' | 'html' | 'text' | 'json' | 'structured',
    engine?: 'lightweight' | 'playwright' | 'puppeteer',
    includeScreenshot?: boolean,
    includePdf?: boolean,
    mobile?: boolean,
    waitTime?: number,      // 0-30 seconds
    javascript?: boolean,
    cookies?: Record<string, string>,
    headers?: Record<string, string>,
    timeout?: number,       // 5-120 seconds
    useCache?: boolean,
    cacheTtl?: number,     // 60-3600 seconds
    webhook?: string       // Webhook URL for completion notification
  }
}
Response: {
  success: true,
  data: {
    content: string,
    metadata: {
      title: string,
      description: string,
      url: string,
      statusCode: number,
      contentType: string,
      extractedAt: string,
      processingTime: number,
      creditsUsed: number
    },
    screenshots?: string[], // S3 URLs
    pdf?: string,          // S3 URL
    files?: JobFile[]
  }
}
```

### **Job Management API (`/v1/jobs/`)**

#### **Job Operations**
```typescript
// List user jobs with filtering and pagination
GET /v1/jobs?status=completed&type=batch&limit=50&offset=0&sort=created_at:desc
Response: {
  success: true,
  data: {
    jobs: ScrapeJob[],
    pagination: {
      total: number,
      page: number,
      totalPages: number,
      hasNext: boolean,
      hasPrev: boolean
    }
  }
}

// Get detailed job information
GET /v1/jobs/:jobId
Response: {
  success: true,
  data: {
    job: ScrapeJob,
    logs: JobLog[],
    files: JobFile[],
    metrics: {
      averageResponseTime: number,
      successRate: number,
      totalDataExtracted: number
    }
  }
}

// Cancel running job
DELETE /v1/jobs/:jobId
Response: { success: true, message: "Job cancelled successfully" }

// Download job results
GET /v1/jobs/:jobId/download?format=json|csv|xml&compressed=true
Response: File download

// Get job real-time updates via WebSocket
WS /v1/jobs/:jobId/stream
Messages: {
  type: 'status_update' | 'progress' | 'completion' | 'error',
  data: {
    status: string,
    progress: number,
    currentUrl?: string,
    pagesCompleted: number,
    creditsUsed: number,
    estimatedTimeRemaining?: number
  }
}
```

### **Admin API (`/v1/admin/`)**

#### **User Management**
```typescript
// List all users with advanced filtering
GET /v1/admin/users?search=john&plan=pro&status=active&sort=created_at:desc&limit=100
Response: {
  success: true,
  data: {
    users: Array<User & {
      stats: {
        totalJobs: number,
        creditsUsed: number,
        lastActive: string,
        registrationSource: string
      }
    }>,
    aggregates: {
      totalUsers: number,
      activeUsers: number,
      paidUsers: number,
      totalRevenue: number
    }
  }
}

// Update user (admin actions)
PUT /v1/admin/users/:userId
Request: {
  plan?: 'free' | 'starter' | 'pro' | 'enterprise',
  monthlyCredits?: number,
  lifetimeCredits?: number,
  isAdmin?: boolean,
  isBanned?: boolean,
  bannedUntil?: string,
  subscriptionStatus?: string,
  notes?: string
}
```

## 📚 Documentation & API Standards

### **API Documentation Requirements**
Every new endpoint must include:

1. **OpenAPI/Swagger documentation** in the endpoint file
2. **Request/Response examples** with real data
3. **Error codes and messages** with descriptions
4. **Rate limiting information**
5. **Authentication requirements**
6. **Credit cost information** (for scraping endpoints)

```typescript
/**
 * @swagger
 * /v1/scrape:
 *   post:
 *     summary: Scrape a single webpage
 *     description: Extracts content from a webpage using specified format and options
 *     tags: [Scraping]
 *     security:
 *       - BearerAuth: []
 *       - ApiKeyAuth: []
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             $ref: '#/components/schemas/ScrapeRequest'
 *           examples:
 *             basic:
 *               summary: Basic scraping
 *               value:
 *                 url: "https://example.com"
 *                 options:
 *                   format: "markdown"
 *             advanced:
 *               summary: Advanced scraping with screenshot
 *               value:
 *                 url: "https://spa-example.com"
 *                 options:
 *                   format: "structured"
 *                   javascript: true
 *                   includeScreenshot: true
 *                   waitTime: 5
 *     responses:
 *       200:
 *         description: Content successfully extracted
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/ScrapeResponse'
 *       400:
 *         $ref: '#/components/responses/BadRequest'
 *       401:
 *         $ref: '#/components/responses/Unauthorized'
 *       429:
 *         $ref: '#/components/responses/RateLimited'
 *     x-credits-cost: 1-5 credits depending on options
 *     x-rate-limit: 30 requests per minute
 */
```

### **Linear Issue Management Workflow**

#### **Issue Creation Template**
When creating Linear issues, always use this format:

**Title**: `[Component] Brief description of task`

**Description**:
```markdown
## 📋 Task Overview
Brief description of what needs to be done

## 🎯 Acceptance Criteria
- [ ] Specific requirement 1
- [ ] Specific requirement 2
- [ ] Specific requirement 3

## 🔧 Technical Details
- API endpoints involved
- Database changes required
- Frontend components affected
- Dependencies and related issues

## 📚 Documentation Updates Needed
- [ ] Update API documentation
- [ ] Update user guides
- [ ] Update developer documentation
- [ ] Update CLAUDE.md (if architecture changes)

## 🧪 Testing Requirements
- [ ] Unit tests
- [ ] Integration tests
- [ ] Manual testing steps

## 💡 Additional Notes
Any additional context, concerns, or considerations
```

#### **Required Labels**
- `type:bug` - Bug fix
- `type:feature` - New feature
- `type:enhancement` - Improvement to existing feature
- `type:documentation` - Documentation update
- `type:api` - API-related changes
- `priority:low|medium|high|urgent` - Priority level
- `component:frontend|backend|database|infrastructure` - Affected component

#### **Status Workflow**
1. **Backlog** → **Todo** (when ready to start)
2. **Todo** → **In Progress** (when actively working)
3. **In Progress** → **In Review** (when development complete)
4. **In Review** → **Done** (after review and deployment)

### **Documentation Update Procedures**

#### **For New API Endpoints**
1. **Update OpenAPI spec** in the route file
2. **Add to Mintlify docs** in the whizoai-docs repo
3. **Update Postman collection** if maintained
4. **Add usage examples** in multiple languages
5. **Update rate limiting documentation**

#### **For New Features**
1. **Update user documentation** in whizoai-docs
2. **Update developer guides** if applicable
3. **Add changelog entries**
4. **Update this CLAUDE.md** if architecture changes

#### **For Bug Fixes**
1. **Update troubleshooting guides** if user-facing
2. **Update API documentation** if behavior changes
3. **Add to changelog** if significant

## 💰 Credit Management System

### **Credit Calculation**
```typescript
// Credit costs for different operations
const CREDIT_COSTS = {
  scraping: {
    base: 1,                    // 1 credit per URL
    javascript: 1,              // +1 for JS rendering
    screenshot: 1,              // +1 for screenshot
    pdf: 1,                     // +1 for PDF generation
    aiExtraction: 2,            // +2 for AI processing
    structured: 1,              // +1 for structured data
    batch: (urls: number) => urls * 0.9  // 10% discount for batch
  }
};
```

### **Plan-Based Features**
```typescript
const PLAN_FEATURES = {
  free: {
    monthlyCredits: 500,
    apiCallsPerHour: 10,
    apiCallsPerDay: 100,
    concurrentJobs: 1,
    maxPagesPerJob: 5,
    features: ['basic_scraping', 'markdown_output'],
    engines: ['lightweight']
  },
  starter: {
    monthlyCredits: 3000,
    apiCallsPerHour: 50,
    apiCallsPerDay: 500,
    concurrentJobs: 3,
    maxPagesPerJob: 25,
    features: ['basic_scraping', 'batch_processing', 'screenshots'],
    engines: ['lightweight', 'playwright']
  },
  pro: {
    monthlyCredits: 10000,
    apiCallsPerHour: 200,
    apiCallsPerDay: 2000,
    concurrentJobs: 10,
    maxPagesPerJob: 100,
    features: ['all_basic', 'ai_extraction', 'pdf_generation'],
    engines: ['all']
  },
  enterprise: {
    monthlyCredits: 50000,
    apiCallsPerHour: 1000,
    apiCallsPerDay: 10000,
    concurrentJobs: 50,
    maxPagesPerJob: 1000,
    features: ['all_features', 'white_label', 'custom_integrations'],
    engines: ['all']
  }
};
```

## 🎨 Frontend UI Patterns & Standards

### **Breadcrumb Navigation Implementation**
All dashboard pages (except the root `/app` page) must include consistent breadcrumb navigation:

```typescript
// Standard breadcrumb implementation pattern
import {
  Breadcrumb,
  BreadcrumbItem,
  BreadcrumbLink,
  BreadcrumbList,
  BreadcrumbPage,
  BreadcrumbSeparator,
} from "@/components/ui/breadcrumb"

// In page component:
<Breadcrumb>
  <BreadcrumbList>
    <BreadcrumbItem>
      <BreadcrumbLink href="/app">Dashboard</BreadcrumbLink>
    </BreadcrumbItem>
    <BreadcrumbSeparator />
    <BreadcrumbItem>
      <BreadcrumbPage>[PageName]</BreadcrumbPage>
    </BreadcrumbItem>
  </BreadcrumbList>
</Breadcrumb>
```

**Pages with Breadcrumbs (Required):**
- `/app/playground` → "Dashboard > Playground"
- `/app/jobs` → "Dashboard > Jobs"
- `/app/analytics` → "Dashboard > Analytics"
- `/app/templates` → "Dashboard > Templates"
- `/app/settings` → "Dashboard > Settings"
- `/app/api-keys` → "Dashboard > API Keys"
- `/app/account` → "Dashboard > Account"
- `/app/notifications` → "Dashboard > Notifications"
- `/app/billing` → "Dashboard > Billing"

**Pages without Breadcrumbs (Correct):**
- `/app` → Root dashboard page

### **Consistent Page Structure**
All dashboard pages must follow this structure pattern:

```typescript
export default function PageName() {
  return (
    <SidebarProvider
      style={
        {
          "--sidebar-width": "calc(var(--spacing) * 72)",
          "--header-height": "calc(var(--spacing) * 12)",
        } as React.CSSProperties
      }
    >
      <AppSidebar variant="inset" />
      <SidebarInset>
        <SiteHeader />
        <div className="flex flex-1 flex-col">
          <div className="@container/main flex flex-1 flex-col gap-2">
            <div className="flex flex-col gap-4 p-4 md:gap-6 md:p-6">
              {/* Breadcrumb here */}
              <Breadcrumb>...</Breadcrumb>
              
              {/* Page header */}
              <div>
                <h1 className="text-3xl font-bold tracking-tight">Page Title</h1>
                <p className="text-muted-foreground">Page description</p>
              </div>
              
              {/* Page content */}
              <div className="grid gap-4">
                {/* Content here */}
              </div>
            </div>
          </div>
        </div>
      </SidebarInset>
    </SidebarProvider>
  )
}
```

## 🔧 Development Workflows & Standards

### **Code Quality Standards**
- **TypeScript strict mode** enabled
- **ESLint** with no warnings allowed
- **Prettier** for code formatting
- **Comprehensive error handling** required
- **Input validation** with Zod schemas
- **Security-first** approach

### **Testing Requirements**
- **Unit tests** for all business logic
- **Integration tests** for API endpoints  
- **E2E tests** for critical user flows
- **Performance tests** for scraping engines
- **Security tests** for authentication

### **Performance Standards**
- **API response time** < 500ms (non-scraping)
- **Database queries** optimized with indexes
- **Frontend bundle size** minimized
- **Image optimization** implemented
- **Caching strategy** in place

## 🚀 Deployment Configuration

### **Environment Variables**
```bash
# Backend (.env)
SUPABASE_URL=your-supabase-project-url
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
JWT_SECRET=your-jwt-secret-key
REDIS_URL=your-redis-connection-string
AWS_ACCESS_KEY_ID=your-aws-access-key
AWS_SECRET_ACCESS_KEY=your-aws-secret-key
STRIPE_SECRET_KEY=your-stripe-secret-key
PORT=8080
NODE_ENV=production

# Frontend (.env.local)  
NEXT_PUBLIC_SUPABASE_URL=your-supabase-project-url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
NEXT_PUBLIC_API_URL=https://api.whizo.ai
NEXT_PUBLIC_APP_URL=https://dashboard.whizo.ai
```

### **Production Checklist**
- ✅ Environment variables configured
- ✅ Database RLS policies enabled
- ✅ Rate limiting configured
- ✅ SSL certificates installed
- ✅ CDN configured
- ✅ Monitoring enabled
- ✅ Backup strategies implemented
- ✅ Security headers configured

## 🛠️ Troubleshooting Guide

### **Common Issues**
1. **Authentication Issues**
   - Check JWT secret configuration
   - Verify database connection
   - Review RLS policies

2. **Database Connection**
   - Verify Supabase credentials
   - Check network connectivity
   - Review connection pool settings

3. **Scraping Engine Issues**
   - Check browser dependencies
   - Verify timeout settings
   - Review proxy configuration

### **Performance Optimization**
1. **Database Query Optimization**
   - Add appropriate indexes
   - Use query explain plans
   - Implement connection pooling

2. **Caching Strategy**
   - Implement Redis caching
   - Use CDN for static assets
   - Cache API responses

3. **Frontend Performance**
   - Code splitting
   - Component memoization
   - Bundle optimization

---

## 🔍 Development Best Practices Summary

### **ALWAYS Required Actions**
1. **Create or update Linear issues** for every development task
2. **Update documentation** when adding new features or endpoints
3. **Run linting and type checking** before committing
4. **Write tests** for new functionality
5. **Follow security best practices** with input validation and error handling
6. **Monitor performance impact** of changes
7. **Update this CLAUDE.md** when architecture changes

### **Never Skip**
- Input validation and sanitization
- Proper error handling and logging
- Authentication and authorization checks
- Rate limiting implementation
- Comprehensive testing
- Documentation updates

### **Code Quality Gates**
- TypeScript strict mode compliance
- ESLint passing without warnings
- All tests passing
- Performance benchmarks met
- Security audit clean
- Documentation updated

This comprehensive guide ensures consistent, high-quality development practices across the WhizoAI platform while maintaining security, performance, and user experience standards.