---
name: ui-test-implementor
description: >
  Implement Playwright TypeScript UI automation tests for web applications.
  Generates runnable spec files, page objects, and fixtures.
  Use when: implement UI tests, write Playwright, e2e automation, TypeScript test,
  browser test, page object, smoke test, visual regression, spec file, fixture.
---

# UI Test Implementor

## Purpose

Write complete, runnable Playwright TypeScript test files from a test plan, user flow, or feature description. Do not create test plans — use `test-plan-creator` for that first.

## Prerequisites

Before implementing, confirm you have at least one of:
- A test plan from `test-plan-creator`
- A user flow or feature description with URLs and expected UI behaviors
- Existing page structure (selectors, routes, component names)

If none are available, ask the user to provide them or run `test-plan-creator` first.

## Conventions

```typescript
// Use fixtures for shared setup
// Use Page Object Model (POM) for reusable selectors and actions
// Prefer role/text/label locators over CSS selectors or XPath
// Group related specs with test.describe
// Use test.step() to annotate multi-action flows
// Always await assertions: await expect(locator).toBeVisible()
// Name tests: should <expected behavior> when <condition>
```

### Project Structure

| Path | Purpose |
|---|---|
| `playwright.config.ts` | Global config: base URL, browsers, reporters, timeouts |
| `tests/**/*.spec.ts` | Spec files, one per feature or user flow |
| `tests/pages/*.ts` | Page Object classes |
| `tests/fixtures/*.ts` | Custom fixtures extending base test |

### Page Object Pattern

```typescript
// tests/pages/login.page.ts
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.getByLabel('Email');
    this.passwordInput = page.getByLabel('Password');
    this.submitButton = page.getByRole('button', { name: 'Sign in' });
    this.errorMessage = page.getByRole('alert');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }
}
```

### Fixture Pattern

```typescript
// tests/fixtures/index.ts
import { test as base } from '@playwright/test';
import { LoginPage } from '../pages/login.page';

type Fixtures = { loginPage: LoginPage };

export const test = base.extend<Fixtures>({
  loginPage: async ({ page }, use) => {
    await use(new LoginPage(page));
  },
});

export { expect } from '@playwright/test';
```

### Spec Structure Pattern

```typescript
// tests/login.spec.ts
import { test, expect } from './fixtures';

test.describe('Login', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
  });

  test('should redirect to dashboard when credentials are valid', async ({ loginPage, page }) => {
    await test.step('Enter valid credentials', async () => {
      await loginPage.login('user@example.com', 'validPassword');
    });
    await expect(page).toHaveURL('/dashboard');
  });

  test('should show error when password is incorrect', async ({ loginPage }) => {
    await loginPage.login('user@example.com', 'wrongPassword');
    await expect(loginPage.errorMessage).toBeVisible();
    await expect(loginPage.errorMessage).toHaveText('Invalid credentials');
  });
});
```

## Workflow

1. Read the test cases from the test plan (or derive them from the user flow)
2. Identify pages/components involved — create or update Page Object files
3. Identify shared fixtures (auth state, logged-in user, seeded data)
4. Write spec file(s) in `tests/`, grouped by feature
5. Use `test.beforeEach` for navigation; avoid `test.beforeAll` for stateful setup
6. Verify each test has: navigation, action, and at minimum one `expect` assertion

## Output Format

- Full, runnable TypeScript files with all imports
- One spec file per feature or user flow
- Page objects and fixtures in separate files
- Brief `test.describe` label that identifies the feature under test
