# Circle Foundation Service Technical Stack

## 1. Introduction

This document outlines the technical stack for the Circle Foundation Service, defining the technologies, architecture patterns, and implementation approaches for building a robust, scalable, and maintainable SaaS platform. The Foundation Service provides core organizational management, user journey management, authentication, and other foundation capabilities as specified in the Circle Foundation Module Features document.

## 2. Core Technology Stack

### 2.1 Frontend
- **Next.js** with App Router (latest stable version)
- **TypeScript** for type safety
- **Zustand** for state management
- **Tailwind CSS** for styling
- **shadcn/ui** for component library and design system
- **Lucide** for icons
- **React Hook Form** for form handling
- **Zod** for validation

### 2.2 Backend
- **Supabase** for database, authentication, and APIs
  - PostgreSQL for data storage
  - Row-Level Security (RLS) for multi-tenant isolation
  - Authentication services
  - Storage services
  - Realtime capabilities (when needed)

### 2.3 Deployment & CI/CD
- **GitHub** for source control
- **GitHub Actions** for CI/CD pipelines
- **Google Cloud Secret Manager** for secrets management

### 2.4 Environments
- **DEV**: Development environment for active development work
- **QA**: Quality Assurance environment for testing
- **PROD**: Production environment

Each environment will have its own Supabase project to ensure complete isolation of data.

## 3. Architectural Approach

### 3.1 Vertical Slice Architecture (VSA)

The Circle Foundation Service will implement Vertical Slice Architecture as the primary architectural pattern. VSA organizes code around business capabilities rather than technical concerns, resulting in:

- **Improved maintainability**: Code that changes together stays together
- **Better developer experience**: Developers can focus on one feature at a time
- **Simplified AI-assisted development**: Clear organization makes Claude Code more effective
- **Reduced coupling**: Changes in one feature rarely affect others
- **Enhanced team coordination**: Different teams can own different slices

### 3.2 Project Structure

The project will be organized using a feature-first approach aligned with Vertical Slice Architecture:

```
src/
├── app/          # Next.js App Router routes
│   ├── layout.tsx
│   ├── page.tsx
│   ├── (auth)/   # Authentication routes
│   ├── (admin)/  # Admin routes
│   └── api/      # API routes
├── features/     # Feature slices 
│   ├── organization-lifecycle/
│   │   ├── components/   # UI components 
│   │   ├── hooks/        # React hooks
│   │   ├── api/          # API functions
│   │   ├── store/        # Zustand store
│   │   ├── utils/        # Feature-specific utilities
│   │   └── types.ts      # TypeScript types
│   ├── user-journey/
│   │   └── ...
│   ├── authentication/
│   │   └── ...
│   ├── role-permission/
│   │   └── ...
│   └── ...
└── shared/       # Truly shared code
    ├── components/
    ├── hooks/
    ├── utils/
    └── types/
```

Each feature slice is organized based on the specific capabilities outlined in the Circle Foundation Module Features document, with Phase 1 features prioritized.

### 3.3 Multi-Tenancy Implementation

The Circle Foundation Service will use a single schema approach with tenant_id for multi-tenant isolation:

- Every tenant-specific table will include a `tenant_id` column
- Row-Level Security (RLS) policies will ensure data isolation
- Tenant context will be stored in user metadata in Supabase Auth

Example RLS policy implementation:

```sql
-- Enable RLS on table
ALTER TABLE organizations ENABLE ROW LEVEL SECURITY;

-- Create policy for data access based on tenant_id
CREATE POLICY "Users can only access their tenant's data" 
ON organizations
FOR ALL
USING (tenant_id = auth.tenant_id());

-- Helper function to get tenant_id from JWT claims
CREATE OR REPLACE FUNCTION auth.tenant_id() 
RETURNS TEXT LANGUAGE SQL STABLE AS $$
  SELECT nullif(
    ((current_setting('request.jwt.claims')::jsonb ->> 'app_metadata')::jsonb ->> 'tenant_id'),
    ''
  )::text
$$;
```

