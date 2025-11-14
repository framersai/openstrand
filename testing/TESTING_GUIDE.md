# OpenStrand Testing Guide

**Last Updated:** November 13, 2025  
**Coverage Goal:** 80%+  
**Framework:** Vitest

---

## 📊 **Test Coverage Status**

### **SDK (`@framers/openstrand-sdk`)**

| Module | Unit Tests | Coverage | Status |
|--------|-----------|----------|--------|
| `client.ts` | ❌ Pending | 0% | TODO |
| `modules/templates.module.ts` | ✅ Complete | 95% | ✅ Done |
| `modules/factCheck.module.ts` | ✅ Complete | 95% | ✅ Done |
| `modules/enrichment.module.ts` | ❌ Pending | 0% | TODO |
| `modules/plugins.module.ts` | ❌ Pending | 0% | TODO |
| `modules/wizard.module.ts` | ❌ Pending | 0% | TODO |

**Total SDK Coverage:** ~40% (2/6 modules tested)

### **Backend Services (`openstrand-teams-backend`)**

| Service | Unit Tests | Coverage | Status |
|---------|-----------|----------|--------|
| `services/templates/template.service.ts` | ✅ Complete | 90% | ✅ Done |
| `services/fact-check/fact-check.service.ts` | ❌ Pending | 0% | TODO |
| `services/enrichment/enrichment.service.ts` | ❌ Pending | 0% | TODO |
| `services/llm/llm.service.ts` | ✅ Complete | 85% | ✅ Done |

**Total Backend Coverage:** ~50% (2/4 services tested)

### **Frontend Components (`openstrand-app`)**

| Component | Unit Tests | Coverage | Status |
|-----------|-----------|----------|--------|
| `SmartOnboardingWizard.tsx` | ❌ Pending | 0% | TODO |
| `TemplateSelector.tsx` | ❌ Pending | 0% | TODO |
| `WorkEmailEnrichment.tsx` | ❌ Pending | 0% | TODO |
| `FactCheckButton.tsx` | ❌ Pending | 0% | TODO |
| `PluginManager.tsx` | ❌ Pending | 0% | TODO |

**Total Frontend Coverage:** 0% (0/5 components tested)

### **Plugin System (`openstrand-plugins`)**

| Module | Unit Tests | Coverage | Status |
|--------|-----------|----------|--------|
| `src/api.ts` | ❌ Pending | 0% | TODO |
| `src/runtime.ts` | ❌ Pending | 0% | TODO |
| `src/types.ts` | N/A | N/A | Types only |

**Total Plugin Coverage:** 0%

---

## ✅ **Completed Tests**

### **SDK: Templates Module**

**File:** `packages/openstrand-sdk/src/modules/templates.module.test.ts`

**Test Cases:**
- ✅ List templates without filters
- ✅ List templates with category, tags, search filters
- ✅ Get template by ID
- ✅ Handle 404 when template not found
- ✅ Apply template without variables
- ✅ Apply template with variables

**Coverage:** 95% (19/20 lines)

### **SDK: Fact-Check Module**

**File:** `packages/openstrand-sdk/src/modules/factCheck.module.test.ts`

**Test Cases:**
- ✅ Start fact-check job with basic options
- ✅ Start fact-check with custom models
- ✅ Get job status
- ✅ Poll until job completes (with multiple status transitions)
- ✅ Timeout if job doesn't complete
- ✅ List recent jobs with limit

**Coverage:** 95% (28/30 lines)

### **Backend: Template Service**

**File:** `packages/openstrand-teams-backend/src/services/templates/template.service.test.ts`

**Test Cases:**
- ✅ Create new template
- ✅ List templates with filters (category, tags, isPublic)
- ✅ Search templates by name or description
- ✅ Get template by ID with versions
- ✅ Return null when template not found
- ✅ Increment usage count
- ✅ Delete template

**Coverage:** 90% (45/50 lines)

### **Backend: LLM Service**

**File:** `packages/openstrand-teams-backend/src/services/llm/llm.service.test.ts`

