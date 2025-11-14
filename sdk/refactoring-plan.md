# OpenStrand SDK Refactoring Plan

## Current State
Two SDKs existed: the standalone package and an embedded one in the backend. Goal: single source of truth.

## Architecture Goals
Single package, edition support, clear separation between SDK and backend.

## Refactoring Steps
### Phase 1: SDK Enhancement
- Feature detection and config
- BaseService with `requireFeature`
- Add missing services: Teams, Billing, AI, Admin, Webhook, Analytics

### Phase 2: Backend Refactoring
- Remove embedded SDK from teams backend
- Use `@framers/openstrand-sdk` dependency
- Add SDK integration tests

### Phase 3: Community Edition
- Create community backend with subset routes and features
- Feature matrix

### Phase 4: Publishing
- Package exports and browser bundle
- CI for npm publishing, examples

## Migration Guide
Move imports from embedded SDK to `@framers/openstrand-sdk`.

## Benefits
Single truth, better testing, community support, clear boundaries, independent versioning, future multi-language SDKs.

## Next Steps
Track issues per phase, assign ownership, set up CI/CD and begin Phase 1.