## 4. Feature Implementation Approach

The following example demonstrates how a feature would be implemented using Vertical Slice Architecture. This pattern serves as a template for all feature implementations in the Circle Foundation Service.

### 4.1 Organization Lifecycle Management (Phase 1) Example

The organization lifecycle management features will be implemented as a vertical slice containing:

```
features/organization-lifecycle/
├── components/
│   ├── OrganizationForm.tsx
│   ├── OrganizationList.tsx
│   └── ...
├── api/
│   ├── createOrganization.ts
│   ├── getOrganizations.ts
│   └── ...
├── hooks/
│   ├── useOrganization.ts
│   └── ...
├── store/
│   ├── organizationStore.ts  # Zustand store
│   └── ...
├── utils/
│   └── ...
└── types.ts
```

The Zustand store will manage organization state:

```typescript
// features/organization-lifecycle/store/organizationStore.ts
import { create } from 'zustand';
import { createClientComponentClient } from '@supabase/auth-helpers-nextjs';

interface Organization {
  id: string;
  name: string;
  status: 'active' | 'inactive';
  // Other properties
}

interface OrganizationState {
  organizations: Record<string, Organization>;
  currentOrganizationId: string | null;
  isLoading: boolean;
  error: string | null;
}

interface OrganizationActions {
  fetchOrganizations: () => Promise<void>;
  createOrganization: (name: string) => Promise<void>;
  // Other actions
}

export const useOrganizationStore = create<OrganizationState & OrganizationActions>((set, get) => ({
  // State
  organizations: {},
  currentOrganizationId: null,
  isLoading: false,
  error: null,
  
  // Actions
  fetchOrganizations: async () => {
    set({ isLoading: true, error: null });
    const supabase = createClientComponentClient();
    
    const { data, error } = await supabase
      .from('organizations')
      .select('*');
      
    if (error) {
      set({ error: error.message, isLoading: false });
      return;
    }
    
    // Normalize data by ID
    const organizationsMap = data.reduce((acc, org) => {
      acc[org.id] = org;
      return acc;
    }, {});
    
    set({ organizations: organizationsMap, isLoading: false });
  },
  
  createOrganization: async (name) => {
    // Implementation
  },
  
  // Other actions
}));
```

This pattern will be repeated for all feature slices, ensuring consistent implementation and organization across the codebase. Each feature slice will have its own components, API functions, hooks, store, and utilities relevant to that specific business capability.


## 5. State Management

### 5.1 Client-Side State with Zustand

Zustand will be used for client-side state management, with stores organized by feature slices:

```typescript
// Example Zustand store in a feature slice
import { create } from 'zustand';

// Define types
interface UserState {
  users: User[];
  isLoading: boolean;
  error: string | null;
}

interface UserActions {
  fetchUsers: () => Promise<void>;
  addUser: (user: User) => Promise<void>;
  removeUser: (id: string) => Promise<void>;
}

// Create store
export const useUserStore = create<UserState & UserActions>((set, get) => ({
  users: [],
  isLoading: false,
  error: null,
  
  fetchUsers: async () => {
    set({ isLoading: true });
    // Implementation
  },
  
  addUser: async (user) => {
    // Implementation
  },
  
  removeUser: async (id) => {
    // Implementation
  }
}));
```

### 5.2 Server-Side State Management

Next.js Server Components with Supabase Server Client

```typescript
// app/dashboard/page.tsx
import { createServerComponentClient } from '@supabase/auth-helpers-nextjs';
import { cookies } from 'next/headers';
import { OrganizationList } from '@/features/organization-lifecycle/components/OrganizationList';

export default async function DashboardPage() {
  const supabase = createServerComponentClient({ cookies });
  
  const { data: organizations } = await supabase
    .from('organizations')
    .select('*');
    
  return <OrganizationList initialData={organizations} />;
}
```


## 6. Database Schema

### 6.1 Core Tables

The following shows the core tables for the Foundation Service with appropriate tenant isolation:

