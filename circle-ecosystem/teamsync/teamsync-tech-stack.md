# TeamSync Tech Stack Document

## Introduction

This document outlines the comprehensive technology stack and architectural decisions for the TeamSync communication app. TeamSync integrates with the Circle Ecosystem to provide text chat, audio/video calls, and remote desktop functionality, with a focus on delivering a premium experience for executive users such as CEOs.

This tech stack is optimized for Claude Code compatibility, ensuring consistent, high-quality code generation throughout the development process.

## Core Technology Choices

### Platform Support
- iOS
- Android
- Windows
- Mac

### Programming Language and Framework
- **Flutter** for cross-platform development

### Backend
- **Supabase** for database, authentication, storage, and serverless functions
- **Direct Supabase Client + Row Level Security (RLS)** for most operations
- Edge Functions reserved for complex features as needed

## Architectural Approach

### Overall Architecture
- **Vertical Slice Architecture** with feature-first folder structure
- Separate logical apps for Chat and Calls brought together via a common UI shell
- Multiple presentation layers sharing common business logic

### Folder Structure
```
/lib
  /chat                 # Chat application
    /slice1_channels    # Vertical slice for channel functionality
      /repository       # PubNub-inspired API implementation
      /presentation
        /ui_components  # Interface definitions
        /whatsapp       # WhatsApp-style implementation
        /slack          # Slack-style implementation
        /teams          # Teams-style implementation
    /slice2_messaging   # Vertical slice for messaging functionality  
      /repository
      /presentation
        /...
  /calls                # Call application (to be developed later)
  /shared               # Truly shared code between chat and calls
    /ui_shell           # Common UI shell with navigation
    /providers          # Shared providers
    /models             # Shared models
```

## State Management

**Riverpod**
- Type-safe dependency injection
- Support for code generation (riverpod_generator)
- Excellent for testability and AI code generation
- Clear provider patterns that create predictable code structure

## API Implementation

### Chat API
- **Modified Repository Pattern**
- Implementation based on PubNub Chat SDK API structure
- Each vertical slice has its own repository implementing the relevant PubNub-inspired interfaces
- Repositories handle both data access and business logic

## Database & Storage

### Remote Database
- **Supabase Database** with Row Level Security (RLS)
- Direct Supabase Client for most operations
- Edge Functions reserved for complex features as needed

### Local Database
- **Drift** (SQLite-based, type-safe)
- Support for structured queries and type-safety
- Data synchronization with **PowerSync** (primary option)
  - Alternate: Custom synchronization layer (secondary option)

### Offline Capabilities
- View-only functionality while offline
- No message queuing or sending while offline

### File/Media Storage
- **Supabase Storage with Local Caching**
- Integration with RLS for security

## Authentication & Security

- **Supabase Auth with External JWT Verification**
- Integration with Circle Foundations for authentication
- TeamSync's Supabase verifies Circle Foundations JWT
- Row Level Security (RLS) for data access control

## UI/UX Implementation

### Component Framework
- **DECISION PENDING** - Requires further analysis for executive user experience
- Options under consideration:
  1. **Material Components + Style Adapter Pattern**:
     - Pros: Well-documented, Claude Code familiar with patterns, consistent cross-platform behavior
     - Cons: May look too "Google-like", requires extensive customization for executive users

  2. **MacOS UI**:
     - Pros: Native Apple aesthetics, premium feel appropriate for executives, authentic macOS experience
     - Cons: Limited to macOS platform, requires custom implementation for other platforms

  3. **Forui**:
     - Pros: Minimalist elegance, platform-agnostic design with premium feel, extensive theming capabilities
     - Cons: Newer library with potentially less community support

  4. **Custom Component Library**:
     - Pros: Complete control over appearance and behavior, perfect alignment with executive expectations
     - Cons: Significant development effort, more code to maintain

### Build Configuration
- **Build Flavors with Product Flavors**
- Flavor-specific assets and configurations
- Clear separation at build time

