# TeamSync Tech Stack Document

## Introduction

This document outlines the technology stack and architectural decisions for the TeamSync chat application. TeamSync integrates with the Circle Ecosystem to provide text chat, audio/video calls, and remote desktop functionality. This document focuses primarily on the chat functionality, with calls to be addressed separately.

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
- **Material Components + Style Adapter Pattern**
- Slice-specific UI adapters for multiple presentation styles
- Support for different UX variants (WhatsApp, Slack, Teams)
- Only one presentation layer compiled into final app

### Build Configuration
- **Build Flavors with Product Flavors**
- Flavor-specific assets and configurations
- Clear separation at build time

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
- **flutter_lints** for standard linting rules
- Potential future expansion with custom_lint as needed
- Dart code formatting enforced in CI

### Documentation
- **Architecture Decision Records (ADRs)** for key decisions
- Documented patterns for Claude Code reference
- Clear API documentation for PubNub-inspired interfaces

### CI/CD Pipeline
- **GitHub Actions with Feature Branch Workflow**
- Automated testing and linting for PRs
- Separate build jobs for each UI variant

## Implementation Notes for Claude Code

1. Each vertical slice should be developed independently with its own repository pattern
2. Follow the PubNub Chat SDK interface patterns for API implementation
3. Use Riverpod providers consistently for dependency injection and state management
4. Implement UI components according to the Style Adapter Pattern
5. Ensure error handling follows established middleware patterns
6. Document new components and decisions with ADRs

## Appendix

### Additional Tools To Consider
- Data analytics integration
- A/B testing framework
- Accessibility compliance testing
- Performance monitoring