```sql
-- Organization table with tenant isolation
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  tenant_id UUID NOT NULL,
  name TEXT NOT NULL,
  status TEXT NOT NULL DEFAULT 'active',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Department/Team table
CREATE TABLE organization_units (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  tenant_id UUID NOT NULL,
  organization_id UUID REFERENCES organizations(id) NOT NULL,
  name TEXT NOT NULL,
  type TEXT NOT NULL, -- 'department', 'team', etc.
  parent_id UUID REFERENCES organization_units(id),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Users table (renamed from user_profiles)
CREATE TABLE users (
  id UUID PRIMARY KEY REFERENCES auth.users(id),
  tenant_id UUID NOT NULL,
  email TEXT NOT NULL,
  display_name TEXT,
  avatar_url TEXT,
  status TEXT NOT NULL DEFAULT 'active',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Reporting structure
CREATE TABLE reporting_lines (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  tenant_id UUID NOT NULL,
  manager_id UUID REFERENCES users(id) NOT NULL,
  report_id UUID REFERENCES users(id) NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(manager_id, report_id)
);
```

### 6.2 Row-Level Security Policies

```sql
-- Enable RLS on all tables
ALTER TABLE organizations ENABLE ROW LEVEL SECURITY;
ALTER TABLE organization_units ENABLE ROW LEVEL SECURITY;
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE reporting_lines ENABLE ROW LEVEL SECURITY;

-- Standard tenant isolation policy for each table
CREATE POLICY "Tenant isolation for organizations" 
ON organizations FOR ALL 
USING (tenant_id::text = auth.tenant_id());

CREATE POLICY "Tenant isolation for organization_units" 
ON organization_units FOR ALL 
USING (tenant_id::text = auth.tenant_id());

CREATE POLICY "Tenant isolation for users" 
ON users FOR ALL 
USING (tenant_id::text = auth.tenant_id());

CREATE POLICY "Tenant isolation for reporting_lines" 
ON reporting_lines FOR ALL 
USING (tenant_id::text = auth.tenant_id());
```

## 7. API Design

### 7.1 RESTful API Structure

The API will follow RESTful principles, with endpoints organized by feature:

```
# Organization management
GET    /api/organizations
POST   /api/organizations
GET    /api/organizations/:id
PATCH  /api/organizations/:id
DELETE /api/organizations/:id

# User journey
GET    /api/users
POST   /api/users/invite
GET    /api/users/:id
PATCH  /api/users/:id
DELETE /api/users/:id
```

### 7.2 API Implementation

API routes will be implemented using Next.js Route Handlers:

```typescript
// app/api/organizations/route.ts
import { createRouteHandlerClient } from '@supabase/auth-helpers-nextjs';
import { cookies } from 'next/headers';
import { NextResponse } from 'next/server';

export async function GET(request: Request) {
  const supabase = createRouteHandlerClient({ cookies });
  
  // Check authentication
  const { data: { session } } = await supabase.auth.getSession();
  if (!session) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }
  
  // Fetch organizations (RLS will automatically filter by tenant)
  const { data, error } = await supabase
    .from('organizations')
    .select('*');
    
  if (error) {
    return NextResponse.json({ error: error.message }, { status: 400 });
  }
  
  return NextResponse.json(data);
}

export async function POST(request: Request) {
  const supabase = createRouteHandlerClient({ cookies });
  
  // Check authentication
  const { data: { session } } = await supabase.auth.getSession();
  if (!session) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }
  
  // Get request body
  const { name } = await request.json();
  
  // Get tenant_id from user metadata
  const { data: { user } } = await supabase.auth.getUser();
  const tenant_id = user?.app_metadata?.tenant_id;
  
  // Create organization
  const { data, error } = await supabase
    .from('organizations')
    .insert([{ name, tenant_id }])
    .select();
    
  if (error) {
    return NextResponse.json({ error: error.message }, { status: 400 });
  }
  
  return NextResponse.json(data[0]);
}
```