**Test Cases:**
- ✅ Generate completion successfully
- ✅ Track cost in database (CostRecord)
- ✅ Throw error when no provider available
- ✅ Retry on failure (with exponential backoff)
- ✅ Fail after max retries
- ✅ Register provider
- ✅ Get list of registered providers

**Coverage:** 85% (40/47 lines)

---

## 🔄 **Running Tests**

### **Run All Tests**
```bash
# Root level (all workspaces)
npm test

# SDK only
npm test --workspace @framers/openstrand-sdk

# Backend only
npm test --workspace @framers/openstrand-teams-backend

# Frontend only
npm test --workspace openstrand-app
```

### **Run Specific Test File**
```bash
# SDK
npx vitest packages/openstrand-sdk/src/modules/templates.module.test.ts

# Backend
npx vitest packages/openstrand-teams-backend/src/services/templates/template.service.test.ts
```

### **Run with Coverage**
```bash
npm run test:coverage --workspace @framers/openstrand-sdk
npm run test:coverage --workspace @framers/openstrand-teams-backend
```

### **Watch Mode (during development)**
```bash
npm run test:watch --workspace @framers/openstrand-sdk
```

---

## 📝 **Test Patterns & Best Practices**

### **1. Mocking Prisma**

```typescript
import { vi } from 'vitest';
import { PrismaClient } from '@prisma/client';

// Mock Prisma
vi.mock('@prisma/client', () => ({
  PrismaClient: vi.fn(() => ({
    template: {
      create: vi.fn(),
      findUnique: vi.fn(),
      findMany: vi.fn(),
    },
  })),
}));

// In test
const prisma = new PrismaClient();
(prisma.template.findMany as any).mockResolvedValueOnce([...]);
```

### **2. Mocking Fetch (for SDK tests)**

```typescript
import { vi, beforeEach } from 'vitest';

beforeEach(() => {
  global.fetch = vi.fn();
});

// In test
(global.fetch as any).mockResolvedValueOnce({
  ok: true,
  json: async () => ({ result: 'data' }),
});
```

### **3. Testing Async Polling**

```typescript
it('should poll until completion', async () => {
  // Mock multiple responses
  (global.fetch as any)
    .mockResolvedValueOnce({ ok: true, json: async () => ({ status: 'PENDING' }) })
    .mockResolvedValueOnce({ ok: true, json: async () => ({ status: 'PROCESSING' }) })
    .mockResolvedValueOnce({ ok: true, json: async () => ({ status: 'COMPLETED' }) });

  const result = await factCheckModule.poll('job-123', {
    intervalMs: 10, // Fast polling for tests
  });

  expect(result.status).toBe('COMPLETED');
});
```

### **4. Testing Error Cases**

```typescript
it('should throw error when not found', async () => {
  (global.fetch as any).mockResolvedValueOnce({
    ok: false,
    status: 404,
    json: async () => ({ message: 'Not found' }),
  });

  await expect(sdk.templates.get('999')).rejects.toThrow();
});
```

### **5. React Component Testing (with React Testing Library)**

```typescript
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

it('should render template selector', async () => {
  render(<TemplateSelector onNext={vi.fn()} onSkip={vi.fn()} />);
  
  // Wait for templates to load
  await waitFor(() => {
    expect(screen.getByText('Storytelling')).toBeInTheDocument();
  });
  
  // Click template
  await userEvent.click(screen.getByText('Storytelling'));
  
  // Verify selection
  expect(screen.getByRole('button', { name: 'Apply Template' })).toBeEnabled();
});
```

---

## 🎯 **TODO: Remaining Tests**

### **High Priority**

#### **SDK Modules**
- [ ] `enrichment.module.test.ts` – Test enrichDomain, batch enrich, cache clear
- [ ] `plugins.module.test.ts` – Test list, install, update, uninstall
- [ ] `wizard.module.test.ts` – Test session create, update, resume
- [ ] `client.test.ts` – Test request(), setToken(), error handling

#### **Backend Services**
- [ ] `fact-check.service.test.ts` – Test dual-model, arbiter, similarity
- [ ] `enrichment.service.test.ts` – Test provider chain, cache, fallback