### Adaptive & Responsive Design Approach
- **DECISION PENDING** - Requires further analysis of UI differences across platforms
- Options under consideration:
  1. **Platform-specific subfolders**:
     - Pros: Clear separation by platform, explicit platform-specific code
     - Cons: Duplication, more folders to maintain

  2. **Adaptive components approach**:
     - Pros: Centralized adaptation logic, less duplication, Claude Code friendly
     - Cons: May not handle fundamental UI paradigm differences

  3. **Platform-specific presentation layers**:
     - Pros: Complete UI independence, platform-optimized patterns, maximum flexibility
     - Cons: Most duplication, separate maintenance for each platform

## Testing Strategy

### Testing Tools
- **Mocktail** for mocking dependencies
- **Riverpod_test** for testing providers
- Flutter test for widget/integration tests (future addition)

### Testing Focus
- Prioritize unit testing of business logic (API layer)
- Later expansion to include widget and integration tests for key flows

## Error Handling & Monitoring

- **Crashlytics + Custom Error Handling Middleware**
- Structured approach to error logging
- Integration with build flavors for environment-specific handling

## Analytics & Monitoring

- **Firebase Analytics + Crashlytics Integration**
- Comprehensive solution for tracking usage and errors
- Good Flutter support
- Integration with the existing error handling approach

## Push Notifications

- **Firebase Cloud Messaging**
- Supabase Function Triggers for notification events
- Integration with Circle Foundations for presence information

## Dependency Management

- **Hybrid Approach: Riverpod + Internal Structure**
- Using Riverpod providers within a structured folder organization
- Clear boundaries maintained through folder structure and import discipline

## Development Tools & Quality Assurance

### Development Tooling
- **Melos + build_runner**
- Powerful tool for monorepo management
- Supports running commands across packages
- Good integration with CI/CD pipelines

### CI/CD Pipeline
- **GitHub Actions + Custom Build Scripts**
- Highly customizable
- Integrated with chosen GitHub workflow
- Capability to deploy to all required app stores (mobile and desktop)

### Auto-Update Solution
- **Shorebird Code Push**
- Flutter-specific solution for partial updates
- Support for controlled rollouts
- Bypasses app store review process
- Works for both mobile and desktop platforms

### Documentation
- **Architecture Decision Records (ADRs)** for key decisions
- Documented patterns for Claude Code reference
- Clear API documentation for PubNub-inspired interfaces

## Sections Pending Decisions

### Security Practices Beyond Authentication
- **DECISION SKIPPED** - To be addressed in future iterations
- Options for future consideration:
  1. **Comprehensive Security Package**:
     - App obfuscation, certificate pinning, secure storage, jailbreak detection
     - Pros: Thorough protection, addresses multiple threat vectors
     - Cons: Implementation complexity, potential performance impact

  2. **Focused Data Security Approach**:
     - Encrypted local storage, secure network communication, minimal data retention
     - Pros: Concentrates on protecting user data, simpler implementation
     - Cons: May leave some attack vectors unaddressed

  3. **Security-as-a-Service Integration** (e.g., AppSweep, Data Theorem):
     - Pros: Ongoing security monitoring, regular vulnerability assessments
     - Cons: Subscription costs, external dependency

### Internationalization Strategy
- **DECISION SKIPPED** - To be addressed in future iterations
- Options for future consideration:
  1. **Flutter Intl Package with ARB Files**:
     - Pros: Official approach, good IDE support, code generation
     - Cons: Manual file management, more complex setup

  2. **Easy Localization with JSON Files**:
     - Pros: Simpler setup, JSON-based translations, good for non-developers
     - Cons: Less IDE integration, potential performance concerns

  3. **GetX Translation Management**:
     - Pros: Integrated with broader GetX ecosystem, simple API
     - Cons: Ties you to GetX, less separation of concerns

## Implementation Notes for Claude Code

1. Each vertical slice should be developed independently with its own repository pattern
2. Follow the PubNub Chat SDK interface patterns for API implementation
3. Use Riverpod providers consistently for dependency injection and state management
4. Implement UI components according to the Style Adapter Pattern
5. Ensure error handling follows established middleware patterns
6. Document new components and decisions with ADRs
7. Use Melos scripts for standardized development commands
8. Follow GitHub Actions workflow patterns for CI/CD

## Appendix

### Additional Tools To Consider
- Accessibility compliance testing
- Performance profiling and optimization
- Integration with advanced analytics for CEO-specific insights