## 8. Development Workflow and Tools

### 8.1 Development Environment

- GitHub Codespaces for Next.js app development
- Supabase Studio for database management
- Supabase Edge Functions dashboard for serverless functions

### 8.2 Testing Strategy

For Phase 1, we'll implement a pragmatic testing approach focused on end-to-end testing:

- **End-to-End Testing with Playwright**: 
  - Test critical user flows and business capabilities
  - Cover key functionality across features
  - Use Supabase seed scripts to prepare the database with test data
  - Focus on happy paths and critical edge cases

Unit testing with Vitest would be overkill at this stage. The end-to-end approach with Playwright allows us to:

1. Verify entire user workflows function correctly
2. Test both frontend and backend integration
3. Detect regression issues efficiently
4. Reduce maintenance overhead compared to numerous unit tests
5. Provide realistic test scenarios with seeded data

As the project matures beyond Phase 1, we can selectively add unit tests for complex business logic or critical utility functions where needed.

### 8.3 CI/CD Pipeline

```yaml
# .github/workflows/main.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install dependencies
        run: npm ci
      - name: Run tests
        run: npm test

  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install dependencies
        run: npm ci
      - name: Build application
        run: npm run build
      # Deployment steps
```

## 9. UI Design Principles

### 9.1 Apple Design Guidelines Alignment

The Circle Foundation Service UI will adhere to Apple's design principles, emphasizing:

- **Clarity**: Focus on essential elements, use plenty of negative space, and employ crisp, legible text
- **Deference**: UI that helps users understand and interact with content while avoiding competing with it
- **Depth**: Visual layers and realistic motion convey hierarchy and facilitate understanding

Key implementation considerations include:

- **Typography**: Use the SF Pro font family (or system fonts that respect user settings)
- **Color**: Implement a color system that supports both light and dark modes
- **Layout**: Employ consistent spacing, alignment, and responsive design patterns
- **Interaction**: Create fluid transitions and meaningful animations
- **Accessibility**: Design for accessibility with proper contrast ratios and support for assistive technologies

### 9.2 Component Design System

The shadcn/ui component library will be customized to align with Apple design principles:

- Modified component styling to match Apple's aesthetic
- Consistent interaction patterns mimicking Apple platforms
- Custom animations and transitions reflecting Apple's motion design
- Color tokens that match Apple's palette while maintaining brand identity

### 9.3 Responsive Design Strategy

The UI will implement a responsive design approach that:

- Prioritizes mobile-first design principles
- Creates consistent experiences across device sizes
- Uses appropriate component variants based on screen size
- Implements adaptive layouts that reorganize for different viewports

## 10. AI-Assisted Development Practices

### 10.1 Code Organization for Claude Code

To optimize for AI-assisted development with Claude Code, the codebase will follow these practices:

- **Clear file naming**: Use descriptive file names that reflect purpose
- **Consistent code patterns**: Maintain consistency across feature slices
- **Explicit typing**: Define comprehensive TypeScript interfaces/types
- **Documentation**: Include JSDoc comments for functions and components
- **Component modularity**: Keep components focused on single responsibilities

### 10.2 Feature Development Process

When implementing features with Claude Code, the development process will follow these steps:

1. Define feature requirements and data model
2. Create TypeScript types for the feature
3. Generate Zustand store with Claude Code
4. Implement UI components with Claude Code
5. Connect components to Zustand store
6. Implement API route handlers
7. Test and refine implementation

## 11. Conclusion

This technical stack document provides a comprehensive blueprint for the Circle Foundation Service, focusing on a Vertical Slice Architecture approach with Supabase as the core backend. By organizing around business capabilities and leveraging TypeScript with Zustand for state management, the implementation will deliver a maintainable, scalable foundation for the Circle Ecosystem.

The technical decisions outlined in this document prioritize developer experience, AI-assisted development, and alignment with the specific requirements of the Circle Foundation Module Features. The approach enables a phased implementation starting with critical Phase 1 features, while laying groundwork for subsequent phases.