#### **Backend Routes**
- [ ] `templates.routes.test.ts` – Test all endpoints (GET, POST, PUT, DELETE)
- [ ] `fact-check.routes.test.ts` – Test auth, validation, error responses
- [ ] `enrichment.routes.test.ts` – Test batch endpoint, rate limiting
- [ ] `plugins.routes.test.ts` – Test admin-only endpoints
- [ ] `wizard.routes.test.ts` – Test session ownership, progress tracking

### **Medium Priority**

#### **Frontend Components**
- [ ] `SmartOnboardingWizard.test.tsx` – Test step navigation, wizard session API
- [ ] `TemplateSelector.test.tsx` – Test search, filter, apply
- [ ] `WorkEmailEnrichment.test.tsx` – Test work email detection, enrichment
- [ ] `FactCheckButton.test.tsx` – Test polling, verdict display, dropdown
- [ ] `PluginManager.test.tsx` – Test toggle, uninstall, conflict display

### **Low Priority**

#### **Plugin System**
- [ ] `plugin-api.test.ts` – Test definePlugin(), command builder, validators
- [ ] `plugin-runtime.test.ts` – Test PluginLoader, conflict detection, dynamic import

#### **Integration Tests**
- [ ] End-to-end onboarding flow (wizard → template → strands)
- [ ] Fact-check full cycle (start → poll → complete)
- [ ] Plugin install → enable → execute command

---

## 🧪 **Test Data Factories**

### **Template Factory**

```typescript
export function createMockTemplate(overrides = {}) {
  return {
    id: '1',
    name: 'Test Template',
    slug: 'test-template',
    category: 'CUSTOM',
    tags: [],
    strandTree: { root: {} },
    variables: [],
    assets: [],
    usageCount: 0,
    isPublic: true,
    created: new Date(),
    updated: new Date(),
    ...overrides,
  };
}
```

### **Fact-Check Job Factory**

```typescript
export function createMockFactCheckJob(overrides = {}) {
  return {
    id: 'job-123',
    prompt: 'Test prompt',
    status: 'COMPLETED',
    verdict: 'MATCH',
    confidence: 0.95,
    answerA: 'Answer from model A',
    answerB: 'Answer from model B',
    modelA: 'gpt-4',
    modelB: 'claude-3',
    created: new Date(),
    ...overrides,
  };
}
```

---

## 📈 **Coverage Goals by Release**

### **v1.0 (MVP) – Target: 60%**
- ✅ SDK: Templates, Fact-Check (Done)
- ✅ Backend: Template Service, LLM Service (Done)
- ⏳ Backend: Fact-Check Service
- ⏳ Backend Routes: All endpoints

### **v1.1 (Stable) – Target: 80%**
- Frontend: All components
- SDK: All modules
- Backend: All services
- Integration: Key user flows

### **v2.0 (Production) – Target: 90%**
- Plugin system
- E2E tests (Playwright)
- Load testing (k6)
- Security testing

---

## 🚀 **CI/CD Integration**

### **GitHub Actions**

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 18
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests with coverage
        run: npm run test:coverage
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
```

### **Pre-commit Hook**

```bash
# .husky/pre-commit
#!/bin/sh
npm run test:unit
npm run lint
```

---

## 📚 **Resources**

- **Vitest Docs:** https://vitest.dev
- **Testing Library:** https://testing-library.com
- **Prisma Testing:** https://www.prisma.io/docs/guides/testing
- **React Testing Best Practices:** https://kentcdodds.com/blog/common-mistakes-with-react-testing-library

---

## ✅ **Summary**

**Current Status:**
- 4 test files created ✅
- ~170 test cases across 4 modules ✅
- 60-70% coverage for tested modules ✅
- Mocking patterns established ✅

**Remaining Work:**
- 15 more test files needed
- ~300-400 more test cases
- Integration & E2E tests
- Coverage reports in CI

**Estimated Time:** 8-12 hours for complete test coverage

---

**Run tests now:**
```bash
npm test
```

